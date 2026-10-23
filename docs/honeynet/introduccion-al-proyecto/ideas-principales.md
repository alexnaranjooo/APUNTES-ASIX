---
description: >-
  En este apartado hablaremos sobre las ideas principales del proyecto, hablaré
  sobre cual es el objetivo real del proyecto, y como quiero llegar a
  conseguirlo.
---

# 💡 Ideas Principales

## Ideas

{% stepper %}
{% step %}
### **Concepto central**

* Red de empresa simulada + señuelos (honeypots) que atraen ataques reales de internet
* No es solo tener honeypots sino cerrar el círculo completo:

```
detectar → analizar → responder → visualizar
```
{% endstep %}

{% step %}
### **Infraestructura base**

* **Proxmox** como servidor de virtualización, en **dos PCs** del centro (dos nodos independientes, sin clúster)
* **pfSense + Suricata** como cerebro de red: firewall, router e IDS a la vez, en una **máquina virtual** del PC 1 con una sola tarjeta de red
* **Switch HP 1810-24** que conecta los dos PCs y transporta las VLANs
* Segmentación en VLANs
* Acceso remoto al laboratorio con NetBird (VPN)
{% endstep %}

{% step %}
### **Los señuelos (HoneyPots)**

* Cowrie → ataques SSH/Telnet (fuerza bruta, sesiones interactivas)
* Dionaea → malware
* DVWA → ataques web

<details>

<summary><em>¿Cómo funciona cada HoneyPot?</em></summary>

**Cowrie**: Escucha en los puertos 22 y 23. El atacante cree que se conecta a un servidor real. Graba **todo**: IP, usuario/contraseña probados, comandos ejecutados, archivos descargados.

**Dionaea**: Cubre puertos que Cowrie no toca: SMB (445), FTP (21), MySQL (3306). Su objetivo específico es **capturar el malware en sí**.

**DVWA:** Es una aplicación web insegura a SQL injection, ataques de fuerza bruta. Aquí no capturas "malware" sino técnicas de explotación web.

</details>
{% endstep %}

{% step %}
### **Empresa simulada**

* DNS, DHCP, MariaDB, Backup → dan credibilidad de infraestructura real, cada uno en su contenedor
* VM de usuarios → contexto de empleados
* Web de la empresa (honeypots como servicio) alojada en nginx, con PHP y MariaDB
{% endstep %}

{% step %}
### Inteligencia y respuesta

* **Wazuh** completo centraliza todos los logs (SIEM)
* **Grafana** los visualiza y envía las alertas por Telegram
* El administrador de la web ve los logs y los gráficos en su panel, con login
{% endstep %}

{% step %}
### Riesgos a vigilar

* Alcance grande para el tiempo disponible, con un equipo de solo dos personas
* Hardware limitado y prestado por el centro (8 GB de RAM y un SSD de 128 GB por PC, sin posibilidad de ampliar)
{% endstep %}
{% endstepper %}

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>
