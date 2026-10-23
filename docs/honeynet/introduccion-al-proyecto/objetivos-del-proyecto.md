# 🎯 Objetivos del proyecto

## **Introducción**

Esta página define qué se quiere conseguir con **HoneyNet** y cómo se comprobará que se ha conseguido. Sirve de referencia durante todo el proyecto: cada decisión técnica debería poder justificarse por alguno de estos objetivos.

***

## **Objetivo general**

Diseñar, desplegar y documentar una **red de laboratorio segmentada** capaz de **atraer, registrar y analizar ataques reales** mediante honeypots, con un sistema SIEM que permita estudiar cómo actúan los atacantes.

## **Objetivos específicos**

### Infraestructura

* Montar **Proxmox** en dos PCs del centro, como base de todas las VMs y contenedores del laboratorio.
* Segmentar la red en **seis VLANs internas** (DMZ, Servidores, Gestión, Usuarios, Atacante y Web) más la WAN, con **pfSense + Suricata** (en una VM) como firewall, router e IDS, conectado todo mediante un switch **HP 1810-24**.
* Aplicar **defensa en profundidad**: que comprometer una pieza no dé acceso a las demás.

El hardware se detalla en [Infraestructura física](infraestructura-fisica.md).

### Captura de ataques

* Desplegar tres honeypots en Docker: **Cowrie** (SSH/Telnet), **Dionaea** (SMB/FTP/MySQL) y **DVWA** (aplicación web vulnerable).
* Conservar los logs y las muestras de malware en volúmenes persistentes.
* Detectar tráfico sospechoso en tiempo real con **Suricata**.

### Análisis y visualización

* Centralizar y correlacionar eventos con **Wazuh** (completo, con su panel).
* Visualizar los datos con **Grafana** (dashboards, tabla de logs y mapa de atacantes) a partir de Wazuh.

### Alertas y automatización

* Enviar alertas por **Telegram** desde Grafana.
* _Extra opcional:_ un **script en Python** que consulte AbuseIPDB y bloquee IPs en pfSense vía API.

### Parte web

* Entregar la actividad **M14UF1A2** y alojar la web de la empresa del proyecto en un servidor **nginx** con PHP y **MariaDB** (ver [Parte web](parte-web/) y [Arquitectura del software](arquitectura-del-software/)).
* Ofrecer un **login de administrador** desde el que ver los logs y los gráficos de Grafana de forma segura.

### Criterios de éxito

El proyecto se considerará logrado si:

* [ ] Las seis VLANs internas y la WAN están operativas y el aislamiento entre ellas se ha **probado** (por ejemplo, desde Kali).
* [ ] Los tres honeypots registran ataques y los logs llegan a Wazuh.
* [ ] Wazuh genera alertas de correlación (como fuerza bruta) ante ataques de prueba.
* [ ] Grafana muestra datos reales de los ataques capturados.
* [ ] La web funciona y el administrador ve los logs y gráficos solo tras iniciar sesión y desde la red interna.
* [ ] Se han capturado ataques reales de forma controlada (desde una VPS) y **ningún atacante ha alcanzado las VLANs internas**.
* [ ] Toda la arquitectura está documentada y justificada.

> El bloqueo automático de IPs vía API es un extra opcional y el objetivo más ambicioso. Su estado y por qué puede quedar fuera de la primera versión se explican en [Alcance y limitaciones](alcance-y-limitaciones.md).
