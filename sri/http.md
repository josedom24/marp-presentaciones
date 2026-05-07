---
marp: true
theme: profesional
paginate: true
header: 'SRI · Unidad 3 — Protocolo HTTP y servidores web'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Protocolo **HTTP**

## Servidores web, proxy y balanceadores

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

## Concepto, mensajes y métodos

---

## ¿Qué es HTTP?

> **HyperText Transfer Protocol** es el protocolo de la **capa de aplicación** que permite la comunicación entre clientes, servidores y proxies en la web.

### Características fundamentales

- Última versión ampliamente desplegada: **HTTP/1.1** (con HTTP/2 y HTTP/3 en uso creciente)
- Basado en el esquema **petición / respuesta**: el cliente pide, el servidor responde
- Mensajes en **texto plano**, legibles y depurables
- Protocolo **sin estado**: el servidor no recuerda quién hizo cada petición

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>La falta de estado se compensa más tarde con <strong>cookies</strong> y <strong>sesiones</strong>.</div>
</div>

---

## HTTP sobre TCP

HTTP usa **TCP** como transporte (puerto **80** por defecto, **443** en HTTPS). Cada conversación se apoya en una conexión TCP previamente establecida.

### Conexiones persistentes (keep-alive)

- En **HTTP/1.0** se abría una conexión TCP **por cada recurso**
- En **HTTP/1.1** una misma conexión sirve **varias peticiones y respuestas**
- Reduce la latencia y el coste de los *handshakes* TCP
- Mejora notable en páginas con muchos recursos (CSS, JS, imágenes…)

---

## HTTP en la pila TCP/IP

| Capa | Protocolo |
|:--|:--|
| Aplicación | **HTTP**, HTTPS, FTP, SMTP… |
| Transporte | **TCP** (puerto 80/443) |
| Red | IP |
| Enlace | Ethernet, Wi-Fi… |

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>HTTPS es <strong>HTTP sobre TLS</strong>: cifra el contenido pero conserva la misma semántica de peticiones y respuestas.</div>
</div>

---

## Estructura de los mensajes

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Petición

- **Línea inicial**: método, URL del recurso y versión
- **Cabeceras** de petición (clave-valor)
- **Cuerpo** opcional (típico en `POST` y `PUT`)

```
GET /index.html HTTP/1.1
Host: www.ejemplo.com
User-Agent: curl/8.5
```

</div>

<div class="card card-green">

### Respuesta

- **Línea de estado**: versión, código y descripción
- **Cabeceras** de respuesta
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
<span>ℹ️</span><div>El protocolo no impone que un método sea seguro: lo es la <strong>aplicación</strong> que lo implementa. Una petición <code>GET</code> nunca debería modificar estado.</div>
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

## Cabeceras HTTP

Información de la forma **clave: valor** asociada a peticiones y respuestas.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Genéricas y de petición

- `Host` — dominio solicitado
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
- `Server` — software del servidor
- `Set-Cookie` — define una cookie
- `Location` — URL destino en redirecciones
- `Cache-Control` — directivas de caché

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Funcionalidades del servidor web

## Virtual Hosts, redirecciones, autenticación y cookies

---

## Virtual Hosts

Un **Virtual Host** permite que un mismo servidor web atienda **varios sitios web** desde una **única dirección IP**.

### Características

- Aloja **múltiples dominios** en el mismo servidor
- Cada sitio tiene su propio contenido, configuración y registros de acceso
- Es una técnica fundamental en **Apache** y **Nginx**
- Tipo más habitual: **basado en nombre** — el servidor distingue el sitio por la cabecera `Host` de la petición

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Sin la cabecera <code>Host</code> el servidor no sabría a qué sitio dirigir la petición: por eso es <strong>obligatoria</strong> en HTTP/1.1.</div>
</div>

---

## Alias

Un **alias** asocia una URL a una **ruta diferente** dentro del servidor, sin mover los archivos físicamente.

```apache
Alias /imagenes/ "/var/www/recursos/img/"

<Directory "/var/www/recursos/img/">
  Options Indexes FollowSymLinks
  AllowOverride None
  Require all granted
</Directory>
```

- Una petición a `http://servidor/imagenes/logo.png` accede realmente a `/var/www/recursos/img/logo.png`
- Útil para **reorganizar** rutas o **proteger** la ubicación real de los recursos

---

## Redirecciones

Cuando un recurso ha **cambiado de ubicación**, el servidor indica al cliente la nueva dirección con un código **3xx**.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Códigos típicos

- **301 Moved Permanently** — cambio definitivo, cacheable
- **302 Found** — cambio temporal
- **307 / 308** — variantes que conservan el método

</div>

<div class="card card-green">

### Cómo funciona

