---
description: >-
  Documentación de lo que se va construyendo en Proxmox, en tres partes:
  plantillas y configuración básica, pruebas y despliegue real con pfSense
---

# Implementación en Proxmox

## <mark style="color:$warning;">**Introducción**</mark>

Esta sección recoge **lo que se hace de verdad en Proxmox**, paso a paso, con las configuraciones explicadas. Está separada de la _Introducción al proyecto_ porque aquí no se describe el diseño, sino la **implementación**: lo que se ha montado, cómo y por qué se ha configurado así.

Se organiza en tres partes, en el orden en que se fue haciendo:

[**1. Plantillas y configuración básica**](1.-plantillas-y-configuracion-basica/)**:** qué es una plantilla, cómo se prepara cada tipo, qué significa cada ajuste de una máquina virtual y de un contenedor, y cómo se protegen las máquinas con copias.

* [Plantillas y configuración en Proxmox](1.-plantillas-y-configuracion-basica/plantillas-y-configuracion-en-proxmox.md): conceptos, preparación de Proxmox y descargas.
* [Plantilla de VM Debian 13](1.-plantillas-y-configuracion-basica/plantilla-de-vm-debian-13.md): creación, instalación y limpieza de la VM.
* [Clones y contenedor LXC](1.-plantillas-y-configuracion-basica/clones-y-contenedor-lxc.md): plantilla, clonado y contenedor.
* [Copias de seguridad y snapshots](1.-plantillas-y-configuracion-basica/copias-de-seguridad-y-snapshots.md): snapshots, copia programada, restauración de prueba y copia fuera del PC.

[**2. Pruebas**](2.-pruebas/)**:** prácticas con máquinas de prueba para entender los mecanismos antes de montarlos de verdad. Se explican igualmente, aunque en la red definitiva las sustituye pfSense.

* [Práctica: VM con nginx y contenedor LXC](2.-pruebas/practica-vm-con-nginx-y-contenedor-lxc.md): una VM y un contenedor conectados por el bridge.
* [Red interna con VLAN](2.-pruebas/red-interna-con-vlan-vmbr1.md): el bridge `vmbr1` y la VLAN 60.
* [Reenvío de puertos con iptables](2.-pruebas/reenvio-de-puertos-con-iptables.md): DNAT, MASQUERADE y denegar por defecto.
* [iptables: persistencia e IP fija](2.-pruebas/iptables-persistencia-e-ip-fija.md): guardar las reglas y fijar la IP externa.

[**3. Despliegue real**](3.-despliegue-real/)**:** la red definitiva con pfSense, el DNS y el DHCP por MAC, los contenedores de la web y de la base de datos, y la publicación de la web.

* [Red real del proyecto](3.-despliegue-real/red-real-del-proyecto.md): pfSense, reglas, contenedores, base de datos y prueba de punta a punta.

{% hint style="info" %}
El hardware y el diseño de la red están en [Infraestructura física](../introduccion-al-proyecto/infraestructura-fisica.md) y en [Diagrama de red](../introduccion-al-proyecto/diagrama-de-red.md). El plan de trabajo por fases está en la [Checklist del proyecto](../introduccion-al-proyecto/checklist-estado-del-proyecto.md).
{% endhint %}
