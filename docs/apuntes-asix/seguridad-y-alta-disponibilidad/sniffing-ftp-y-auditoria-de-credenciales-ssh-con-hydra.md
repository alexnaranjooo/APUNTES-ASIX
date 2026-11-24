# Sniffing FTP y Auditoría de Credenciales SSH con Hydra

## Introducción

Esta guía documenta un ejercicio práctico de pentesting realizado en clase por el alumno Alex. El objetivo del ejercicio es doble: por un lado, demostrar mediante **Wireshark** los riesgos de usar protocolos de transferencia de ficheros en texto claro como **FTP**, capturando credenciales y ficheros sensibles que circulan por la red sin cifrar; por otro lado, realizar una auditoría de seguridad de un servicio **SSH** mediante un ataque de fuerza bruta con **Hydra**, usando como diccionarios los propios ficheros de usuarios y contraseñas obtenidos durante la captura.

La máquina objetivo del ejercicio es `192.168.111.55`.

## 1. Captura de tráfico FTP con Wireshark

Se inicia una captura general de tráfico en Wireshark sobre la interfaz de red correspondiente, observando el conjunto de paquetes que circulan por la red de prácticas.

Para aislar únicamente el tráfico del protocolo FTP, se aplica el filtro de visualización `ftp` en la barra de filtros de Wireshark.

Con el filtro aplicado se observa cómo un cliente FTP se conecta al servidor y, en un momento dado, ejecuta un comando `STOR` para subir un fichero al servidor. Entre los ficheros transferidos se identifican `password.txt` y `usuarios.txt`, ambos viajando en texto claro dentro del propio tráfico FTP.

Además del envío de los ficheros, también se puede observar en la captura un inicio de sesión FTP completado con éxito, con el usuario y la contraseña visibles en claro en los paquetes correspondientes.

## 2. Extracción del contenido mediante "Follow TCP Stream"

Wireshark permite reconstruir el flujo completo de una conexión TCP mediante la opción **Follow TCP Stream**. Aplicando esta opción sobre la conexión FTP capturada, es posible leer el contenido íntegro de los ficheros transferidos tal y como viajaron por la red, sin necesidad de interceptar el fichero en el propio servidor.

De esta forma se recupera el contenido completo del fichero de contraseñas:

Y de igual manera se recupera el contenido completo del fichero de usuarios:

Con ambos ficheros reconstruidos, se dispone de dos listas (`usuarios.txt` y `password.txt`) que servirán como diccionarios para el siguiente paso del ejercicio: un ataque de fuerza bruta contra el servicio SSH de la máquina objetivo.

## 3. Fuerza bruta contra SSH con Hydra

Con los diccionarios de usuarios y contraseñas ya disponibles, se lanza un primer intento de ataque de fuerza bruta contra el servicio SSH de `192.168.111.55` utilizando **Hydra**:

```
sudo hydra -L usuarios.txt -P password.txt -t 4 192.168.111.55 ssh
```

Este primer intento falla, mostrando el siguiente error:

```
[ERROR] all children were disabled due too many connection errors
```

Este tipo de error suele indicar que el servidor SSH está limitando o rechazando conexiones simultáneas, por lo que antes de seguir insistiendo con Hydra se procede a comprobar manualmente el estado del servicio en el puerto 22, usando `nc` y `nmap`:

```
nc -zv 192.168.111.55 22
nmap -p 22 192.168.111.55
```

Ambas comprobaciones confirman que el puerto 22 está abierto y el servicio SSH responde con normalidad, por lo que el error anterior se debe al exceso de conexiones simultáneas lanzadas por Hydra y no a que el servicio esté caído o filtrado.

Con esta información, se reduce la concurrencia del ataque para evitar saturar el servicio, añadiendo también un tiempo de espera entre reintentos:

```
sudo hydra -L usuarios.txt -P password.txt -t 2 -W 3 192.168.111.55 ssh
```

Tras este ajuste, se repite el ataque con una concurrencia algo mayor (`-t 4`), obteniendo esta vez resultados:

```
sudo hydra -L usuarios.txt -P password.txt -t 4 192.168.111.55 ssh
```

Como resultado del proceso (ya sea directamente a través del ataque de Hydra o gracias a las credenciales ya observadas durante el sniffing FTP), se obtienen unas credenciales válidas para el servicio SSH:

* **Usuario:** `aladar`
* **Contraseña:** `mezga`

## 4. Verificación de acceso por SSH

Para comprobar que las credenciales obtenidas son válidas, se realiza una conexión SSH directa contra la máquina objetivo:

```
ssh aladar@192.168.111.55
```

La conexión se establece correctamente, confirmando que el usuario `aladar` con contraseña `mezga` tiene acceso al sistema.

Una vez dentro del sistema, se exploran brevemente el directorio de trabajo y el sistema de ficheros con los comandos:

```
pwd
ls
```

Navegando hasta el directorio `Documents/SEGiAD` se pueden observar ficheros dejados por otros compañeros de clase como prueba de haber completado el mismo ejercicio. Como prueba propia de la realización del ejercicio, se crea un fichero vacío con el nombre del alumno:

```
touch alexnaranjo2
```

## Conclusión

Este ejercicio ilustra de forma práctica dos problemas de seguridad habituales:

* **Uso de FTP en texto claro.** Al no cifrar ni las credenciales ni los datos transferidos, cualquier atacante con capacidad de capturar tráfico en la red (sniffing) puede obtener usuarios, contraseñas y el contenido íntegro de los ficheros transferidos, como se ha demostrado con Wireshark y la opción Follow TCP Stream. La recomendación es sustituir FTP por alternativas cifradas como **SFTP** o **FTPS**.
* **Servicio SSH sin protección frente a fuerza bruta.** El hecho de que un ataque con Hydra, con un diccionario relativamente pequeño, consiga encontrar credenciales válidas pone de manifiesto la necesidad de reforzar la configuración del servicio SSH mediante medidas como:
  * Uso de **fail2ban** u otras herramientas de bloqueo automático de IPs tras varios intentos fallidos.
  * Limitación del número de intentos de autenticación por conexión (`MaxAuthTries`) y de conexiones simultáneas.
  * Migración hacia **autenticación basada en claves públicas** en lugar de contraseñas, deshabilitando el acceso por contraseña siempre que sea posible.

En conjunto, el ejercicio demuestra cómo una mala práctica en un servicio (FTP en claro) puede facilitar el compromiso de otro servicio completamente distinto (SSH), reforzando la importancia de aplicar buenas prácticas de seguridad de forma consistente en todos los servicios expuestos en una red.
