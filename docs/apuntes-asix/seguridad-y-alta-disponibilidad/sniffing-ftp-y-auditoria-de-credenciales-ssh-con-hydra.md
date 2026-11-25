# Sniffing FTP y Auditoría de Credenciales SSH con Hydra

## 1. Captura de tráfico FTP con Wireshark

Se inicia una captura general de tráfico en Wireshark sobre la interfaz de red correspondiente, observando el conjunto de paquetes que circulan por la red de prácticas.

<figure><img src="../../.gitbook/assets/Captura de pantalla 2026-11-24 184812.png" alt=""><figcaption></figcaption></figure>



Para aislar únicamente el tráfico del protocolo FTP, se aplica el filtro de visualización `ftp` en la barra de filtros de Wireshark.

<figure><img src="../../.gitbook/assets/Captura de pantalla 2026-11-24 184732.png" alt=""><figcaption></figcaption></figure>



Con el filtro aplicado se observa cómo un cliente FTP se conecta al servidor y, en un momento dado, ejecuta un comando `STOR` para subir un fichero al servidor. Entre los ficheros transferidos se identifican `password.txt` y `usuarios.txt`, ambos viajando en texto claro dentro del propio tráfico FTP.

<figure><img src="../../.gitbook/assets/wireshark con el filtrado ftp (1).png" alt=""><figcaption></figcaption></figure>

## 2. Extracción del contenido mediante "Follow TCP Stream"

Wireshark permite reconstruir el flujo completo de una conexión TCP mediante la opción **Follow TCP Stream**. Aplicando esta opción sobre la conexión FTP capturada, es posible leer el contenido íntegro de los ficheros transferidos tal y como viajaron por la red, sin necesidad de interceptar el fichero en el propio servidor.

De esta forma se recupera el contenido completo del fichero de contraseñas:

<figure><img src="../../.gitbook/assets/paquete con las contraseñas (1).png" alt=""><figcaption></figcaption></figure>



Y de igual manera se recupera el contenido completo del fichero de usuarios:

<figure><img src="../../.gitbook/assets/paquete con los usuarios.png" alt=""><figcaption></figcaption></figure>



Con ambos ficheros reconstruidos, se dispone de dos listas (`usuarios.txt` y `password.txt`) que servirán como diccionarios para el siguiente paso del ejercicio.

## 3. Fuerza bruta contra SSH con Hydra

Con los diccionarios de usuarios y contraseñas ya disponibles, se lanza un primer intento de ataque de fuerza bruta contra el servicio SSH de `192.168.111.55` utilizando **Hydra**:

```
sudo hydra -L usuarios.txt -P password.txt -t 4 192.168.111.55 ssh
```

Este primer intento falla, mostrando el siguiente error:

<figure><img src="../../.gitbook/assets/Captura de pantalla 2026-11-24 185236.png" alt=""><figcaption></figcaption></figure>

Este tipo de error suele indicar que el servidor SSH está limitando o rechazando conexiones simultáneas, por lo que antes de seguir insistiendo con Hydra se procede a comprobar manualmente el estado del servicio en el puerto 22, usando `nc` y `nmap`:

```
nc -zv 192.168.111.55 22
nmap -p 22 192.168.111.55
```

<figure><img src="../../.gitbook/assets/Captura de pantalla 2026-11-24 185509.png" alt=""><figcaption></figcaption></figure>

\
Ambas comprobaciones confirman que el puerto 22 está abierto y el servicio SSH también.

Con esta información, se reduce la concurrencia del ataque para evitar saturar el servicio, añadiendo también un tiempo de espera entre reintentos:

Tras este ajuste, se repite el ataque con una concurrencia algo mayor (`-t 4`), obteniendo esta vez resultados:

```
sudo hydra -L usuarios.txt -P password.txt -t 4 192.168.111.55 ssh
```

<figure><img src="../../.gitbook/assets/Captura de pantalla 2026-11-24 192101.png" alt=""><figcaption></figcaption></figure>



Como resultado del proceso (ya sea directamente a través del ataque de Hydra o gracias a las credenciales ya observadas durante el sniffing FTP), se obtienen unas credenciales válidas para el servicio SSH:

* **Usuario:** `aladar`
* **Contraseña:** `mezga`

## 4. Verificación de acceso por SSH

Para comprobar que las credenciales obtenidas son válidas, se realiza una conexión SSH directa contra la máquina objetivo:

```
ssh aladar@192.168.111.55
```

La conexión se establece correctamente, confirmando que el usuario `aladar` con contraseña `mezga` tiene acceso al sistema.

<figure><img src="../../.gitbook/assets/Captura de pantalla 2026-11-24 192211.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Captura de pantalla 2026-11-24 192217.png" alt=""><figcaption></figcaption></figure>



Una vez dentro del sistema, se exploran brevemente el directorio de trabajo y el sistema de ficheros con los comandos:

```
pwd
ls
```

Como prueba propia de la realización del ejercicio, se crea un fichero vacío con el mi nombre.

```
touch alexnaranjo2
```

<figure><img src="../../.gitbook/assets/Captura de pantalla 2026-11-24 192308 (1).png" alt=""><figcaption></figcaption></figure>

