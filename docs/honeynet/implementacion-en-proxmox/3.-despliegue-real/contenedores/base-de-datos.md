---
description: >-
  MariaDB en ct-db: instalación, carga del script, usuario de la web con mínimo
  privilegio y pruebas con datos de ejemplo.
---

# Base de datos

En `ct-db` se instala MariaDB, que escucha solo en su IP interna, y se carga el script `honeypot_web.sql`, que crea las tablas, las vistas, los procedimientos, los triggers y los eventos. La web usa un usuario que solo puede ejecutar procedimientos (ver [Diagrama de la base de datos](../../../introduccion-al-proyecto/arquitectura-del-software/3.-diagrama-de-la-base-de-datos.md)).

<figure><img src="../../../../.gitbook/assets/11-ct-db-tablas.png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$warning;">**Pruebas con datos de ejemplo**</mark>

Para comprobar que todo funciona se usa el script `honeypot_web_pruebas.sql`. Inventa datos (mensajes, un administrador, intentos de login, planes nuevos) y llama a cada procedimiento y a cada trigger. También lanza a propósito los casos que deben fallar, que son los marcados con `-- error` en el script: un error en esos casos es el resultado correcto.

Se ejecuta desde la consola de `ct-db`, fuera de `mysql`, en la carpeta donde esté el script:

```bash
mariadb -t --force -u root < honeypot_web_pruebas.sql | less -S
```

`-t` muestra los resultados en tablas, `--force` hace que siga aunque una prueba dé error a propósito y `-S` evita que se parta la línea en las tablas anchas. Se sale de `less` con `q`.

### <mark style="color:$warning;">**Qué comprueba cada bloque**</mark>

| Bloque | Qué se prueba                                           | Resultado esperado                                                                                                                                                                 |
| ------ | ------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1      | Zona pública: `sp_listar_planes` y `sp_listar_sectores` | Salen los planes con sus características y los sectores                                                                                                                            |
| 2      | Función `fn_precio_anual`                               | Precio anual con descuento, y vacío para el plan «a consultar»                                                                                                                     |
| 3      | Formulario con mensajes correctos                       | Se guardan los dos mensajes y el trigger limpia los espacios del nombre                                                                                                            |
| 4      | Formulario con datos incorrectos                        | Error por mensaje demasiado corto, teléfono no válido y plan inexistente                                                                                                           |
| 5      | Anti-spam                                               | Los tres primeros mensajes del mismo teléfono pasan y el cuarto da error                                                                                                           |
| 6      | Alta de administradores                                 | El admin se crea con el email normalizado. Una contraseña sin cifrar y un email falso dan error                                                                                    |
| 7      | Bloqueo del login                                       | Con cuatro fallos el usuario aún entra. Con el quinto la cuenta se bloquea. Al desbloquearla se puede entrar y se anota el último acceso                                           |
| 8      | Panel de administración                                 | Los mensajes cambian de estado y se anota quién y cuándo. Un mensaje que no existe da error. Salen las estadísticas                                                                |
| 9      | Gestión de planes                                       | Se crea un plan, se le asocian características y se cambian los precios, con su historial. Borrar un plan, usar uno desactivado o cambiar el precio de uno que no existe dan error |
| 10     | Último administrador                                    | Desactivarlo y borrarlo dan error                                                                                                                                                  |
| 11     | Contraseñas y auditoría                                 | El cambio con un hash válido funciona y con texto plano da error. La auditoría lista los cambios                                                                                   |
| 12     | Mantenimiento                                           | Purgar con menos de 30 días da error. Con 30 días borra el mensaje archivado antiguo                                                                                               |
| 13     | Eventos y permisos                                      | Salen los dos eventos programados y los permisos de `web_app` y `web_admin`, que solo tienen `EXECUTE` y algún `SELECT` sobre vistas                                               |

{% hint style="info" %}
CAPTURA PENDIENTE: la salida de las pruebas, con alguno de los errores esperados (por ejemplo el bloqueo de la cuenta en el bloque 7).
{% endhint %}

{% hint style="info" %}
CAPTURA PENDIENTE: el bloque 13 con los permisos de `web_app` y `web_admin`.
{% endhint %}

### <mark style="color:$warning;">**Antes y después de las pruebas**</mark>

* **Repetirlas:** el anti-spam recuerda los teléfonos de la ejecución anterior durante 10 minutos, así que hay que esperar o limpiar antes.
* **Los datos de prueba no se quedan:** el script crea un administrador `admin` con un hash inventado. Antes de exponer la web hay que volver a dejar la base limpia.
* **Limpiar:** recargar `honeypot_web.sql` borra la base y la recrea con sus datos de ejemplo originales.

```bash
mariadb -u root < honeypot_web.sql
```
