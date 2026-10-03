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

## HTTP sobre TCP

HTTP es un protocolo de **aplicación** que usa **TCP** como transporte.

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### En la pila TCP/IP

- **Aplicación**: HTTP, HTTPS, SMTP…
- **Transporte**: TCP 80 (HTTP) y 443 (HTTPS)
- **Red**: IP
- **Enlace**: Ethernet, Wi-Fi…

</div>

<div class="card card-green">

### Conexiones persistentes

- **HTTP/1.0**: una conexión **por recurso**
- **HTTP/1.1** (*keep-alive*): una conexión para **varias peticiones**
- **HTTP/3**: cambia TCP por **QUIC** sobre **UDP** 443

</div>

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
https://www.ejemplo.com:8080/docs/manual.html?idioma=es#capitulo2
```

- **Esquema** `https` — protocolo (y puerto por defecto: 80 / 443)
- **Host** `www.ejemplo.com` — servidor; viaja en la cabecera **`Host`**
- **Puerto** `8080` — opcional
- **Ruta** `/docs/manual.html` y **consulta** `?idioma=es` — viajan en la **línea de petición**
- **Fragmento** `#capitulo2` — **no se envía**: lo usa el navegador

```
GET /docs/manual.html?idioma=es HTTP/1.1
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

- **`*`** información de `curl` (conexión TCP) · **`>`** lo que envía el cliente · **`<`** lo que responde el servidor
- La **línea vacía** marca el final de las cabeceras; después iría el cuerpo
- Respuesta **301** con `Location`: el recurso está en la versión HTTPS

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
