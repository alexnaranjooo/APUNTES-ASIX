---
description: >-
  Qué es una plantilla en Proxmox, cómo se prepara cada tipo y qué significa
  cada ajuste de la máquina virtual y del contenedor.
---

# Plantillas y configuración en Proxmox

## Introducción

Antes de montar los servidores del laboratorio conviene dejar preparadas unas **plantillas**: copias limpias de un sistema ya instalado y actualizado, desde las que se crean máquinas nuevas en segundos. Esta página explica los conceptos, cómo se prepara Proxmox y qué hay que descargar. Después se crean la plantilla de VM y el contenedor en las dos páginas siguientes.

Todo se hace desde la interfaz web de Proxmox (`https://IP-DEL-NODO:8006`). En este laboratorio el nodo se llama `honeynet` y usa **Proxmox VE 9**, que se basa en Debian 13.

***

## 1. Conceptos previos

### 1.1 Máquina virtual y contenedor

|                        | Máquina virtual (VM)                                            | Contenedor LXC                        |
| ---------------------- | --------------------------------------------------------------- | ------------------------------------- |
| Sistema operativo      | Tiene su propio kernel y un sistema completo                    | Comparte el kernel de Proxmox         |
| Arranque               | Decenas de segundos                                             | Pocos segundos                        |
| Memoria                | Mayor, porque carga un sistema entero                           | Muy poca (desde unos 128 MB)          |
| Aislamiento            | Alto: hardware virtualizado                                     | Menor: comparte el kernel con el host |
| Herramienta de consola | `qm`                                                            | `pct`                                 |
| Cuándo usarla          | Sistemas que no son Linux o cuando se quiere aislamiento fuerte | Servicios Linux ligeros               |

### 1.2 Tipos de recursos que se guardan en Proxmox

| Recurso                     | Qué es                                                             | Dónde se guarda                       | Para qué sirve                           |
| --------------------------- | ------------------------------------------------------------------ | ------------------------------------- | ---------------------------------------- |
| **ISO**                     | Imagen del instalador de un sistema                                | `local` → ISO Images                  | Instalar un sistema desde cero en una VM |
| **Plantilla de contenedor** | Sistema Linux ya preparado, en un archivo comprimido (`.tar.zst`)  | `local` → CT Templates                | Crear contenedores LXC                   |
| **Plantilla de VM**         | VM ya instalada que se marca como plantilla y no se puede encender | Árbol de Proxmox, con un icono propio | Clonar VMs nuevas                        |

### 1.3 Dónde se guarda cada cosa

* **`local`**: una carpeta del sistema. Guarda las ISOs, las plantillas de contenedor y las copias de seguridad.
* **`local-lvm`**: un volumen LVM-thin. Guarda los **discos** de las VMs y de los contenedores.

{% hint style="info" %}
En el PC 1 quedan unos **54 GB** en `local-lvm` para todas las máquinas. Por eso los discos de las plantillas y de los contenedores son pequeños.
{% endhint %}

***

## 2. Preparar Proxmox

### 2.1 Entrar en la interfaz

Al entrar con `root` aparece el aviso **«No valid subscription»**: es normal, Proxmox es gratuito y solo la versión con soporte es de pago. Se pulsa **OK**. El navegador también marca «No es seguro» porque Proxmox usa un certificado propio.

### 2.2 Repositorios y actualización

**Concepto:** Proxmox viene configurado con el repositorio de pago (_enterprise_). Sin suscripción da error al actualizar. Hay que desactivarlo y activar el gratuito (_no-subscription_); los paquetes son los mismos.

1. Hacer clic en el nodo **honeynet** y entrar en **Updates → Repositories**.
2. Seleccionar las filas _enterprise_ (la de `pve` y la de `ceph`) y pulsar **Disable**.
3. Pulsar **Add**, elegir **No-Subscription** y confirmar.
4. En **Updates**, pulsar **Refresh** y luego **Upgrade**. Al terminar, reiniciar el nodo.

Comprobación, en **honeynet → Shell**:

```bash
apt update
pveversion
```

`apt update` no debe mostrar errores `401` y `pveversion` debe indicar una versión 9.x.

<figure><img src="../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

### 2.3 Usuario administrador propio

`root` lo puede todo y es el usuario que más se ataca, así que se trabaja con un usuario propio:

1. **Datacenter → Permissions → Users → Add**, con el _Realm_ **Proxmox VE authentication server**.
2. **Permissions → Add → User Permission**: ruta `/`, el usuario y el rol `Administrator`.
3. Opcionalmente, activar un segundo factor en **Permissions → Two Factor**.

***

## 3. Descargar lo necesario

### 3.1 ISO de Debian 13

1. En debian.org, copiar el enlace de la imagen **netinst** de 64 bits (amd64).
2. **honeynet → local → ISO Images → Download from URL**, pegar el enlace, pulsar **Query URL** y después **Download**.

### 3.2 Plantilla de contenedor Debian 13

**honeynet → local → CT Templates → Templates**, buscar `debian-13` y descargar la **standard**.

### 3.3 ISO de pfSense (para más adelante)

Se descarga en el portátil desde pfsense.org en formato **ISO (DVD)**, se descomprime, se comprueba el hash con `certutil -hashfile fichero.iso SHA256` y se sube con **Upload**. No se usa en la práctica, pero se deja preparada para la Fase 2.



***

## Siguiente

* [Plantilla de VM Debian 13](plantilla-de-vm-debian-13.md): crear, instalar, preparar y limpiar la VM.
* [Clones y contenedor LXC](clones-y-contenedor-lxc.md): convertirla en plantilla, clonarla y crear un contenedor.
* [Práctica: VM con nginx y contenedor LXC](practica-vm-con-nginx-y-contenedor-lxc.md): la práctica que usa todo esto.
