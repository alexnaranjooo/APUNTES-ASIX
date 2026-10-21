# ✔️ Checklist / estado del proyecto

## Introducción

Seguimiento del estado de **HoneyNet** fase a fase. Se actualiza a medida que se avanza: marca cada casilla cuando esté hecha y cambia el estado de la fase en la tabla resumen.

> Las subtareas de cada fase son una propuesta de trabajo basada en el stack del proyecto; ajústalas según vayas descubriendo qué hace falta de verdad en cada paso.

## Resumen de estado

| Fase   | Contenido                                                           | Estado      |
| ------ | ------------------------------------------------------------------- | ----------- |
| **F0** | Profundización: Proxmox, Docker, VLANs de seguridad, stack completo | ✅ Hecha     |
| **F1** | Inventario y Proxmox en los dos PCs                                 | 🟡 En curso |
| **F2** | VLANs, switch y pfSense (en VM) + Suricata                          | ⚪ Pendiente |
| **F3** | Servidores internos (DNS, DHCP, MariaDB, Backup) y web              | ⚪ Pendiente |
| **F4** | Honeypots: Cowrie, Dionaea, DVWA                                    | ⚪ Pendiente |
| **F5** | Wazuh completo (SIEM)                                               | ⚪ Pendiente |
| **F6** | Grafana y alertas por Telegram                                      | ⚪ Pendiente |
| **F7** | Exposición controlada a internet (VPS)                              | ⚪ Pendiente |
| **F8** | Memoria y presentación                                              | ⚪ Pendiente |

### Fase 0 — Profundización

* [x] Bloque 1 — Proxmox (node, VM, LXC, storage, bridge, snapshot, template)
* [x] Bloque 2 — Docker (imagen, contenedor, volumen, Compose, red Docker)
* [x] Bloque 3 — VLANs aplicadas a seguridad (razonamiento de cada VLAN y defensa en profundidad)
* [x] Bloque 4 — Cada servicio del stack (qué hace y por qué está ahí)
* [x] Páginas de la Fase 0 y diagrama de red publicados en el GitBook

### Fase 1 — Inventario y Proxmox en los dos PCs

* [x] Instalar Proxmox VE en el PC 1 y acceder a la interfaz web (`https://IP-DEL-PC:8006`)
* [x] Inventario del PC 1 (i5-8500, 8 GB de RAM, SSD de 128 GB, una NIC)
* [ ] Inventario del PC 2 (RAM, CPU, disco y tarjetas de red)
* [ ] Preguntar al centro por escrito: salida a internet, permiso para Kali, NetBird e instalar Proxmox en el PC 2
* [ ] Instalar Proxmox en el PC 2
* [ ] Repositorios gratuitos y actualizar los dos nodos
* [ ] Subir las ISOs (Debian 13, pfSense, Kali) y la plantilla de contenedor Debian 13
* [ ] Usuario administrador propio y contraseñas fuertes
* [ ] Plantilla de VM Debian 13 y prueba de clonado
* [ ] Snapshots y copia programada (con una copia fuera de los PCs)
* [ ] Configurar NetBird para trabajar en remoto

### Fase 2 — VLANs, switch y pfSense

* [ ] Configurar en el switch HP 1810-24 las VLANs 99, 10, 20, 30, 40, 50 y 60 y los puertos en trunk
* [ ] Pasar `vmbr0` a VLAN-aware en los dos nodos y mover la gestión a la VLAN 30
* [ ] Crear la VM de pfSense en el PC 1 (una tarjeta con todas las VLANs, arranque automático)
* [ ] Definir las VLANs sobre la tarjeta de pfSense (VLAN 99 como WAN) y desactivar el checksum offload
* [ ] Escribir las reglas de firewall según la tabla de tráfico
* [ ] Instalar y activar Suricata con reglas ET Open
* [ ] Probar el aislamiento entre VLANs

### Fase 3 — Servidores internos y web

* [ ] LXC de servidores: DNS con Bind9
* [ ] DHCP para la VLAN de Usuarios (con relay en pfSense)
* [ ] MariaDB para guardar los ataques capturados
* [ ] Backup automático con rsync + cron
* [ ] Comprobar que la VLAN de Usuarios obtiene IP y resuelve nombres
* [ ] LXC de web con nginx, HTTPS y endurecimiento básico

### Fase 4 — Honeypots

* [ ] Instalar Docker en la VM de la DMZ
* [ ] Escribir el `docker-compose.yml` con Cowrie, Dionaea y DVWA
* [ ] Configurar volúmenes para logs y muestras de malware
* [ ] Atacar los honeypots desde Kali para comprobar que registran

### Fase 5 — Wazuh completo (SIEM)

* [ ] Desplegar Wazuh completo en la VLAN de Gestión (PC 2), con las contraseñas por defecto cambiadas
* [ ] Conectar los logs de Cowrie, Dionaea, Suricata y la web
* [ ] Crear reglas de correlación (por ejemplo, fuerza bruta)
* [ ] Fijar una retención corta de datos
* [ ] Verificar que las alertas aparecen al atacar desde Kali

### Fase 6 — Grafana y alertas

* [ ] Desplegar Grafana y conectarlo a Wazuh y MariaDB
* [ ] Crear dashboards y mapa de atacantes
* [ ] Configurar alertas por Telegram desde Grafana
* [ ] _Extra opcional:_ bloqueo automático de IPs en pfSense _(puede no llegar en la primera versión)_

### Fase 7 — Exposición a internet

* [ ] Revisar y endurecer las reglas antes de exponer nada
* [ ] Desplegar los honeypots en la VPS y abrir el túnel VPN desde pfSense
* [ ] Monitorizar los primeros ataques reales
* [ ] Verificar que ningún atacante alcanza las VLANs internas

### Fase 8 — Memoria y presentación

* [ ] Redactar la memoria del proyecto
* [ ] Recopilar capturas, datos y estadísticas de ataques reales
* [ ] Preparar la presentación
* [ ] Ensayar la defensa

### Parte web

Detalle en [Parte web](parte-web/).

* [x] Mapa del sitio
* [x] Mockups / wireframes con notas (a partir del mockup de Figma)
* [x] Pruebas por perfil de usuario
* [x] Datos por pantalla
* [x] Webgrafía
* [ ] Servidor nginx para alojar la web (en la Fase 3)
