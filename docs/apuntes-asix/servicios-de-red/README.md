---
description: Apuntes Servicios de Red
---

# 🖥️ Servicios de Red

¡Bienvenido a la sección de Servicios de Red! En este espacio recopilo las guías, despliegues y configuraciones paso a paso de los principales servicios de red implementados sobre entornos GNU/Linux (Debian 13 Trixie), orientados a laboratorios de administración de sistemas.

### 📂 Índice de Servicios

Selecciona el servicio que deseas consultar o configurar:

#### 🗄️ 1. Servidor DNS (Bind9)

* Descripción: Implementación de un servidor DNS maestro autoritativo con gestión de zonas directas e inversas, reenvío de consultas y aislamiento por redes internas.
* Estado: ✅ Completado y documentado.

#### 🔌 2. Servidor DHCP (Próximamente)

* Descripción: Configuración del servicio de asignación dinámica de direcciones IP y entrega automática de parámetros de red (puerta de enlace, servidores DNS) a los clientes de la red interna.
* Estado: 🚧 En desarrollo / Pendiente.

#### 📁 3. Servidor de Ficheros - NFS / Samba (Próximamente)

* Descripción: Compartición de recursos en red entre sistemas Linux y mixtos mediante protocolos NFS y SMB.
* Estado: 🚧 En desarrollo / Pendiente.

#### 🔒 4. Servicio Web y Seguridad (Próximamente)

* Descripción: Despliegue de servidores web (Apache/Nginx) y endurecimiento de servicios.
* Estado: 🚧 En desarrollo / Pendiente.

### 🛠️ Entorno de Laboratorio Común

La mayoría de los servicios de esta sección se despliegan bajo las siguientes características base de virtualización:

* Sistema Operativo: Debian GNU/Linux 13 (Trixie)
* Esquema de Red:
  * Adaptador 1 (NAT): Salida a internet para actualizaciones y resolución externa.
  * Adaptador 2 (Red Interna): Comunicación aislada con las máquinas cliente de la red del laboratorio (`192.168.6.0/24`).
* Herramientas de Diagnóstico: `ip`, `systemctl`, `journalctl`, `bind9-utils` (`dig`, `nslookup`, `named-checkzone`).

> 💡 _Nota: Esta documentación está en constante actualización conforme avanzamos en las prácticas del módulo de ASIX._
