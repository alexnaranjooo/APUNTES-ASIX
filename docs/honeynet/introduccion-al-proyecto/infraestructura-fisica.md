# Infraestructura física

## Introducción

Esta página describe el **hardware real** del laboratorio de HoneyNet: qué equipos se usan, cómo se conectan y qué decisiones se han tomado. Complementa al [Diagrama de red](diagrama-de-red.md), que muestra la arquitectura lógica (VLANs y reglas de tráfico).

***

## Inventario de hardware

| Equipo                                | Papel en el proyecto                                                                                          | Estado                                   |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| **PC con Proxmox**                    | Hipervisor instalado como sistema operativo. Aloja **todas las VMs** del laboratorio. Está siempre encendido. | ✅ En uso                                 |
| **PC dedicado a pfSense + Suricata**  | Firewall, router e IDS en **hardware real** (no en VM). Tiene **una sola tarjeta de red**.                    | 🟡 Reservado (se configura en la Fase 2) |
| **Switch HP 1810-24** (J9801A)        | Conecta los dos PCs y transporta las VLANs. Se encuentra en el centro.                                        | 🟡 Reservado (se configura en la Fase 2) |
| **Juniper NetScreen-25** (NS-025-005) | Firewall físico prestado por el centro.                                                                       | ⏸️ Aparcado                              |

{% hint style="warning" %}
Los dos PCs, el switch y el firewall son **material del centro**, no propio.
{% endhint %}

***

## Topología física

```mermaid
flowchart LR
    subgraph LAB["Laboratorio (centro)"]
        SW["Switch HP 1810-24<br/>VLANs 10, 20, 30, 40 y 99"]
        PFPC["PC pfSense + Suricata<br/>1 NIC · trunk"]
        PVE["PC Proxmox<br/>todas las VMs"]
    end

    ADMIN(("Administrador<br/>en remoto"))

    PFPC <-- "trunk" --> SW
    PVE <--> SW
    ADMIN -. "NetBird (VPN)" .-> LAB
```

***

## Cómo funciona con una sola tarjeta de red

El PC de pfSense solo tiene **una NIC**, y un firewall normalmente necesita al menos dos (una para WAN y otra para LAN). La solución es usar **VLANs sobre un único cable**:

* La NIC va conectada al switch en modo **trunk**: por el mismo cable viajan todas las VLANs **etiquetadas** (802.1Q).
* La **VLAN 99** actúa como la **WAN**, y las VLANs 10, 20, 30 y 40 son las redes internas.
* pfSense crea una interfaz virtual por cada VLAN sobre esa única NIC y enruta el tráfico entre ellas.

Esta técnica se conoce como **router on a stick**.

```mermaid
flowchart LR
    subgraph PFPC["PC pfSense + Suricata (una sola NIC)"]
        direction TB
        W["VLAN 99 · WAN"]
        D["VLAN 10 · DMZ"]
        S["VLAN 20 · Servidores"]
        G["VLAN 30 · Gestión"]
        U["VLAN 40 · Usuarios"]
    end
    SW["Switch HP 1810-24"]
    PVE["PC Proxmox<br/>VMs en sus VLANs"]

    PFPC == "1 cable · trunk 802.1Q" ==> SW
    SW == "VLANs etiquetadas" ==> PVE
```

{% hint style="info" %}
**Contrapartida**: todo el tráfico entre VLANs pasa por un único enlace físico. Para un laboratorio es suficiente, pero sería un cuello de botella en una red de producción.
{% endhint %}

### Recorrido de un ataque

Aunque Kali y el honeypot estén en el mismo PC Proxmox, el tráfico entre VLANs siempre tiene que pasar por pfSense:

```mermaid
sequenceDiagram
    participant K as Kali (VM, VLAN 99)
    participant SW as Switch HP 1810-24
    participant PF as pfSense + Suricata
    participant H as Honeypot (VM, VLAN 10)

    K->>SW: ataque etiquetado VLAN 99
    SW->>PF: por el trunk
    Note over PF: reglas de firewall + análisis de Suricata
    PF->>SW: reenvía a la VLAN 10 si las reglas lo permiten
    SW->>H: el tráfico llega a la DMZ
```

***

## Decisiones tomadas

* **pfSense + Suricata va en hardware real**, en el PC dedicado, no en una VM de Proxmox.
* **Un solo PC Proxmox** con todas las VMs (honeypots, servidores, gestión, usuarios y Kali).
* **WAN como VLAN 99** y resto de VLANs en trunk sobre la única NIC de pfSense.
* El **NetScreen-25 queda aparcado**: no forma parte de la arquitectura por ahora.
* **No hay portátil de administración fijo.**

***

## Acceso remoto con NetBird

Se puede trabajar en el laboratorio **desde casa** mediante **NetBird**, una VPN. Como el PC con Proxmox está siempre encendido, el laboratorio está disponible en cualquier momento.

***

## Por definir

* Reparto de puertos del switch (qué puerto va a cada PC) y configuración de las VLANs en el HP 1810-24.
* Dónde va conectada físicamente la salida a internet del laboratorio.

## Próximos pasos

* Documentar la configuración del switch en la **Fase 2**.
* Instalar y configurar NetBird para el acceso remoto.
* Actualizar esta página si cambia algún equipo o conexión.
