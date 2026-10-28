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

<figure><img src="../../.gitbook/assets/Captura de pantalla 2026-10-28 154819.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Captura de pantalla 2026-10-28 154808.png" alt=""><figcaption></figcaption></figure>

***

## 2. Copia de seguridad programada

En **Datacenter → Backup → Add**:

<table><thead><tr><th width="189">Campo</th><th width="281">Valor</th><th>Por qué</th></tr></thead><tbody><tr><td>Node</td><td><code>honeynet</code></td><td>El nodo del laboratorio</td></tr><tr><td>Storage</td><td><code>local</code></td><td>Donde se guardan las copias</td></tr><tr><td>Schedule</td><td>Todos los días a las 03:00</td><td>De madrugada, sin carga en el equipo</td></tr><tr><td>Selection mode</td><td><em>Include selected VMs</em>: <code>vm-nginx</code> y <code>ct-prueba</code></td><td>Las plantillas no hace falta copiarlas</td></tr><tr><td>Mode</td><td>Snapshot</td><td>Copia la máquina sin apagarla</td></tr><tr><td>Compression</td><td>ZSTD</td><td>Rápida y ocupa poco</td></tr><tr><td>Retention (<em>Keep Last</em>)</td><td>3</td><td>Conserva las últimas tres y borra las anteriores para no llenar el disco</td></tr></tbody></table>

Para no esperar a las 03:00, se selecciona el trabajo y se pulsa **Run now**. Al terminar (`TASK OK`), en **honeynet → local → Backups** aparecen los archivos de las dos máquinas.

<figure><img src="../../.gitbook/assets/Captura de pantalla 2026-10-28 154617.png" alt=""><figcaption></figcaption></figure>

***

## 3. Probar la restauración

Una copia que no se ha restaurado nunca no es de fiar, así que se prueba:

1. En **local → Backups**, seleccionar la copia de `ct-prueba` → **Restore**.
2. Poner un ID nuevo (por ejemplo `199`) y desmarcar _Start after restore_.
3. Comprobar que aparece el contenedor restaurado y borrarlo después (**More → Remove**).

<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

***
