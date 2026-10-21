# ✂️ Alcance y limitaciones

## Introducción

Delimita con claridad **qué incluye** HoneyNet, **qué queda fuera a propósito** y qué factores limitan el proyecto. Dejarlo por escrito evita que una decisión consciente parezca un olvido.

***

## Dentro del alcance

* Infraestructura sobre dos PCs del centro, cada uno con **Proxmox**: el **PC 1** con pfSense + Suricata (en una VM), honeypots, servidores, web y usuario, y el **PC 2** con el SIEM y Kali. Conectados con un switch **HP 1810-24**.
* Segmentación en seis VLANs internas (más la WAN) con **pfSense + Suricata**.
* Servicios internos: DNS, DHCP, MariaDB y copias de seguridad.
* Tres honeypots en Docker: **Cowrie**, **Dionaea** y **DVWA**.
* SIEM con **Wazuh completo** y visualización con **Grafana**.
* Alertas por **Telegram** desde Grafana.
* Exposición **controlada** de honeypots a internet mediante una VPS.
* Acceso remoto al laboratorio con **NetBird** (VPN).
* Parte web del proyecto, alojada en un servidor **nginx**.
* Documentación completa en este GitBook, memoria y defensa.

## Fuera del alcance

* Alta disponibilidad real o clúster: los dos Proxmox son **independientes**, sin clúster.
* Entornos de producción: es un laboratorio de estudio, no una red para proteger una empresa real.
* Análisis forense profundo del malware capturado (se guardan las muestras, pero no se hace ingeniería inversa).
* El firewall físico **Juniper NetScreen-25**, prestado por el centro, queda aparcado.
* **Bloqueo automático de IPs** en pfSense: es un extra opcional si sobra tiempo.
* Atribución de los atacantes: se registran IPs y comportamiento, no se investiga quién hay detrás.

***

## Limitaciones

### Técnicas

* **Poca memoria y disco**: cada PC tiene **8 GB de RAM y un SSD de 128 GB**, y el centro no puede ampliarlos. Obliga a usar contenedores ligeros, a limitar la retención de datos de Wazuh y a encender Kali solo durante las pruebas.
* **pfSense en una VM** del PC 1, con una sola tarjeta de red: WAN y redes internas viajan por el mismo cable en trunk, con la VLAN 99 como WAN. Todo el tráfico entre VLANs pasa por un único enlace.
* **Wazuh completo es la pieza más pesada**: funciona, pero con poco margen. El plan B es una versión ligera (solo el manager con Loki y Grafana).
* **Hardware prestado**: los PCs, el switch y el firewall son del centro, y podrían reiniciarse o reinstalarse. Las configuraciones se guardan fuera de los equipos.
* **Sin portátil de administración fijo**: se administra desde un puerto del switch en la VLAN 30.
* **Herramientas nuevas**: Proxmox, Docker, pfSense, Wazuh y Grafana se aprenden durante el proyecto, por lo que cada fase incluye curva de aprendizaje.

### De equipo y tiempo

* Equipo de **dos personas**, sin un especialista dedicado a monitorización.
* Fecha de entrega fija (febrero-marzo), lo que obliga a priorizar.

### Ambición del proyecto

* El **bloqueo automático de IPs** es la parte más ambiciosa. Puede no completarse en esta primera versión; si ocurre, las fases anteriores siguen siendo un proyecto completo y funcional por sí mismas.

***

## Consideraciones de seguridad

* La exposición a internet se hace **solo al final** (Fase 7), desde una VPS y tras revisar y endurecer las reglas.
* La DMZ **nunca puede iniciar conexiones** hacia otras VLANs, salvo dos flujos documentados: logs a Wazuh y MariaDB.
* Los datos capturados (IPs, sesiones, malware) se tratan como material de estudio dentro del laboratorio.
* Las muestras de malware **no deben ejecutarse** fuera de un entorno aislado.

> El razonamiento detrás de estas reglas está en Diagrama de red y en la Fase 0.
