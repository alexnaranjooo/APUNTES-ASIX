# ✂️ Alcance y limitaciones

## Introducción

Delimita con claridad **qué incluye** HoneyNet, **qué queda fuera a propósito** y qué factores limitan el proyecto. Dejarlo por escrito evita que una decisión consciente parezca un olvido.

***

## Dentro del alcance

* Infraestructura sobre dos PCs del centro —uno con **Proxmox** (todas las VMs) y otro dedicado a **pfSense + Suricata**— conectados con un switch **HP 1810-24**.
* Segmentación en cinco VLANs con **pfSense + Suricata**.
* Servicios internos: DNS, DHCP, MySQL y copias de seguridad.
* Tres honeypots en Docker: **Cowrie**, **Dionaea** y **DVWA**.
* SIEM con **Wazuh** y visualización con **Grafana**.
* Alertas por **Telegram**.
* Exposición **controlada** de la DMZ a internet.
* Acceso remoto al laboratorio con **NetBird** (VPN).
* Parte web del proyecto, alojada en un servidor **nginx**.
* Documentación completa en este GitBook, memoria y defensa.

## Fuera del alcance

* Alta disponibilidad real o clúster de varios nodes Proxmox (el proyecto usa un único node).
* Entornos de producción: es un laboratorio de estudio, no una red para proteger una empresa real.
* Análisis forense profundo del malware capturado (se guardan las muestras, pero no se hace ingeniería inversa).
* El firewall físico **Juniper NetScreen-25**, prestado por el centro, queda aparcado.
* Atribución de los atacantes: se registran IPs y comportamiento, no se investiga quién hay detrás.

***

## Limitaciones

### Técnicas

* **Un solo node Proxmox**: todas las VMs comparten los recursos de ese PC (RAM, CPU, disco), lo que limita cuántas pueden correr a la vez y con qué carga.
* **Una sola tarjeta de red en el PC de pfSense**: WAN y redes internas viajan por el mismo cable en trunk, con la VLAN 99 como WAN. Todo el tráfico entre VLANs pasa por un único enlace.
* **Hardware prestado**: los PCs, el switch y el firewall son del centro.
* **Sin portátil de administración fijo**.
* **Herramientas nuevas**: Proxmox, Docker, pfSense, Wazuh y Grafana se aprenden durante el proyecto, por lo que cada fase incluye curva de aprendizaje.

### De equipo y tiempo

* Equipo de **dos personas**, sin un especialista dedicado a monitorización.
* Fecha de entrega fija, lo que obliga a priorizar.

### Ambición del proyecto

* El **script de respuesta automática** es la parte más ambiciosa. Puede no completarse en esta primera versión; si ocurre, las fases anteriores siguen siendo un proyecto completo y funcional por sí mismas.

***

## Consideraciones de seguridad

* La exposición a internet se hace **solo al final** (Fase 7), tras revisar y endurecer las reglas.
* La DMZ **nunca puede iniciar conexiones** hacia otras VLANs; solo envía logs a Gestión.
* Los datos capturados (IPs, sesiones, malware) se tratan como material de estudio dentro del laboratorio.
* Las muestras de malware **no deben ejecutarse** fuera de un entorno aislado.

> El razonamiento detrás de estas reglas está en Diagrama de red y en la Fase 0.
