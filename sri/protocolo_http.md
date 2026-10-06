---
marp: true
title: Protocolo HTTP
theme: profesional
paginate: true
header: 'SRI · Unidad 2 — Protocolo HTTP'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Protocolo **HTTP**

## Mensajes, métodos, códigos de estado y cabeceras

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI · Servicios de Red e Internet</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# El protocolo HTTP

## Mensajes, cabeceras y estado

---

## ¿Qué es HTTP?

> **HyperText Transfer Protocol** es el protocolo de la **capa de aplicación** que permite la comunicación entre clientes, servidores y proxies en la web.

### Características fundamentales

- Basado en el esquema **petición / respuesta**: el cliente pide, el servidor responde
- Versiones en uso: **HTTP/1.1**, **HTTP/2** y **HTTP/3**; cambian la forma de transportar los mensajes, no su significado
- En **HTTP/1.1** los mensajes son **texto legible**; HTTP/2 y HTTP/3 son **binarios**
- Protocolo **sin estado**: el servidor no recuerda quién hizo cada petición

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>La falta de estado se compensa más tarde con <strong>cookies</strong> y <strong>sesiones</strong>.</div>
</div>

---

## El modelo TCP/IP: cuatro niveles

Cada nivel resuelve **un problema** y tiene **su propia dirección**. Cada uno se apoya en el de abajo.

<div style="display:grid;grid-template-columns:150px 1.5fr 1fr 1fr 1.1fr;gap:4px;font-size:0.8em;margin-top:0.8rem">
<div style="font-weight:700;color:#475569;padding:0.2rem 0.6rem">Nivel</div>
<div style="font-weight:700;color:#475569;padding:0.2rem 0.6rem">¿Qué resuelve?</div>
<div style="font-weight:700;color:#475569;padding:0.2rem 0.6rem">Dirección</div>
<div style="font-weight:700;color:#475569;padding:0.2rem 0.6rem">Protocolos</div>
<div style="font-weight:700;color:#475569;padding:0.2rem 0.6rem">¿Quién lo hace?</div>
<div style="background:#d97706;color:white;font-weight:700;border-radius:6px 0 0 6px;padding:0.5rem 0.6rem">Aplicación</div>
<div style="background:#fef3c7;padding:0.5rem 0.6rem">¿Qué se pide y qué se responde?</div>
<div style="background:#fef3c7;padding:0.5rem 0.6rem">URL, cabecera <code>Host</code></div>
<div style="background:#fef3c7;padding:0.5rem 0.6rem">HTTP, DNS, SMTP</div>
<div style="background:#fef3c7;border-radius:0 6px 6px 0;padding:0.5rem 0.6rem">El programa: <code>curl</code>, Apache</div>
<div style="background:#16a34a;color:white;font-weight:700;border-radius:6px 0 0 6px;padding:0.5rem 0.6rem">Transporte</div>
<div style="background:#dcfce7;padding:0.5rem 0.6rem">¿A qué <strong>programa</strong> del equipo va?</div>
<div style="background:#dcfce7;padding:0.5rem 0.6rem">Puerto</div>
<div style="background:#dcfce7;padding:0.5rem 0.6rem">TCP, UDP</div>
<div style="background:#dcfce7;border-radius:0 6px 6px 0;padding:0.5rem 0.6rem">Sistema operativo</div>
<div style="background:#1e56b0;color:white;font-weight:700;border-radius:6px 0 0 6px;padding:0.5rem 0.6rem">Red</div>
<div style="background:#dbeafe;padding:0.5rem 0.6rem">¿A qué <strong>equipo</strong> va y por qué camino?</div>
<div style="background:#dbeafe;padding:0.5rem 0.6rem">IP</div>
<div style="background:#dbeafe;padding:0.5rem 0.6rem">IP</div>
<div style="background:#dbeafe;border-radius:0 6px 6px 0;padding:0.5rem 0.6rem">Sistema operativo y routers</div>
<div style="background:#64748b;color:white;font-weight:700;border-radius:6px 0 0 6px;padding:0.5rem 0.6rem">Enlace</div>
<div style="background:#f1f5f9;padding:0.5rem 0.6rem">¿A qué <strong>tarjeta</strong> de la red local va?</div>
<div style="background:#f1f5f9;padding:0.5rem 0.6rem">MAC</div>
<div style="background:#f1f5f9;padding:0.5rem 0.6rem">Ethernet, Wi-Fi</div>
<div style="background:#f1f5f9;border-radius:0 6px 6px 0;padding:0.5rem 0.6rem">Tarjeta de red y switch</div>
</div>

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>HTTP solo se ocupa de <strong>qué se pide</strong>. Que el mensaje llegue al equipo y al programa correctos es trabajo de los niveles de abajo.</div>
</div>

