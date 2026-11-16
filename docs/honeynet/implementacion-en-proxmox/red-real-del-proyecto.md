---
description: >-
  La red que se dibujó en la pizarra, ya montada: pfSense con DNS, DHCP por MAC
  y reenvío de puertos, los contenedores de la web y la base de datos, y la
  prueba de punta a punta.
---

# Red real del proyecto

## Objetivo

Montar la red que se dibujó en la pizarra: la red del centro entra por `vmbr0` a **pfSense**, que hace el reenvío de puertos, las VLANs y el DHCP por MAC, y por `vmbr1` cuelgan los servicios (NGINX, base de datos y copias). El **DNS lo hace el propio pfSense**, así que no hace falta un contenedor para él.

{% hint style="info" %}
CAPTURA PENDIENTE: la foto de la pizarra con el diseño objetivo (archivo 01).
{% endhint %}

```mermaid
flowchart TB
    CENTRO(("Red del centro<br/>192.168.135.0/24"))
    PF{{"pfSense (VM)<br/>DNS · DHCP por MAC · reenvío de puertos · firewall"}}
    WEB["ct-web<br/>nginx + PHP · VLAN 60"]
    DB[("ct-db<br/>MariaDB · VLAN 20")]
    BK["ct-backup<br/>rsync + cron · VLAN 20"]
    CL["vm-cliente<br/>VLAN 40"]

    CENTRO -- "vmbr0" --> PF
    PF -- "vmbr1" --> WEB
    PF -- "vmbr1" --> DB
    PF -- "vmbr1" --> BK
    PF -- "vmbr1" --> CL
```

## Estado

| Elemento                                       | Resultado                        |
| ---------------------------------------------- | -------------------------------- |
| Red del centro → `vmbr0` → pfSense             | Hecho: la WAN recibe IP por DHCP |
| VLANs 10, 20, 30, 40, 50 y 60 sobre `vmbr1`    | Hecho                            |
| Reglas del firewall por VLAN                   | Hecho                            |
| DHCP por MAC (VLAN 40) y DNS en pfSense        | Hecho                            |
| `ct-web` (NGINX y PHP-FPM) y `ct-db` (MariaDB) | Hecho                            |
| Reenvío de puertos hacia NGINX                 | Hecho                            |
| `ct-backup`                                    | Pendiente                        |

***

## 1. pfSense

La WAN va por `vmbr0` y recibe su IP por DHCP del centro. Las seis VLANs internas llegan por una única tarjeta virtual, que no lleva etiqueta de VLAN, y pfSense crea una interfaz por cada una.

{% hint style="info" %}
CAPTURA PENDIENTE: el estado de la WAN de pfSense, con su IP, puerta de enlace y DNS (archivo 02).
{% endhint %}

| Interfaz | Descripción     | IP             |
| -------- | --------------- | -------------- |
| WAN      | Red del centro  | Por DHCP       |
| LAN      | MGMT (VLAN 30)  | `10.0.30.1/24` |
| OPT1     | DMZ (VLAN 10)   | `10.0.10.1/24` |
| OPT2     | SERV (VLAN 20)  | `10.0.20.1/24` |
| OPT3     | USERS (VLAN 40) | `10.0.40.1/24` |
| OPT4     | ATK (VLAN 50)   | `10.0.50.1/24` |
| OPT5     | WEB (VLAN 60)   | `10.0.60.1/24` |

### Reglas del firewall

pfSense bloquea por defecto todo lo que no esté permitido, y las reglas se evalúan de arriba abajo. En cada VLAN se permite el ping a la puerta de enlace y el DNS de pfSense, se bloquea el acceso a la administración de pfSense y se permite la salida a internet (todo lo que no sea la red interna, con el alias `RED_INTERNA` invertido). La VLAN 40 puede abrir la web y la VLAN 60 puede consultar la base de datos. La WAN, la DMZ y la VLAN del atacante no llevan reglas, así que lo bloquean todo.

| VLAN       | Reglas, de arriba abajo                                                                                        |
| ---------- | -------------------------------------------------------------------------------------------------------------- |
| SERV (20)  | Ping a pfSense, DNS a pfSense, bloqueo a pfSense, salida a internet                                            |
| USERS (40) | Ping a pfSense, DNS a pfSense, web (puertos 80 y 443) hacia `10.0.60.10`, bloqueo a pfSense, salida a internet |
| WEB (60)   | Ping a pfSense, DNS a pfSense, base de datos (3306) hacia `10.0.20.11`, bloqueo a pfSense, salida a internet   |

{% hint style="info" %}
CAPTURA PENDIENTE: la lista de reglas de SERV, de USERS y de WEB (archivos 03, 04 y 05).
{% endhint %}

### DHCP por MAC y DNS

