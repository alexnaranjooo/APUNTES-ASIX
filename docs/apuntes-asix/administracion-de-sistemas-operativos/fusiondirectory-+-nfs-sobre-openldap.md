# FusionDirectory + NFS sobre OpenLDAP

Playbook de instalación y configuración de FusionDirectory (panel web de administración de OpenLDAP) integrado con un servidor NFS cuyo acceso se restringe por grupo del directorio.

**Datos usados en esta guía:**

* VM Debian 12 (Bookworm) o 13 (Trixie): `192.168.111.55`
* Base DN: `dc=honeynet,dc=asix,dc=com`
* OUs: `ou=usuarios`, `ou=grupos`
* Grupo SOC: `soc` (gidNumber 5000)
* Usuarios de prueba: `analista1` (uid 10001), `analista2` (uid 10002)
* Recurso NFS: `/srv/nfs/soc`, permisos `770`, grupo `soc`



***

## Parte 0 — Prerrequisito: OpenLDAP + NSS/SSSD

### 0.1 Instalar OpenLDAP

```bash
sudo apt update
sudo apt install slapd ldap-utils -y
sudo dpkg-reconfigure slapd
```

Respuestas del asistente:

<table data-search="false"><thead><tr><th>Pregunta</th><th>Respuesta</th></tr></thead><tbody><tr><td>¿Omitir configuración del servidor?</td><td>No</td></tr><tr><td>Nombre de dominio DNS</td><td><code>honeynet.asix.com</code></td></tr><tr><td>Nombre de la organización</td><td><code>HoneyNet</code></td></tr><tr><td>Contraseña administrador</td><td>elige una (anótala)</td></tr><tr><td>Motor de base de datos</td><td>MDB</td></tr><tr><td>¿Eliminar la BD al purgar slapd?</td><td>No</td></tr><tr><td>¿Mover la BD antigua?</td><td>Sí</td></tr></tbody></table>

```bash
sudo systemctl status slapd
```

### 0.2 Confirmar la IP de la VM

```bash
ip a
```

Debe coincidir con `192.168.111.55`. Si no coincide, usa la IP real que te devuelva este comando en el resto de la guía.

### 0.3 Crear estructura (OUs, grupo, usuarios)

```bash
nano ~/estructura.ldif
```

```
dn: ou=usuarios,dc=honeynet,dc=asix,dc=com
objectClass: organizationalUnit
ou: usuarios

dn: ou=grupos,dc=honeynet,dc=asix,dc=com
objectClass: organizationalUnit
ou: grupos
```

```bash
ldapadd -x -D "cn=admin,dc=honeynet,dc=asix,dc=com" -W -f ~/estructura.ldif
```

```bash
nano ~/grupo-soc.ldif
```

```
dn: cn=soc,ou=grupos,dc=honeynet,dc=asix,dc=com
objectClass: posixGroup
cn: soc
gidNumber: 5000
```

```bash
ldapadd -x -D "cn=admin,dc=honeynet,dc=asix,dc=com" -W -f ~/grupo-soc.ldif
```

```bash
nano ~/usuarios.ldif
```

```
dn: uid=analista1,ou=usuarios,dc=honeynet,dc=asix,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: analista1
sn: Analista1
givenName: Analista
cn: Analista Uno
displayName: Analista Uno
uidNumber: 10001
gidNumber: 5000
userPassword: {CRYPT}x
loginShell: /bin/bash
homeDirectory: /home/analista1

dn: uid=analista2,ou=usuarios,dc=honeynet,dc=asix,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: analista2
sn: Analista2
givenName: Analista
cn: Analista Dos
displayName: Analista Dos
uidNumber: 10002
gidNumber: 5000
userPassword: {CRYPT}x
loginShell: /bin/bash
homeDirectory: /home/analista2
```

```bash
ldapadd -x -D "cn=admin,dc=honeynet,dc=asix,dc=com" -W -f ~/usuarios.ldif
ldappasswd -x -D "cn=admin,dc=honeynet,dc=asix,dc=com" -W -S "uid=analista1,ou=usuarios,dc=honeynet,dc=asix,dc=com"
ldappasswd -x -D "cn=admin,dc=honeynet,dc=asix,dc=com" -W -S "uid=analista2,ou=usuarios,dc=honeynet,dc=asix,dc=com"
```