---

## Encapsulamiento

Al enviar, cada nivel **añade su cabecera** a lo que le entrega el nivel de arriba. Así viaja una petición HTTP:

<div style="border:2px solid #64748b;background:#f1f5f9;border-radius:8px;padding:0.4rem;display:flex;gap:0.5rem;align-items:stretch;font-size:0.8em;margin-top:0.6rem">
<div style="padding:0.3rem 0.5rem;min-width:130px"><strong style="color:#475569">Ethernet</strong><br>MAC origen<br>MAC destino</div>
<div style="flex:1;border:2px solid #1e56b0;background:#dbeafe;border-radius:8px;padding:0.4rem;display:flex;gap:0.5rem">
<div style="padding:0.3rem 0.5rem;min-width:150px"><strong style="color:#1e56b0">IP</strong><br>192.168.1.10<br>→ 10.0.0.5</div>
<div style="flex:1;border:2px solid #16a34a;background:#dcfce7;border-radius:8px;padding:0.4rem;display:flex;gap:0.5rem">
<div style="padding:0.3rem 0.5rem;min-width:120px"><strong style="color:#15803d">TCP</strong><br>puerto 51234<br>→ 80</div>
<div style="flex:1;border:2px solid #d97706;background:#fef3c7;border-radius:8px;padding:0.3rem 0.6rem">
<strong style="color:#b45309">HTTP</strong> (los datos)<br><code>GET /index.html HTTP/1.1</code><br><code>Host: www.ejemplo.org</code>
</div>
</div>
</div>
</div>

- Al **recibir** se hace al revés: cada nivel **quita su cabecera** y pasa el resto al nivel de arriba
- Cada nivel **solo lee su cabecera**: lo que va dentro son datos que no entiende
- El nivel de red ve IPs; el de transporte, puertos. **Solo la aplicación ve el `GET` y el `Host`**

---

## ¿Quién mira cada cabecera?

El recorrido de la petición anterior hasta el servidor web:

<div style="display:flex;align-items:flex-start;justify-content:center;gap:0.5rem;margin:0.8rem 0;font-size:0.82em">
<div style="text-align:center;width:150px"><div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem;background:#f1f5f9;font-weight:600">Cliente<br><code>curl</code></div><div style="margin-top:0.4rem">crea las<br>4 cabeceras</div></div>
<div style="font-size:1.6rem;color:#1e56b0;padding-top:0.6rem">→</div>
<div style="text-align:center;width:150px"><div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem;background:#f1f5f9;font-weight:600">Switch<br>&nbsp;</div><div style="margin-top:0.4rem"><span style="background:#64748b;color:white;border-radius:4px;padding:0 0.4rem">Ethernet</span></div></div>
<div style="font-size:1.6rem;color:#1e56b0;padding-top:0.6rem">→</div>
<div style="text-align:center;width:170px"><div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem;background:#f1f5f9;font-weight:600">Router<br>&nbsp;</div><div style="margin-top:0.4rem"><span style="background:#1e56b0;color:white;border-radius:4px;padding:0 0.4rem">IP</span><br><span style="background:#16a34a;color:white;border-radius:4px;padding:0 0.4rem">puertos</span> si hace NAT</div></div>
<div style="font-size:1.6rem;color:#1e56b0;padding-top:0.6rem">→</div>
<div style="text-align:center;width:170px"><div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem;background:#f1f5f9;font-weight:600">Servidor<br>sistema operativo</div><div style="margin-top:0.4rem"><span style="background:#1e56b0;color:white;border-radius:4px;padding:0 0.4rem">IP</span> <span style="background:#16a34a;color:white;border-radius:4px;padding:0 0.4rem">TCP</span><br>puerto 80 → Apache</div></div>
<div style="font-size:1.6rem;color:#1e56b0;padding-top:0.6rem">→</div>
<div style="text-align:center;width:150px"><div style="border:2px solid #d97706;border-radius:8px;padding:0.4rem;background:#fef3c7;font-weight:600">Apache<br>&nbsp;</div><div style="margin-top:0.4rem"><span style="background:#d97706;color:white;border-radius:4px;padding:0 0.4rem">HTTP</span><br><code>GET</code>, <code>Host</code></div></div>
</div>

