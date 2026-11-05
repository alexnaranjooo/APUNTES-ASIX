---
description: >-
  Respuestas a las preguntas sobre Proxmox: clúster, alta disponibilidad,
  hipervisores, QEMU, KVM, noVNC, Ceph, roles y almacenamiento.
---

# Preguntas sobre Proxmox

## 1. ¿Qué es un clúster de Proxmox?

Un **clúster** es un conjunto de nodos Proxmox unidos para funcionar como un solo sistema. Se gestionan desde cualquier nodo con la misma interfaz web, comparten la misma configuración y pueden mover máquinas de uno a otro.

* La comunicación entre nodos la hace **Corosync**, y la configuración compartida se guarda en un sistema de archivos propio del clúster (`pmxcfs`).
* Todos los nodos son iguales: no hay un nodo «jefe» fijo.
* Funciona por **quórum**: el clúster solo actúa si ve a la mayoría de sus nodos. Por eso se recomiendan **tres nodos como mínimo**.
* Permite **migrar máquinas** entre nodos y, con almacenamiento compartido, hacerlo con la máquina encendida.
* Se crea con `pvecm create` en el primer nodo y se une el resto con `pvecm add`.
* Necesita una red rápida y estable entre los nodos.

En HoneyNet **no hay clúster**: son dos nodos independientes.

## 2. ¿De qué otro modo se puede emular un clúster de Proxmox?

Cuando no hay varios equipos físicos, se puede practicar con **virtualización anidada**: instalar cada nodo Proxmox dentro de una máquina virtual.

| Forma                                    | Cómo funciona                                                                                     |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------- |
| Nodos en VirtualBox o VMware Workstation | Cada VM lleva Proxmox y se unen en un clúster, como si fueran PCs distintos                       |
| Nodos dentro de otro Proxmox             | Se activa la virtualización anidada en una VM para que pueda ejecutar a su vez máquinas virtuales |
| Almacenamiento compartido simulado       | Otra VM ofrece una carpeta NFS o un disco iSCSI, o varios nodos forman un Ceph de prueba          |

Para probar el reparto de carga o la migración funciona bien, pero no sirve para medir el rendimiento real, y exige bastante memoria. Para gestionar varios nodos o clústeres desde un solo panel sin unirlos, Proxmox ofrece también una herramienta aparte, **Proxmox Datacenter Manager**.

## 3. ¿Qué tecnologías permiten crear un clúster de alta disponibilidad?

| Tecnología                                       | Qué aporta                                                                     |
| ------------------------------------------------ | ------------------------------------------------------------------------------ |
| **Corosync**                                     | Comunicación entre nodos y control del quórum                                  |
| **Gestor de alta disponibilidad de Proxmox**     | Vigila las máquinas marcadas como HA y las arranca en otro nodo si el suyo cae |
| **Fencing** (con _watchdog_)                     | Aparta un nodo averiado para que no use los mismos discos que su sustituto     |
| **Almacenamiento compartido**: Ceph, NFS o iSCSI | Todos los nodos ven los mismos discos de las máquinas                          |
| **Replicación de almacenamiento** (ZFS)          | Copia los discos entre nodos cada cierto tiempo, sin almacenamiento compartido |
| **Redes redundantes**                            | Evitan que un cable o un switch dejen al clúster aislado                       |
| **QDevice**                                      | Un voto externo para poder decidir el quórum cuando solo hay dos nodos         |

## 4. ¿Qué son los hipervisores?

Un **hipervisor** es el software que crea y ejecuta máquinas virtuales. Hace de intermediario entre el hardware real (procesador, memoria, disco y red) y las máquinas virtuales, repartiendo esos recursos entre ellas y manteniéndolas aisladas unas de otras.

## 5. ¿Qué significa que sean de tipo 1 o de tipo 2?

|              | Tipo 1 (_bare-metal_)                                                 | Tipo 2 (_hosted_)                                     |
| ------------ | --------------------------------------------------------------------- | ----------------------------------------------------- |
| Dónde corre  | Directamente sobre el hardware                                        | Como un programa sobre un sistema operativo           |
| Rendimiento  | Mayor                                                                 | Menor, porque el sistema base consume recursos        |
| Uso habitual | Servidores y centros de datos                                         | Escritorio, pruebas y aprendizaje                     |
| Ejemplos     | **Proxmox VE** (KVM), **VMware ESXi**, **Microsoft Hyper-V**, **Xen** | **VirtualBox**, **VMware Workstation**, **Parallels** |

## 6. ¿En qué sistema operativo se basa Proxmox?

En **Debian GNU/Linux**. Proxmox VE 9, que es el que se usa en el laboratorio, se basa en Debian 13 (_Trixie_), con un kernel Linux propio, derivado del de Ubuntu y adaptado para virtualización.