El servidor DHCP de pfSense reparte las IPs de la VLAN 40 del `.100` al `.200` y asigna una IP fija a cada equipo conocido según su dirección MAC. Los servidores llevan IP fija configurada en Proxmox, para que sigan funcionando si el DHCP falla.

El DNS Resolver de pfSense reenvía las consultas al DNS del centro y resuelve los nombres del laboratorio (`honeynet.lan`) con _host overrides_: `ct-web`, `www`, `ct-db` y `ct-backup`.

La prueba desde la VM cliente: recibe `10.0.40.10`, la IP asignada a su MAC, y el DNS de pfSense resuelve `www.honeynet.lan` a `10.0.60.10`, con una respuesta autoritativa.

{% hint style="info" %}
CAPTURA PENDIENTE: el `ip -br a` del cliente y la consulta `dig` a `10.0.30.1` (archivo 06) y a `10.0.40.1` (archivo 12).
{% endhint %}

### Reenvío de puertos

La web se publica por la WAN de pfSense con una regla de NAT (puerto 80 de la WAN hacia `10.0.60.10`) y su regla asociada en la WAN, limitada a las redes del centro con un alias.

{% hint style="info" %}
CAPTURA PENDIENTE: la regla de NAT y la regla asociada de la WAN (archivos 13 y 14).
{% endhint %}

Desde un PC de otra red del centro:

```
PS> Test-NetConnection 192.168.135.31 -Port 80
TcpTestSucceeded : True
```

***

## 2. Contenedores `ct-db` y `ct-web`

Los dos son contenedores LXC sin privilegios, con 256 MiB de RAM, en `vmbr1` con su VLAN y su IP fija, y usan a pfSense como puerta de enlace y como DNS.

| Contenedor | ID  | VLAN | IP              | Disco |
| ---------- | --- | ---- | --------------- | ----- |
| `ct-db`    | 211 | 20   | `10.0.20.11/24` | 3 GiB |
| `ct-web`   | 610 | 60   | `10.0.60.10/24` | 2 GiB |

{% hint style="info" %}
CAPTURA PENDIENTE: los ajustes de creación de `ct-db` y de `ct-web` (archivos 07 y 08).
{% endhint %}

Salen a internet a través de pfSense, sin tarjetas temporales: el `ping` a la puerta de enlace y a `1.1.1.1` responde y `apt update` funciona.

{% hint style="info" %}
CAPTURA PENDIENTE: la consola de cada contenedor con su IP y los `ping` (archivos 09 y 10).
{% endhint %}

## 3. Base de datos

En `ct-db` se instala MariaDB, que escucha solo en su IP interna, y se carga el script `honeypot_web.sql`, que crea las tablas, las vistas, los procedimientos, los triggers y los eventos. La web usa un usuario que solo puede ejecutar procedimientos (ver [Diagrama de la base de datos](../introduccion-al-proyecto/arquitectura-del-software/3.-diagrama-de-la-base-de-datos.md)).

{% hint style="info" %}
CAPTURA PENDIENTE: el resultado de `SHOW TABLES`, con las tablas y las vistas creadas (archivo 11).
{% endhint %}

## 4. Prueba de punta a punta

Desde la VM cliente (VLAN 40), el navegador abre `www.honeynet.lan` escribiendo `http://` delante: el DHCP da la IP, el DNS de pfSense resuelve el nombre, el firewall deja pasar el puerto 80, NGINX ejecuta PHP y PHP llama a un procedimiento de la base de datos, que devuelve los tres planes con sus características.

{% hint style="info" %}
CAPTURA PENDIENTE: la web abierta en el navegador de la VM cliente, con los tres planes.
{% endhint %}

***

## Problemas encontrados y soluciones

| Problema                                               | Causa                                                          | Solución                                                                                     |
| ------------------------------------------------------ | -------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| El DNS Resolver no se podía activar en modo reenvío    | Faltaba un servidor DNS en la configuración general de pfSense | Escribir el DNS del centro en _System → General Setup_                                       |
| Los alias no aparecían en el desplegable de las reglas | Los alias no salen en la lista de tipos                        | Elegir _Address or Alias_ y escribir el nombre                                               |
| `www.honeynet.lan` no cargaba en el navegador          | Firefox lo buscaba en Google                                   | Escribirlo con `http://` delante                                                             |
| Error 500 en la web                                    | PHP no podía conectar a la base de datos: _Access denied_      | Cambiar la contraseña del usuario de la web y desactivar la resolución de nombres en MariaDB |
| La web no se veía desde otro PC                        | La regla de NAT estaba deshabilitada                           | Activarla                                                                                    |
| El origen del reenvío no coincidía                     | En la regla, el origen era una dirección suelta, sin máscara   | Usar un alias con las redes del centro                                                       |

## Pendiente

* `ct-backup` (rsync y cron).
* Borrar el contenedor `ct-dns-dhcp`, que ya no se usa.
* Pedir al centro una reserva de IP para la WAN de pfSense.
