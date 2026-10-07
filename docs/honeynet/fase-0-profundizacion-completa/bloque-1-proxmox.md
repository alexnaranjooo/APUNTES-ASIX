# Bloque 1 — Proxmox

### Por qué existe y qué problema resuelve

Con VirtualBox, el sistema operativo base ya consume RAM y CPU de fondo, y VirtualBox reparte entre las VMs lo que sobra. Para un proyecto que necesitamos **10 o más VMs corriendo a la vez**, esto es ineficiente e insuficiente.

**Proxmox** elimina esa capa intermedia, se instala como si fuera el propio sistema operativo del PC. No hay "Windows + programa de virtualización" — hay un servidor dedicado exclusivamente a crear y gestionar máquinas virtuales.&#x20;

Todo el hardware (RAM, CPU, disco) se reparte directamente entre las VMs y se gestiona automáticamente.



<figure><img src="../../.gitbook/assets/1200_628_Proxmox-vs.-Vmware@2x.png" alt=""><figcaption></figcaption></figure>



### Conceptos clave

<details>

<summary></summary>



</details>

* **Node**: el PC físico. En un entorno de empresa puede haber un clúster de varios nodes; en este proyecto hay uno solo, y todo se gestiona "dentro" de él.
* **VM (máquina virtual)**: un ordenador completo simulado, con su propia CPU asignada, RAM, disco virtual y sistema operativo instalado desde cero. Pesada pero total: permite instalar Ubuntu, pfSense, Kali, lo que sea.
* **LXC Container**: un "mini-sistema" que comparte el kernel de Linux de Proxmox en vez de tener el suyo propio. Mucho más ligero (puede arrancar con 128–256 MB de RAM frente a 1–2 GB de una VM). A cambio, solo sirve para sistemas Linux y está algo menos aislado. Ideal para servicios poco exigentes como **DNS** o **DHCP**.
* **Storage**: dónde se guardan los discos virtuales de las VMs — en este caso, el SSD del PC. Proxmox permite organizar varios storages si hay varios discos.
* **Bridge (vmbr)**: concepto clave para todo el proyecto. Un bridge es un **switch virtual**: cuando se conecta la tarjeta de red de una VM a un bridge, esa VM queda en la misma red que cualquier otra VM conectada al mismo bridge. Se creará un bridge por cada VLAN (`vmbr0` para WAN, `vmbr1` para DMZ, etc.) en la Fase 2, pero el concepto nace aquí.
*   **Snapshot**: una foto congelada del estado completo de una VM (disco + configuración). Si algo se rompe probando pfSense, se restaura el snapshot y se vuelve atrás en segundos sin reinstalar nada.

    > **Buena práctica**: tomar snapshot antes de cualquier cambio importante. Ahorra horas de frustración.
* **Template**: una VM "maestra" ya configurada que se clona para crear VMs nuevas rápidamente. Por ejemplo: instalar Ubuntu Server una vez, dejarlo limpio y actualizado, convertirlo en template, y clonar desde ahí cada vez que se necesite una VM Ubuntu nueva.

#### Cómo se ve en la práctica

Todo se gestiona desde el navegador, accediendo a `https://IP-DEL-PC:8006`. Ahí aparece un árbol a la izquierda con el node y, dentro de él, todas las VMs y containers creados.
