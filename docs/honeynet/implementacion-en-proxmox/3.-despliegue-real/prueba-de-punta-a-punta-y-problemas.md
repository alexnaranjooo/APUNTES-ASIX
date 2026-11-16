---
description: >-
  La prueba completa desde la VM cliente hasta la base de datos y los problemas
  encontrados durante el montaje.
---

# Prueba de punta a punta y problemas

## Prueba de punta a punta

Desde la VM cliente (VLAN 40), el navegador abre `www.honeynet.lan` escribiendo `http://` delante: el DHCP da la IP, el DNS de pfSense resuelve el nombre, el firewall deja pasar el puerto 80, NGINX ejecuta PHP y PHP llama a un procedimiento de la base de datos, que devuelve los tres planes con sus características.

<figure><img src="../../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

## Problemas encontrados y soluciones

| Problema                                            | Causa                                                          | Solución                                                                                     |
| --------------------------------------------------- | -------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| El DNS Resolver no se podía activar en modo reenvío | Faltaba un servidor DNS en la configuración general de pfSense | Escribir el DNS del centro en _System → General Setup_                                       |
| `www.honeynet.lan` no cargaba en el navegador       | Firefox lo buscaba en Google                                   | Escribirlo con `http://` delante                                                             |
| Error 500 en la web                                 | PHP no podía conectar a la base de datos: _Access denied_      | Cambiar la contraseña del usuario de la web y desactivar la resolución de nombres en MariaDB |
| La web no se veía desde otro PC                     | La regla de NAT estaba deshabilitada                           | Activarla                                                                                    |
