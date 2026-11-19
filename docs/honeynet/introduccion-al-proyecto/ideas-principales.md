---
description: >-
  En este apartado hablaremos sobre las ideas principales del proyecto, hablaré
  sobre cual es el objetivo real del proyecto, y como quiero llegar a
  conseguirlo.
---

# 💡 Ideas Principales

## <mark style="color:$warning;">**Ideas**</mark>

{% stepper %}
{% step %}
### <mark style="color:$warning;">**Concepto central**</mark>

* Red de empresa simulada + señuelos (honeypots) que atraen ataques reales de internet
* No es solo tener honeypots sino cerrar el círculo completo:

```
detectar → analizar → responder → visualizar
```
{% endstep %}

{% step %}
### <mark style="color:$warning;">**Infraestructura base**</mark>

* **Proxmox** como servidor de virtualización, en **dos PCs** del centro (dos nodos independientes, sin clúster)
* **pfSense + Suricata** como cerebro de red: firewall, router e IDS a la vez, y también DNS y DHCP, en una **máquina virtual** del PC 1 con una sola tarjeta de red
* **Switch HP 1810-24** que conecta los dos PCs y transporta las VLANs
* Segmentación en VLANs
* Acceso remoto al laboratorio con NetBird (VPN)
{% endstep %}

{% step %}
### <mark style="color:$warning;">**Los señuelos (HoneyPots)**</mark>

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
### <mark style="color:$warning;">**Empresa simulada**</mark>

* DNS y DHCP (en pfSense), MariaDB y Backup → dan credibilidad de infraestructura real; la base de datos y las copias van cada una en su contenedor
* VM de usuarios → contexto de empleados
* Web de la empresa (honeypots como servicio) alojada en nginx, con PHP y MariaDB
{% endstep %}

{% step %}
### <mark style="color:$warning;">**Inteligencia y respuesta**</mark>

* **Wazuh** completo centraliza todos los logs (SIEM)
* **Grafana** los visualiza y envía las alertas por Telegram
* El administrador de la web ve los logs y los gráficos en su panel, con login
{% endstep %}

{% step %}
### <mark style="color:$warning;">**Riesgos a vigilar**</mark>

* Alcance grande para el tiempo disponible, con un equipo de solo dos personas
* Hardware limitado y prestado por el centro (8 GB de RAM por PC, sin posibilidad de ampliar)
{% endstep %}
{% endstepper %}

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>
