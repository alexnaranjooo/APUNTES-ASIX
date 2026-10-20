# ✔️ Checklist / estado del proyecto

## Introducción

Seguimiento del estado de **HoneyNet** fase a fase. Se actualiza a medida que se avanza: marca cada casilla cuando esté hecha y cambia el estado de la fase en la tabla resumen.

> Las subtareas de cada fase son una propuesta de trabajo basada en el stack del proyecto; ajústalas según vayas descubriendo qué hace falta de verdad en cada paso.

## Resumen de estado

<table data-search="false"><thead><tr><th width="76">Fase</th><th width="562">Contenido</th><th>Estado</th></tr></thead><tbody><tr><td><strong>F0</strong></td><td>Profundización: Proxmox, Docker, VLANs de seguridad, stack completo</td><td>🟡 En curso</td></tr><tr><td><strong>F1</strong></td><td>Instalación de Proxmox</td><td>🟡 En curso</td></tr><tr><td><strong>F2</strong></td><td>VLANs, switch y pfSense (+ Suricata)</td><td>⚪ Pendiente</td></tr><tr><td><strong>F3</strong></td><td>Servidores internos: DNS, DHCP, MySQL, Backup</td><td>⚪ Pendiente</td></tr><tr><td><strong>F4</strong></td><td>Honeypots: Cowrie, Dionaea, DVWA</td><td>⚪ Pendiente</td></tr><tr><td><strong>F5</strong></td><td>Wazuh (SIEM)</td><td>⚪ Pendiente</td></tr><tr><td><strong>F6</strong></td><td>Grafana, Telegram y script de respuesta automática</td><td>⚪ Pendiente</td></tr><tr><td><strong>F7</strong></td><td>Exposición controlada a internet</td><td>⚪ Pendiente</td></tr><tr><td><strong>F8</strong></td><td>Memoria y presentación</td><td>⚪ Pendiente</td></tr></tbody></table>

### Fase 0 — Profundización

* [x] Bloque 1 — Proxmox (node, VM, LXC, storage, bridge, snapshot, template)
* [x] Bloque 2 — Docker (imagen, contenedor, volumen, Compose, red Docker)
* [ ] Bloque 3 — VLANs aplicadas a seguridad (razonamiento de cada VLAN y defensa en profundidad)
* [ ] Bloque 4 — Cada servicio del stack (qué hace y por qué está ahí)
* [ ] Páginas de la Fase 0 y diagrama de red publicados en el GitBook

### Fase 1 — Instalación de Proxmox

* [x] Preparar el PC y el medio de instalación
* [x] Instalar Proxmox VE como sistema base
* [ ] Acceder a la interfaz web (`https://IP-DEL-PC:8006`)
* [ ] Revisar el storage y subir las ISOs necesarias
* [ ] Crear y probar una VM de prueba con snapshot
* [ ] Configurar NetBird para trabajar en remoto

### Fase 2 — VLANs, switch y pfSense

* [ ] Crear un bridge (`vmbr`) por cada VLAN
* [ ] Configurar en el switch HP 1810-24 las VLANs 10, 20, 30, 40 y 99 y los puertos en trunk
* [ ] Instalar pfSense en el PC físico dedicado (1 NIC)
* [ ] Definir las VLANs sobre la única NIC de pfSense (VLAN 99 como WAN, resto en trunk)
* [ ] Escribir las reglas de firewall según la tabla de tráfico
* [ ] Instalar y activar Suricata con reglas ET Open
* [ ] Probar el aislamiento entre VLANs

### Fase 3 — Servidores internos

* [ ] DNS con Bind9
* [ ] DHCP con isc-dhcp-server (VLAN de Usuarios)
* [ ] MySQL para guardar los ataques capturados
* [ ] Backup automático con rsync + cron
* [ ] Comprobar que la VLAN de Usuarios obtiene IP y resuelve nombres

### Fase 4 — Honeypots

* [ ] Instalar Docker en la VM de la DMZ
* [ ] Escribir el `docker-compose.yml` con Cowrie, Dionaea y DVWA
* [ ] Configurar volúmenes para logs y muestras de malware
* [ ] Atacar los honeypots desde Kali para comprobar que registran

### Fase 5 — Wazuh (SIEM)

* [ ] Desplegar Wazuh en la VLAN de Gestión
* [ ] Conectar los logs de Cowrie, Dionaea y Suricata
* [ ] Crear reglas de correlación (por ejemplo, fuerza bruta)
* [ ] Verificar que las alertas aparecen al atacar desde Kali

### Fase 6 — Grafana, Telegram y script de respuesta

* [ ] Desplegar Grafana y conectarlo a MySQL/Wazuh
* [ ] Crear dashboards y mapa de atacantes
* [ ] Configurar alertas por Telegram
* [ ] Script en Python: leer logs de Cowrie, consultar AbuseIPDB y guardar en MySQL
* [ ] Bloqueo automático de IPs en pfSense vía API _(parte más ambiciosa; puede no llegar en la primera versión)_

### Fase 7 — Exposición a internet

* [ ] Revisar y endurecer las reglas antes de exponer nada
* [ ] Abrir la DMZ a internet de forma controlada
* [ ] Monitorizar los primeros ataques reales
* [ ] Verificar que ningún atacante alcanza las VLANs internas

### Fase 8 — Memoria y presentación

* [ ] Redactar la memoria del proyecto
* [ ] Recopilar capturas, datos y estadísticas de ataques reales
* [ ] Preparar la presentación
* [ ] Ensayar la defensa

### Parte web

Detalle en [Parte web](parte-web.md).

* [ ] Mapa del sitio
* [ ] Mockups / wireframes con notas (a partir del mockup de Figma)
* [ ] Pruebas por perfil de usuario
* [ ] Datos por pantalla
* [ ] Webgrafía
* [ ] Servidor nginx para alojar la web
* [ ] Entregar la actividad M14UF1A2
