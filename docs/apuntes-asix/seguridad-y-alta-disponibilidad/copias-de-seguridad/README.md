---
description: >-
  Preguntas y respuestas sobre copias de seguridad: clasificación de la
  información, tipos de copia, almacenamiento, estrategia 3-2-1, cifrado, nube,
  SAN y NAS.
---

# Copias de seguridad

## 1. ¿Qué son las copias de seguridad? ¿Por qué debemos hacer copias?

Una **copia de seguridad** (_backup_) es una copia de los datos guardada en otro soporte o lugar, para poder **recuperarlos** si los originales se pierden o se dañan.

Se hacen porque los datos pueden perderse de muchas formas:

| Causa             | Ejemplo                                           |
| ----------------- | ------------------------------------------------- |
| Error humano      | Borrar o sobrescribir un archivo por equivocación |
| Fallo de hardware | Un disco que se estropea                          |
| Ataque            | Un _ransomware_ que cifra los archivos            |
| Fallo de software | Una actualización que corrompe una base de datos  |
| Desastre          | Incendio, inundación o robo del equipo            |

Además de evitar pérdidas, permiten **continuar la actividad** tras un incidente y ayudan a **cumplir la ley**: el RGPD (artículo 32) exige poder restaurar la disponibilidad de los datos personales tras un incidente.

## 2. ¿Qué criterios de clasificación de la información determinan qué datos salvaguardar?

No todo merece la misma protección. Se clasifica la información según varios criterios:

| Criterio                 | Pregunta que responde              | Ejemplos                                                            |
| ------------------------ | ---------------------------------- | ------------------------------------------------------------------- |
| **Criticidad**           | ¿Qué pasa si se pierde?            | Crítica (base de datos de clientes), importante, útil, prescindible |
| **Confidencialidad**     | ¿Quién puede verla?                | Pública, interna, confidencial, secreta                             |
| **Frecuencia de cambio** | ¿Con qué frecuencia cambia?        | Una base de datos activa frente a un archivo histórico              |
| **Obligación legal**     | ¿Hay que conservarla por ley?      | Facturas, datos personales                                          |
| **Coste de recrearla**   | ¿Cuánto cuesta volver a generarla? | Un documento escrito a mano frente a un programa instalable         |

Con eso se decide **qué copiar** (datos de usuarios, bases de datos, configuraciones, correo, código), **qué no** (archivos temporales, el sistema operativo si se puede reinstalar) y con **qué frecuencia**. Dos valores guían esa decisión: el **RPO** (cuántos datos se pueden perder, medido en tiempo) y el **RTO** (cuánto se puede tardar en recuperar el servicio).

## 3. ¿De qué depende el tiempo de duración de una copia completa?

El tiempo es, aproximadamente, el **volumen de datos dividido entre la velocidad del eslabón más lento** de la cadena. Depende de:

* **La cantidad de datos** a copiar.
* **La velocidad de lectura** del disco de origen y la de **escritura** del destino.
* **La red**, si la copia es remota (el ancho de banda).
* **La compresión y el cifrado**, que gastan procesador.
* **El número de archivos**: muchos archivos pequeños tardan más que uno grande del mismo tamaño.
* **La carga del equipo** durante la copia y la **herramienta** usada.
* **La verificación** posterior, si se hace.

## 4. ¿Qué tipos de discos pueden usarse? ¿Influyen en el tiempo? ¿Cuándo se hacen los respaldos?

| Soporte               | Velocidad aproximada              | Uso típico                               |
| --------------------- | --------------------------------- | ---------------------------------------- |
| **Disco duro (HDD)**  | Unos 100 a 200 MB/s               | Copias de gran volumen y bajo coste      |
| **SSD SATA**          | Unos 500 MB/s                     | Copias rápidas                           |
| **SSD NVMe**          | De 2 a varios GB/s                | Entornos donde el tiempo importa         |
| **Cinta (LTO)**       | Cientos de MB/s                   | Archivo a largo plazo, muy barato por GB |
| **Disco externo USB** | Depende del disco y del puerto    | Copias personales o de pequeña empresa   |
| **NAS o SAN**         | Depende de la red y de los discos | Copias centralizadas                     |

