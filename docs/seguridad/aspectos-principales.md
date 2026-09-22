# ASPECTOS PRINCIPALES

## Seguridad Informática

### Introducción

La **seguridad informática** es el conjunto de medidas y procedimientos orientados a proteger los sistemas informáticos frente a posibles ataques y daños, ya sean intencionados o accidentales. Estos ataques pueden dirigirse contra el **hardware**, el **software** o los **datos** de una organización.

Para hacer frente a estas amenazas se distinguen dos grandes enfoques:

* **Seguridad activa**: conjunto de medidas orientadas a **evitar** que se produzca un daño en el sistema.
* **Seguridad pasiva**: conjunto de medidas orientadas a **minimizar** el impacto una vez que el daño ya se ha producido.

Esta página recoge los apuntes de la unidad de seguridad: mecanismos de protección, la tríada CID, elementos vulnerables, seguridad física y lógica, seguridad de red, control de acceso y análisis forense.

***

### Mecanismos de seguridad de un sistema informático

Son los procedimientos que se llevan a cabo para garantizar la seguridad del sistema. Se agrupan en cuatro fases:

1. **Prevención** — acciones para evitar una posible intrusión, ya sea a nivel de software, hardware o red.
2. **Detección** — identificar el momento en que se produce un ataque y tomar medidas para frenarlo (eliminar procesos, filtrar puertos, apagar el equipo, etc.).
3. **Restauración** — recuperación del sistema mediante copias de seguridad, instantáneas (snapshots), etc.
4. **Análisis forense** — determinar qué acciones ha realizado el atacante, con el objetivo de blindar el sistema ante ataques futuros.

### Aspectos que incluye la seguridad

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

***

### Confidencialidad, integridad y disponibilidad (CID)

#### Confidencialidad

El acceso a la información se produce **solo por personas autorizadas**, estableciendo relaciones de confianza y seguridad en las comunicaciones.

* **Ejemplo**: cifrar un email para que solo el destinatario pueda leerlo.
* **Técnicas**: contraseñas seguras, cifrado, control de accesos, autenticación.

#### Disponibilidad

Asegura que la información y los sistemas estén **accesibles cuando se necesitan**, evitando interrupciones.

* **Ejemplo**: la web de un banco debe estar operativa 24/7 sin caídas.
* **Técnicas**: copias de seguridad, redundancia, balanceo de carga, protección frente a ataques DoS/DDoS.

#### Integridad

Garantiza que la información **no sea alterada de manera indebida**, ya sea por accidente o por un ataque.

* **Ejemplo**: verificar mediante un hash que la ISO de un sistema operativo descargada no ha sido manipulada.
* **Técnicas**: sumas de verificación, firmas digitales, control de versiones.

***

### Elementos vulnerables

#### Hardware

* Servidores, PC, routers, switches, firewalls.
* Discos duros, SSD, NAS.
* Dispositivos móviles: portátiles, smartphones.
* **Riesgos**: robos, fallos eléctricos, incendios, humedad, manipulación física.

#### Software

* Sistemas operativos.
* Aplicaciones de usuario.
* Software de red: protocolos, servicios de correo, bases de datos, servidores web.
* **Riesgos**: errores de programación, falta de actualizaciones, malware, configuraciones inseguras.

#### Datos

* Archivos confidenciales.
* Bases de datos.
* Información en tránsito por la red: mensajes, contraseñas, emails.
* **Riesgos**: pérdidas, corrupción, robo, alteraciones, interceptación en la red.

#### Red

* Conexiones a Internet.
* Redes internas (LAN, WLAN).
* Dispositivos de comunicación: antenas, puntos de acceso Wi-Fi.
* **Riesgo principal**: acceso no autorizado.

#### Usuarios

* Administradores del sistema.
* Empleados con acceso a información sensible.
* Proveedores y terceros con permisos de acceso.
* **Riesgos**: ingeniería social, errores humanos, contraseñas débiles, uso indebido de privilegios.

> **Resumen — medidas según el tipo de elemento**
>
> * **Medidas preventivas (seguridad activa)**: evitan que se produzca el daño en el sistema (actualizaciones automáticas/parches, protección de S.O. y aplicaciones).
> * **Medidas correctivas (seguridad pasiva)**: minimizan los daños una vez producido el incidente.

***

### Vulnerabilidades

Aunque suelen asociarse solo al software, una vulnerabilidad puede tener consecuencias graves:

* Un intruso puede conseguir **permisos de administración** en el sistema.
* Un virus puede tomar el **control de los equipos** de la empresa.
* Un atacante puede **borrar, alterar o cifrar** datos de la empresa.

Todas las vulnerabilidades detectadas se publican en informes **CVE** (_Common Vulnerability Exposure_) — [cve.org](https://www.cve.org/).

#### Otras vulnerabilidades destacadas

* **A nivel de microprocesador**: la CPU se ve obligada a leer datos a los que el usuario no debería tener acceso (ejecución especulativa).
* **Inyecciones SQL** (_SQL injection_): afectan a aplicaciones basadas en bases de datos.
* **HTML/JS injects**: otro tipo de ataque común en aplicaciones web.

***

### Amenazas y medidas de seguridad física (seguridad pasiva)

#### Amenazas físicas

* Cortes de suministro, robos, incendios, desastres atmosféricos.
* Condiciones adversas: temperaturas extremas, humedad excesiva, inundaciones, terremotos.

**Consecuencias**: falta de disponibilidad de servicios, pérdida de información, pérdida o mal funcionamiento del hardware.

#### Barreras y protecciones físicas

* Servidores cerrados con llave y acceso restringido.
* Controles de acceso con tarjeta, guardia de seguridad o biometría.
* Puertas con apertura programada.
* Protección eléctrica y protección antiincendios adecuada.

> **Nota**: no se deben ubicar los centros de datos en plantas bajas o sótanos, por el riesgo de inundación.

#### Medidas en CPD y grandes organizaciones

* **SAI** (Sistema de Alimentación Ininterrumpida) y generadores eléctricos.
* Fuentes de alimentación redundantes.
* Personal de vigilancia y cámaras de seguridad.
* Centro de respaldo en ubicación diferente, con información sincronizada.
* **Alta disponibilidad y monitorización**: objetivo típico de 99,99 %, servicio 24x7x365.
* Redundancia y diversificación: almacenamiento externo de datos, tomas eléctricas independientes, telecomunicaciones con balanceo de carga.

#### La sala fría

1. Concentra los grandes servidores, con sistema propio de refrigeración (**21–23 ºC** y humedad relativa entre **40 % y 60 %**).
2. Medidas estrictas de control de acceso físico.
3. Sistemas de extinción de incendios.

***

### Amenazas y medidas de seguridad lógica (seguridad pasiva)

#### Amenazas lógicas

Virus, ataques DoS, phishing y troyanos son ejemplos de amenazas lógicas que pueden descargarse o propagarse por la red, afectando no solo a un dispositivo sino a toda la infraestructura.

La instalación de programas no testeados puede provocar **desbordamientos de búfer** (_buffer overflow_) y condiciones de carrera, permitiendo a un atacante obtener privilegios y leer o escribir ficheros protegidos.

#### Medidas lógicas principales

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

***

### Seguridad de la red

Para defender una red primero hay que entender de qué está compuesta, cómo se comunican sus elementos y cuál es su arquitectura. Las áreas clave son:

* **Segmentación de la red.**
* **Controles de acceso** adecuados.
* **Defensa en profundidad** (_Defense in Depth_, DiD).
* **Privilegios mínimos** desde el diseño.

#### Amenazas y medidas por capa OSI

| Capa OSI            | Amenaza                                                                                                                         | Medida                                                                                                                       |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **Física**          | _Sniffing_: interceptación de señales físicas (cables, Wi-Fi) para capturar datos sin cifrar (ej. Wireshark). Robo de cables.   | Cifrado robusto como **WPA3** en redes Wi-Fi. Fibra óptica en tubos sellados con gas inerte a presión (uso militar).         |
| **Enlace de datos** | Suplantación de direcciones **MAC** para evitar el control de acceso (ej. _ARP spoofing_ para redirigir tráfico).               | Configuraciones **anti-spoofing** y detección de ataques ARP/DHCP. Uso de **VLAN** para segmentar la red.                    |
| **Red**             | **Man in the Middle**: interceptar o modificar el tráfico entre dos dispositivos (ej. redes Wi-Fi abiertas o mal configuradas). | Uso de **VPN** para cifrar la información en redes vulnerables o monitorizadas.                                              |
| **Transporte**      | Escaneo de puertos y ataques de denegación de servicio (ej. **Nmap**, usado tanto en hacking ético como malicioso).             | Auditar puertos y protocolos expuestos. Uso de **firewalls** (filtrado de paquetes, de red o de nueva generación) y **IDS**. |
| **Sesión**          | _**Hijacking**_: secuestro de sesión activa (ej. robo de cookies de sesión).                                                    | Tokens de sesión seguros o de un solo uso. Autenticación multifactor (**MFA**).                                              |
| **Presentación**    | **Phishing**: obtención de información mediante engaño e ingeniería social (ej. emails falsos).                                 | Formación de usuarios y herramientas de filtrado/anti-phishing.                                                              |
| **Aplicación**      | **Exploits**: aprovechamiento de vulnerabilidades en aplicaciones (ej. explotación de CVE en software desactualizado).          | Mantener el software actualizado y usar soluciones antimalware contra vulnerabilidades de día cero.                          |
| **Factor humano**   | Malas configuraciones, contraseñas débiles, caídas por error humano, falta de formación en ciberseguridad.                      | Formación continua y concienciación en ciberseguridad.                                                                       |

***

### Control de acceso

#### Autenticación vs. autorización

* **Autenticación**: verifica la **identidad** de un usuario en el sistema y determina si puede entrar o se queda fuera.
* **Autorización**: determina si el usuario **puede acceder** al recurso solicitado, consultando la base de datos de autorización (mantenida por el administrador), donde se especifica el tipo de acceso de cada usuario a cada recurso.

El flujo típico es: **usuario del sistema → autenticación → control de acceso → recursos del sistema** (ficheros, bases de datos, aplicaciones), todo ello validado contra la base de datos de autorización que gestiona el administrador.

#### Objetos, sujetos y derechos de acceso

* **Objeto**: recurso cuyo acceso está controlado (una entidad para almacenar y/o recibir información). Ejemplos: ficheros, directorios y aplicaciones.
* **Sujeto**: entidad capaz de acceder a un objeto, ligada al concepto de **proceso**. Habitualmente se habla de "usuarios" como sujetos, aunque en realidad el acceso se realiza a través de un proceso que representa a ese usuario o aplicación.

Un sistema de control de acceso básico define **tres clases de sujetos**:

| Clase           | Descripción                                                                                                          |
| --------------- | -------------------------------------------------------------------------------------------------------------------- |
| **Propietario** | Creador del recurso, fichero o directorio.                                                                           |
| **Grupo**       | Conjunto de usuarios con privilegios de acceso a determinados recursos; un usuario puede pertenecer a varios grupos. |
| **Otros**       | Usuarios que han entrado en el sistema pero no son propietarios ni pertenecen al grupo del recurso.                  |

#### Derechos de acceso

* **Lectura**: visualizar información de un recurso (permite copiar o imprimir).
* **Escritura**: agregar, modificar o eliminar datos de un recurso.
* **Ejecución**: ejecutar programas específicos.
* **Borrado**: eliminar recursos como ficheros o registros.
* **Creación**: crear nuevos ficheros, registros o directorios.

***

### Listas de control de acceso (ACL)

Las **ACL** se utilizan en sistemas y redes para administrar los permisos de acceso a recursos (archivos, directorios, dispositivos de red, etc.), determinando quién puede acceder a qué y qué acciones puede realizar. Son una pieza fundamental de la seguridad de la información.

#### DAC — Control de acceso discrecional

El **propietario del recurso** tiene control total sobre quién accede y con qué permisos. Por ejemplo, un usuario puede definir quién puede leer, escribir o ejecutar un archivo de su propiedad.

**Ejemplo de matriz DAC:**

| Sujeto     | /home/albert         | /home/alicia         | /home/ricard         | /etc/passwd |
| ---------- | -------------------- | -------------------- | -------------------- | ----------- |
| **Albert** | Read, Write, Execute | —                    | —                    | Read        |
| **Alicia** | —                    | Read, Write, Execute | —                    | Read        |
| **Ricard** | Read, Write, Execute | Read, Write, Execute | Read, Write, Execute | Read, Write |

#### MAC — Control de acceso obligatorio

Los sistemas operativos implementan mecanismos de control mediante **ACL** incorporadas en cada objeto (directorios, ficheros, recursos de red). Cuando un usuario intenta acceder a un objeto, el **núcleo del sistema operativo** comprueba su ACL para determinar si tiene derecho de acceso.

* La ACL es una lista de entradas con: **usuario o grupo**, **operación** (lectura, escritura) y **permiso** (denegar, permitir).
* Los permisos se asignan según **políticas y etiquetas de seguridad**.
* Es habitual en sistemas de seguridad de alto nivel (gobierno, ejército), donde se requiere un control más estricto.

> **ACL vs. permisos tradicionales**: las ACL controlan los permisos de archivos y directorios con **mayor precisión** que `chmod`, `chown` o `chgrp`, especificando qué usuarios o procesos tienen acceso a qué objetos y qué operaciones tienen permitidas.

**Herramientas por sistema operativo:**

* **Linux**: gestión de ACL mediante `setfacl` y `getfacl`.
* **Windows**: las ACL son la base de los permisos **NTFS**.

***

### Análisis forense en sistemas informáticos

Consiste en realizar una búsqueda detallada para **reconstruir los acontecimientos** ocurridos hasta el momento de detección de un ataque, asegurando en todo momento la conservación intacta de la información del sistema comprometido.

El análisis se apoya en ficheros log, el estudio del sistema de ficheros (FS) del equipo afectado y la reconstrucción de la secuencia de eventos. Es una ciencia **sistemática**, basada en hechos, orientada a recabar pruebas que después serán analizadas.

> **Nota**: no existe un procedimiento único aplicable "a rajatabla". Las técnicas generales deben combinarse con la experiencia del analista para desarrollar métodos propios.

#### El análisis termina cuando el forense conoce

1. Cómo se produjo el compromiso.
2. Bajo qué circunstancias.
3. La identidad del/los posibles atacantes.
4. Su procedencia y origen.
5. Fechas clave.
6. Los objetivos del/los atacantes.
7. La reconstrucción de la línea temporal de los eventos.

> Los archivos guardan metadatos (autor, compañía, fecha...) que pueden llegar a identificar el equipo usado para crearlos. Una imagen o archivo audiovisual puede protegerse mediante **marcas de agua digitales**, que permiten determinar su origen incluso si ha sido modificado.

#### Políticas de seguridad de la organización

Antes de un incidente, una organización debe definir:

* Activos de la empresa y qué aspectos de seguridad deben considerarse.
* Responsabilidades y planes de contingencia.
* Medidas de control y auditorías periódicas de los sistemas.
* Estimación preliminar de riesgos a partir de los puntos débiles de la infraestructura.
* El coste de una pérdida de información frente al coste de protegerla.

#### Información a recopilar del equipo afectado

* Sistema operativo e inventario del software instalado.
* Hardware y periféricos conectados.
* Presencia de firewall, si está en una DMZ o conectado a Internet.
* Configuración del equipo y parches/actualizaciones instaladas.
* Si el almacenamiento está cifrado.
* Si existe un IDS.
* El resto de equipos de la red.

#### Las cinco fases del análisis forense informático

1. **Identificación** — localizar los sistemas y dispositivos que pueden contener evidencia digital relevante (PC, servidores, móviles, medios de almacenamiento), para acotar el alcance del análisis.
2. **Adquisición** — crear imágenes forenses (copia bit a bit) de los dispositivos originales, capturando también ficheros borrados y datos ocultos, sin alterar la evidencia.
3. **Preservación** — almacenar los datos de forma segura sin modificarlos, usando cifrado y medios seguros. Mantener la **cadena de custodia** es esencial para que la evidencia sea válida ante un tribunal.
4. **Análisis** — examinar los datos preservados con herramientas y técnicas específicas: recuperación de archivos borrados, revisión de registros, búsqueda de malware y patrones relevantes.
5. **Documentación y presentación** — registrar cada paso del proceso (capturas, informes, notas de herramientas y hallazgos) y comunicar los resultados de forma clara a las partes interesadas (abogados, jueces, auditores), incluyendo la preparación de testimonios si es necesario.

***

### Enlaces de referencia

* [Access Control Lists — Arch Wiki (Español)](https://wiki.archlinux.org/index.php/Access_Control_Lists_\(Espa%C3%B1ol\))
* [Crear directivas de seguridad con ACL extendidas — Microsoft Docs](https://docs.microsoft.com/es-es/windows-server/virtualization/hyper-v-virtual-switch/create-security-policies-with-extended-port-access-control-lists)
* [Pautas de seguridad informática — apuntes SAD](https://oscarmaestre.github.io/apuntes_sad/tema_pautas_seguridad_informatica/tema_pautas_seguridad_informatica.html)

### Próximos pasos

* Ampliar con ejemplos prácticos de configuración de ACL en Linux (`setfacl`/`getfacl`) y en Windows (permisos NTFS).
* Relacionar esta unidad con el proyecto de [HoneyNet del TFG](https://claude.ai/chat/honeynet-tfg.md), aplicando conceptos de segmentación, IDS (Wazuh) y defensa en profundidad.
* Documentar un caso práctico de análisis forense siguiendo las cinco fases descritas.