- El **router nunca ve la URL ni el `Host`**: decide solo con IPs (y puertos si hace NAT)
- El sistema operativo del servidor entrega los datos al programa que **escucha en el puerto 80**: `ss -tlnp` lo muestra
- Solo Apache **entiende la petición**: por eso un **virtual host** puede elegir el sitio por el nombre

---

## HTTP sobre TCP

HTTP es un protocolo de **aplicación**: se apoya en **TCP** (transporte) para que sus mensajes lleguen.

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### Lo que TCP le da a HTTP

- Entrega **fiable y en orden**: si se pierde un segmento, TCP lo reenvía
- **Orientado a conexión**: antes de enviar la petición hay que **establecer la conexión**
- Puertos: **80** (HTTP) y **443** (HTTPS)

</div>

<div class="card card-green">

### HTTP/3 cambia el transporte

- Usa **QUIC** sobre **UDP 443**
- QUIC hace el trabajo de TCP (fiabilidad, orden) y **establece la conexión más rápido**
- Los mensajes HTTP **significan lo mismo**

</div>

</div>

---

## Establecimiento de la conexión TCP

<div class="cols-60-40" style="align-items:center">

<div>
<svg viewBox="0 0 640 380" style="width:100%;max-height:470px" font-family="sans-serif" font-size="15">
<defs>
<marker id="hs-tcp" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="8" markerHeight="8" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#16a34a"/></marker>
<marker id="hs-http" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="8" markerHeight="8" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#d97706"/></marker>
<marker id="hs-fin" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="8" markerHeight="8" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#94a3b8"/></marker>
</defs>
<text x="150" y="24" text-anchor="middle" font-weight="700" fill="#1e293b">Cliente</text>
<text x="560" y="24" text-anchor="middle" font-weight="700" fill="#1e293b">Servidor :80</text>
<line x1="150" y1="36" x2="150" y2="365" stroke="#94a3b8" stroke-width="2"/>
<line x1="560" y1="36" x2="560" y2="365" stroke="#94a3b8" stroke-width="2"/>
<line x1="150" y1="55" x2="560" y2="95" stroke="#16a34a" stroke-width="2.5" marker-end="url(#hs-tcp)"/>
<text x="355" y="68" text-anchor="middle" fill="#15803d" font-weight="700" style="paint-order:stroke;stroke:#fff;stroke-width:5px">SYN</text>
<line x1="560" y1="95" x2="150" y2="135" stroke="#16a34a" stroke-width="2.5" marker-end="url(#hs-tcp)"/>
<text x="355" y="108" text-anchor="middle" fill="#15803d" font-weight="700" style="paint-order:stroke;stroke:#fff;stroke-width:5px">SYN + ACK</text>
<line x1="150" y1="135" x2="560" y2="175" stroke="#16a34a" stroke-width="2.5" marker-end="url(#hs-tcp)"/>
<text x="355" y="148" text-anchor="middle" fill="#15803d" font-weight="700" style="paint-order:stroke;stroke:#fff;stroke-width:5px">ACK</text>
<line x1="150" y1="180" x2="560" y2="220" stroke="#d97706" stroke-width="2.5" marker-end="url(#hs-http)"/>
<text x="355" y="193" text-anchor="middle" fill="#b45309" font-weight="700" style="paint-order:stroke;stroke:#fff;stroke-width:5px">GET /index.html</text>
<line x1="560" y1="220" x2="150" y2="260" stroke="#d97706" stroke-width="2.5" marker-end="url(#hs-http)"/>
<text x="355" y="233" text-anchor="middle" fill="#b45309" font-weight="700" style="paint-order:stroke;stroke:#fff;stroke-width:5px">200 OK + HTML</text>
<line x1="150" y1="295" x2="560" y2="320" stroke="#94a3b8" stroke-width="2" stroke-dasharray="6 4" marker-end="url(#hs-fin)"/>
<line x1="560" y1="320" x2="150" y2="345" stroke="#94a3b8" stroke-width="2" stroke-dasharray="6 4" marker-end="url(#hs-fin)"/>
<text x="355" y="290" text-anchor="middle" fill="#64748b" style="paint-order:stroke;stroke:#fff;stroke-width:5px">FIN: cierre de la conexión</text>
<path d="M132,55 L124,55 L124,135 L132,135" fill="none" stroke="#15803d" stroke-width="2"/>
<text x="116" y="92" text-anchor="end" fill="#15803d" font-weight="700">1 RTT</text>
<text x="116" y="110" text-anchor="end" fill="#15803d" font-size="13">conexión</text>
<path d="M132,180 L124,180 L124,260 L132,260" fill="none" stroke="#b45309" stroke-width="2"/>
<text x="116" y="217" text-anchor="end" fill="#b45309" font-weight="700">1 RTT</text>
<text x="116" y="235" text-anchor="end" fill="#b45309" font-size="13">petición</text>
</svg>
</div>