**Sí influyen**: el tipo de disco, junto con la red, marca la velocidad del respaldo y de la restauración. Son cifras orientativas.

**¿Cuándo se hacen?** Lo más habitual es **fuera del horario laboral**, por la noche o en fin de semana, porque hay menos carga, menos usuarios y menos cambios mientras se copia. Las **restauraciones** se hacen cuando hace falta, tras un incidente; además se programan **pruebas de restauración periódicas**, en horas de poca actividad, para comprobar que las copias sirven.

## 5. ¿Qué es una copia incremental y qué diferencias tiene con la diferencial?

* **Completa**: copia todo.
* **Incremental**: copia solo lo que ha cambiado **desde la última copia, sea del tipo que sea**.
* **Diferencial**: copia lo que ha cambiado **desde la última copia completa**.

|                        | Completa  | Incremental                                            | Diferencial                             |
| ---------------------- | --------- | ------------------------------------------------------ | --------------------------------------- |
| Tamaño de cada copia   | El mayor  | El menor                                               | Crece cada día                          |
| Tiempo de copia        | El mayor  | El menor                                               | Intermedio                              |
| Restaurar necesita     | Solo ella | La completa y **todas** las incrementales, en orden    | La completa y **la última** diferencial |
| Tiempo de restauración | El menor  | El mayor                                               | Intermedio                              |
| Riesgo                 | Ninguno   | Si falla una copia de la cadena, se rompe lo posterior | Menor que con incrementales             |

**Ejemplo con una semana**: completa el lunes. Con incrementales, el martes guarda lo del martes, el miércoles lo del miércoles, y así. Con diferenciales, el martes guarda lo del martes, el miércoles lo del martes y el miércoles juntos, y así.

## 6. ¿Dónde podemos almacenar nuestra copia de seguridad?

| Ubicación                               | Ventaja                                  | Inconveniente                      |
| --------------------------------------- | ---------------------------------------- | ---------------------------------- |
| Otro disco o partición del mismo equipo | Muy rápido y barato                      | Si falla el equipo, se pierde todo |
| Disco externo USB                       | Portátil, fácil de guardar en otro sitio | Hay que acordarse de conectarlo    |
| NAS o SAN                               | Centralizado y automático                | Sigue en la misma sede             |
| Otro servidor, por red o SSH            | Protege contra el fallo del equipo       | Depende de la red                  |
| Cinta                                   | Muy duradera y barata por GB             | Lenta de restaurar                 |
| Nube                                    | Fuera de la sede, escalable              | Coste, privacidad y ancho de banda |

Lo ideal es combinar varias, con una **fuera de la sede** (_offsite_).

## 7. ¿En qué consiste la estrategia 3-2-1?

Es una regla básica de las copias:

* **3** copias de los datos: el original y dos copias.
* **2** tipos de soporte distintos (por ejemplo, disco y nube).
* **1** copia fuera de la sede.

Así, un solo fallo (un disco, un incendio, un ataque) no destruye todas las copias. Hay una versión ampliada, la **3-2-1-1-0**: una copia _offline_ o inmutable, y **cero errores** al verificar las copias.

En HoneyNet se aplica así: el original en el nodo de Proxmox, una copia programada en el propio nodo, y otra copia fuera de los PCs (ver [Copias de seguridad y snapshots](../../../honeynet/implementacion-en-proxmox/copias-de-seguridad-y-snapshots.md)).

## 8. ¿Qué herramientas de cifrado protegen las copias de seguridad?

