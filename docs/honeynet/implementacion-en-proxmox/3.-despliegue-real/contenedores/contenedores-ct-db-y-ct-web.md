---
description: >-
  Los contenedores de la base de datos y de la web: recursos, red y salida a
  internet a través de pfSense.
---

# Contenedores ct-db y ct-web

Los dos son contenedores LXC sin privilegios, con 256 MiB de RAM, en `vmbr1` con su VLAN y su IP fija, y usan a pfSense como puerta de enlace y como DNS.

| Contenedor | ID  | VLAN | IP              | Disco |
| ---------- | --- | ---- | --------------- | ----- |
| `ct-db`    | 211 | 20   | `10.0.20.11/24` | 3 GiB |
| `ct-web`   | 610 | 60   | `10.0.60.10/24` | 2 GiB |

<figure><img src="../../../../.gitbook/assets/07-ct-db-ajustes.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/08-ct-web-ajustes.png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$warning;">**Salida a internet**</mark>

Salen a internet a través de pfSense, sin tarjetas temporales: el `ping` a la puerta de enlace y a `1.1.1.1` responde y `apt update` funciona.

<figure><img src="../../../../.gitbook/assets/09-ct-db-red.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/10-ct-web-red.png" alt=""><figcaption></figcaption></figure>
