---
description: Guía de como crear un servidor DNS con Bind9 y un Debian 13.
---

# Servidor DNS con Bind9 en Debian 13 (Trixie)

Esta guía detalla el proceso completo para desplegar un servidor DNS maestro utilizando Bind9 sobre una máquina virtual con Debian 13 (Trixie), configurado con doble interfaz de red (NAT para salida a internet y Red Interna para dar servicio a los clientes).

<figure><img src="../../../.gitbook/assets/DNS ESQUEMA.png" alt=""><figcaption></figcaption></figure>



### **1. Características de la Máquina Virtual**

* **Sistema Operativo**: Debian&#x20;
* **Red**:
  * Adaptador 1: NAT (para conexión a internet).
  * Adaptador 2: Red Interna (para dar servicio a la red local y clientes).
* **Disco Duro**: 25 GB
* **Memoria RAM**: 2 GB
* **IP Estática del Servidor** (Red Interna): `192.168.6.10/24` _(ejemplo)_

### **2. Configuración de Red en Debian 13**

Configuramos las interfaces de red estáticas editando el fichero correspondiente según tu gestor de red (por ejemplo, en `/etc/network/interfaces`):

```bash
auto enp0s3
iface enp0s3 inet dhcp

auto enp0s8
iface enp0s8 inet static
    address 192.168.6.10
    netmask 255.255.255.0
```

### **3. Instalación de Bind9**

Actualizamos los repositorios e instalamos Bind9 junto con sus utilidades y herramientas de consulta (`dig`, `nslookup`):

```bash
sudo apt update
sudo apt install bind9 bind9-utils bind9-dnsutils
```

### **4. Configuración de Zonas (Bind9)**

Creamos el directorio para almacenar los ficheros de zona y editamos la configuración local:

```bash
sudo mkdir -p /etc/bind/zones
sudo nano /etc/bind/named.conf.local
```

Añadimos las declaraciones para la zona directa y la zona inversa de `honeypot.com`:

```bash
zone "honeypot.com" {
    type master;
    file "/etc/bind/zones/db.honeypot.com";
};

zone "6.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.6.168.192";
};
```

Verificamos la sintaxis general:

```bash
sudo named-checkconf
```

### **5. Creación de los Ficheros de Zona**

#### A. Zona Directa

Creamos y editamos el fichero de zona directa (`/etc/bind/zones/db.honeypot.com`):

```bash
$TTL    86400
@   IN  SOA ns1.honeypot.com. hostmaster.honeypot.com. (
            2026092801  ; Serial (incrementar al modificar)
            3600        ; Refresh
            1800        ; Retry
            604800      ; Expire
            86400 )     ; Minimum TTL

@       IN  NS      ns1.honeypot.com.
ns1     IN  A       192.168.6.10
cliente IN  A       192.168.6.20
```

#### B. Zona Inversa

Creamos y editamos el fichero de zona inversa (`/etc/bind/zones/db.6.168.192`):

```bash
$TTL    86400
@   IN  SOA ns1.honeypot.com. hostmaster.honeypot.com. (
            2026092801  ; Serial
            3600
            1800
            604800
            86400 )

@   IN  NS   ns1.honeypot.com.
10  IN  PTR  ns1.honeypot.com.
20  IN  PTR  cliente.honeypot.com.
```

#### Verificación de Zonas

Comprobamos que ambos ficheros sean correctos (deben devolver `OK`):

```bash
sudo named-checkzone honeypot.com /etc/bind/zones/db.honeypot.com
sudo named-checkzone 6.168.192.in-addr.arpa /etc/bind/zones/db.6.168.192
```

### **6. Opciones Globales y Forzar IPv4**

Editamos el fichero de opciones globales (`/etc/bind/named.conf.options`):

```bash
options {
    directory "/var/cache/bind";

    listen-on { 127.0.0.1; 192.168.6.10; };
    listen-on-v6 { none; };

    recursion yes;
    allow-recursion { 127.0.0.1; 192.168.6.0/24; };

    forwarders { 8.8.8.8; 1.1.1.1; };

    dnssec-validation auto;
};
```

Para evitar errores y ruido en entornos sin IPv6, forzamos el uso de IPv4 editando `/etc/default/named`:

```bash
OPTIONS="-u bind -4"
```

### **7. Reinicio del Servicio y Pruebas**

Reiniciamos el servicio y comprobamos su estado:

```bash
sudo systemctl restart bind9
sudo systemctl status bind9
```

Ejecuta las comprobaciones:

```bash
nslookup ns1.honeypot.com
nslookup 192.168.6.10
nslookup google.com
```

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>
