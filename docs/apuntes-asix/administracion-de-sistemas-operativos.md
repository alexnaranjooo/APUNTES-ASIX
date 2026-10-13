# 💿 Administración de Sistemas Operativos

## Servidor LDAP con OpenLDAP en Ubuntu Server

### Introducción

Guía para montar un **servidor de directorio LDAP** con **OpenLDAP (slapd)** sobre Ubuntu Server, dentro de una máquina virtual de VirtualBox. Se parte de la red y la resolución de nombres, se instala y verifica el servicio y se deja preparada la estructura inicial del directorio.

**LDAP** (_Lightweight Directory Access Protocol_) es un protocolo para consultar y mantener un directorio de información organizada en árbol: usuarios, grupos, equipos, etc. Es la base de muchos sistemas de autenticación centralizada.

***

### Datos del laboratorio

| Parámetro           | Valor                                                 |
| ------------------- | ----------------------------------------------------- |
| Hipervisor          | VirtualBox                                            |
| Sistema operativo   | Ubuntu Server                                         |
| Red de la VM        | Red NAT `RedLDAP` (`20.0.0.0/24`, DHCP deshabilitado) |
| IP del servidor     | `20.0.0.5/24`                                         |
| Puerta de enlace    | `20.0.0.1`                                            |
| DNS                 | `8.8.8.8`, `1.1.1.1`                                  |
| Nombre del servidor | `ldapserver`                                          |
| FQDN                | `ldapserver.planetafp.local`                          |
| Dominio LDAP        | `planetafp.local`                                     |
| DN base             | `dc=planetafp,dc=local`                               |
| Organización        | `planetafp`                                           |

***

### 1. Red NAT en VirtualBox

Primero se crea una **red NAT** propia para el laboratorio, con un rango fijo y el DHCP **deshabilitado**, porque la IP del servidor se asignará a mano.

En VirtualBox: **Herramientas → Redes → Redes NAT → Crear**, con estos valores:

* **Nombre**: `RedLDAP`
* **Prefijo IPv4**: `20.0.0.0/24`
* **Habilitar DHCP**: desmarcado

Después, en la configuración de la VM, **Red → Adaptador 1** se conecta a esa red:

* **Conectar a**: Red NAT
* **Nombre**: `RedLDAP`

> **Por qué una red NAT y no la NAT normal**: la red NAT de VirtualBox permite elegir el rango de direcciones y que varias VMs se vean entre sí, además de salir a internet.

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

***

### 2. Instalación de Ubuntu Server

Durante la instalación se definen el nombre del servidor y el usuario administrador:

* **Nombre del servidor**: `ldapserver`
* **Usuario**: `honeynet`

Una vez instalado, se comprueba la configuración de red con:

```bash
ip -br a
```

***

### 3. IP estática con Netplan

En Ubuntu Server la red se configura con **Netplan**. El archivo es `/etc/netplan/00-installer-config.yaml`:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 20.0.0.5/24
      routes:
        - to: default
          via: 20.0.0.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```

Para aplicar los cambios sin riesgo se usa `netplan try`, que los revierte solos si no se confirman a tiempo:

```bash
sudo netplan try
```

Hay que pulsar **Enter** antes de que acabe la cuenta atrás (120 segundos por defecto) para aceptar la configuración.

> **Buena práctica**: `netplan try` evita quedarse sin red por un error en el YAML, algo especialmente molesto en una máquina a la que solo se accede por consola.

#### Verificación

Se comprueba que la IP está asignada y que hay salida a internet:

```bash
ip -br a
ping -c 3 8.8.8.8
```

***

### 4. Resolución de nombres: `/etc/hosts`

OpenLDAP deduce el dominio a partir del nombre completo (FQDN) del servidor, así que conviene que el equipo se resuelva a sí mismo correctamente. Se edita `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

Se añade una línea con la IP, el FQDN y el nombre corto:

```
20.0.0.5   ldapserver.planetafp.local   ldapserver
```

Se comprueba que el sistema la resuelve con `getent`:

```bash
getent hosts ldapserver.planetafp.local
```

***

### 5. Instalación de OpenLDAP

Se instalan el servidor (`slapd`) y las utilidades de cliente (`ldap-utils`):