Verificación:

```bash
ldapsearch -x -b "dc=honeynet,dc=asix,dc=com" "(objectClass=*)" -LLL
```

### 0.4 NSS/SSSD

```bash
sudo apt install sssd sssd-ldap libnss-sss libpam-sss -y
sudo nano /etc/sssd/sssd.conf
```

```
[sssd]
config_file_version = 2
services = nss, pam
domains = honeynet.asix.com

[domain/honeynet.asix.com]
id_provider = ldap
auth_provider = ldap
ldap_uri = ldap://192.168.111.55
ldap_search_base = dc=honeynet,dc=asix,dc=com
ldap_default_bind_dn = cn=admin,dc=honeynet,dc=asix,dc=com
ldap_default_authtok_type = password
ldap_default_authtok = TU_CONTRASEÑA_ADMIN
ldap_id_use_start_tls = False

cache_credentials = true
enumerate = true
```



```bash
sudo chmod 600 /etc/sssd/sssd.conf
sudo chown root:root /etc/sssd/sssd.conf
sudo nano /etc/nsswitch.conf
```

Añadir `sss` al final de `passwd` y `group`:

```
passwd:         files systemd sss
group:          files systemd sss
```

```bash
sudo systemctl enable sssd
sudo systemctl restart sssd
```

Verificación:

```bash
getent passwd analista1
getent group soc
```

Si `getent` no devuelve nada (sin error), confirma con `sudo systemctl status sssd` si el backend aparece como `offline`, y revisa el log con `sudo tail -40 /var/log/sssd/sssd_honeynet.asix.com.log`. Si corriges `sssd.conf`, limpia la caché antes de reintentar:

```bash
sudo systemctl stop sssd
sudo rm -rf /var/lib/sss/db/*
sudo systemctl start sssd
```

***

## Módulo A — Instalación de FusionDirectory (paquete nativo Debian)

### A.1 Dependencias: Apache, PHP

```bash
sudo apt install apache2 php libapache2-mod-php -y
sudo apt install php-ldap php-mbstring php-xml php-curl php-gd php-zip php-intl -y
```

Verificación:

```bash
php -v
php -m | grep -i ldap
```

### A.2 Repositorio oficial de FusionDirectory

Comprueba tu versión de Debian:

```bash
cat /etc/os-release
```

Añade la clave y el repositorio (ajusta `bookworm` si tu Debian es otra versión):

```bash
sudo apt install wget gnupg apt-transport-https -y
wget -O - https://www.fusiondirectory.org/download/debian/fusiondirectory.gpg | sudo gpg --dearmor -o /usr/share/keyrings/fusiondirectory.gpg
echo "deb [signed-by=/usr/share/keyrings/fusiondirectory.gpg] https://www.fusiondirectory.org/download/debian/ bookworm main" | sudo tee /etc/apt/sources.list.d/fusiondirectory.list
sudo apt update
```

Si `apt update` falla con el repo de FusionDirectory, verifica el codename exacto de tu Debian (`VERSION_CODENAME` en `/etc/os-release`) y sustitúyelo en la línea de arriba.

```bash
sudo apt install fusiondirectory fusiondirectory-schema -y
```

### A.3 Cargar los esquemas

```bash
sudo fusiondirectory-insert-schema
```

Este comando detecta `slapd` en ejecución e inyecta automáticamente todos los esquemas necesarios directamente en `cn=config`.

Verificación:

```bash
sudo fusiondirectory-schema-manager -l
```

### A.4 Asistente web de configuración — Las 6 etapas

Desde el navegador del host:

```
http://192.168.111.55/fusiondirectory/setup
```

**Etapa 1 — Welcome/Checks.** Comprueba PHP, extensiones y permisos automáticamente. Resuelve cualquier aviso en rojo antes de continuar.

**Etapa 2 — Security token.** Genera un token temporal que confirmas en la web.

**Etapa 3 — Language.** Selecciona `es` (español) y zona horaria `Europe/Madrid`.

