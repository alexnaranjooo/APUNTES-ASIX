---
description: Aquí podremos ver aspectos principales de la seguridad.
---

# Aspectos Principales

### _<mark style="color:$danger;">**Introducción**</mark>_

La **seguridad informática** es el conjunto de medidas y procedimientos orientados a proteger los sistemas informáticos frente a posibles ataques y daños, ya sean intencionados o accidentales. Estos ataques pueden dirigirse contra el **hardware**, el **software** o los **datos** de una organización.

Para hacer frente a estas amenazas se distinguen dos grandes enfoques:

* **Seguridad activa**: conjunto de medidas orientadas a **evitar** que se produzca un daño en el sistema.
* **Seguridad pasiva**: conjunto de medidas orientadas a **minimizar** el impacto una vez que el daño ya se ha producido.

Esta página recoge los apuntes de la unidad de seguridad: mecanismos de protección, la tríada CID, elementos vulnerables, seguridad física y lógica, seguridad de red, control de acceso y análisis forense.

***

### _<mark style="color:$danger;">**Mecanismos de seguridad de un sistema informático**</mark>_

Son los procedimientos que se llevan a cabo para garantizar la seguridad del sistema. Se agrupan en cuatro fases:

1. **Prevención** — acciones para evitar una posible intrusión, ya sea a nivel de software, hardware o red.
2. **Detección** — identificar el momento en que se produce un ataque y tomar medidas para frenarlo (eliminar procesos, filtrar puertos, apagar el equipo, etc.).
3. **Restauración** — recuperación del sistema mediante copias de seguridad, instantáneas (snapshots), etc.
4. **Análisis forense** — determinar qué acciones ha realizado el atacante, con el objetivo de blindar el sistema ante ataques futuros.

#### _**Aspectos que incluye la seguridad**_

* **CID**: confidencialidad, integridad y disponibilidad.
* Seguridad **física** y **lógica**.
* Prevención de amenazas y ataques.
* Seguridad perimetral: **Firewall** y **Proxy**.
* Alta disponibilidad.
* Legislación aplicable.

> **Nota — Índice de la unidad (pautas de seguridad)**
>
> 1. Confidencialidad, integridad y disponibilidad
> 2. Elementos vulnerables: aplicaciones, dispositivos y datos
> 3. Principales vulnerabilidades de un sistema informático
> 4. Tipos de amenazas físicas y lógicas
> 5. Seguridad física y ambiental
> 6. Seguridad lógica (listas de control de acceso, políticas de contraseñas, políticas de almacenamiento, copias de seguridad, medios de almacenamiento)
> 7. Análisis forense en sistemas informáticos

<figure><img src="../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

***

### <mark style="color:$danger;">**Confidencialidad, integridad y disponibilidad (CID)**</mark>

#### _**Confidencialidad**_

El acceso a la información se produce **solo por personas autorizadas**, estableciendo relaciones de confianza y seguridad en las comunicaciones.

* **Ejemplo**: cifrar un email para que solo el destinatario pueda leerlo.
* **Técnicas**: contraseñas seguras, cifrado, control de accesos, autenticación.

#### _**Disponibilidad**_

Asegura que la información y los sistemas estén **accesibles cuando se necesitan**, evitando interrupciones.

* **Ejemplo**: la web de un banco debe estar operativa 24/7 sin caídas.
* **Técnicas**: copias de seguridad, redundancia, balanceo de carga, protección frente a ataques DoS/DDoS.

#### _**Integridad**_

Garantiza que la información **no sea alterada de manera indebida**, ya sea por accidente o por un ataque.

* **Ejemplo**: verificar mediante un hash que la ISO de un sistema operativo descargada no ha sido manipulada.
* **Técnicas**: sumas de verificación, firmas digitales, control de versiones.

<figure><img src="../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

***

### _<mark style="color:$danger;">**Elementos vulnerables**</mark>_

#### _**Hardware**_

* Servidores, PC, routers, switches, firewalls.
* Discos duros, SSD, NAS.
* Dispositivos móviles: portátiles, smartphones.
* **Riesgos**: robos, fallos eléctricos, incendios, humedad, manipulación física.

#### _**Software**_

