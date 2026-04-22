---
marp: true
theme: profesional
paginate: true
header: 'Protocolo DHCP'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Protocolo DHCP

## Servidor KEA DHCP

José Domingo Muñoz
Abril 2026

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Protocolo DHCP

---

## DHCP

El protocolo de **configuración dinámica de host DHCP** (Dynamic Host Configuration Protocol) es un estándar TCP/IP que simplifica la administración de la configuración IP haciéndola **automática**. Un servidor gestiona la concesión de direcciones IP de un determinado segmento de red y mantiene una lista actualizada de la  correspondencia entre estas direcciones IP ylas direcciones MAC de los equipos que las han solicitado. En el protocolo DHCP, el servidor utiliza el puerto **67/udp** y el cliente el **68/udp**.

---

## ¿Cómo funciona el protocolo?

* **DHCP discover**: El cliente envía una petición a la dirección de **broadcast 255.255.255.255** **solicitando** una configuración de red única dentro de la red en la que se encuentra.
* **DHCP offer**: El servidor DHCP de la red responde la petición anterior **enviándole** al cliente (identificado por su dirección MAC) los parámetros de la red.

En principio podría terminar aquí, pero hay dos pasos más para evitar conflictos en el caso de que hubiera más de un servidor DHCP en la red:

* **DHCP request**: El cliente **confirma** al servidor DHCP (identificado por su dirección IP) que va a utilizar los parámetros que éste le ha enviado.
* **DHCP acknowledgement**: El servidor **confirma** los parámetros enviados y el **tiempo de validez de la concesión** de dirección IP (*DHCP lease time*).

---

## Estados del protocolo DHCP

![dhcp](img/dhcp.png)
