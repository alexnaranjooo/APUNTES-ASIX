---
description: >-
  Cómo guardar las reglas de iptables para que sobrevivan a un reinicio y cómo
  poner IP fija a la tarjeta externa del contenedor.
---

# iptables: persistencia e IP fija

## Introducción

Continuación de [Reenvío de puertos con iptables](reenvio-de-puertos-con-iptables.md). Las reglas funcionan, pero viven en memoria: al reiniciar el contenedor se pierden. Además, la IP externa la reparte el DHCP del centro y puede cambiar. Aquí se resuelven las dos cosas.

***

## 1. Que las reglas sobrevivan a un reinicio

El paquete `iptables-persistent` guarda las reglas en un archivo y las recarga al arrancar:

```bash
apt install iptables-persistent -y
netfilter-persistent save
```

Al instalarlo pregunta si guardar las reglas actuales (IPv4 e IPv6): se responde que sí. El comando `netfilter-persistent save` se ejecuta siempre **después** de cualquier cambio en las reglas.

| Qué                       | Dónde se guarda                 | Quién lo recupera al arrancar |
| ------------------------- | ------------------------------- | ----------------------------- |
| Las reglas de `iptables`  | `/etc/iptables/rules.v4`        | `iptables-persistent`         |
| El reenvío (`ip_forward`) | `/etc/sysctl.d/99-forward.conf` | El propio sistema             |

Hacen falta las dos para que la web siga funcionando tras reiniciar.

**Comprobación:** se apaga el contenedor del todo (**Shutdown**), se vuelve a encender y se miran `sysctl net.ipv4.ip_forward` (debe dar `1`), `iptables -t nat -L -n`, `iptables -L FORWARD -n` y `ip -br a`. Después se abre la web.

{% hint style="info" %}
CAPTURA PENDIENTE: las reglas guardadas y el contenedor tras reiniciar.
{% endhint %}

Si no se cargan solas, se comprueba el servicio con `systemctl status netfilter-persistent` y se activa con `systemctl enable netfilter-persistent`.

## 2. IP fija en la tarjeta externa

**Concepto:** una IP fija evita que la dirección de la web cambie. Hay que elegir una que el DHCP del centro no pueda dar a otro equipo; lo ideal es pedir una reservada. Los datos de la red del centro son: máscara `/24`, puerta de enlace `192.168.111.1` y DNS `192.168.111.1`.

Con el contenedor apagado del todo, en **ct-prueba**:

| Pantalla              | Ajuste      | Valor               |
| --------------------- | ----------- | ------------------- |
| Network, `eth0`, Edit | IPv4        | Static              |
| Network, `eth0`, Edit | IPv4/CIDR   | `192.168.111.54/24` |
| Network, `eth0`, Edit | Gateway     | `192.168.111.1`     |
| DNS, Edit             | DNS servers | `192.168.111.1`     |

Después se pulsa **Start** y se comprueba con `ip -br a`, `ip route` y `apt update`.

Las reglas de `iptables` siguen valiendo porque usan los **nombres** de las tarjetas (`eth0` y `eth1`), no las IPs.

{% hint style="warning" %}
Una IP fija puede chocar con la que el DHCP dé a otro equipo, y los dos fallarían a ratos. Si hay dudas, se pide al centro una dirección reservada.
{% endhint %}

{% hint style="info" %}
CAPTURA PENDIENTE: la pantalla _Network_ con `eth0` en Static, y la web abierta con la IP fija.
{% endhint %}

***

## Problemas frecuentes

| Síntoma                                    | Qué mirar                                                                                         |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| `iptables` da «Permission denied»          | Un contenedor sin privilegios puede no dejar tocar el cortafuegos; en ese caso se hace en el nodo |
| El navegador no carga y las reglas existen | Comprobar `ip_forward` y que los contadores de `PREROUTING` suban                                 |
| Carga pero se corta                        | Falta la regla `MASQUERADE`                                                                       |
| Tras reiniciar no hay reglas               | Faltaba `netfilter-persistent save` o el servicio no está activo                                  |
| Sin ruta por defecto con IP fija           | No se hizo _Shutdown_ y _Start_ del todo                                                          |

## Relación con el proyecto

En la Fase 2 esto lo hará **pfSense**: una regla de reenvío de puertos desde la WAN hacia `ct-web` (`10.0.60.10`) y reglas de firewall que solo dejan pasar el 80 y el 443. Aquí se ha hecho a mano con `iptables` para entender el mecanismo.
