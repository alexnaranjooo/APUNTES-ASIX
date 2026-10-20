---
description: >-
  Antes de tocar nada en Proxmox, esta fase sienta las bases teóricas del
  proyecto HoneyNet: qué es cada pieza del stack, para qué sirve y por qué está
  ahí. No se instala nada todavía.
---

# 0️⃣ Fase 0 — Profundización completa

## **Bloque 1 — Proxmox**

#### Por qué existe y qué problema resuelve

Con VirtualBox o VMWare, el sistema operativo base ya consume RAM y CPU de fondo, y VirtualBox reparte entre las VMs lo que sobra. Para un proyecto que necesitamos **10 o más VMs corriendo a la vez**, esto es ineficiente e insuficiente.

**Proxmox** elimina esa capa intermedia, se instala como si fuera el propio sistema operativo del PC. No hay "Windows + programa de virtualización" — hay un servidor dedicado exclusivamente a crear y gestionar máquinas virtuales. Todo el hardware (RAM, CPU, disco) se reparte directamente entre las VMs.

<figure><img src="../.gitbook/assets/image (37).png" alt="" width="563"><figcaption></figcaption></figure>
