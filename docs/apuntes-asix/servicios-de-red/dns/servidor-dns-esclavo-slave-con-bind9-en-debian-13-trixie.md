---
description: >-
  Esta guía detalla el proceso para desplegar un entorno de alta disponibilidad
  con un servidor DNS maestro y un servidor DNS esclavo (Slave), partiendo de la
  clonación de la máquina virtual maestra.
---

# Servidor DNS Esclavo (Slave) con Bind9 en Debian 13 (Trixie)

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

### 1. Esquema de Red y Direccionamiento

* Sistema Operativo: Debian GNU/Linux 13 (Trixie)
* Red:
  * Adaptador 1: NAT (internet y _forwarders_).
  * Adaptador 2: Red Interna (`192.168.6.0/24`).
* Asignación de IPs:
  * Servidor Master: `192.168.6.100/24`
  * Servidor Slave (Clon): `192.168.6.101/24`
  * Cliente: `192.168.6.20/24`

### 2. Ajustes en el Servidor Maestro (`192.168.6.100`)

Dado que el maestro ya está operativo, solo necesitamos permitir que le transfiera las zonas al esclavo y registrar el nuevo servidor de nombres (`ns2`).

#### A. Permitir la Transferencia en `named.conf.local`

Edita el fichero de configuración local del maestro:

```bash
sudo nano /etc/bind/named.conf.local
```

Añade las directivas `allow-transfer` y `also-notify` apuntando a la IP del esclavo (`192.168.6.101`):

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

```bash
zone "honeypot.com" {
    type master;
    file "/etc/bind/zones/db.honeypot.com";
    allow-transfer { 192.168.6.101; };
    also-notify { 192.168.6.101; };
};

zone "6.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.6.168.192";
    allow-transfer { 192.168.6.101; };
    also-notify { 192.168.6.101; };
};
```

#### B. Actualizar los Ficheros de Zona del Maestro

1. Zona Directa (`/etc/bind/zones/db.honeypot.com`): Añade el registro del esclavo (`ns2`) y no olvides incrementar el número Serial (por ejemplo, súmele 1):

<figure><img src="../../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

```bash
$TTL    86400
@   IN  SOA ns1.honeypot.com. hostmaster.honeypot.com. (
            2026092802  ; Serial (¡Incrementado!)
            3600        ; Refresh
            1800        ; Retry
            604800      ; Expire
            86400 )     ; Minimum TTL

@       IN  NS      ns1.honeypot.com.
@       IN  NS      ns2.honeypot.com.
ns1     IN  A       192.168.6.100
ns2     IN  A       192.168.6.101
cliente IN  A       192.168.6.20
```

1. Zona Inversa (`/etc/bind/zones/db.6.168.192`): Añade el puntero PTR para el esclavo e incrementa también su Serial:

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

```bash
$TTL    86400
@   IN  SOA ns1.honeypot.com. hostmaster.honeypot.com. (
            2026092802  ; Serial
            3600
            1800
            604800
            86400 )

@   IN  NS   ns1.honeypot.com.
@   IN  NS   ns2.honeypot.com.
100 IN  PTR  ns1.honeypot.com.
101 IN  PTR  ns2.honeypot.com.
20  IN  PTR  cliente.honeypot.com.
```

#### C. Validar y Reiniciar el Maestro

Comprueba que todo es correcto y reinicia Bind9:

```bash
sudo named-checkconf
sudo named-checkzone honeypot.com /etc/bind/zones/db.honeypot.com
sudo systemctl restart bind9
```

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

### 3. Configuración del Servidor Esclavo (`192.168.6.101`)

Como has clonado la máquina del maestro, ya tiene instalado Bind9. Ahora debes adaptar sus parámetros de red, su identidad y transformar las zonas a modo esclavo.

#### A. Modificar la Red y el Hostname

1.  Edita el fichero de red (ej. `/etc/network/interfaces`) para cambiar la IP estática interna a `192.168.6.101`:

    ```bash
    auto enp0s3
    iface enp0s3 inet dhcp

    auto enp0s8
    iface enp0s8 inet static
        address 192.168.6.101
        netmask 255.255.255.0
    ```
2.  Cambia el nombre del host para identificarlo como esclavo:

    ```bash
    sudo hostnamectl set-hostname dns-slave
    ```

    _(Reinicia la máquina o aplica los cambios de red para que coja la nueva IP `192.168.6.101`)._

<figure><img src="../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

#### B. Limpiar y Configurar `named.conf.local` en el Esclavo

Como heredó los ficheros de zona estáticos del master, limpia o sobrescribe el fichero `/etc/bind/named.conf.local` para que actúe como `slave` y descargue las zonas del maestro (`192.168.6.100`):

```bash
sudo nano /etc/bind/named.conf.local
```

Contenido:

```bash
zone "honeypot.com" {
    type slave;
    file "/var/cache/bind/db.honeypot.com";
    masters { 192.168.6.100; };
};

zone "6.168.192.in-addr.arpa" {
    type slave;
    file "/var/cache/bind/db.6.168.192";
    masters { 192.168.6.100; };
};
```

#### C. Actualizar Opciones Globales del Esclavo

Edita `/etc/bind/named.conf.options` para asegurarte de que escucha en su propia IP (`192.168.6.101`):

```bash
options {
    directory "/var/cache/bind";

    listen-on { 127.0.0.1; 192.168.6.101; };
    listen-on-v6 { none; };

    recursion yes;
    allow-recursion { 127.0.0.1; 192.168.6.0/24; };

    forwarders { 8.8.8.8; 1.1.1.1; };

    dnssec-validation auto;
};
```

Verifica que en `/etc/default/named` se mantiene el parámetro para forzar IPv4:

Bash

```bash
OPTIONS="-u bind -4"
```

### 4. Puesta en Marcha y Verificación

Reinicia el servicio Bind9 en el servidor esclavo:

```bash
sudo systemctl restart bind9
sudo systemctl status bind9
```

#### Comprobación de la Sincronización (Transferencia de Zona)

1.  Revisa los logs de Bind9 en el esclavo para comprobar que se ha conectado al maestro y ha descargado las zonas:

    ```bash
    sudo journalctl -u named -n 50 --no-pager
    ```

    _(Deberías ver avisos indicando que se ha transferido la zona `honeypot.com` desde `192.168.6.100`)._
2.  Comprueba que los archivos de zona ya aparecen físicamente en la carpeta de caché del esclavo:

    ```bash
    ls -l /var/cache/bind/
    ```

#### Pruebas de Funcionamiento

Realiza una consulta dirigida explícitamente al servidor esclavo (`192.168.6.101`):

```bash
nslookup ns2.honeypot.com 192.168.6.101
nslookup cliente.honeypot.com 192.168.6.101
nslookup 192.168.6.100 192.168.6.101
```

* Verificación: Si el esclavo responde con las IPs correctas, la sincronización y la alta disponibilidad están funcionando a la perfección.

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>
