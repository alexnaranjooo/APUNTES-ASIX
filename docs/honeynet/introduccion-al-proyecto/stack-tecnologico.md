# 🗃️ Stack tecnológico

## Introducción

Tabla única con todas las herramientas del proyecto: qué hace cada una, en qué fase se instala y dónde está su documentación oficial. Funciona como índice rápido del laboratorio.

***

## Resumen por capas

### Virtualización e infraestructura

| Herramienta          | Función                                                       | Fase  | Documentación                                                 |
| -------------------- | ------------------------------------------------------------- | ----- | ------------------------------------------------------------- |
| **Proxmox VE**       | Hipervisor bare-metal sobre el que corre todo el laboratorio. | F1    | [pve.proxmox.com/pve-docs](https://pve.proxmox.com/pve-docs/) |
| **Docker + Compose** | Empaqueta y despliega honeypots, Wazuh y Grafana.             | F4–F6 | [docs.docker.com](https://docs.docker.com/)                   |

### Red y seguridad perimetral

| Herramienta  | Función                                                             | Fase | Documentación                                                   |
| ------------ | ------------------------------------------------------------------- | ---- | --------------------------------------------------------------- |
| **pfSense**  | Firewall y router entre todas las VLANs (en un PC físico dedicado). | F2   | [docs.netgate.com](https://docs.netgate.com/pfsense/en/latest/) |
| **Suricata** | IDS dentro de pfSense; alerta con firmas ET Open.                   | F2   | [suricata.io](https://suricata.io/documentation/)               |

### Servicios internos

| Herramienta         | Función                                                | Fase | Documentación                                                 |
| ------------------- | ------------------------------------------------------ | ---- | ------------------------------------------------------------- |
| **Bind9**           | Servidor DNS interno.                                  | F3   | [bind9.readthedocs.io](https://bind9.readthedocs.io/)         |
| **isc-dhcp-server** | Reparte IPs en la VLAN de Usuarios.                    | F3   | [kb.isc.org](https://kb.isc.org/docs/isc-dhcp-server)         |
| **MySQL**           | Base de datos donde se guardan los ataques capturados. | F3   | [dev.mysql.com/doc](https://dev.mysql.com/doc/)               |
| **rsync + cron**    | Copias de seguridad automáticas nocturnas.             | F3   | [rsync.samba.org](https://rsync.samba.org/documentation.html) |

### Honeypots

| Herramienta | Función                                               | Fase | Documentación                                                        |
| ----------- | ----------------------------------------------------- | ---- | -------------------------------------------------------------------- |
| **Cowrie**  | Honeypot SSH/Telnet con grabación de sesiones.        | F4   | [github.com/cowrie/cowrie](https://github.com/cowrie/cowrie)         |
| **Dionaea** | Honeypot multi-protocolo para capturar malware.       | F4   | [github.com/DinoTools/dionaea](https://github.com/DinoTools/dionaea) |
| **DVWA**    | Aplicación web vulnerable para registrar ataques web. | F4   | [github.com/digininja/DVWA](https://github.com/digininja/DVWA)       |

### Monitorización y respuesta

| Herramienta          | Función                                                                 | Fase | Documentación                                                                 |
| -------------------- | ----------------------------------------------------------------------- | ---- | ----------------------------------------------------------------------------- |
| **Wazuh**            | SIEM: centraliza logs y aplica reglas de correlación.                   | F5   | [documentation.wazuh.com](https://documentation.wazuh.com/current/index.html) |
| **Grafana**          | Dashboards y mapas a partir de MySQL/Wazuh.                             | F6   | [grafana.com/docs](https://grafana.com/docs/grafana/latest/)                  |
| **Telegram Bot API** | Envío de alertas al móvil.                                              | F6   | [core.telegram.org/bots/api](https://core.telegram.org/bots/api)              |
| **Python**           | Script de respuesta: logs de Cowrie, AbuseIPDB, MySQL y API de pfSense. | F6   | [docs.python.org](https://docs.python.org/3/)                                 |
| **AbuseIPDB**        | Consulta de reputación de IPs atacantes.                                | F6   | [docs.abuseipdb.com](https://docs.abuseipdb.com/)                             |

### Hardware y acceso remoto

| Elemento                  | Función                                                 | Fase        | Documentación                                   |
| ------------------------- | ------------------------------------------------------- | ----------- | ----------------------------------------------- |
| **PC con Proxmox**        | Aloja todas las VMs del laboratorio.                    | F1          | —                                               |
| **PC pfSense + Suricata** | Firewall, router e IDS en hardware real (1 NIC, trunk). | F2          | —                                               |
| **Switch HP 1810-24**     | Conecta los PCs y transporta las VLANs.                 | F2          | —                                               |
| **NetBird**               | VPN para trabajar en el laboratorio desde casa.         | F1          | [docs.netbird.io](https://docs.netbird.io/)     |
| **nginx**                 | Servidor web para la parte web del proyecto.            | Por decidir | [nginx.org/en/docs](https://nginx.org/en/docs/) |

> Detalle del hardware en [Infraestructura física](infraestructura-fisica.md). El firewall Juniper NetScreen-25 está aparcado y no forma parte del stack.

### Máquinas auxiliares

| Herramienta          | Función                                                     | Fase           | Documentación                               |
| -------------------- | ----------------------------------------------------------- | -------------- | ------------------------------------------- |
| **Kali Linux**       | Máquina atacante en la VLAN WAN para probar el laboratorio. | F2 en adelante | [kali.org/docs](https://www.kali.org/docs/) |
| **VM Windows/Linux** | Equipo de empleado simulado en la VLAN de Usuarios.         | F3             | —                                           |

***

> Para ver dónde vive cada herramienta dentro de la red, consulta el [Diagrama de red](diagrama-de-red.md). Para entender por qué cada pieza está ahí, consulta [F0 — Profundización completa](../f0-profundizacion-completa/).

## Próximos pasos

* Añadir la versión exacta de cada herramienta cuando se instale.
* Anotar aquí cualquier herramienta nueva que se incorpore durante el proyecto.
