# Bloque 1 — Proxmox

## **Proxmox**

### Por qué existe y qué problema resuelve

Con VirtualBox, el sistema operativo base ya consume RAM y CPU de fondo, y VirtualBox reparte entre las VMs lo que sobra. Para un proyecto que necesita **varias máquinas virtuales y contenedores corriendo a la vez**, esto es ineficiente e insuficiente.

**Proxmox** elimina esa capa intermedia, se instala como si fuera el propio sistema operativo del PC. No hay "Windows + programa de virtualización" — hay un servidor dedicado exclusivamente a crear y gestionar máquinas virtuales.

Todo el hardware (RAM, CPU, disco) se reparte directamente entre las VMs y se gestiona automáticamente.

<figure><img src="../../../.gitbook/assets/1200_628_Proxmox-vs.-Vmware@2x.png" alt=""><figcaption></figcaption></figure>

### **Conceptos clave**

{% tabs %}
{% tab title="Node" %}
**Node**: el PC físico que ejecuta Proxmox. En este proyecto hay **dos nodos independientes**, uno en cada PC: `honeynet` (PC 1), con pfSense, los honeypots, los servidores y la web, y `honeynet2` (PC 2), con el SIEM y Kali.

<figure><img src="../../../.gitbook/assets/images.jpg" alt="" width="506"><figcaption></figcaption></figure>
{% endtab %}

{% tab title="VM" %}
**VM (máquina virtual)**: un ordenador completo simulado, con su propia CPU asignada, RAM, disco virtual y sistema operativo instalado desde cero, permite instalar Ubuntu, Debian, Kali, lo que sea.

<figure><img src="../../../.gitbook/assets/virtual_machines.png" alt="" width="548"><figcaption></figcaption></figure>
{% endtab %}

{% tab title="Storage" %}
**Storage**: dónde se guardan los discos virtuales de las VMs.

<figure><img src="../../../.gitbook/assets/shared_storage_XenServer_blog_virtualizacion.png" alt="" width="346"><figcaption></figcaption></figure>
{% endtab %}

{% tab title="Bridge " %}
**Bridge:** un bridge es un **switch virtual,** cuando se conecta la tarjeta de red de una VM a un bridge, esa VM queda en la misma red que cualquier otra VM conectada al mismo bridge.

En este proyecto se usan dos: `vmbr0`, unido a la tarjeta física y a la red del centro, y `vmbr1`, una red interna sin tarjeta física y compatible con VLANs, donde cada máquina elige su VLAN con una etiqueta.

<figure><img src="../../../.gitbook/assets/vn-Bridged-Mode-Diagram.png" alt="" width="563"><figcaption></figcaption></figure>
{% endtab %}

{% tab title="Snapshot" %}
**Snapshot**: una foto congelada del estado completo de una VM.

Si algo se rompe probando, se restaura el snapshot y se vuelve atrás en segundos sin reinstalar nada.

<figure><img src="../../../.gitbook/assets/vmware-snapshot-monitoring-banner.svg" alt="" width="531"><figcaption></figcaption></figure>
{% endtab %}

{% tab title="Template" %}
**Template**: una VM "maestra" ya configurada que se clona para crear VMs nuevas rápidamente.

Por ejemplo: instalar Ubuntu Server una vez, dejarlo limpio y actualizado, convertirlo en template, y clonar desde ahí cada vez que se necesite una VM Ubuntu nueva.

<figure><img src="../../../.gitbook/assets/1069465476.svg" alt="" width="513"><figcaption></figcaption></figure>
{% endtab %}
{% endtabs %}

### **Cómo se ve en la práctica**

Todo se gestiona desde el navegador, accediendo a `https://IP-DEL-PC:8006`.

<figure><img src="../../../.gitbook/assets/proxmox-intro.png" alt=""><figcaption></figcaption></figure>

### Para ampliar

* [Proxmox en detalle](/broken/pages/wSx1MTVYad00x4kcO1jh): cómo funciona por dentro, almacenamiento, red, copias, usuarios y clúster.
* [Preguntas sobre Proxmox](preguntas-sobre-proxmox.md): clúster, alta disponibilidad, hipervisores, QEMU, KVM, noVNC, Ceph, roles y almacenamiento.
