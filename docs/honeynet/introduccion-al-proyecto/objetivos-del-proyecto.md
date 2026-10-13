# 🎯 Objetivos del proyecto

## **Introducción**

Esta página define qué se quiere conseguir con **HoneyNet** y cómo se comprobará que se ha conseguido. Sirve de referencia durante todo el proyecto: cada decisión técnica debería poder justificarse por alguno de estos objetivos.

***

## **Objetivo general**

Diseñar, desplegar y documentar una **red de laboratorio segmentada** capaz de **atraer, registrar y analizar ataques reales** mediante honeypots, con un sistema SIEM que permita estudiar cómo actúan los atacantes.



## **Objetivos específicos**

### Infraestructura

* Montar un **Proxmox** como base de todo el laboratorio.
* Segmentar la red en **cinco VLANs** (DMZ, Servidores, Gestión, Usuarios y WAN) con **pfSense** como firewall y router.
* Aplicar **defensa en profundidad**: que comprometer una pieza no dé acceso a las demás.

### Captura de ataques

* Desplegar tres honeypots en Docker: **Cowrie** (SSH/Telnet), **Dionaea** (SMB/FTP/MySQL) y **DVWA** (aplicación web vulnerable).
* Conservar los logs y las muestras de malware en volúmenes persistentes.
* Detectar tráfico sospechoso en tiempo real con **Suricata**.

### Análisis y visualización

* Centralizar y correlacionar eventos con **Wazuh**.
* Guardar cada ataque en **MySQL** para poder consultarlos de forma estructurada.
* Visualizar los datos con **Grafana** (dashboards y mapa de atacantes).

### Automatización

* Enviar alertas por **Telegram**.
* Desarrollar un **script en Python** que lea los logs de Cowrie, consulte AbuseIPDB, guarde los datos en MySQL y bloquee IPs en pfSense vía API.

### Criterios de éxito

El proyecto se considerará logrado si:

* [ ] Las cinco VLANs están operativas y el aislamiento entre ellas se ha **probado** (por ejemplo, desde Kali).
* [ ] Los tres honeypots registran ataques y los logs llegan a Wazuh.
* [ ] Wazuh genera alertas de correlación (como fuerza bruta) ante ataques de prueba.
* [ ] Grafana muestra datos reales de los ataques capturados.
* [ ] Se ha expuesto la DMZ a internet de forma controlada y **ningún atacante ha alcanzado las VLANs internas**.
* [ ] Toda la arquitectura está documentada y justificada.

> El script de respuesta automática (bloqueo de IPs vía API) es el objetivo más ambicioso. Su estado y por qué puede quedar fuera de la primera versión se explican en Alcance y limitaciones.
