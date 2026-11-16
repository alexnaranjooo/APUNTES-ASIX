---
description: >-
  MariaDB en ct-db: instalación, carga del script y usuario de la web con mínimo
  privilegio.
---

# Base de datos

En `ct-db` se instala MariaDB, que escucha solo en su IP interna, y se carga el script `honeypot_web.sql`, que crea las tablas, las vistas, los procedimientos, los triggers y los eventos. La web usa un usuario que solo puede ejecutar procedimientos (ver [Diagrama de la base de datos](../../../introduccion-al-proyecto/arquitectura-del-software/3.-diagrama-de-la-base-de-datos.md)).

<figure><img src="../../../../.gitbook/assets/11-ct-db-tablas.png" alt=""><figcaption></figcaption></figure>

## Limpieza y contraseñas antes de entregar

**Concepto:** durante las pruebas la web de ejemplo (`index.php`) llevaba la contraseña de la base de datos escrita en claro. Se borra y se cambian las contraseñas, para que no quede ninguna credencial de prueba en el sistema.

**En `ct-web`:** se comprueba que ningún otro archivo usa los usuarios de la base de datos, se borra el `index.php` de prueba y se deja una página inocua para que nginx no dé un 403.

```bash
grep -rln "web_app\|web_admin" /var/www /etc/nginx 2>/dev/null
rm /var/www/honeynet.lan/public/index.php
echo "HoneyNet - en construccion" > /var/www/honeynet.lan/public/index.html
curl -s localhost
```

**En `ct-db`:** los usuarios `web_app` y `web_admin` solo existen para el host `10.0.60.10` (`ct-web`), así que no se puede entrar con ellos desde ninguna otra máquina. Se generan contraseñas al azar de 32 caracteres y se cambian:

```bash
P1=$(openssl rand -hex 16); P2=$(openssl rand -hex 16)
mariadb -e "ALTER USER 'web_app'@'10.0.60.10' IDENTIFIED BY '$P1'; ALTER USER 'web_admin'@'10.0.60.10' IDENTIFIED BY '$P2'; FLUSH PRIVILEGES"
```

Las contraseñas se guardan en un gestor de contraseñas, nunca en el repositorio, en la documentación ni en capturas.
