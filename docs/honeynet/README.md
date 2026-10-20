---
description: Hecho por Álex Naranjo Blázquez y David Álvarez Fernández
cover: ../.gitbook/assets/image (16).png
coverY: 0
layout:
  width: default
  cover:
    visible: true
    size: hero
    mask: none
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
  anchors:
    visible: true
---

# 🍯 HoneyNet

Este repositorio de Gitbook, es un repositorio orientado a mi proyecto del segundo grado del ciclo de Administración de Sistemas Informáticos en Red (ASIR) con perfil en ciberseguridad.

{% hint style="info" %}
**Estado actual del proyecto**: en desarrollo activo

**Fase**: F0 — Profundización y F1 — Instalación de Proxmox (en curso)
{% endhint %}

En el repositorio se incluirá la memoria del proyecto, junto a todos los aspectos y acciones que he ido tomando durante el transcurso del mismo.

## Infraestructura de un vistazo

```mermaid
flowchart LR
    NET(("Internet / WAN"))
    SW["Switch HP 1810-24<br/>VLANs 10, 20, 30, 40 y 99"]
    PF["PC pfSense + Suricata<br/>firewall · router · IDS<br/>(1 NIC, trunk)"]
    PVE["PC Proxmox<br/>todas las VMs:<br/>honeypots, servidores,<br/>gestión, usuarios y Kali"]

    NET -.-> PF
    PF <== "trunk" ==> SW
    SW <== "VLANs" ==> PVE
```

Más detalle en [Infraestructura física](introduccion-al-proyecto/infraestructura-fisica.md) y en el [Diagrama de red](introduccion-al-proyecto/diagrama-de-red.md).