<div>

- <span style="color:#15803d;font-weight:700">Transporte (TCP)</span>: **saludo en tres pasos** (*three-way handshake*). Ningún dato HTTP viaja hasta que termina
- <span style="color:#b45309;font-weight:700">Aplicación (HTTP)</span>: la petición y la respuesta van **dentro** de esa conexión
- **RTT** (*round-trip time*): lo que tarda un mensaje en ir y volver. Lo ves con `ping`
- Cada conexión nueva cuesta **1 RTT más** antes de poder pedir nada

</div>

</div>

---

## Conexiones persistentes (*keep-alive*)

Una página con **3 recursos** (`index.html`, `estilo.css`, `logo.png`), con un RTT de 50 ms:

<div style="font-size:0.8em;margin-top:0.6rem">
<div style="font-weight:700;margin-bottom:0.3rem">Sin keep-alive (HTTP/1.0): una conexión por recurso → <span style="color:#dc2626">6 RTT ≈ 300 ms</span></div>
<div style="display:flex;gap:3px;color:white;text-align:center;font-weight:600">
<div style="width:15%;background:#16a34a;border-radius:4px;padding:0.3rem 0">conexión</div>
<div style="width:15%;background:#d97706;border-radius:4px;padding:0.3rem 0">index.html</div>
<div style="width:15%;background:#16a34a;border-radius:4px;padding:0.3rem 0">conexión</div>
<div style="width:15%;background:#d97706;border-radius:4px;padding:0.3rem 0">estilo.css</div>
<div style="width:15%;background:#16a34a;border-radius:4px;padding:0.3rem 0">conexión</div>
<div style="width:15%;background:#d97706;border-radius:4px;padding:0.3rem 0">logo.png</div>
</div>
<div style="font-weight:700;margin:0.8rem 0 0.3rem">Con keep-alive (HTTP/1.1): una conexión para todas las peticiones → <span style="color:#15803d">4 RTT ≈ 200 ms</span></div>
<div style="display:flex;gap:3px;color:white;text-align:center;font-weight:600">
<div style="width:15%;background:#16a34a;border-radius:4px;padding:0.3rem 0">conexión</div>
<div style="width:15%;background:#d97706;border-radius:4px;padding:0.3rem 0">index.html</div>
<div style="width:15%;background:#d97706;border-radius:4px;padding:0.3rem 0">estilo.css</div>
<div style="width:15%;background:#d97706;border-radius:4px;padding:0.3rem 0">logo.png</div>
</div>
<div style="color:#64748b;margin-top:0.2rem">tiempo →</div>
</div>

