---
description: Python y DNS.
---

# Práctica: Seguridad en el DNS

## Objetivo de la Práctica

El objetivo fundamental de este laboratorio es implementar, analizar y auditar un **canal encubierto (**_**covert channel**_**) mediante Tunelización DNS (DNS Tunneling)**. Esta técnica avanzada de exfiltración aprovecha que el tráfico DNS convencional (Puerto 53 UDP) suele estar permitido en los cortafuegos perimetrales para transportar información confidencial camuflada dentro de solicitudes de resolución de nombres legítimas, evadiendo los sistemas de seguridad tradicionales.

***

## Análisis y Funcionamiento de los Scripts

El diseño del laboratorio se basa en una arquitectura Cliente-Servidor utilizando scripts personalizados en Python mediante la biblioteca de abstracción de red `dnspython`.

### 1. El Cliente DNS (`dns_client_1.py`)

Actúa como el agente de exfiltración dentro de la máquina comprometida (víctima).

* Toma el string confidencial previamente convertido a formato hexadecimal (`6461746f73206f63756c746f73`).
* Lo concatena por la izquierda para estructurar un subdominio falso apuntando a una zona bajo control (`.secreto.com`).
* Utiliza la directiva `dns.message.make_query()` para empaquetar de forma estricta una **petición de registro A** y la despacha mediante protocolo no orientado a conexión UDP al puerto 53.

### 2. El Servidor DNS (`dns_server_1.py`)

Actúa como la autoridad de nombres falsa controlada por el atacante en el exterior de la infraestructura.

* Abre un socket UDP puro en todas las interfaces (`0.0.0.0`) vinculándose al puerto crítico de DNS (`53`).
* Al capturar el paquete crudo entrante, lo de serializa con `dns.message.from_wire()`.
* Realiza un filtrado quirúrgico aislando el primer bloque del subdominio mediante `split(".")`.
* Revierte la ingeniería de la cadena aplicando `bytes.fromhex().decode()` para **revelar y exponer el secreto robado en texto plano**.
* Genera una respuesta formal clonando el ID de transacción e inyectando un registro A con la IP falsa `4.3.2.1` y código `NOERROR` para simular ante la red que la resolución ha sido legítima.

***

## Metodología de Ejecución&#x20;

### Paso a Paso

#### Paso 1: Configuración de Dependencias de Python

Para preparar las capacidades de análisis de red del laboratorio en el entorno **Debian**, se instaló el núcleo del intérprete y sus librerías asociadas:

```bash
sudo apt update
sudo apt install python3 python3-pip python3-dnspython -y
```

#### Paso 2: Resolución de Bloqueos en el Puerto 53

Durante el despliegue del servidor DNS en Debian (`named.service`), el arranque fallaba con el error `unable to listen on any configured interfaces` debido a una colisión de sockets en el puerto 53. El procedimiento para liberar la interfaz fue:

1.  **Rastreo e Identificación del proceso intruso:**

    ```bash
    sudo ss -tulnp | grep :53
    ```

    Se detectó una instancia huérfana de Python reteniendo el socket bajo el identificador **PID 5478**.
2.  **Liberación del socket:**

    ```bash
    sudo kill -9 5478
    ```

    Usa el código con precaución.
3. **Verificación Sintáctica:** Se validó la integridad de las zonas locales ejecutando `sudo named-checkconf -z`, asegurando que la máquina base poseía las directivas estructurales funcionales para convivir en el entorno de pruebas.\
   **Nota de Privilegios:** Debido a que el script del Servidor DNS debe enlazar y abrir el puerto de red 53 (puerto privilegiado e inferior al 1024), es estrictamente obligatorio ejecutarlo con privilegios de superusuario (`sudo`).

#### Paso 3: Inicialización del Canal Oculto

Se procedió a simular el escenario interactivo abriendo dos terminales en paralelo (simulando dos VMs con direccionamiento de red IP independiente):En la **Terminal del Servidor** (Atacante):

```bash
sudo python3 dns_server_1_peticion.py
```

La consola entra en modo de escucha pasiva indicando: _“Esperando 1 peticion DNS...”_.En la **Terminal del Cliente** (Víctima):

```bash
python3 dns_client_1_peticion.py
```

Al procesarse el envío, la consola del servidor interceptó y decodificó de manera instantánea el payload, exponiendo con éxito el secreto en claro:

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

***

## Auditoría de Tráfico con Wireshark

Para evaluar la efectividad de la técnica de evasión, se monitorizó la interfaz de red activa en tiempo real con el analizador **Wireshark** durante el intercambio de tramas. Miramos el adaptador de Lookback.

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

***