* Sistemas operativos.
* Aplicaciones de usuario.
* Software de red: protocolos, servicios de correo, bases de datos, servidores web.
* **Riesgos**: errores de programación, falta de actualizaciones, malware, configuraciones inseguras.

#### _**Datos**_

* Archivos confidenciales.
* Bases de datos.
* Información en tránsito por la red: mensajes, contraseñas, emails.
* **Riesgos**: pérdidas, corrupción, robo, alteraciones, interceptación en la red.

#### _**Red**_

* Conexiones a Internet.
* Redes internas (LAN, WLAN).
* Dispositivos de comunicación: antenas, puntos de acceso Wi-Fi.
* **Riesgo principal**: acceso no autorizado.

#### _**Usuarios**_

* Administradores del sistema.
* Empleados con acceso a información sensible.
* Proveedores y terceros con permisos de acceso.
* **Riesgos**: ingeniería social, errores humanos, contraseñas débiles, uso indebido de privilegios.

> **Resumen — medidas según el tipo de elemento**
>
> * **Medidas preventivas (seguridad activa)**: evitan que se produzca el daño en el sistema (actualizaciones automáticas/parches, protección de S.O. y aplicaciones).
> * **Medidas correctivas (seguridad pasiva)**: minimizan los daños una vez producido el incidente.

<figure><img src="../../.gitbook/assets/image (52).png" alt="" width="544"><figcaption></figcaption></figure>

***

### _<mark style="color:$danger;">**Vulnerabilidades**</mark>_

Aunque suelen asociarse solo al software, una vulnerabilidad puede tener consecuencias graves:

* Un intruso puede conseguir **permisos de administración** en el sistema.
* Un virus puede tomar el **control de los equipos** de la empresa.
* Un atacante puede **borrar, alterar o cifrar** datos de la empresa.

