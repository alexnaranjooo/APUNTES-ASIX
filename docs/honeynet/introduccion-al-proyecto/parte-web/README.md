# 📄 Parte Web

## Introducción

Además de la infraestructura, el proyecto incluye una **parte web**: la página de una empresa de ciberseguridad que ofrece **honeypots como servicio**, es decir, desplegar señuelos para estudiar cómo atacan los hackers y ayudar a las empresas a defenderse.

***

## Referencia y diseño

* **Web de referencia**: `tworockets.consulting`.
* **Mockup de baja fidelidad**: ya realizado en **Figma**.

## Entregable: actividad M14UF1A2

La actividad **M14UF1A2** incluye:

* [x] **Mapa del sitio**.
* [x] **Mockups / wireframes** con notas.
* [x] **Pruebas por perfil de usuario**.
* [x] **Datos por pantalla**.
* [x] **Webgrafía**.

El documento completo está en [Diseño y plan de producción](diseno-y-plan-de-produccion.md).

## Alojamiento

La web se aloja en un servidor **nginx** dentro del contenedor `ct-web`, en la **VLAN 60**. El detalle de la arquitectura, el direccionamiento y el reenvío de puertos está en el informe [A3 — Arquitectura del software](../../a3-arquitectura-del-software/).

> nginx es un servidor web ligero y muy usado. Documentación oficial: [nginx.org/en/docs](https://nginx.org/en/docs/).
