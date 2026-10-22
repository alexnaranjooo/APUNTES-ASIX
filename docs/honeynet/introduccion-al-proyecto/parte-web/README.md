# 📄 Parte Web

## Introducción

Además de la infraestructura, el proyecto incluye una **parte web**: la página de una empresa de ciberseguridad que ofrece **honeypots como servicio**, es decir, desplegar señuelos para estudiar cómo atacan los hackers y ayudar a las empresas a defenderse.

***

## Referencia y diseño

* **Web de referencia**: `tworockets.consulting`.
* **Mockup de baja fidelidad**: ya realizado en **Figma**.

## Documentación del diseño

El diseño de la web incluye:

* **Mapa del sitio**.
* **Mockups / wireframes** con notas.
* **Pruebas por perfil de usuario**.
* **Datos por pantalla**.
* **Webgrafía**.

Todo ello está recogido en [Diseño y plan de producción](diseno-y-plan-de-produccion.md).

## Alojamiento

La web se aloja en un servidor **nginx** dentro del contenedor `ct-web`, en la **VLAN 60**. El detalle de la arquitectura, el direccionamiento y el reenvío de puertos está en [Arquitectura del software](../arquitectura-del-software/).

> nginx es un servidor web ligero y muy usado. Documentación oficial: [nginx.org/en/docs](https://nginx.org/en/docs/).