## 7. ¿Qué tipos de virtualización están disponibles por defecto en Proxmox?

| Tipo                                | Tecnología | Características                                                                      |
| ----------------------------------- | ---------- | ------------------------------------------------------------------------------------ |
| **Virtualización completa**         | KVM y QEMU | Máquinas virtuales con su propio kernel; admiten cualquier sistema, incluido Windows |
| **Virtualización por contenedores** | LXC        | Comparten el kernel del nodo; son más ligeros, pero solo ejecutan sistemas Linux     |

Además, las VMs pueden usar **VirtIO**, controladores virtuales de disco y red pensados para ir más rápido.

## 8. ¿Qué son QEMU, KVM, noVNC y Ceph, y qué características tienen?

| Nombre                                   | Qué es                                                                                                               | Características principales                                                                                                            |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| **QEMU**                                 | Programa libre que emula el hardware de un ordenador (procesador, disco, tarjeta de red, BIOS)                       | Admite muchas arquitecturas. Solo, es lento; con KVM llega a casi la velocidad nativa. Cada VM de Proxmox es un proceso QEMU           |
| **KVM** (_Kernel-based Virtual Machine_) | Módulo del kernel de Linux que lo convierte en hipervisor usando las extensiones del procesador (Intel VT-x o AMD-V) | Rendimiento casi nativo, aislamiento entre máquinas, migración en caliente y soporte de virtualización anidada                         |
| **noVNC**                                | Cliente VNC escrito en HTML5 y JavaScript                                                                            | Muestra la consola de una VM o contenedor dentro del navegador, sin instalar nada. Es la consola que ofrece la interfaz web de Proxmox |
| **Ceph**                                 | Sistema de almacenamiento distribuido y de código abierto, integrado en Proxmox                                      | Replica los datos entre nodos, se repara solo si falla un disco, escala añadiendo nodos y sirve discos de VM, archivos y objetos       |

Sobre **Ceph**: sus piezas son los monitores (MON), los gestores (MGR) y los discos (OSD). Pide al menos tres nodos y una red rápida, así que en HoneyNet no se usa.

## 9. ¿Qué roles de usuarios y grupos se pueden configurar en Proxmox?

El control de acceso se basa en cuatro conceptos:

| Concepto    | Qué es                                                                                               |
| ----------- | ---------------------------------------------------------------------------------------------------- |
| **Usuario** | Una cuenta, que vive en un _realm_ (Linux PAM, servidor de Proxmox, LDAP, Active Directory u OpenID) |
| **Grupo**   | Un conjunto de usuarios al que se le dan permisos de una vez                                         |
| **Rol**     | Un conjunto de privilegios con nombre                                                                |
| **Permiso** | La unión de una ruta (`/`, una VM, un almacenamiento...), un usuario o grupo y un rol                |

Roles predefinidos más habituales:

| Rol                                          | Qué permite                                      |
| -------------------------------------------- | ------------------------------------------------ |
| **Administrator**                            | Todos los privilegios                            |
| **PVEAdmin**                                 | Casi todo, salvo administrar permisos            |
| **PVEVMAdmin**                               | Gestionar por completo las VMs y contenedores    |
| **PVEVMUser**                                | Usar las VMs: consola, encender y apagar         |
| **PVEAuditor**                               | Solo lectura                                     |
| **PVEDatastoreAdmin** y **PVEDatastoreUser** | Administrar o usar un almacenamiento             |
| **PVEPoolAdmin** y **PVEPoolUser**           | Administrar o usar un grupo de recursos (_pool_) |
| **PVETemplateUser**                          | Ver y clonar plantillas                          |
| **PVEUserAdmin**                             | Administrar usuarios                             |
| **NoAccess**                                 | Ningún acceso                                    |

También se pueden crear **roles propios** con los privilegios que se necesiten, y **tokens de API** para acceso por scripts.

## 10. ¿Dónde se almacenan las ISOs y las plantillas de Proxmox?

En los **almacenamientos** (_storage_) que tengan activado ese tipo de contenido. Por defecto, en `local`, que es una carpeta del nodo:

| Qué                      | Carpeta en el nodo           | Dónde se ve en la web  |
| ------------------------ | ---------------------------- | ---------------------- |
| ISOs de instalación      | `/var/lib/vz/template/iso`   | `local` → ISO Images   |
| Plantillas de contenedor | `/var/lib/vz/template/cache` | `local` → CT Templates |
| Copias de seguridad      | `/var/lib/vz/dump`           | `local` → Backups      |

Los **discos** de las VMs y los contenedores no van ahí, sino en `local-lvm` (o en otro almacenamiento de discos). Las plantillas de VM no son un archivo: son una VM marcada como plantilla, y se ven en el árbol de Proxmox.

Se pueden añadir otros almacenamientos (NFS, un segundo disco) y elegir qué tipos de contenido guarda cada uno.