- El servidor responde con el código y la cabecera **`Location:`**
- El cliente realiza una **nueva petición** a esa URL
- Las herramientas como `curl` necesitan `-L` para seguir la redirección

</div>

</div>

---

## Autenticación

Mecanismo para que el cliente se identifique antes de obtener un recurso.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Básica (*Basic*)

- Credenciales codificadas en **Base64**
- Cabecera: `Authorization: Basic dXNlcjpwYXNz`
- **No cifra**: sólo segura sobre **HTTPS**

</div>

<div class="card card-green">

### Resumen (*Digest*)

- Las credenciales viajan como **hash**
- Cabecera: `Authorization: Digest username="...", response="..."`
- Más segura que Basic en HTTP plano

</div>

</div>

---

## Control de acceso

Restringe quién puede acceder a determinados recursos o directorios. Puede combinar varios criterios:

- **Dirección IP** o red de origen
- **Nombre de usuario y contraseña**
- **Nombres de dominio** del cliente
- **Cabeceras** HTTP o autenticación previa

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>En Apache se configura con directivas como <code>Require ip</code>, <code>Require valid-user</code> o <code>Require host</code>; en Nginx con bloques <code>allow</code> / <code>deny</code> y <code>auth_basic</code>.</div>
</div>

---

## Cookies

> Pequeños fragmentos de información que el **navegador** guarda a petición del **servidor** mediante la cabecera `Set-Cookie`.

### ¿Para qué sirven?

- Guardar información de **sesión**
- **Comercio electrónico**: carrito de la compra
- **Personalización**: idioma, tema, preferencias
- **Seguimiento** de visitas y publicidad
- Recordar **login** y contraseña

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Las cookies permiten <strong>mantener estado</strong> entre peticiones que, por diseño, son independientes.</div>
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

<p class="numero">03</p>

# Servidores web

## Apache, Nginx y comparativa

---

## ¿Qué es un servidor web?

- Programa que **implementa el protocolo HTTP**
- Sirve **páginas estáticas** (HTML, CSS, JS, imágenes…)
- Sirve **páginas dinámicas** generadas por lenguajes como **PHP**, **Python**, **Java**…
- Suele apoyarse en un **servidor de aplicaciones** para ejecutar el código
- Implementa funcionalidades del protocolo: **redirecciones, autenticación, negociación de contenido…**

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>En la práctica, muchas de esas funcionalidades se acaban implementando en la propia <strong>aplicación web</strong>.</div>
</div>

---

## Apache HTTP Server

- Servidor web **HTTP de código abierto**, multiplataforma
- Implementa **HTTP/1.1** y soporta **HTTP/2** mediante módulos
- Desarrollado por la **Apache Software Foundation** (proyecto *httpd*)
- Surgió en 1995 a partir del servidor **NCSA HTTPd**
- **Servidor web más usado en Internet entre 1996 y 2020**
- Destaca por su **estabilidad**, **modularidad** y comunidad
- Integración sencilla con **PHP, Python, Perl, CGI**…
- Versión estable actual: **2.4** (la 2.5 en desarrollo)

---

## Nginx

- Servidor **web y proxy inverso** ligero, de **alto rendimiento**
- Actúa también como **proxy de correo** (IMAP/POP3)
- **Software libre** (BSD simplificada); existe versión comercial **Nginx Plus**
- Creado por **Igor Sysoev** en 2004 para el portal ruso *Rambler*
- Diseñado para **superar a Apache** en archivos estáticos y muchas conexiones simultáneas
- Usa **menos memoria** y procesa más peticiones por segundo
- **Menos flexible** en configuraciones dinámicas (no hay `.htaccess`)
- Menos módulos integrados, pero **mejor escalabilidad**

---

## Comparativa Apache vs Nginx

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Apache

- Modelo basado en **procesos / hilos**
- Configuración por **directorio** (`.htaccess`)
- Enorme catálogo de **módulos**
- Ideal para entornos con **PHP** y configuración heterogénea

</div>

<div class="card card-green">

### Nginx

- Modelo **asíncrono** orientado a eventos
- Configuración **centralizada**
- Mejor rendimiento sirviendo **estáticos**
- Excelente como **proxy inverso** y balanceador

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# Proxy, proxy inverso y balanceador

## Intermediarios HTTP

---

## Proxy

Servidor que **intermedia las peticiones** entre los clientes de una red y los servidores de Internet.

### ¿Para qué se usa?

- Cuando los equipos de la red **no acceden directamente a Internet**
- **Filtrar** y controlar el tráfico HTTP: por dominio, URL, contenido, horario…
- Hacer de **caché** de respuestas para acelerar accesos posteriores
- Mejorar la **seguridad**, anonimato y rendimiento

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Software clásico: <strong>Squid</strong>.</div>
</div>

---

## Proxy inverso

