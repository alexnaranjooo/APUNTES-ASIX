# Servidor DNS Esclavo (Slave) con Bind9 en Debian 13 (Trixie)

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

### 1. Preparación de la Máquina Virtual (Esclavo)

* Sistema Operativo: Debian GNU/Linux 13 (Trixie) _(Clon de la máquina master)_
* Red:
  * Adaptador 1: NAT (conexión a internet).
  * Adaptador 2: Red Interna (servicio local).
* IP Estática del Servidor Esclavo: `192.168.6.101/24` _(evitando el conflicto con el master `192.168.6.100`)_

### 2. Configuración de Red en el Servidor Esclavo

Modifica la interfaz de red interna en el fichero de configuración de red (por ejemplo, `/etc/network/interfaces`) para asignarle la IP `192.168.6.11`:

Plaintext

```
auto enp0s3
iface enp0s3 inet dhcp

auto enp0s8
iface enp0s8 inet static
    address 192.168.6.101
    netmask 255.255.255.0
```

* Verificación: Aplica los cambios de red o reinicia el servicio de red, y comprueba con `ip a` que la interfaz `enp0s8` tiene asignada correctamente la IP `192.168.6.11`.

### 3. Configuración en el Servidor Maestro (`192.168.6.10`)

Antes de configurar el esclavo, el servidor maestro debe permitir la transferencia de zona (_zone transfer_) hacia la IP del esclavo.

Edita el fichero `/etc/bind/named.conf.local` en el servidor maestro:

Fragmento de código

```
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

Reinicia el servicio Bind9 en el maestro:

Bash

```
sudo systemctl restart bind9
```

* Verificación: Comprueba que el maestro no muestra errores ejecutando `sudo systemctl status bind9`.

### 4. Configuración del Servidor Esclavo (`192.168.6.11`)

En el servidor esclavo, edita el fichero `/etc/bind/named.conf.local` para declarar las zonas como de tipo `slave` e indicar dónde está el maestro (`masters`):

Fragmento de código

```
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

Verifica la sintaxis general de la configuración:

Bash

```
sudo named-checkconf
```

* Verificación: El comando `named-checkconf` no debe devolver ningún error en pantalla.

### 5. Opciones Globales y Forzar IPv4 en el Esclavo

Asegúrate de que el fichero de opciones globales `/etc/bind/named.conf.options` en el esclavo permite escuchar en su IP (`192.168.6.11`) y realizar consultas recursivas para la red local:

Fragmento de código

```
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

Fuerza el uso exclusivo de IPv4 en `/etc/default/named`:

Bash

```
OPTIONS="-u bind -4"
```

* Verificación: Comprueba que el parámetro `OPTIONS="-u bind -4"` está presente en `/etc/default/named`.

### 6. Reinicio del Servicio y Sincronización

Reinicia el servicio Bind9 en el servidor esclavo:

Bash

```
sudo systemctl restart bind9
sudo systemctl status bind9
```

* Verificación: Revisa los logs del sistema con `sudo journalctl -u named -n 30` para confirmar que el servidor esclavo se ha conectado al maestro (`192.168.6.10`) y ha descargado las zonas correctamente (deberás ver mensajes indicando que las zonas se han transferido y guardado en `/var/cache/bind/`).

### 7. Pruebas de Funcionamiento

Apunta un cliente de la red interna (o el propio esclavo temporalmente en `/etc/resolv.conf`) al servidor esclavo (`192.168.6.11`):

Bash

```
nslookup ns1.honeypot.com 192.168.6.101
nslookup 192.168.6.100 192.168.6.101
```

* Verificación: Ambas consultas deben resolverse correctamente utilizando el servidor esclavo, demostrando que la transferencia de zona entre el maestro y el esclavo ha funcionado.
