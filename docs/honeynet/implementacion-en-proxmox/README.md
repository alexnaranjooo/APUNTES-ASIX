---
description: >-
  Documentación de lo que se va construyendo en Proxmox: cómo se preparan las
  plantillas y las máquinas, las redes internas y las prácticas que se hacen con
  ellas.
---

# Implementación en Proxmox

## Introducción

Esta sección recoge **lo que se hace de verdad en Proxmox**, paso a paso, con las configuraciones explicadas. Está separada de la _Introducción al proyecto_ porque aquí no se describe el diseño, sino la **implementación**: lo que se ha montado, cómo y por qué se ha configurado así.

Se organiza en tres partes:

**Plantillas y configuración en Proxmox:** qué es una plantilla, cómo se prepara cada tipo y qué significa cada ajuste de una máquina virtual y de un contenedor, y cómo se protegen las máquinas con copias.

* [Plantillas y configuración en Proxmox](plantillas-y-configuracion-en-proxmox.md): conceptos, preparación de Proxmox y descargas.
* [Plantilla de VM Debian 13](plantilla-de-vm-debian-13.md): creación, instalación y limpieza de la VM.
* [Clones y contenedor LXC](clones-y-contenedor-lxc.md): plantilla, clonado y contenedor.
* [Copias de seguridad y snapshots](copias-de-seguridad-y-snapshots.md): snapshots, copia programada, restauración de prueba y copia fuera del PC.

**Práctica:** una práctica pequeña que usa esas plantillas para demostrar cómo funcionan y cómo se conectan una VM y un contenedor.

* [Práctica: VM con nginx y contenedor LXC](practica-vm-con-nginx-y-contenedor-lxc.md)

**Red interna y reenvío de puertos:** una red aislada con VLANs dentro de Proxmox y cómo se publica una web interna a través de un contenedor, como hará pfSense.

* [Red interna con VLAN](red-interna-con-vlan-vmbr1.md): el bridge `vmbr1` y la VLAN 60.
* [Reenvío de puertos con iptables](reenvio-de-puertos-con-iptables.md): DNAT, MASQUERADE y denegar por defecto.
* [iptables: persistencia e IP fija](iptables-persistencia-e-ip-fija.md): guardar las reglas y fijar la IP externa.

{% hint style="info" %}
El hardware y el diseño de la red están en [Infraestructura física](../introduccion-al-proyecto/infraestructura-fisica.md) y en [Diagrama de red](../introduccion-al-proyecto/diagrama-de-red.md). El plan de trabajo por fases está en la [Checklist del proyecto](../introduccion-al-proyecto/checklist-estado-del-proyecto.md).
{% endhint %}