Todas las vulnerabilidades detectadas se publican en informes **CVE** (_Common Vulnerability Exposure_) — [cve.org](https://www.cve.org/).

#### Otras vulnerabilidades destacadas

* **A nivel de microprocesador**: la CPU se ve obligada a leer datos a los que el usuario no debería tener acceso (ejecución especulativa).
* **Inyecciones SQL** (_SQL injection_): afectan a aplicaciones basadas en bases de datos.
* **HTML/JS injects**: otro tipo de ataque común en aplicaciones web.

<figure><img src="../../.gitbook/assets/image (53).png" alt="" width="542"><figcaption></figcaption></figure>

***

### _<mark style="color:$danger;">**Amenazas y medidas de seguridad física (seguridad pasiva)**</mark>_

#### _**Amenazas físicas**_

* Cortes de suministro, robos, incendios, desastres atmosféricos.
* Condiciones adversas: temperaturas extremas, humedad excesiva, inundaciones, terremotos.

**Consecuencias**: falta de disponibilidad de servicios, pérdida de información, pérdida o mal funcionamiento del hardware.

#### _**Barreras y protecciones físicas**_

* Servidores cerrados con llave y acceso restringido.
* Controles de acceso con tarjeta, guardia de seguridad o biometría.
* Puertas con apertura programada.
* Protección eléctrica y protección antiincendios adecuada.

> **Nota**: no se deben ubicar los centros de datos en plantas bajas o sótanos, por el riesgo de inundación.

#### _**Medidas en CPD y grandes organizaciones**_

* **SAI** (Sistema de Alimentación Ininterrumpida) y generadores eléctricos.
* Fuentes de alimentación redundantes.
* Personal de vigilancia y cámaras de seguridad.
* Centro de respaldo en ubicación diferente, con información sincronizada.
* **Alta disponibilidad y monitorización**: objetivo típico de 99,99 %, servicio 24x7x365.
* Redundancia y diversificación: almacenamiento externo de datos, tomas eléctricas independientes, telecomunicaciones con balanceo de carga.

#### _**La sala fría**_

1. Concentra los grandes servidores, con sistema propio de refrigeración (**21–23 ºC** y humedad relativa entre **40 % y 60 %**).
2. Medidas estrictas de control de acceso físico.
3. Sistemas de extinción de incendios.

<figure><img src="../../.gitbook/assets/image (54).png" alt="" width="563"><figcaption></figcaption></figure>

***

### _<mark style="color:$danger;">**Amenazas y medidas de seguridad lógica (seguridad pasiva)**</mark>_

#### _**Amenazas lógicas**_

Virus, ataques DoS, phishing y troyanos son ejemplos de amenazas lógicas que pueden descargarse o propagarse por la red, afectando no solo a un dispositivo sino a toda la infraestructura.

La instalación de programas no testeados puede provocar **desbordamientos de búfer** (_buffer overflow_) y condiciones de carrera, permitiendo a un atacante obtener privilegios y leer o escribir ficheros protegidos.

#### _**Medidas lógicas principales**_

* Limitar el acceso a programas y archivos.
* Restringir el acceso a datos según el usuario: claves de acceso, listas de control de acceso, roles, firewalls.
* Garantizar que la información llegue únicamente al destinatario previsto.
* Establecer sistemas alternativos para compartir información y redundancia de sistemas.
* **Políticas de almacenamiento** y copias de seguridad.
* **Cifrado** de datos y comunicaciones.
* **Políticas de contraseñas** robustas.
* Protección del software (S.O. y aplicaciones) y permisos de usuario.
* Actualizaciones y filtrado de conexiones en aplicaciones de red.
* Software antimalware.

<figure><img src="../../.gitbook/assets/image (55).png" alt="" width="450"><figcaption></figcaption></figure>

***

### _<mark style="color:$danger;">**Seguridad de la red**</mark>_

Para defender una red primero hay que entender de qué está compuesta, cómo se comunican sus elementos y cuál es su arquitectura. Las áreas clave son:

* **Segmentación de la red.**
* **Controles de acceso** adecuados.
* **Defensa en profundidad** (_Defense in Depth_, DiD).
* **Privilegios mínimos** desde el diseño.

#### _**Amenazas y medidas por capa OSI**_

<table><thead><tr><th width="154">Capa OSI</th><th width="296">Amenaza</th><th>Medida</th></tr></thead><tbody><tr><td><strong>Física</strong></td><td><em>Sniffing</em>: interceptación de señales físicas (cables, Wi-Fi) para capturar datos sin cifrar (ej. Wireshark). Robo de cables.</td><td>Cifrado robusto como <strong>WPA3</strong> en redes Wi-Fi. Fibra óptica en tubos sellados con gas inerte a presión (uso militar).</td></tr><tr><td><strong>Enlace de datos</strong></td><td>Suplantación de direcciones <strong>MAC</strong> para evitar el control de acceso (ej. <em>ARP spoofing</em> para redirigir tráfico).</td><td>Configuraciones <strong>anti-spoofing</strong> y detección de ataques ARP/DHCP. Uso de <strong>VLAN</strong> para segmentar la red.</td></tr><tr><td><strong>Red</strong></td><td><strong>Man in the Middle</strong>: interceptar o modificar el tráfico entre dos dispositivos (ej. redes Wi-Fi abiertas o mal configuradas).</td><td>Uso de <strong>VPN</strong> para cifrar la información en redes vulnerables o monitorizadas.</td></tr><tr><td><strong>Transporte</strong></td><td>Escaneo de puertos y ataques de denegación de servicio (ej. <strong>Nmap</strong>, usado tanto en hacking ético como malicioso).</td><td>Auditar puertos y protocolos expuestos. Uso de <strong>firewalls</strong> (filtrado de paquetes, de red o de nueva generación) y <strong>IDS</strong>.</td></tr><tr><td><strong>Sesión</strong></td><td><em><strong>Hijacking</strong></em>: secuestro de sesión activa (ej. robo de cookies de sesión).</td><td>Tokens de sesión seguros o de un solo uso. Autenticación multifactor (<strong>MFA</strong>).</td></tr><tr><td><strong>Presentación</strong></td><td><strong>Phishing</strong>: obtención de información mediante engaño e ingeniería social (ej. emails falsos).</td><td>Formación de usuarios y herramientas de filtrado/anti-phishing.</td></tr><tr><td><strong>Aplicación</strong></td><td><strong>Exploits</strong>: aprovechamiento de vulnerabilidades en aplicaciones (ej. explotación de CVE en software desactualizado).</td><td>Mantener el software actualizado y usar soluciones antimalware contra vulnerabilidades de día cero.</td></tr><tr><td><strong>Factor humano</strong></td><td>Malas configuraciones, contraseñas débiles, caídas por error humano, falta de formación en ciberseguridad.</td><td>Formación continua y concienciación en ciberseguridad.</td></tr></tbody></table>

<figure><img src="../../.gitbook/assets/image (56).png" alt="" width="563"><figcaption></figcaption></figure>

***

### _<mark style="color:$danger;">**Control de acceso**</mark>_

#### _**Autenticación vs. autorización**_

* **Autenticación**: verifica la **identidad** de un usuario en el sistema y determina si puede entrar o se queda fuera.
* **Autorización**: determina si el usuario **puede acceder** al recurso solicitado, consultando la base de datos de autorización (mantenida por el administrador), donde se especifica el tipo de acceso de cada usuario a cada recurso.

El flujo típico es: **usuario del sistema → autenticación → control de acceso → recursos del sistema** (ficheros, bases de datos, aplicaciones), todo ello validado contra la base de datos de autorización que gestiona el administrador.

#### _**Objetos, sujetos y derechos de acceso**_

* **Objeto**: recurso cuyo acceso está controlado (una entidad para almacenar y/o recibir información). Ejemplos: ficheros, directorios y aplicaciones.
* **Sujeto**: entidad capaz de acceder a un objeto, ligada al concepto de **proceso**. Habitualmente se habla de "usuarios" como sujetos, aunque en realidad el acceso se realiza a través de un proceso que representa a ese usuario o aplicación.

Un sistema de control de acceso básico define **tres clases de sujetos**:

| Clase           | Descripción                                                                                                          |
| --------------- | -------------------------------------------------------------------------------------------------------------------- |
| **Propietario** | Creador del recurso, fichero o directorio.                                                                           |
| **Grupo**       | Conjunto de usuarios con privilegios de acceso a determinados recursos; un usuario puede pertenecer a varios grupos. |
| **Otros**       | Usuarios que han entrado en el sistema pero no son propietarios ni pertenecen al grupo del recurso.                  |

#### _**Derechos de acceso**_

* **Lectura**: visualizar información de un recurso (permite copiar o imprimir).
* **Escritura**: agregar, modificar o eliminar datos de un recurso.
* **Ejecución**: ejecutar programas específicos.
* **Borrado**: eliminar recursos como ficheros o registros.
* **Creación**: crear nuevos ficheros, registros o directorios.

***

### _<mark style="color:$danger;">**Listas de control de acceso (ACL)**</mark>_

Las **ACL** se utilizan en sistemas y redes para administrar los permisos de acceso a recursos (archivos, directorios, dispositivos de red, etc.), determinando quién puede acceder a qué y qué acciones puede realizar. Son una pieza fundamental de la seguridad de la información.

#### _**DAC — Control de acceso discrecional**_

El **propietario del recurso** tiene control total sobre quién accede y con qué permisos. Por ejemplo, un usuario puede definir quién puede leer, escribir o ejecutar un archivo de su propiedad.

**Ejemplo de matriz DAC:**[Listas de control de acceso (ACL)](https://app.gitbook.com/o/QNZ7kWO4LQvYzNxlDie4/sites/site_oM9mv/s/4kWIiYmAwL2bY5HlTph8/~/edit/~/changes/20/seguridad/aspectos-principales#listas-de-control-de-acceso-acl)

| Sujeto     | /home/albert         | /home/alicia         | /home/ricard         | /etc/passwd |
| ---------- | -------------------- | -------------------- | -------------------- | ----------- |
| **Albert** | Read, Write, Execute | —                    | —                    | Read        |
| **Alicia** | —                    | Read, Write, Execute | —                    | Read        |
| **Ricard** | Read, Write, Execute | Read, Write, Execute | Read, Write, Execute | Read, Write |

#### _**MAC — Control de acceso obligatorio**_

Los sistemas operativos implementan mecanismos de control mediante **ACL** incorporadas en cada objeto (directorios, ficheros, recursos de red). Cuando un usuario intenta acceder a un objeto, el **núcleo del sistema operativo** comprueba su ACL para determinar si tiene derecho de acceso.

* La ACL es una lista de entradas con: **usuario o grupo**, **operación** (lectura, escritura) y **permiso** (denegar, permitir).
* Los permisos se asignan según **políticas y etiquetas de seguridad**.
* Es habitual en sistemas de seguridad de alto nivel (gobierno, ejército), donde se requiere un control más estricto.

> **ACL vs. permisos tradicionales**: las ACL controlan los permisos de archivos y directorios con **mayor precisión** que `chmod`, `chown` o `chgrp`, especificando qué usuarios o procesos tienen acceso a qué objetos y qué operaciones tienen permitidas.

**Herramientas por sistema operativo:**

* **Linux**: gestión de ACL mediante `setfacl` y `getfacl`.
* **Windows**: las ACL son la base de los permisos **NTFS**.

<figure><img src="../../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

***

### _<mark style="color:$danger;">**Análisis forense en sistemas informáticos**</mark>_

Consiste en realizar una búsqueda detallada para **reconstruir los acontecimientos** ocurridos hasta el momento de detección de un ataque, asegurando en todo momento la conservación intacta de la información del sistema comprometido.

El análisis se apoya en ficheros log, el estudio del sistema de ficheros (FS) del equipo afectado y la reconstrucción de la secuencia de eventos. Es una ciencia **sistemática**, basada en hechos, orientada a recabar pruebas que después serán analizadas.

> **Nota**: no existe un procedimiento único aplicable "a rajatabla". Las técnicas generales deben combinarse con la experiencia del analista para desarrollar métodos propios.

#### _**El análisis termina cuando el forense conoce**_

1. Cómo se produjo el compromiso.
2. Bajo qué circunstancias.
3. La identidad del/los posibles atacantes.
4. Su procedencia y origen.
5. Fechas clave.
6. Los objetivos del/los atacantes.
7. La reconstrucción de la línea temporal de los eventos.

> Los archivos guardan metadatos (autor, compañía, fecha...) que pueden llegar a identificar el equipo usado para crearlos. Una imagen o archivo audiovisual puede protegerse mediante **marcas de agua digitales**, que permiten determinar su origen incluso si ha sido modificado.

#### _**Políticas de seguridad de la organización**_

Antes de un incidente, una organización debe definir:

* Activos de la empresa y qué aspectos de seguridad deben considerarse.
* Responsabilidades y planes de contingencia.
* Medidas de control y auditorías periódicas de los sistemas.
* Estimación preliminar de riesgos a partir de los puntos débiles de la infraestructura.
* El coste de una pérdida de información frente al coste de protegerla.

#### _**Información a recopilar del equipo afectado**_

* Sistema operativo e inventario del software instalado.
* Hardware y periféricos conectados.
* Presencia de firewall, si está en una DMZ o conectado a Internet.
* Configuración del equipo y parches/actualizaciones instaladas.
* Si el almacenamiento está cifrado.
* Si existe un IDS.
* El resto de equipos de la red.

#### _**Las cinco fases del análisis forense informático**_

1. **Identificación** — localizar los sistemas y dispositivos que pueden contener evidencia digital relevante (PC, servidores, móviles, medios de almacenamiento), para acotar el alcance del análisis.
2. **Adquisición** — crear imágenes forenses (copia bit a bit) de los dispositivos originales, capturando también ficheros borrados y datos ocultos, sin alterar la evidencia.
3. **Preservación** — almacenar los datos de forma segura sin modificarlos, usando cifrado y medios seguros. Mantener la **cadena de custodia** es esencial para que la evidencia sea válida ante un tribunal.
4. **Análisis** — examinar los datos preservados con herramientas y técnicas específicas: recuperación de archivos borrados, revisión de registros, búsqueda de malware y patrones relevantes.
5. **Documentación y presentación** — registrar cada paso del proceso (capturas, informes, notas de herramientas y hallazgos) y comunicar los resultados de forma clara a las partes interesadas (abogados, jueces, auditores), incluyendo la preparación de testimonios si es necesario.

***