```bash
sudo apt update
sudo apt install slapd ldap-utils
```

Durante la configuración de `slapd` se indican el **dominio DNS** y la **organización**. Con ellos se construye el DN base del directorio.

Si hay que repetir este asistente más adelante:

```bash
sudo dpkg-reconfigure slapd
```

**Nombre de dominio DNS**: `planetafp.local`, que genera el DN base `dc=planetafp,dc=local`.

**Nombre de la organización**: `planetafp`.

> **Cómo se forma el DN base**: cada parte del dominio pasa a ser un componente `dc`. El dominio `planetafp.local` se convierte en `dc=planetafp,dc=local`.

***

### 6. Verificación del servicio

Se comprueba que el servicio está activo y habilitado en el arranque:

```bash
sudo systemctl status slapd
```

Debe aparecer **`active (running)`** y **`enabled`**.

Se comprueba también que escucha en el puerto **389**, el de LDAP sin cifrar:

```bash
sudo ss -tlnp | grep slapd
```

***

### 7. Comprobar los esquemas cargados

Los **esquemas** definen qué clases de objeto y atributos admite el directorio. Se consultan con acceso local como root mediante `-Y EXTERNAL` sobre el socket `ldapi:///`:

```bash
sudo ldapsearch -Y EXTERNAL -H ldapi:/// -b "cn=schema,cn=config" dn
```

Deben aparecer cuatro esquemas: `core`, `cosine`, `nis` e `inetorgperson`. Este último aporta la clase `inetOrgPerson`, que se usará para los usuarios.

***

### 8. Consulta del directorio

Se consulta la raíz del directorio con una búsqueda anónima (`-x`) de ámbito `base`, que devuelve solo la entrada indicada:

```bash
ldapsearch -x -H ldap://ldapserver.planetafp.local -b "dc=planetafp,dc=local" -s base
```

La respuesta muestra la entrada raíz `dc=planetafp,dc=local`, con las clases `top`, `dcObject` y `organization`, y termina en `result: 0 Success`.

***

### 9. Estructura inicial del directorio (LDIF)

Para poblar el directorio se usan archivos **LDIF**, un formato de texto que describe las entradas. Se crea `estructura.ldif` con una **unidad organizativa** (`ou=alumno`) y un primer usuario:

```bash
nano estructura.ldif
```

```ldif
dn: ou=alumno,dc=planetafp,dc=local
objectClass: organizationalUnit
ou: alumno

dn: uid=jperez,ou=alumno,dc=planetafp,dc=local
objectClass: inetOrgPerson
cn: Juan Perez
sn: Perez
uid: jperez
mail: jperez@planetafp.local
```

> **Atención al sufijo**: el `dn` de cada entrada debe terminar exactamente en el DN base (`dc=planetafp,dc=local`). Si un `dn` usa otro sufijo (por ejemplo `dc=planeta`), el servidor rechazará la entrada.

***

### Resumen de comandos

| Comando                               | Para qué sirve                                    |
| ------------------------------------- | ------------------------------------------------- |
| `ip -br a`                            | Ver las interfaces y sus IPs de forma resumida    |
| `sudo netplan try`                    | Aplicar la red con reversión automática si falla  |
| `getent hosts <nombre>`               | Comprobar cómo resuelve el sistema un nombre      |
| `sudo dpkg-reconfigure slapd`         | Repetir el asistente de configuración de OpenLDAP |
| `sudo systemctl status slapd`         | Estado del servicio                               |
| `sudo ss -tlnp \| grep slapd`         | Comprobar que escucha en el puerto 389            |
| `ldapsearch -Y EXTERNAL -H ldapi:///` | Consultar la configuración como root              |
| `ldapsearch -x -H ldap://... -b <DN>` | Consultar el directorio de forma anónima          |

### Próximos pasos

* Cargar `estructura.ldif` en el directorio con `ldapadd` y comprobar que las entradas aparecen.
* Añadir más usuarios y grupos.
* Cifrar las conexiones con **StartTLS** o **LDAPS**, ya que el puerto 389 envía los datos sin cifrar.
* Conectar un cliente Linux para que autentique contra este servidor.