- Se ahorra **un saludo TCP por recurso**: con 50 recursos, 100 RTT frente a 51
- El servidor también gana: abre y cierra **menos conexiones**
- En HTTP/1.1 es lo normal: lo indica la cabecera `Connection: keep-alive`, y el servidor cierra la conexión si no recibe peticiones durante un tiempo

<div class="alerta alerta-info" style="margin-top:0.2rem">
<span>ℹ️</span><div>Con <code>curl -v &lt;url1&gt; &lt;url2&gt;</code> (mismo servidor) verás que la segunda petición <strong>reutiliza la conexión</strong>.</div>
</div>

---

## Versiones de HTTP

| Versión | Año | Transporte | Formato | Tráfico web (aprox.) |
|:--|:--|:--|:--|:--|
| **HTTP/1.1** | 1997 | TCP | Texto | ~30 % |
| **HTTP/2** | 2015 | TCP + TLS | Binario, varias peticiones a la vez por una conexión, cabeceras comprimidas | ~50 % |
| **HTTP/3** | 2022 | QUIC (UDP) con TLS 1.3 | Binario | ~20 % |

- La **semántica es la misma** en las tres: métodos, códigos de estado y cabeceras
- HTTP/1.1 sigue muy presente: servidores pequeños y comunicación **proxy → backend**
- Con `curl -I` a un sitio moderno verás `HTTP/2 200` (sin descripción y cabeceras en minúscula)

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Porcentajes aproximados del tráfico web en 2026 (Cloudflare Radar).</div>
</div>

---

## Anatomía de una URL

```
https://www.ejemplo.com:8080/docs/manual.html?idioma=es&pagina=2#capitulo2
```

- **Esquema** `https` — protocolo (y puerto por defecto: 80 / 443)
- **Host** `www.ejemplo.com` — servidor; viaja en la cabecera **`Host`**
- **Puerto** `8080` — opcional
- **Ruta** `/docs/manual.html` y **consulta** `?idioma=es&pagina=2` — viajan en la **línea de petición**
  - `?` inicia la consulta, `&` separa los parámetros y `=` separa nombre y valor
- **Fragmento** `#capitulo2` — **no se envía**: lo usa el navegador

```
GET /docs/manual.html?idioma=es&pagina=2 HTTP/1.1
Host: www.ejemplo.com:8080
```

---

## Estructura de los mensajes

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Petición

- **Línea inicial**: método, URL del recurso y versión
- **Cabeceras** de petición (clave-valor)
- **Línea vacía**
- **Cuerpo** opcional (típico en `POST` y `PUT`)

```
GET /index.html HTTP/1.1
Host: www.ejemplo.com
User-Agent: curl/8.14.1
```

</div>

<div class="card card-green">

### Respuesta

