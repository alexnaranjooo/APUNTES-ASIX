---
description: >-
  Cómo se convierte la VM en plantilla, cómo se clona y cómo se crea un
  contenedor LXC, con cada ajuste explicado.
---

# Clones y contenedor LXC

## Introducción

Continuación de [Plantilla de VM Debian 13](plantilla-de-vm-debian-13.md). La VM `debian13-base` ya está instalada, actualizada y limpia. Aquí se convierte en plantilla, se clona y se explica cómo se crea un **contenedor LXC** con la plantilla de contenedor.

***

## 1. Convertir en plantilla

Clic derecho sobre la VM → **Convert to template**. El icono cambia y la máquina ya no se puede encender: solo sirve para clonar.

## 2. Clonar la plantilla

Clic derecho sobre la plantilla → **Clone**. Hay dos modos:

| Modo             | Cómo funciona                                                         | Cuándo usarlo                                                                 |
| ---------------- | --------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| **Full Clone**   | Copia el disco entero y la máquina nueva es independiente             | Para máquinas definitivas. Es lo que usamos                                   |
| **Linked Clone** | Comparte el disco base con la plantilla y solo guarda las diferencias | Para pruebas rápidas; la plantilla no se puede borrar mientras existan clones |

Tras clonar, hay que darle identidad propia a la máquina nueva. Dentro del clon se regeneran las claves SSH, que se borraron al preparar la plantilla, y se cambia el nombre del equipo:

```bash
ssh-keygen -A
systemctl restart ssh
hostnamectl set-hostname NOMBRE-NUEVO
```

Además, en el fichero `hosts` (dentro de la carpeta `etc`) se sustituye el nombre antiguo, `debian13-base`, por el nuevo. Y en **Hardware → Memory** se ajusta la RAM a lo que necesite cada clon.

<figure><img src="../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

***

## 3. Contenedor LXC

Se crea con **Create CT**. Cada ajuste significa lo siguiente:

| Pestaña  | Ajuste                 | Valor                  | Por qué                                                                           |
| -------- | ---------------------- | ---------------------- | --------------------------------------------------------------------------------- |
| General  | Hostname               | `ct-prueba`            | Nombre del contenedor                                                             |
| General  | Unprivileged container | Marcado                | El `root` del contenedor no es `root` en el host: es más seguro                   |
| General  | Password               | Una contraseña de root | Para entrar por la consola                                                        |
| Template | Plantilla              | `debian-13-standard`   | La plantilla de contenedor descargada                                             |
| Disks    | Tamaño                 | 2 GiB en `local-lvm`   | Un servicio ligero ocupa poco                                                     |
| CPU      | Núcleos                | 1                      | Suficiente para la práctica                                                       |
| Memory   | RAM y swap             | 256 MiB y 256 MiB      | Los contenedores arrancan con muy poca memoria                                    |
| Network  | Nombre, bridge e IP    | `eth0`, `vmbr0`, DHCP  | La tarjeta del contenedor se conecta al mismo switch virtual y recibe IP del DHCP |
| DNS      | Servidores             | Por defecto            | Hereda el DNS del host                                                            |

{% hint style="warning" %}
El campo **Name** de la red debe ser `eth0`, que es el nombre habitual de la interfaz en Linux. Si se escribe otro, el contenedor puede quedarse sin IP (ver el apartado de problemas).
{% endhint %}

Al crearlo puede aparecer el aviso **«Systemd 257 detected. You may need to enable nesting»**. No es un error: el contenedor se crea igualmente. Si algún servicio fallara al arrancar, se activa en **Options → Features → Nesting**.

<figure><img src="../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

***
