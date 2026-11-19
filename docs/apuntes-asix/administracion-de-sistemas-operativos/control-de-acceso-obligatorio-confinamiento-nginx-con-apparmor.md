# Control de Acceso Obligatorio: confinamiento Nginx con AppArmor

## **Objetivo**

Configurar un servidor web **Nginx** en una máquina Linux (Kali) y crear un perfil de **AppArmor** que lo confine, primero en modo **complain** (aprendizaje) y después en modo **enforce** (aplicación), comprobando que el servicio sigue funcionando correctamente tras el confinamiento.

AppArmor es un framework de **Control de Acceso Obligatorio (MAC)** para Linux que restringe, por perfil y por binario, a qué recursos del sistema puede acceder un proceso, con independencia de los permisos UNIX habituales (DAC).

***

## **Entorno utilizado**

| Parámetro                           | Valor                                       |
| ----------------------------------- | ------------------------------------------- |
| Máquina                             | Kali Linux (VM existente, `alexASIX2-kali`) |
| Servicio confinado                  | Nginx (`/usr/sbin/nginx`)                   |
| Herramienta de generación de perfil | `apparmor-utils` (`aa-genprof`)             |
| Módulo AppArmor                     | Cargado en el kernel (`apparmor_status`)    |

***

## **1. Configuración inicial de Nginx**

Instalación de Nginx y de las utilidades de AppArmor:

```bash
sudo apt install nginx apparmor-utils apparmor-profiles python3-apparmor
```

Se comprueba el estado del módulo AppArmor antes de empezar (sin ningún perfil propio cargado todavía):

```bash
sudo apparmor_status
```

```
apparmor module is loaded.
No policy loaded into the kernel.
```

Se habilita y arranca Nginx:

```bash
sudo systemctl enable --now nginx
sudo systemctl status nginx
```

El servicio queda **active (running)**. Verificación de que responde correctamente antes de aplicar ningún confinamiento:

```bash
curl -I http://localhost
```

```
HTTP/1.1 200 OK
Server: nginx
...
```

{% hint style="info" %}
📷 **CAPTURA 1 AQUÍ** — la del `curl -I` repetido + `systemctl reload nginx` (la de las 15:29)
{% endhint %}

> **Nota — dependencia de `rsyslog`**: `aa-genprof` necesita leer `/var/log/syslog`. En Kali, por defecto solo está activo `journald` y ese archivo no existe. Se resuelve instalando y habilitando `rsyslog`:
>
> ```bash
> sudo apt install -y rsyslog
> sudo systemctl enable --now rsyslog
> ```

***

## **2. Creación y ajuste del perfil de AppArmor (modo complain)**

Se lanza el generador de perfiles sobre el binario de Nginx:

```bash
sudo aa-genprof nginx
```

`aa-genprof` pone el perfil en **modo complain** (aprendizaje: registra los eventos pero no bloquea nada) y queda esperando a que se genere tráfico real contra el servicio.

En una segunda terminal se genera tráfico variado para que AppArmor capture los accesos reales que necesita Nginx:

```bash
curl http://localhost
curl -I http://localhost
curl http://localhost/noexiste123
sudo systemctl reload nginx
```

De vuelta en `aa-genprof`, se escanea el log del sistema (**S**) y la herramienta va mostrando los eventos AppArmor detectados, pidiendo decidir sobre cada uno (**A**llow / **D**eny / **I**gnore / **Audit** / **F**inalizar). Entre los eventos detectados:

* `capability dac_override` → se permite (**Allow**), necesaria porque Nginx arranca como root para poder enlazar el puerto 80 y accede a archivos que no le pertenecen directamente.
* Acceso a `/var/log/nginx/access.log` (modo `mrwk`) → añadido automáticamente, necesario para que Nginx pueda escribir el log de peticiones.
* Acceso a `/run/nginx.pid` (`owner`, `mrwk`) → añadido automáticamente, necesario para que `systemd` pueda gestionar el proceso por su PID.

Tras decidir sobre los eventos, se guardan los cambios en el perfil:

```
= Changed Local Profiles =
(S)ave Changes / Save Selec(t)ed Profile / [(V)er cambios] / Abo(r)t
```