- **Línea de estado**: versión, código y descripción
- **Cabeceras** de respuesta
- **Línea vacía**
- **Cuerpo** con el recurso solicitado

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1024
```

</div>

</div>

---

## Métodos de petición

- **GET** — Solicita un recurso. Los datos viajan en la URL (*query string*)
- **HEAD** — Igual que `GET` pero pide **sólo las cabeceras**
- **POST** — Envía datos al servidor en el **cuerpo** del mensaje
- **PUT** — Almacena el documento enviado en la URL indicada
- **DELETE** — Elimina el recurso referenciado por la URL
- **OPTIONS**, **PATCH**, **TRACE**, **CONNECT** — usos más específicos

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div><code>GET</code> y <code>HEAD</code> son métodos <strong>seguros</strong>: no deben modificar nada en el servidor. Es la <strong>aplicación</strong> la que tiene que respetarlo.</div>
</div>

---

## GET: los datos viajan en la URL

```
GET /login.php?usuario=pepe&clave=1234 HTTP/1.1
Host: www.ejemplo.com
```

Los datos forman parte de la URL, así que quedan en el **historial** del navegador y en el **`access.log`** del servidor:

```
192.168.1.20 - - [06/Oct/2026:10:15:32 +0200] "GET /login.php?usuario=pepe&clave=1234 HTTP/1.1" 200 512
```

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Nunca se envían contraseñas ni datos personales por <code>GET</code>: cualquiera que lea los logs los ve.</div>
</div>

---

## POST: los datos viajan en el cuerpo

```
POST /login.php HTTP/1.1
Host: www.ejemplo.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 23

usuario=pepe&clave=1234
```

Los datos van **después de la línea vacía**. En el `access.log` solo aparece la ruta:

```
192.168.1.20 - - [06/Oct/2026:10:16:05 +0200] "POST /login.php HTTP/1.1" 200 512
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div><code>POST</code> no cifra nada: con <code>curl -v</code> o capturando el tráfico se ve el cuerpo. Para protegerlo hace falta <strong>HTTPS</strong>.</div>
</div>

---

## Códigos de estado

Cifra de tres dígitos que clasifica la respuesta. La primera cifra indica el grupo:

| Código | Familia | Significado |
|:--|:--|:--|
| **1xx** | Informativo | Información intermedia |
| **2xx** | Éxito | La petición se procesó correctamente (`200 OK`, `201 Created`) |
| **3xx** | Redirección | El recurso está en otra URL (`301`, `302`, `304`) |
| **4xx** | Error del cliente | Petición incorrecta (`400`, `401`, `403`, `404`) |
| **5xx** | Error del servidor | Fallo en el servidor (`500`, `502`, `503`) |

---

## Códigos que verás administrando un servidor

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### 3xx y 4xx

- **301 / 302** — redirección
- **304** — se usa la copia en caché
- **401** — faltan credenciales
- **403** — acceso prohibido
- **404** — el recurso no existe

</div>

<div class="card card-green">

### 5xx

- **500** — fallo de la aplicación o la configuración
- **502** — el proxy no recibe respuesta válida del backend
- **503** — sin servicio (backends caídos)
- **504** — el backend tarda demasiado

</div>

</div>

---

## Cabeceras HTTP

Información de la forma **clave: valor** asociada a peticiones y respuestas.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Genéricas y de petición

- `Host` — nombre del sitio solicitado
- `User-Agent` — cliente que hace la petición
- `Accept` — tipos MIME aceptados
- `Accept-Language` — idiomas
- `Cookie` — cookies enviadas
- `Authorization` — credenciales

</div>

<div class="card card-green">

### De respuesta

- `Content-Type` — tipo MIME del cuerpo
- `Content-Length` — tamaño en bytes
- `Server` — software (y versión) del servidor
- `Set-Cookie` — define una cookie
- `Location` — URL destino en redirecciones
- `Cache-Control` — directivas de caché

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div><code>Host</code> es <strong>obligatoria</strong> en HTTP/1.1: gracias a ella un servidor con una sola IP aloja varios sitios (<strong>virtual hosts</strong>). <code>Server</code> revela el software y la versión: en producción se suele ocultar.</div>
</div>

---

## Una petición real con `curl -v`

