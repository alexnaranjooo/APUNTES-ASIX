---
description: >-
  Diseño propuesto para que la web de la empresa tenga un login de administrador
  desde el que ver los logs y los paneles de Grafana, sin debilitar el
  aislamiento de la red.
---

# Acceso de administrador desde la web

## Idea

La web de la empresa (ver [Parte Web](parte-web.md)) tendrá un **login de administrador**. Al iniciar sesión, el administrador podrá ver los **paneles de Grafana** (gráficos y logs de ataques) sin salir de la web.

Estado: **propuesta**, se implementa en la Fase 6, cuando Grafana ya funcione. Se asume que el servidor web va en una VLAN propia (VLAN 60), separada de los honeypots.

{% hint style="warning" %}
**El problema de seguridad:** la web vive en una zona pensada para que la vea gente, y Grafana vive en la VLAN 30 (gestión), la zona más protegida. Si la web puede llegar a Grafana, un atacante que comprometa la web tiene un camino hacia la gestión. Todo el diseño consiste en abrir ese camino lo mínimo posible.
{% endhint %}

## Dos maneras de hacerlo

|                            | **A. Proxy inverso**                                             | **B. Instantáneas empujadas**                                       |
| -------------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------- |
| Cómo funciona              | nginx reenvía `/grafana/` a Grafana, solo si has iniciado sesión | La zona de gestión genera copias de los paneles y las sube a la web |
| Flujo nuevo en el firewall | Web → Grafana (TCP 3000)                                         | Ninguno: lo inicia gestión, que ya puede entrar a todo              |
| Datos                      | En vivo                                                          | Con unos minutos de retraso                                         |
| Dificultad                 | Media                                                            | Más alta (hay que automatizar la subida)                            |

**Opción elegida: la A**, con las protecciones de abajo. La B es más segura, pero necesita más piezas.

## Protecciones de la opción A

{% stepper %}
{% step %}
### Una sola regla de firewall

El servidor web puede llegar a Grafana por el puerto 3000, con log. Nada más de la VLAN 30.
{% endstep %}

{% step %}
### Zona de administración solo accesible desde dentro

Aunque la web pública se vea desde fuera, la ruta `/admin` en nginx solo responde a las IPs de la red interna y de NetBird. Es la protección más importante.
{% endstep %}

{% step %}
### Login bien hecho

* Contraseñas guardadas con `password_hash` de PHP.
* Consultas preparadas contra inyección SQL.
* Cookies de sesión seguras (`httponly`, `secure`, `samesite`).
* Límite de intentos con nginx y, a ser posible, segundo factor.
{% endstep %}

{% step %}
### Grafana en solo lectura

Usuario de solo lectura y acceso anónimo desactivado. El proxy solo deja pasar las rutas de los paneles, no las de administración de Grafana.
{% endstep %}

{% step %}
### No mostrar logs en bruto

Los logs de los honeypots contienen texto que escribe el atacante (usuarios, comandos). Si se muestra sin escapar en nuestra propia página, es una vía para atacar al administrador. Se muestran paneles de Grafana, que ya escapa el contenido.
{% endstep %}

{% step %}
### No exponer Wazuh por la web

Los logs se ven a través de paneles de Grafana, no del dashboard de Wazuh.
{% endstep %}
{% endstepper %}

## Qué cambia en el plan

* La matriz de reglas del firewall gana una excepción nueva: **Web → Grafana (TCP 3000)**.
* Se hace en la **Fase 6**. Hasta entonces, la web se despliega sin la zona de administración.
* El desarrollo del login es trabajo de la web y se puede ir haciendo en paralelo.
