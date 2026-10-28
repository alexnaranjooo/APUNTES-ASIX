---
description: >-
  Snapshots y copias de seguridad programadas de las máquinas de Proxmox, con la
  restauración de prueba y una copia fuera del PC.
---

# Copias de seguridad y snapshots

## Introducción

Los PCs del laboratorio son del centro y pueden reiniciarse o reinstalarse, y un cambio mal hecho puede romper una máquina. Por eso, antes de seguir construyendo, se configuran dos protecciones distintas:

|                           | Snapshot                                               | Copia de seguridad (_backup_)                                |
| ------------------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| Qué es                    | Una foto instantánea de una máquina, en el mismo disco | Un archivo completo de la máquina que se puede guardar fuera |
| Velocidad                 | Inmediata                                              | Más lenta                                                    |
| Sirve para                | Volver atrás tras un cambio que sale mal               | Recuperar una máquina si se pierde o se rompe el disco       |
| Protege si el disco falla | No                                                     | Sí, si la copia está fuera del disco                         |

{% hint style="info" %}
Documentado el 28 de octubre de 2026. Las máquinas usadas son `vm-nginx` y `ct-prueba`, creadas en la [práctica](practica-vm-con-nginx-y-contenedor-lxc.md).
{% endhint %}

***

## 1. Snapshots

1. Seleccionar `vm-nginx` → **Snapshots** → **Take Snapshot**.
2. Nombre `practica-ok` (sin espacios) y sin marcar _Include RAM_. Pulsar **Take Snapshot**.
3. Repetir con `ct-prueba`.

**Comprobación con un rollback:** dentro de la VM se borra la página de nginx (`rm /var/www/html/index.html`) y después se elige el snapshot → **Rollback**. La página vuelve a estar.

{% hint style="info" %}
CAPTURA PENDIENTE: la pestaña _Snapshots_ de `vm-nginx` con `practica-ok`.
{% endhint %}

{% hint style="info" %}
CAPTURA PENDIENTE: la pestaña _Snapshots_ de `ct-prueba` con `practica-ok`.
{% endhint %}

***

## 2. Copia de seguridad programada

En **Datacenter → Backup → Add**:

| Campo                   | Valor                                            | Por qué                                                                  |
| ----------------------- | ------------------------------------------------ | ------------------------------------------------------------------------ |
| Node                    | `honeynet`                                       | El nodo del laboratorio                                                  |
| Storage                 | `local`                                          | Donde se guardan las copias                                              |
| Schedule                | Todos los días a las 03:00                       | De madrugada, sin carga en el equipo                                     |
| Selection mode          | _Include selected VMs_: `vm-nginx` y `ct-prueba` | Las plantillas no hace falta copiarlas                                   |
| Mode                    | Snapshot                                         | Copia la máquina sin apagarla                                            |
| Compression             | ZSTD                                             | Rápida y ocupa poco                                                      |
| Retention (_Keep Last_) | 3                                                | Conserva las últimas tres y borra las anteriores para no llenar el disco |

Para no esperar a las 03:00, se selecciona el trabajo y se pulsa **Run now**. Al terminar (`TASK OK`), en **honeynet → local → Backups** aparecen los archivos de las dos máquinas.

{% hint style="info" %}
CAPTURA PENDIENTE: el trabajo de copia en _Datacenter → Backup_, con su programación.
{% endhint %}

{% hint style="info" %}
CAPTURA PENDIENTE: _local → Backups_ con los archivos creados y su tamaño.
{% endhint %}

***

## 3. Probar la restauración

Una copia que no se ha restaurado nunca no es de fiar, así que se prueba:

1. En **local → Backups**, seleccionar la copia de `ct-prueba` → **Restore**.
2. Poner un ID nuevo (por ejemplo `199`) y desmarcar _Start after restore_.
3. Comprobar que aparece el contenedor restaurado y borrarlo después (**More → Remove**).

{% hint style="info" %}
CAPTURA PENDIENTE: el asistente de restauración y el contenedor restaurado en el árbol.
{% endhint %}

***

## 4. Una copia fuera del PC

**Por qué:** las copias guardadas en `local` están en el mismo disco que las máquinas. Si el centro reinstala el PC, se pierden con él. Hay que llevar una copia a otro sitio.

1. Desde el portátil, conectar por SFTP (WinSCP o `scp`) al nodo con el usuario `root`.
2. Entrar en la carpeta `/var/lib/vz/dump` y copiar los archivos de copia a un USB o a un disco propio.
3. Guardar también la configuración de Proxmox, que está en la carpeta `/etc/pve` del nodo, copiándola con `cp -r` a una carpeta de `root` y descargándola por SFTP.

{% hint style="warning" %}
Nunca se suben contraseñas ni claves a GitHub. Las copias y las configuraciones se guardan en un sitio privado.
{% endhint %}

{% hint style="info" %}
CAPTURA PENDIENTE: la conexión SFTP con los archivos de copia descargados.
{% endhint %}

***

## 5. Resultados

| Comprobación                                              | Resultado    |
| --------------------------------------------------------- | ------------ |
| Snapshot `practica-ok` creado en la VM y en el contenedor | Por rellenar |
| Rollback de la VM devuelve la página de nginx             | Por rellenar |
| Copia programada creada y ejecutada con **Run now**       | Por rellenar |
| Restauración de prueba del contenedor                     | Por rellenar |
| Copia llevada fuera del PC                                | Por rellenar |

## 6. Problemas encontrados

| Síntoma      | Causa        | Solución     |
| ------------ | ------------ | ------------ |
| Por rellenar | Por rellenar | Por rellenar |
