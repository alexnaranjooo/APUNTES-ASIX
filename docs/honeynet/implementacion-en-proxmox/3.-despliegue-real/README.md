---
description: >-
  Tercera parte: la red definitiva del proyecto con pfSense, DNS, DHCP, la web,
  la base de datos y las copias.
---

# 3. Despliegue real

La red definitiva, la que se entrega: pfSense con sus VLANs, firewall, DNS y DHCP, y los servidores `ct-web`, `ct-db` y `ct-backup`.

* [Red real del proyecto](red-real-del-proyecto.md): visión general, diagrama y estado.
* [Red con pfSense](red-con-pfsense/): interfaces y reglas, DHCP y DNS, y reenvío de puertos.
* [Contenedores](contenedores/): ct-db y ct-web, base de datos y copias de seguridad.
* [Prueba de punta a punta y problemas](prueba-de-punta-a-punta-y-problemas.md): la web completa desde la VM cliente y lo que falló.
