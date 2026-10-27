---
description: >-
  Cómo se crea, prepara y clona la plantilla de VM Debian 13, con cada ajuste
  explicado.
---

# Plantilla de VM Debian 13

## Introducción

Continuación de [Plantillas y configuración en Proxmox](plantillas-y-configuracion-en-proxmox.md): una vez preparado Proxmox y descargada la ISO de Debian 13, aquí se crea la **plantilla de VM** `debian13-base`, explicando qué significa cada ajuste.

***

<figure><img src="../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

## 1. Crear la VM

Se pulsa **Create VM** y se rellenan las pestañas:

| Pestaña | Ajuste                  | Valor                                    | Por qué                                                                    |
| ------- | ----------------------- | ---------------------------------------- | -------------------------------------------------------------------------- |
| General | Nombre y VM ID          | `debian13-base`, `9000`                  | El ID 9000 distingue la plantilla del resto de máquinas                    |
| OS      | ISO image               | La ISO de Debian 13                      | Es el instalador                                                           |
| System  | Qemu Agent              | Marcado                                  | Permite a Proxmox ver la IP de la VM y apagarla limpiamente                |
| Disks   | Tamaño y almacenamiento | 8 GiB en `local-lvm`                     | El sistema mínimo ocupa poco                                               |
| CPU     | Núcleos                 | 2                                        | Suficiente para instalar y para los clones                                 |
| Memory  | RAM                     | 768 MiB al instalar; 512 MiB al terminar | La plantilla no se enciende, así que no gasta RAM. Cada clon lleva la suya |
| Network | Bridge y modelo         | `vmbr0`, VirtIO                          | `vmbr0` es el switch virtual; VirtIO es la tarjeta virtual más rápida      |

Se desmarca _Start after created_ antes de terminar.

{% hint style="info" %}
CAPTURA PENDIENTE: resumen del asistente de creación de la VM.
{% endhint %}

## 2. Instalar Debian

La VM se enciende y se abre la **Console**. Se usa el instalador normal con idioma, país y teclado en español, el nombre de equipo `debian13-base`, una contraseña de root y un usuario normal. El particionado es _Guiado: usar todo el disco_ con todos los ficheros en una partición.

En la **selección de software**, se desmarcan _Debian desktop environment_ y _GNOME_, y se dejan solo **SSH server** y **standard system utilities**. Así el sistema queda ligero y sin interfaz gráfica.

Al terminar, en **Hardware → CD/DVD Drive** se elige _Do not use any media_, para que la VM no vuelva a arrancar el instalador.

## 3. Preparar el sistema

Como `root`, dentro de la VM:

```bash
apt update && apt full-upgrade -y
apt install qemu-guest-agent sudo curl vim -y
```

El paquete `qemu-guest-agent` es la parte de la VM que se comunica con Proxmox.

## 4. Limpiar la VM antes de convertirla

**Concepto:** si no se limpia, todos los clones compartirían el mismo identificador de equipo y las mismas claves SSH, y se comportarían como una sola máquina.

```bash
apt clean
truncate -s 0 /etc/machine-id
rm -f /etc/ssh/ssh_host_*
history -c
poweroff
```

| Comando                         | Qué hace                                                                      |
| ------------------------------- | ----------------------------------------------------------------------------- |
| `apt clean`                     | Borra los paquetes descargados, para que el disco ocupe menos                 |
| `truncate -s 0 /etc/machine-id` | Vacía el identificador único del equipo; cada clon genera el suyo al arrancar |
| `rm -f /etc/ssh/ssh_host_*`     | Borra las claves SSH del equipo, para que no se repitan en los clones         |
| `history -c`                    | Borra el historial de comandos                                                |
| `poweroff`                      | Apaga la VM                                                                   |

Con la VM apagada, se baja la memoria a 512 MiB en **Hardware → Memory**.