**Etapa 4 — LDAP connection.**

| Campo           | Valor                                 |
| --------------- | ------------------------------------- |
| LDAP URI        | `ldap://192.168.111.55`               |
| Base DN         | `dc=honeynet,dc=asix,dc=com`          |
| Bind DN (admin) | `cn=admin,dc=honeynet,dc=asix,dc=com` |
| Contraseña      | la del administrador LDAP (paso 0.1)  |

Pulsa **Test connection** y confirma éxito antes de avanzar.

**Etapa 5 — Branches / Plugins.**

| Tipo de objeto | Rama                                     |
| -------------- | ---------------------------------------- |
| Usuarios       | `ou=usuarios,dc=honeynet,dc=asix,dc=com` |
| Grupos         | `ou=grupos,dc=honeynet,dc=asix,dc=com`   |

Activa al menos el plugin **POSIX**.

{% hint style="success" %}
Esta etapa crea automáticamente toda la estructura de ACL y configuración interna (`ou=fusiondirectory`, `cn=config`, roles, permisos de administrador) que necesita el panel para funcionar, sin tener que tocar LDAP a mano.
{% endhint %}

**Etapa 6 — Finish.** El asistente escribe `/etc/fusiondirectory/fusiondirectory.conf`. Verifica:

```bash
sudo cat /etc/fusiondirectory/fusiondirectory.conf
```

Por seguridad, bloquea el acceso público al setup una vez terminado:

```bash
sudo mv /var/www/fusiondirectory/html/setup/setup.php /var/www/fusiondirectory/html/setup/setup.php.disabled
```

(la ruta exacta puede variar ligeramente; si no existe ahí, comprueba con `ls /var/www/fusiondirectory/html/setup/`)

### A.5 Primer login y administración visual

```
http://192.168.111.55/fusiondirectory
```

El propio asistente, en una de sus últimas etapas, permite crear o designar el primer usuario administrador con ACL completa automáticamente.

Una vez dentro con permisos de admin:

**Dar de alta un grupo adicional vía interfaz:**

1. **Groups → Create → Group**
2. Rama `ou=grupos,dc=honeynet,dc=asix,dc=com`
3. Pestaña **POSIX** → marcar como grupo POSIX

**Dar de alta un usuario vía interfaz:**

1. **Users → Create → User**
2. Pestaña **User** → nombre, apellidos, `uid`
3. Pestaña **POSIX** → cuenta POSIX, `uidNumber`, grupo primario (`soc`), `homeDirectory`, `loginShell`
4. Establecer contraseña

***

## Módulo B — Servidor NFS

### B.1 Instalación

```bash
sudo apt install nfs-kernel-server -y
sudo systemctl enable nfs-kernel-server
```

### B.2 Directorio compartido y permisos

```bash
sudo mkdir -p /srv/nfs/soc
sudo chown root:soc /srv/nfs/soc
sudo chmod 770 /srv/nfs/soc
ls -ld /srv/nfs/soc
```

Debe mostrar: `drwxrwx--- root soc ...`

### B.3 Exportar el recurso

```bash
sudo nano /etc/exports
```

```
/srv/nfs/soc 172.16.0.0/24(rw,sync,no_subtree_check)
```

```bash
sudo exportfs -ra
sudo exportfs -v
```

### B.4 Montaje desde un cliente

En la VM cliente:

```bash
sudo apt install nfs-common -y
sudo mkdir -p /mnt/soc
sudo mount 192.168.111.55:/srv/nfs/soc /mnt/soc
df -h | grep soc
```

Persistente en `/etc/fstab` del cliente:

```
192.168.111.55:/srv/nfs/soc  /mnt/soc  nfs  defaults  0  0
```

***

## Módulo C — Verificación y troubleshooting

### C.1 Comandos de comprobación

```bash
sudo systemctl status nfs-kernel-server
sudo exportfs -v
sudo showmount -e 192.168.111.55
rpcinfo -p 192.168.111.55
```

### C.2 Prueba de segregación de acceso

&#x20;**Usuario del grupo `soc` (debe funcionar):**

```bash
su - analista1
cd /mnt/soc
touch prueba_analista1.txt
ls -l
```

