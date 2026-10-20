# 🏷️ Glosario de términos

## Introducción

Definiciones breves de los conceptos que aparecen a lo largo del proyecto, ordenadas por bloques. Desde cualquier otra página se puede enlazar aquí en vez de repetir explicaciones.

***

## Seguridad

| Término                    | Definición                                                                                                                                                               |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Honeypot**               | Sistema señuelo, deliberadamente expuesto y vulnerable, cuyo único propósito es atraer atacantes y registrar lo que hacen.                                               |
| **HoneyNet**               | Red completa de honeypots y sistemas de monitorización que simula un entorno real para estudiar ataques.                                                                 |
| **SIEM**                   | _Security Information and Event Management_: plataforma que centraliza los logs de varios sistemas y aplica reglas para detectar incidentes (en el proyecto, **Wazuh**). |
| **IDS**                    | _Intrusion Detection System_: sistema que analiza el tráfico y genera alertas al detectar patrones de ataque conocidos (en el proyecto, **Suricata**).                   |
| **Regla de correlación**   | Regla que relaciona varios eventos para detectar un patrón; por ejemplo, muchos intentos fallidos desde la misma IP en poco tiempo indican fuerza bruta.                 |
| **Defensa en profundidad** | Estrategia de usar varias capas de protección independientes, de modo que el fallo de una no comprometa todo.                                                            |
| **Hardening**              | Endurecer un sistema reduciendo su superficie de ataque (servicios innecesarios, configuraciones por defecto, permisos excesivos).                                       |
| **Malware**                | Software malicioso. Dionaea intenta capturar muestras reales que los bots intentan descargar.                                                                            |
| **Fuerza bruta**           | Ataque que prueba muchas combinaciones de usuario y contraseña hasta acertar.                                                                                            |
| **SQLi / XSS**             | Inyección SQL y _Cross-Site Scripting_: ataques web habituales que DVWA permite practicar y registrar.                                                                   |
| **Log**                    | Registro de eventos de un sistema o aplicación.                                                                                                                          |

## Redes

| Término               | Definición                                                                                                                                              |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **VLAN**              | Red local virtual: separa lógicamente equipos que comparten la misma infraestructura física.                                                            |
| **802.1Q**            | Estándar de etiquetado de VLANs en las tramas Ethernet.                                                                                                 |
| **Trunk**             | Enlace entre dos dispositivos (por ejemplo, un PC y un switch) que transporta varias VLANs a la vez por el mismo cable, cada una etiquetada con 802.1Q. |
| **NIC**               | _Network Interface Card_: tarjeta de red de un equipo. El PC de pfSense solo tiene una.                                                                 |
| **Router on a stick** | Técnica que usa una única NIC en trunk para enrutar entre varias VLANs, creando una interfaz virtual por cada una.                                      |
| **Switch**            | Dispositivo que conecta equipos en una red local y, si es gestionable, permite configurar VLANs y trunks (en el proyecto, un **HP 1810-24**).           |
| **DMZ**               | Zona desmilitarizada: segmento expuesto al exterior, aislado del resto de la red interna.                                                               |
| **Segmentación**      | Dividir una red en zonas con reglas de tráfico distintas entre ellas.                                                                                   |
| **Firewall**          | Sistema que decide qué tráfico se permite o se bloquea según reglas (en el proyecto, **pfSense**).                                                      |
| **DNS**               | Sistema que traduce nombres de dominio a direcciones IP.                                                                                                |
| **DHCP**              | Protocolo que asigna direcciones IP y otros parámetros de red automáticamente.                                                                          |
| **VPN**               | Red privada virtual: conexión cifrada para acceder a una red desde fuera; en el proyecto se usa **NetBird** para trabajar desde casa.                   |
| **API**               | Interfaz que permite a un programa controlar otro; el script del proyecto usará la de pfSense para bloquear IPs.                                        |

## Virtualización y contenedores

| Término                 | Definición                                                                                                                                                         |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Hipervisor**          | Software que crea y gestiona máquinas virtuales. _Bare-metal_ significa que se instala directamente sobre el hardware, sin sistema operativo debajo (**Proxmox**). |
| **Node**                | El PC físico que ejecuta Proxmox.                                                                                                                                  |
| **VM**                  | Máquina virtual: ordenador completo simulado, con su propia CPU, RAM, disco y sistema operativo.                                                                   |
| **LXC**                 | Contenedor ligero de Linux que comparte el kernel del host; menos aislado pero mucho más ligero que una VM.                                                        |
| **Bridge (vmbr)**       | Switch virtual de Proxmox al que se conectan las tarjetas de red de las VMs.                                                                                       |
| **Snapshot**            | Foto del estado completo de una VM para poder volver atrás.                                                                                                        |
| **Template**            | VM maestra ya configurada que se clona para crear VMs nuevas rápidamente.                                                                                          |
| **Imagen (Docker)**     | Plantilla de solo lectura a partir de la cual se crean contenedores.                                                                                               |
| **Contenedor (Docker)** | Imagen puesta en marcha.                                                                                                                                           |
| **Volumen**             | Carpeta fuera del contenedor que conserva los datos aunque este se borre.                                                                                          |
| **Docker Compose**      | Archivo YAML que describe varios contenedores y cómo se conectan entre sí.                                                                                         |

## Automatización, copias y web

| Término                | Definición                                                                                            |
| ---------------------- | ----------------------------------------------------------------------------------------------------- |
| **cron**               | Planificador de tareas de Linux; ejecuta comandos a horas fijas.                                      |
| **rsync**              | Herramienta de copia y sincronización de archivos, usada para las copias de seguridad nocturnas.      |
| **Playlog**            | Función de Cowrie que permite reproducir una sesión de atacante como un vídeo de terminal.            |
| **nginx**              | Servidor web ligero; aloja la web de la empresa del proyecto.                                         |
| **Mockup / wireframe** | Boceto visual de una pantalla de la web, de baja o alta fidelidad (en el proyecto, se hace en Figma). |

## Próximos pasos

* Añadir términos nuevos según aparezcan en cada fase.
* Enlazar este glosario desde las páginas donde se usen conceptos técnicos por primera vez.