Se pulsa **S** para guardar, confirmando con:

```
Escribiendo perfil actualizado para /usr/sbin/nginx.
```

Se repite el ciclo de generar tráfico → escanear (**S**) una vez más para cubrir casos adicionales, y finalmente se pulsa **F** para finalizar `aa-genprof`.

***

## **3. Evidencias de los registros en modo complain**

Con el perfil todavía en complain, se consultan los eventos AppArmor registrados en el log del sistema:

```bash
sudo grep -i apparmor /var/log/syslog | tail -40
```

Extracto representativo de los eventos capturados (operaciones de red sobre IPv6 no cubiertas explícitamente por el perfil, permitidas por estar en modo complain):

```
apparmor="ALLOWED" operation="accept" profile="/usr/sbin/nginx" pid=10779 comm="nginx" family="inet6" sock_type="stream" protocol=6 requested_mask="accept" denied_mask="accept"
apparmor="ALLOWED" operation="recvmsg" profile="/usr/sbin/nginx" pid=10779 comm="nginx" family="inet6" sock_type="stream" protocol=6 requested_mask="receive" denied_mask="receive"
apparmor="ALLOWED" operation="sendmsg" profile="/usr/sbin/nginx" pid=10779 comm="nginx" family="inet6" sock_type="stream" protocol=6 requested_mask="send" denied_mask="send"
apparmor="ALLOWED" operation="setsockopt" profile="/usr/sbin/nginx" pid=10779 comm="nginx" family="inet6" sock_type="stream" protocol=6 requested_mask="setopt" denied_mask="setopt"
apparmor="STATUS" operation="profile_replace" profile="unconfined" name="/usr/sbin/nginx" comm="apparmor_parser"
```

Estos eventos `ALLOWED` con `info="failed af match"` indican que el tráfico se permitió (porque el perfil estaba en complain), pero que la regla de red del perfil no cubría explícitamente ese tipo de conexión (IPv6). Es precisamente la utilidad del modo complain: observar qué pide realmente la aplicación antes de pasar a bloquear en modo enforce.

{% hint style="info" %}
📷 **CAPTURA 2 AQUÍ** — la de los logs largos `apparmor="ALLOWED"` con el intento fallido de `cat` sin sudo (la de las 15:33, la primera)
{% endhint %}

***

## **4. Perfil final en modo enforce**

Se activa el modo enforce sobre el perfil de Nginx:

```bash
sudo aa-enforce /usr/sbin/nginx
```

Se confirma el cambio de modo:

```bash
sudo aa-status
```

```
3 processes are in enforce mode.
   /usr/sbin/nginx (10778)
   /usr/sbin/nginx (10779)
   /usr/sbin/nginx (10780)
0 processes are in complain mode.
```

{% hint style="info" %}
📷 **CAPTURA 3 AQUÍ** — la del `aa-status` con "3 processes are in enforce mode" (la de las 15:31)
{% endhint %}

Contenido final del perfil (`/etc/apparmor.d/usr.sbin.nginx`):

```
# Last Modified: Tue Nov 17 19:26:43 2026
abi <abi/4.0>,

include <tunables/global>

/usr/sbin/nginx {
  include <abstractions/base>
  include <abstractions/dovecot-common>
  include <abstractions/lightdm>
  include <abstractions/postfix-common>

  capability dac_override,

  /usr/sbin/nginx mr,
  /var/** mrwk,
  owner /proc/sys/kernel/random/boot_id r,
  owner /{,var/}run/** mrwk,
}
```

{% hint style="info" %}
📷 **CAPTURA 4 AQUÍ** — la del `sudo cat /etc/apparmor.d/usr.sbin.nginx` con el perfil completo (la de las 15:33, la segunda)
{% endhint %}

> **Observación**: las reglas `/var/** mrwk` y `owner /{,var/}run/** mrwk` son más amplias de lo estrictamente necesario — `aa-genprof` generalizó en vez de limitarse a `/var/log/nginx/access.log` y `/run/nginx.pid`, que eran los accesos concretos detectados durante el aprendizaje. En un entorno de producción convendría ajustar manualmente estas rutas a los archivos exactos que usa Nginx.

***

## **5. Comprobación de Nginx tras el confinamiento (modo enforce)**