| Herramienta                       | Qué cifra                                    | Ejemplo                                                            |
| --------------------------------- | -------------------------------------------- | ------------------------------------------------------------------ |
| **GPG**                           | Archivos, con contraseña o con clave pública | `gpg -c copia.tar.gz`                                              |
| **OpenSSL**                       | Archivos                                     | `openssl enc -aes-256-cbc -pbkdf2 -in copia.tar.gz -out copia.enc` |
| **7-Zip**                         | Archivos comprimidos con AES-256             | Opción de contraseña en el archivo `.7z`                           |
| **LUKS (dm-crypt)**               | Un disco o partición completa en Linux       | El disco externo de las copias                                     |
| **VeraCrypt**                     | Volúmenes cifrados, multiplataforma          | Un contenedor cifrado en un USB                                    |
| **BitLocker**                     | Discos completos en Windows                  | Una unidad de copias                                               |
| **restic, BorgBackup, duplicity** | Programas de copia con cifrado integrado     | Copias cifradas y deduplicadas                                     |
| **rclone (crypt)**                | Los archivos antes de subirlos a la nube     | Copias en un servicio externo                                      |

La **clave** debe guardarse **aparte** de la copia: si se pierde, los datos no se pueden recuperar, y si está junto a la copia, el cifrado no sirve de nada. Durante el envío, SSH o TLS cifran el transporte.

## 9. ¿Qué debemos considerar al hacer copias de seguridad en la nube?

* **Cifrar antes de subir** (cifrado en el cliente), además del cifrado en tránsito y el del proveedor.
* **Privacidad y ley**: dónde están los datos (idealmente en la Unión Europea) y qué dice el contrato sobre el RGPD.
* **Ancho de banda**: lo que tarda en subirse la primera copia y, sobre todo, en **restaurarla**.
* **Coste**: almacenamiento, peticiones y, a menudo, la **descarga** de los datos.
* **Disponibilidad**: el acuerdo de servicio (SLA) y qué pasa si el proveedor falla o cierra.
* **Versionado y retención**: poder recuperar versiones antiguas, y copias inmutables contra el _ransomware_.
* **Acceso**: autenticación con doble factor y claves de API con los mínimos permisos.
* **Dependencia del proveedor**: lo difícil que sería cambiarse.
* **Pruebas**: comprobar de vez en cuando que se puede restaurar.

## 10. ¿Qué es una SAN?

Una **SAN** (_Storage Area Network_) es una **red dedicada de alta velocidad** que ofrece almacenamiento a los servidores a **nivel de bloque**: cada servidor ve un disco como si fuera suyo, aunque esté en un equipo de almacenamiento aparte.

* Se conecta con **Fibre Channel** o con **iSCSI** (sobre Ethernet).
* Se usa para virtualización y bases de datos, donde importan el rendimiento y la fiabilidad.
* Suele ser cara y compleja.

## 11. ¿Qué es un NAS? Ejemplos de pago y libres

Un **NAS** (_Network Attached Storage_) es un equipo de almacenamiento conectado a la red que comparte **archivos y carpetas** (a **nivel de archivo**) mediante protocolos como SMB, NFS o FTP. Es más sencillo y barato que una SAN.

|       | SAN                                        | NAS                         |
| ----- | ------------------------------------------ | --------------------------- |
| Nivel | Bloque (disco)                             | Archivo (carpetas)          |
| Red   | Dedicada (Fibre Channel o iSCSI)           | La red local normal         |
| Uso   | Servidores, bases de datos, virtualización | Compartir archivos y copias |

| Tipo        | Ejemplos                                                                                      |
| ----------- | --------------------------------------------------------------------------------------------- |
| **De pago** | Synology, QNAP, Western Digital My Cloud, Netgear ReadyNAS (equipos con software propietario) |
| **Libres**  | TrueNAS, OpenMediaVault, XigmaNAS, o un servidor Linux con Samba y NFS                        |

La práctica con `rsync` y `tar.gz` está en el apartado [rsync + tar.gz](rsync-+-tar.gz.md).
