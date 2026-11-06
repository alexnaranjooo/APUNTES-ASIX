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

<figure><img src="../../../.gitbook/assets/image (58).png" alt="" width="528"><figcaption></figcaption></figure>

## 2. ¿De qué otro modo se puede emular un clúster de Proxmox?

Cuando no hay varios equipos físicos, se puede practicar con **virtualización anidada**: instalar cada nodo Proxmox dentro de una máquina virtual.

<table><thead><tr><th width="316">Forma</th><th>Cómo funciona</th></tr></thead><tbody><tr><td>Nodos en VirtualBox o VMware Workstation</td><td>Cada VM lleva Proxmox y se unen en un clúster, como si fueran PCs distintos</td></tr><tr><td>Nodos dentro de otro Proxmox</td><td>Se activa la virtualización anidada en una VM para que pueda ejecutar a su vez máquinas virtuales</td></tr><tr><td>Almacenamiento compartido simulado</td><td>Otra VM ofrece una carpeta NFS o un disco iSCSI, o varios nodos forman un Ceph de prueba</td></tr></tbody></table>

## 3. ¿Qué tecnologías permiten crear un clúster de alta disponibilidad?

<table data-search="false"><thead><tr><th width="399">Tecnología</th><th>Qué aporta</th></tr></thead><tbody><tr><td><strong>Corosync</strong></td><td>Comunicación entre nodos y control del quórum</td></tr><tr><td><strong>Gestor de alta disponibilidad de Proxmox</strong></td><td>Vigila las máquinas marcadas como HA y las arranca en otro nodo si el suyo cae</td></tr><tr><td><strong>Fencing</strong> (con <em>watchdog</em>)</td><td>Aparta un nodo averiado para que no use los mismos discos que su sustituto</td></tr><tr><td><strong>Almacenamiento compartido</strong>: Ceph, NFS o iSCSI</td><td>Todos los nodos ven los mismos discos de las máquinas</td></tr><tr><td><strong>Replicación de almacenamiento</strong> (ZFS)</td><td>Copia los discos entre nodos cada cierto tiempo, sin almacenamiento compartido</td></tr><tr><td><strong>Redes redundantes</strong></td><td>Evitan que un cable o un switch dejen al clúster aislado</td></tr><tr><td><strong>QDevice</strong></td><td>Un voto externo para poder decidir el quórum cuando solo hay dos nodos</td></tr></tbody></table>

## 4. ¿Qué son los hipervisores?

Un **hipervisor** es el software que crea y ejecuta máquinas virtuales. Hace de intermediario entre el hardware real (procesador, memoria, disco y red) y las máquinas virtuales, repartiendo esos recursos entre ellas y manteniéndolas aisladas unas de otras.

<figure><img src="../../../.gitbook/assets/image (59).png" alt="" width="563"><figcaption></figcaption></figure>

## 5. ¿Qué significa que sean de tipo 1 o de tipo 2?

<table data-header-hidden><thead><tr><th width="125"></th><th width="273"></th><th></th></tr></thead><tbody><tr><td></td><td>Tipo 1 (<em>bare-metal</em>)</td><td>Tipo 2 (<em>hosted</em>)</td></tr><tr><td>Dónde corre</td><td>Directamente sobre el hardware</td><td>Como un programa sobre un sistema operativo</td></tr><tr><td>Rendimiento</td><td>Mayor</td><td>Menor, porque el sistema base consume recursos</td></tr><tr><td>Uso habitual</td><td>Servidores y centros de datos</td><td>Escritorio, pruebas y aprendizaje</td></tr><tr><td>Ejemplos</td><td><strong>Proxmox VE</strong> (KVM), <strong>VMware ESXi</strong>, <strong>Microsoft Hyper-V</strong>, <strong>Xen</strong></td><td><strong>VirtualBox</strong>, <strong>VMware Workstation</strong>, <strong>Parallels</strong></td></tr></tbody></table>

## 6. ¿En qué sistema operativo se basa Proxmox?

En **Debian GNU/Linux**. Proxmox VE 9, que es el que se usa en el laboratorio, se basa en Debian 13 (_Trixie_), con un kernel Linux propio, derivado del de Ubuntu y adaptado para virtualización.



<figure><img src="../../../.gitbook/assets/image (60).png" alt="" width="563"><figcaption></figcaption></figure>

## 7. ¿Qué tipos de virtualización están disponibles por defecto en Proxmox?