```
* Connected to dit.gonzalonazareno.org (5.196.224.198) port 80
> GET / HTTP/1.1
> Host: dit.gonzalonazareno.org
> User-Agent: curl/8.14.1
> Accept: */*
>
< HTTP/1.1 301 Moved Permanently
< Server: nginx/1.22.1
< Content-Type: text/html
< Content-Length: 169
< Connection: keep-alive
< Location: https://dit.gonzalonazareno.org/
```

- **`*`** información de `curl`: la conexión TCP (**red y transporte**)
- **`>`** envía el cliente · **`<`** responde el servidor: el mensaje HTTP (**aplicación**)
- La **línea vacía** cierra las cabeceras · **301** con `Location`: el recurso está en HTTPS

---

## Cookies

> Pequeños fragmentos de información que el **navegador** guarda a petición del **servidor** mediante la cabecera `Set-Cookie`.

### ¿Para qué sirven?

- Guardar información de **sesión**
- **Comercio electrónico**: carrito de la compra
- **Personalización**: idioma, tema, preferencias
- **Seguimiento** de visitas y publicidad
- Mantener la **sesión iniciada** (login)

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Las cookies permiten <strong>mantener estado</strong> entre peticiones que, por diseño, son independientes.</div>
</div>

---

## Cookies: cómo funcionan

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### 1ª respuesta del servidor

```
< Set-Cookie: sesion=a3f9c2; Path=/;
    Max-Age=3600; HttpOnly; Secure
```

### Peticiones siguientes del navegador

```
> Cookie: sesion=a3f9c2
```

</div>

<div class="card card-green">

### Atributos

- **`Max-Age`** / **`Expires`** — cuándo caduca
- **`Path`** / **`Domain`** — a qué URLs se envía
- **`HttpOnly`** — JavaScript no puede leerla
- **`Secure`** — solo se envía por HTTPS

</div>

</div>

---

## Sesiones

HTTP es **stateless**: las sesiones añaden la noción de estado a las aplicaciones web.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Lo que guarda el servidor

- **Identificador** de sesión
- **Usuario** asociado
- **Tiempo de expiración**
- Datos temporales de la aplicación

</div>

<div class="card card-green">

### Lo que guarda el cliente

- Sólo el **identificador** de sesión
- Habitualmente en una **cookie**
- En cada petición lo envía al servidor
- El servidor recupera el resto de su lado

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Aplicaciones que usan HTTP

## Servidor web, proxy, proxy inverso y balanceador

---

## Servidor web

Programa que **implementa el protocolo HTTP**: recibe las peticiones de los clientes y les devuelve los recursos solicitados.

- Sirve **páginas estáticas** (HTML, CSS, JS, imágenes…)
- Sirve **páginas dinámicas** generadas por lenguajes como **PHP**, **Python**, **Java**…, normalmente apoyándose en un **servidor de aplicaciones**
- Implementa funcionalidades del protocolo: **virtual hosts, redirecciones, autenticación, control de acceso…**

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Ejemplos: <strong>Apache</strong> y <strong>Nginx</strong>, que estudiaremos en la siguiente presentación.</div>
</div>

---

## Proxy

Servidor que **intermedia las peticiones** entre los clientes de una red y los servidores de Internet. Actúa **en nombre de los clientes**.

<div style="display:flex;align-items:center;justify-content:center;gap:0.8rem;margin:0.8rem 0">
<div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem 0.9rem;background:#f1f5f9;font-weight:600;text-align:center">Clientes de la red</div>
<div style="font-size:1.6rem;color:var(--teal-600)">→</div>
<div style="border:2px solid var(--teal-600);border-radius:8px;padding:0.4rem 0.9rem;background:var(--teal-50);font-weight:600;text-align:center">Proxy</div>
<div style="font-size:1.6rem;color:var(--teal-600)">→</div>
<div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem 0.9rem;background:#f1f5f9;font-weight:600;text-align:center">Servidores de Internet</div>
</div>