Con el perfil ya en enforce, se repite la verificación del servicio:

```bash
sudo systemctl status nginx
curl -I http://localhost
curl http://localhost
```

```
HTTP/1.1 200 OK
Server: nginx
Date: Thu, 19 Nov 2026 14:34:49 GMT
Content-Type: text/html
Content-Length: 10703
...
```

{% hint style="info" %}
📷 **CAPTURA 5 AQUÍ** — la del `curl -I` con 200 OK ya en enforce (la de las 15:35, la primera)
{% endhint %}

{% hint style="info" %}
📷 **CAPTURA 6 AQUÍ** — la del `curl` con el HTML completo (la de las 15:35, la segunda)
{% endhint %}

Nginx sigue respondiendo con normalidad (**200 OK**, página por defecto servida correctamente), confirmando que el perfil AppArmor en modo enforce no bloquea ninguna operación necesaria para el funcionamiento del servicio.

***

## **6. Reflexión: accesos necesarios y por qué**

Mirando el perfil final, estos son los permisos que hizo falta dar y para qué sirve cada uno:

* **`dac_override`**: Nginx arranca como root porque necesita el puerto 80, que es un puerto "privilegiado" (los puertos por debajo del 1024 solo los puede abrir root). Por eso, aunque no sea el dueño de algunos archivos, necesita este permiso para poder tocarlos igualmente.
* **Escribir en `/var/log/nginx/access.log`**: es el log donde Nginx apunta cada petición que le llega (quién entra, a qué hora, qué pide). Sin poder escribir ahí, no podría registrar nada.
* **Acceso a `/run/nginx.pid`**: en este archivo Nginx guarda su propio PID. Lo necesita para que el sistema (con `systemctl`) pueda pararlo o reiniciarlo sin problema.
* **Leer y ejecutar `/usr/sbin/nginx`**: es el propio programa de Nginx, necesita poder leerlo y ejecutarlo para arrancar.

En los logs en modo complain también salieron eventos relacionados con IPv6 en operaciones de red (`accept`, `recvmsg`, `sendmsg`). No estaban permitidos explícitamente en el perfil, pero como estaba en complain, AppArmor solo los apuntó sin bloquear nada — para eso sirve este modo: ver qué pide la aplicación antes de cerrar el grifo. Al pasar a enforce, Nginx siguió respondiendo bien, así que esos accesos de red no eran imprescindibles para las pruebas realizadas.

En resumen, la idea es darle a Nginx solo lo justo y necesario para funcionar, en vez de dejarlo con vía libre por todo el sistema — el principio de **mínimo privilegio**.

***

## **Complain vs. Enforce**

|                        | Modo complain                                                           | Modo enforce                                                        |
| ---------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------- |
| Comportamiento         | Registra las violaciones del perfil en el log, pero **no bloquea nada** | **Bloquea** activamente cualquier acceso no permitido por el perfil |
| Uso típico             | Fase de aprendizaje/ajuste del perfil (`aa-genprof`, `aa-logprof`)      | Producción, una vez el perfil está validado                         |
| Riesgo                 | Ninguno de romper el servicio, pero tampoco protege                     | Puede romper el servicio si el perfil está incompleto               |
| Comando para activarlo | `sudo aa-complain <perfil>`                                             | `sudo aa-enforce <perfil>`                                          |

### **Resumen de comandos**

| Comando                                   | Para qué sirve                                                                                |
| ----------------------------------------- | --------------------------------------------------------------------------------------------- |
| `sudo apparmor_status` / `sudo aa-status` | Ver el estado de AppArmor y los perfiles cargados, y en qué modo está cada uno                |
| `sudo aa-genprof <binario>`               | Generar/ajustar un perfil de forma interactiva en modo complain, a partir del log del sistema |
| `sudo aa-logprof`                         | Revisar eventos nuevos y seguir ajustando un perfil ya existente                              |
| `sudo aa-complain <perfil>`               | Pasar un perfil a modo complain                                                               |
| `sudo aa-enforce <perfil>`                | Pasar un perfil a modo enforce                                                                |
| `sudo grep -i apparmor /var/log/syslog`   | Ver los eventos AppArmor registrados                                                          |