| Tipo                                | Tecnología | Características                                                                      |
| ----------------------------------- | ---------- | ------------------------------------------------------------------------------------ |
| **Virtualización completa**         | KVM y QEMU | Máquinas virtuales con su propio kernel; admiten cualquier sistema, incluido Windows |
| **Virtualización por contenedores** | LXC        | Comparten el kernel del nodo; son más ligeros, pero solo ejecutan sistemas Linux     |

Además, las VMs pueden usar **VirtIO**, controladores virtuales de disco y red pensados para ir más rápido.

## 8. ¿Qué son QEMU, KVM, noVNC y Ceph, y qué características tienen?

<table><thead><tr><th width="87">Nombre</th><th width="288">Qué es</th><th>Características principales</th></tr></thead><tbody><tr><td><strong>QEMU</strong></td><td>Programa libre que emula el hardware de un ordenador (procesador, disco, tarjeta de red, BIOS)</td><td>Admite muchas arquitecturas. Solo, es lento; con KVM llega a casi la velocidad nativa. Cada VM de Proxmox es un proceso QEMU</td></tr><tr><td><strong>KVM</strong> </td><td>Módulo del kernel de Linux que lo convierte en hipervisor usando las extensiones del procesador (Intel VT-x o AMD-V)</td><td>Rendimiento casi nativo, aislamiento entre máquinas, migración en caliente y soporte de virtualización anidada</td></tr><tr><td><strong>noVNC</strong></td><td>Cliente VNC escrito en HTML5 y JavaScript</td><td>Muestra la consola de una VM o contenedor dentro del navegador, sin instalar nada. Es la consola que ofrece la interfaz web de Proxmox</td></tr><tr><td><strong>Ceph</strong></td><td>Sistema de almacenamiento distribuido y de código abierto, integrado en Proxmox</td><td>Replica los datos entre nodos, se repara solo si falla un disco, escala añadiendo nodos y sirve discos de VM, archivos y objetos</td></tr></tbody></table>

## 9. ¿Qué roles de usuarios y grupos se pueden configurar en Proxmox?

El control de acceso se basa en cuatro conceptos:

<table><thead><tr><th width="90">Concepto</th><th>Qué es</th></tr></thead><tbody><tr><td><strong>Usuario</strong></td><td>Una cuenta, que vive en un <em>realm</em> (Linux PAM, servidor de Proxmox, LDAP, Active Directory u OpenID)</td></tr><tr><td><strong>Grupo</strong></td><td>Un conjunto de usuarios al que se le dan permisos de una vez</td></tr><tr><td><strong>Rol</strong></td><td>Un conjunto de privilegios con nombre</td></tr><tr><td><strong>Permiso</strong></td><td>La unión de una ruta (<code>/</code>, una VM, un almacenamiento...), un usuario o grupo y un rol</td></tr></tbody></table>

Roles predefinidos más habituales:

<table><thead><tr><th width="352">Rol</th><th>Qué permite</th></tr></thead><tbody><tr><td><strong>Administrator</strong></td><td>Todos los privilegios</td></tr><tr><td><strong>PVEAdmin</strong></td><td>Casi todo, salvo administrar permisos</td></tr><tr><td><strong>PVEVMAdmin</strong></td><td>Gestionar por completo las VMs y contenedores</td></tr><tr><td><strong>PVEVMUser</strong></td><td>Usar las VMs: consola, encender y apagar</td></tr><tr><td><strong>PVEAuditor</strong></td><td>Solo lectura</td></tr><tr><td><strong>PVETemplateUser</strong></td><td>Ver y clonar plantillas</td></tr><tr><td><strong>PVEUserAdmin</strong></td><td>Administrar usuarios</td></tr><tr><td><strong>NoAccess</strong></td><td>Ningún acceso</td></tr></tbody></table>

## 10. ¿Dónde se almacenan las ISOs y las plantillas de Proxmox?

En los **almacenamientos** (_storage_) que tengan activado ese tipo de contenido. Por defecto, en `local`, que es una carpeta del nodo:

| Qué                      | Carpeta en el nodo           | Dónde se ve en la web  |
| ------------------------ | ---------------------------- | ---------------------- |
| ISOs de instalación      | `/var/lib/vz/template/iso`   | `local` → ISO Images   |
| Plantillas de contenedor | `/var/lib/vz/template/cache` | `local` → CT Templates |
| Copias de seguridad      | `/var/lib/vz/dump`           | `local` → Backups      |

Los **discos** de las VMs y los contenedores no van ahí, sino en `local-lvm` (o en otro almacenamien-to de discos). Las plantillas de VM no son un archivo: son una VM marcada como plantilla, y se ven en el árbol de Proxmox.
