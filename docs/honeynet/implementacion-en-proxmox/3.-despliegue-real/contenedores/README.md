---
description: >-
  Los contenedores del despliegue real: la web, la base de datos y las copias de
  seguridad.
---

# Contenedores

Los servicios del proyecto viven en contenedores LXC ligeros, conectados a `vmbr1` y protegidos por pfSense.

* [Contenedores ct-db y ct-web](contenedores-ct-db-y-ct-web.md): recursos, red y salida a internet.
* [Base de datos](base-de-datos.md): MariaDB y el script de la base de datos.
* [Copias de seguridad con ct-backup](copias-de-seguridad-con-ct-backup.md): copias nocturnas de la base de datos y de la web.
