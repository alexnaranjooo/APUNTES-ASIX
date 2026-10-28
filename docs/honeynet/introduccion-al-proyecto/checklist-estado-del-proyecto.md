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
| **F6** | Grafana, alertas por Telegram y paneles en la web                   | ⚪ Pendiente |
| **F7** | Exposición controlada a internet (VPS)                              | ⚪ Pendiente |
| **F8** | Memoria y presentación                                              | ⚪ Pendiente |

### Fase 0 — Profundización

* [x] Bloque 1 — Proxmox (node, VM, LXC, storage, bridge, snapshot, template)
* [x] Bloque 2 — Docker (imagen, contenedor, volumen, Compose, red Docker)
* [x] Bloque 3 — VLANs aplicadas a seguridad (razonamiento de cada VLAN y defensa en profundidad)
* [x] Bloque 4 — Cada servicio del stack (qué hace y por qué está ahí)
* [x] Páginas de la Fase 0 y diagrama de red publicados en el GitBook

### Fase 1 — Inventario y Proxmox en los dos PCs

Lo que se va haciendo se documenta en [Implementación en Proxmox](../implementacion-en-proxmox/).

* [x] Instalar Proxmox VE en el PC 1 y acceder a la interfaz web (`https://IP-DEL-PC:8006`)
* [x] Inventario del PC 1 (i5-8500, 8 GB de RAM, SSD de 128 GB, una NIC)
* [ ] Inventario del PC 2 (RAM, CPU, disco y tarjetas de red)
* [x] El centro permite instalar Proxmox en el PC 2
* [ ] Preguntar al centro por escrito: salida a internet, permiso para Kali y NetBird, y quién reinicia los equipos
* [ ] Instalar Proxmox en el PC 2
* [ ] Repositorios gratuitos y actualizar los dos nodos
* [x] Subir las ISOs de Debian 13 y pfSense y la plantilla de contenedor Debian 13
* [ ] Subir la ISO de Kali (en el PC 2)
* [ ] Usuario administrador propio y contraseñas fuertes
* [x] Plantilla de VM Debian 13 y prueba de clonado
* [x] Práctica: VM con nginx y contenedor LXC conectados ([Práctica](../implementacion-en-proxmox/practica-vm-con-nginx-y-contenedor-lxc.md))
* [ ] Snapshots y copia programada, con una copia fuera de los PCs ([Copias de seguridad y snapshots](../implementacion-en-proxmox/copias-de-seguridad-y-snapshots.md))
* [ ] Configurar NetBird para trabajar en remoto

### Fase 2 — VLANs, switch y pfSense

* [ ] Configurar en el switch HP 1810-24 las VLANs 99, 10, 20, 30, 40, 50 y 60 y los puertos en trunk
* [ ] Pasar `vmbr0` a VLAN-aware en los dos nodos y mover la gestión a la VLAN 30
* [ ] Crear la VM de pfSense en el PC 1 (una tarjeta con todas las VLANs, arranque automático)
* [ ] Definir las VLANs sobre la tarjeta de pfSense (VLAN 99 como WAN) y desactivar el checksum offload
* [ ] Escribir las reglas de firewall según la tabla de tráfico de [Seguridad](arquitectura-del-software/6.-seguridad.md)
* [ ] Instalar y activar Suricata con reglas ET Open
* [ ] Probar el aislamiento entre VLANs

### Fase 3 — Servidores internos y web

* [ ] `ct-dns-dhcp`: Bind9 (zona `honeypot.lan`) y DHCP para la VLAN de Usuarios, con relay en pfSense
* [ ] `ct-db`: MariaDB con el esquema `honeypot_web` y el usuario `web_app` con permisos mínimos
* [ ] `ct-backup`: copias automáticas con rsync + cron
* [ ] `ct-web`: nginx + PHP-FPM, HTTPS, UFW y SSH con claves
* [ ] Comprobar que la VLAN de Usuarios obtiene IP, resuelve nombres y abre la web (prueba «Hello world»)

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

### Fase 6 — Grafana, alertas y paneles en la web

* [ ] Desplegar Grafana y conectarlo a Wazuh
* [ ] Crear dashboards, tabla de logs y mapa de atacantes
* [ ] Configurar alertas por Telegram desde Grafana
* [ ] Incrustar los paneles en `/admin` mediante el proxy de nginx, con la excepción Web → Grafana en pfSense
* [ ] Probar el login de administrador y que `/admin` solo responde desde la red interna
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

Detalle en [Parte web](parte-web/) y en [Arquitectura del software](arquitectura-del-software/).

* [x] Mapa del sitio
* [x] Mockups / wireframes con notas (a partir del mockup de Figma)
* [x] Pruebas por perfil de usuario
* [x] Datos por pantalla
* [x] Webgrafía
* [x] Arquitectura del software documentada (requisitos, sistema, base de datos, UI/UX, infraestructura y seguridad)
* [ ] Servidor nginx para alojar la web (en la Fase 3)
* [ ] Login de administrador y panel `/admin` (en la Fase 6)
