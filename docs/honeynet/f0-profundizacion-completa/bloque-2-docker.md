# Bloque 2 — Docker

### La analogía que lo explica mejor

Montar un bar tiene dos caminos. **Opción A**: construir la cocina desde cero, instalar los fogones, conectar el gas, comprar los utensilios uno a uno. **Opción B**: pedir un módulo de cocina prefabricado, enchufarlo, y ya funciona.

**Docker es la opción B para software.** Cowrie, Wazuh, Grafana... cada uno viene empaquetado con todo lo que necesita (librerías, dependencias, configuración base). Se arranca con un comando y ya está corriendo, sin pelearse con "me falta tal librería" o "esta versión no es compatible".

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

### Conceptos clave

* **Imagen**: la plantilla de solo lectura, el plano del contenedor. Docker Hub tiene miles de imágenes públicas ya hechas (`cowrie/cowrie`, `grafana/grafana`, etc.) — no se construyen desde cero, se descargan.
* **Contenedor**: una imagen puesta en marcha. Pueden coexistir varios contenedores de la misma imagen (por ejemplo, dos instancias de un servicio web).
*   **Volumen**: una carpeta que vive fuera del contenedor, en el disco real de la VM. Si el contenedor se reinicia o se borra, lo que esté en un volumen sobrevive.

    > **Por qué importa aquí**: los logs de Cowrie y las muestras de malware de Dionaea van en volúmenes.
* **Docker Compose**: un archivo que describe varios contenedores a la vez — qué imagen usa cada uno, qué puertos expone, qué volúmenes monta y cómo se conectan entre sí. Con un solo comando (`docker-compose up -d`) se levantan, por ejemplo, los tres honeypots a la vez.
* **Red Docker**: los contenedores definidos en el mismo `docker-compose` pueden hablarse entre sí usando su nombre, como un mini-DNS interno de Docker. Así, Cowrie y MySQL se comunican sin necesidad de buscar IPs.

### Por qué importa para este proyecto

Se desplegarán **Cowrie + Dionaea + DVWA** con un solo `docker-compose`, y después **Wazuh + Grafana** con otro.&#x20;



<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>
