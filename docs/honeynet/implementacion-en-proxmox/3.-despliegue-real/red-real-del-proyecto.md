---
description: >-
  Visión general de la red definitiva del proyecto: qué se ha montado, cómo
  encaja y en qué estado está cada pieza.
---

# Red real del proyecto

## Objetivo

Montar la red que se dibujó en la pizarra y que será la red del proyecto: la red del centro entra por `vmbr0` a **pfSense**, que hace el reenvío de puertos, las VLANs y el DHCP por MAC, y por `vmbr1` cuelgan los servicios (NGINX, base de datos y copias). El **DNS lo hace el propio pfSense**, así que no hace falta un contenedor para él.

<figure><img src="../../../.gitbook/assets/image (63).png" alt="" width="431"><figcaption></figcaption></figure>

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
| `ct-backup`                                    | Hecho                            |

## Cómo se organiza esta parte

1. [Red con pfSense](red-con-pfsense/): las VLANs y el firewall, el DHCP por MAC y el DNS, y el reenvío de puertos que publica la web.
2. [Contenedores](contenedores/): los servidores `ct-db` y `ct-web`, la base de datos y las copias de seguridad con `ct-backup`.
3. [Prueba de punta a punta y problemas](prueba-de-punta-a-punta-y-problemas.md): la web completa desde la VM cliente y lo que falló por el camino.