- Da salida a equipos que **no acceden directamente a Internet**
- **Filtra** el tráfico HTTP: por dominio, URL, contenido, horario…
- Hace de **caché** de respuestas para acelerar accesos posteriores

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Software clásico: <strong>Squid</strong>.</div>
</div>

---

## Proxy inverso

Recibe las peticiones de los clientes y las **reenvía a uno o varios servidores internos** (backends). Actúa **en nombre de los servidores**: el cliente no accede directamente a ellos.

<div style="display:flex;align-items:center;justify-content:center;gap:0.8rem;margin:0.8rem 0">
<div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem 0.9rem;background:#f1f5f9;font-weight:600;text-align:center">Clientes de Internet</div>
<div style="font-size:1.6rem;color:var(--teal-600)">→</div>
<div style="border:2px solid var(--teal-600);border-radius:8px;padding:0.4rem 0.9rem;background:var(--teal-50);font-weight:600;text-align:center">Proxy inverso</div>
<div style="font-size:1.6rem;color:var(--teal-600)">→</div>
<div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem 0.9rem;background:#f1f5f9;font-weight:600;text-align:center">Servidores internos (backends)</div>
</div>

- **Protege** los servidores internos (ocultando su ubicación)
- Publica **varias aplicaciones** con un único nombre o IP
- Centraliza **TLS/SSL**, caché, compresión y autenticación

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Ejemplos: <strong>Apache</strong>, <strong>Nginx</strong>, <strong>Varnish</strong>, <strong>Traefik</strong>.</div>
</div>

---

## Nivel de red o nivel de aplicación

En la Unidad 1 usamos **DNAT** para llegar a un servidor interno, y el **proxy inverso** también lo hace. Parecen lo mismo, pero no trabajan en el mismo nivel:

| | DNAT (iptables / nftables en el router) | Proxy inverso (Apache, Nginx) |
|:--|:--|:--|
| **Nivel** | Red y transporte | Aplicación |
| **Decide por** | IP y puerto de destino | Cabecera `Host`, URL |
| **Conexiones TCP** | **Una**: del cliente al servidor | **Dos**: del cliente al proxy y del proxy al servidor |
| **¿Entiende HTTP?** | No: cambia direcciones y reenvía | Sí: lee y cambia cabeceras, puede guardar en caché |
| **Varios sitios en el puerto 80** | No puede distinguirlos | Los reparte por nombre |

<div class="alerta alerta-warning" style="margin-top:0.4rem">
<span>💡</span><div>Pregúntate <strong>qué mira para decidir</strong>: si va <strong>dentro de la petición HTTP</strong> (<code>Host</code>, URL), es el nivel de <strong>aplicación</strong>.</div>
</div>

---

## Balanceador de carga

Distribuye las peticiones entre **varios backends que sirven lo mismo**. Normalmente es un **proxy inverso con varios backends**.

<div style="display:flex;align-items:center;justify-content:center;gap:0.8rem;margin:0.8rem 0">
<div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem 0.9rem;background:#f1f5f9;font-weight:600;text-align:center">Clientes</div>
<div style="font-size:1.6rem;color:var(--teal-600)">→</div>
<div style="border:2px solid var(--teal-600);border-radius:8px;padding:0.4rem 0.9rem;background:var(--teal-50);font-weight:600;text-align:center">Balanceador</div>
<div style="font-size:1.6rem;color:var(--teal-600)">→</div>
<div style="display:flex;flex-direction:column;gap:0.4rem">
<div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem 0.9rem;background:#f1f5f9;font-weight:600;text-align:center">backend1</div>
<div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem 0.9rem;background:#f1f5f9;font-weight:600;text-align:center">backend2</div>
</div>
</div>

- Reparte la carga según un **algoritmo de distribución**
- Evita la **sobrecarga** de un único servidor
- Si cae un backend, sigue dando servicio con el resto

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Ejemplos: <strong>Nginx</strong>, <strong>HAProxy</strong>, <strong>Apache mod_proxy_balancer</strong>.</div>
</div>

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## HTTP

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI</span>
</div>