Recibe las peticiones de los clientes y las **redirige a uno o varios servidores internos**. El cliente no accede directamente a los servidores backend.

### Usos habituales

- **Proteger** los servidores internos (ocultando su ubicación)
- **Distribuir carga** entre varios backend
- **Caché** de contenidos estáticos o respuestas frecuentes
- Centralizar **TLS/SSL**, compresión y autenticación

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Ejemplos: <strong>Apache</strong>, <strong>Nginx</strong>, <strong>Varnish</strong>, <strong>Traefik</strong>.</div>
</div>

---

## Balanceador de carga

Distribuye las peticiones entrantes entre **varios servidores backend** para optimizar el rendimiento, la disponibilidad y la tolerancia a fallos.

### Características

- Puede implementarse por **hardware** o **software**
- Reparte la carga según un **algoritmo de distribución**
- Evita la **sobrecarga** de un único servidor
- Permite ofrecer servicio aunque caiga uno de los nodos

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Ejemplos: <strong>Nginx</strong>, <strong>HAProxy</strong>, <strong>Apache mod_proxy_balancer</strong>.</div>
</div>

---

## Algoritmos de balanceo — estáticos

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Round Robin

Las peticiones se reparten **secuencialmente** entre los servidores. Requiere instancias equivalentes y sin estado.

### Sticky Round Robin

Variante con **afinidad**: un mismo cliente se dirige siempre al mismo servidor.

</div>

<div class="card card-green">

### Round Robin ponderado

Cada servidor tiene un **peso**: los más potentes reciben más peticiones.

### Hash

Se aplica una función *hash* (sobre IP, URL…) que **determina** qué servidor atiende cada petición.

</div>

</div>

---

## Algoritmos de balanceo — dinámicos

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Conexiones mínimas

La nueva petición se envía al servidor con **menos conexiones activas** en ese momento.

</div>

<div class="card card-green">

### Menor tiempo de respuesta

Se elige el servidor con el **tiempo de respuesta más bajo** según mediciones recientes.

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Los algoritmos dinámicos se adaptan al estado real del sistema, pero requieren más <strong>monitorización</strong>.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">05</p>

# Ejercicio práctico

## Peticiones HTTP con `curl`

---

## La herramienta `curl`

`curl` es una herramienta de línea de comandos para hacer peticiones HTTP (y muchos otros protocolos). Permite **ver y depurar** el tráfico HTTP sin un navegador.

### Opciones básicas

| Opción | Para qué sirve |
|:--|:--|
| `curl <url>` | Petición **GET** |
| `curl -L <url>` | Sigue **redirecciones** |
| `curl -I <url>` | Petición **HEAD** (sólo cabeceras) |
| `curl -X POST -d "campo=valor" <url>` | Petición **POST** con datos en el cuerpo |
| `curl -v <url>` | Modo *verbose*: muestra cabeceras enviadas y recibidas |

---

## Ejercicios

1. Realiza una petición para ver las **cabeceras** de `https://dit.gonzalonazareno.org`. ¿Qué **código de estado** devuelve? ¿Qué significa? ¿En qué cabecera se encuentra la URL a la que hay que acceder para obtener el recurso?

2. Realiza una petición **GET** a `https://dit.gonzalonazareno.org`. ¿Qué tipo de **redirección** devuelve? Realiza una nueva petición que **siga** la redirección.

3. Con las **herramientas para desarrolladores** del navegador (en Firefox: *Herramientas para desarrolladores → Red*), inspecciona `https://dit.gonzalonazareno.org/gestiona/`. ¿Cuántas peticiones se han realizado para mostrar la página? Identifica las cabeceras más importantes.

---

## Ejercicios (continuación)

4. Obtén el **cuerpo** de la respuesta de `https://dit.gonzalonazareno.org/gestiona/`.

5. Usando el método **GET**, manda tu nombre a la página `https://http.josedomingo.org/index2.php`.

6. Usando el método **POST** (que envía el contenido en el cuerpo), manda tu nombre a la misma página.

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Compara las dos últimas peticiones: el contenido viaja en lugares <strong>distintos</strong> y la página debería distinguirlas.</div>
</div>

---

## Pistas para resolverlos

```bash
# 1 — Cabeceras (HEAD)
curl -I https://dit.gonzalonazareno.org

# 2 — GET y seguir redirección
curl https://dit.gonzalonazareno.org
curl -L https://dit.gonzalonazareno.org

# 4 — Cuerpo de la respuesta
curl https://dit.gonzalonazareno.org/gestiona/

# 5 — GET con parámetros en la URL
curl "https://http.josedomingo.org/index2.php?nombre=Pepe"

# 6 — POST con datos en el cuerpo
curl -X POST -d "nombre=Pepe" https://http.josedomingo.org/index2.php
```

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## HTTP → Apache y Nginx en la práctica

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI</span>
</div>
