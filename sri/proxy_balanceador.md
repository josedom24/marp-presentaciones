---
marp: true
title: Proxy inverso y balanceador de carga
theme: profesional
paginate: true
header: 'SRI · Unidad 2 — Proxy inverso y balanceador de carga'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# **Proxy inverso** y **balanceador de carga**

## Apache, Nginx y HAProxy

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI · Servicios de Red e Internet</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Proxy inverso

## Concepto y problemas comunes

---

## ¿Qué es un proxy inverso?

> Un **proxy inverso** acepta las peticiones de los clientes, las **reenvía** a uno o varios servidores internos (**backends**) y devuelve sus respuestas como si las hubiera generado él mismo.

- El cliente **no accede directamente** a los backends: solo ve al proxy
- Los backends pueden estar en una **red privada** sin acceso desde fuera
- Para el backend, **el cliente es el proxy**: la conexión TCP la abre el proxy

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Un <strong>proxy</strong> actúa en nombre de los clientes; un <strong>proxy inverso</strong>, en nombre de los servidores (lo vimos en <em>Protocolo HTTP</em>).</div>
</div>

---

## Casos de uso

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Protección y centralización

- **Ocultar** la infraestructura backend
- Publicar varias aplicaciones internas bajo **un único nombre o IP**
- Aplicar **filtros** y **autenticación** en un solo punto
- Centralizar las **cabeceras** de seguridad

</div>

<div class="card card-green">

### Rendimiento y operación

- **Distribuir carga** entre varios backends
- **Terminación TLS**: HTTPS solo en el proxy
- **Caché** de respuestas estáticas
- Cambiar los backends **sin que el cliente lo note**

</div>

</div>

---

## ¿Qué `Host` recibe el backend?

El proxy hace una **petición nueva** al backend. ¿Qué pone en la cabecera `Host`?

| Opción | `Host` que llega al backend |
|:--|:--|
| Por defecto | El nombre del **destino del proxy** (`interno.example1.org`) |
| Conservar el del cliente | El nombre **público** que pidió el cliente (`www.app1.org`) |

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Si el backend tiene <strong>virtual hosts por nombre</strong>, elige el sitio por la cabecera <code>Host</code>. Si recibe un nombre que no tiene en ningún <code>ServerName</code>, responde su <strong>sitio por defecto</strong>, no el que esperabas.</div>
</div>

- Se conserva el `Host` del cliente cuando el backend **sí** tiene configurado el nombre público
- En el log del backend se ve qué virtual host ha atendido la petición

---

## El problema de las redirecciones

Si el backend responde con una redirección, la cabecera `Location` lleva **su nombre interno**:

```
HTTP/1.1 301 Moved Permanently
Location: http://interno.example1.org/nuevodirectorio
```

El navegador intenta ir a `interno.example1.org`, un nombre que **no resuelve** (o que no es accesible) desde fuera.

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El proxy tiene que <strong>reescribir la cabecera <code>Location</code></strong> de la respuesta para que apunte al nombre público: <code>ProxyPassReverse</code> en Apache, <code>proxy_redirect</code> en Nginx.</div>
</div>

---

## Publicar en una subruta

Publicar una aplicación en `www.ejemplo.org/shop` tiene **dos trampas**:

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### La barra final

Si el proxy está configurado para `/shop/`, una petición a `/shop` (sin barra) **no coincide**.

Solución: redirigir `/shop` a `/shop/` (Nginx lo hace solo).

</div>

<div class="card card-green">

### Enlaces absolutos

Si la página del backend enlaza `/css/estilo.css`, el navegador lo pide al proxy **sin** `/shop`: la página sale **sin estilos** o rota.

Solución: que la aplicación use enlaces relativos o permita indicar su ruta base, o **reescribir el HTML** en el proxy.

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Proxy inverso con Apache

## mod_proxy y ProxyPass

---

## Activar los módulos necesarios

Apache implementa el proxy inverso con los módulos **`mod_proxy`** y **`mod_proxy_http`**.

```bash
sudo a2enmod proxy proxy_http
sudo systemctl restart apache2
```

### Otros módulos relacionados

- **`mod_headers`** — añadir o cambiar cabeceras de la petición y de la respuesta
- **`mod_proxy_html`** — reescribir los enlaces del HTML que devuelve el backend

---

## Configuración básica

Un virtual host por cada nombre público (`www.app1.org`, `www.app2.org`…):

```apache
<VirtualHost *:80>
    ServerName www.app1.org
    ProxyPass        "/" "http://interno.example1.org/"
    ProxyPassReverse "/" "http://interno.example1.org/"
</VirtualHost>
```

- **`ProxyPass`** — reenvía las peticiones al backend
- **`ProxyPassReverse`** — reescribe `Location` en las respuestas (redirecciones)
- El proxy tiene que **resolver** `interno.example1.org` (DNS o `/etc/hosts`)
- **`ProxyRequests Off`** (por defecto): con `On`, Apache sería un proxy **abierto** a Internet

---

## El `Host` y las redirecciones en Apache

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### `ProxyPreserveHost`

- **`Off`** (por defecto): el backend recibe el nombre de `ProxyPass` (`interno.example1.org`)
- **`On`**: recibe el nombre que pidió el cliente (`www.app1.org`)

```apache
ProxyPreserveHost On
```

</div>

<div class="card card-green">

### `ProxyPassReverse`

Cambia en la respuesta

`http://interno.example1.org/nuevodirectorio`

por

`http://www.app1.org/nuevodirectorio`

Sin esta línea, el cliente recibe el nombre interno.

</div>

</div>

---

## Proxy de subrutas en Apache

```apache
ProxyPass        "/shop/" "http://192.168.100.10:3000/"
ProxyPassReverse "/shop/" "http://192.168.100.10:3000/"

ProxyPass        "/game/" "http://192.168.100.10:8080/"
ProxyPassReverse "/game/" "http://192.168.100.10:8080/"

# /shop y /game sin barra no coinciden con ProxyPass
RedirectMatch "^/(shop|game)$" "/$1/"
```

- Las dos URL de `ProxyPass` tienen que acabar **igual**: las dos con barra o las dos sin ella
- Sin el `RedirectMatch`, `/shop` lo sirve el `DocumentRoot` del proxy: **404**
- Para los enlaces absolutos: `mod_proxy_html` (`ProxyHTMLEnable On`, `ProxyHTMLURLMap`)

---

## Cabeceras hacia el backend

Para el backend, el cliente es el proxy. `mod_proxy_http` añade **automáticamente**:

| Cabecera | Contenido |
|:--|:--|
| `X-Forwarded-For` | IP del cliente original (se van acumulando si hay varios proxies) |
| `X-Forwarded-Host` | `Host` que pidió el cliente |
| `X-Forwarded-Server` | `ServerName` del proxy |

Otras cabeceras se añaden con `mod_headers` (`sudo a2enmod headers`):

```apache
RequestHeader set X-Real-IP         "expr=%{REMOTE_ADDR}"
RequestHeader set X-Forwarded-Proto "http"
```

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# Proxy inverso con Nginx

## proxy_pass y proxy_params

---

## Configuración básica

Nginx incluye el proxy inverso de forma **nativa**, sin activar módulos.

```nginx
server {
    listen 80;
    server_name www.app1.org;

    location / {
        proxy_pass http://interno.example1.org/;
    }
}
```

- **`proxy_pass`** — destino al que se reenvía la petición
- Si Nginx **no resuelve** el nombre de `proxy_pass`, no arranca (`host not found in upstream`)

---

## El `Host` en Nginx

Por defecto Nginx envía `Host: $proxy_host`, el nombre de `proxy_pass` (`interno.example1.org`).

Para enviar el que pidió el cliente:

```nginx
location / {
    proxy_pass       http://interno.example1.org/;
    proxy_set_header Host $host;
}
```

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div><code>include proxy_params</code> también cambia el <code>Host</code> por el del cliente. Si el backend tiene virtual hosts con los <strong>nombres internos</strong>, responderá su sitio por defecto.</div>
</div>

---

## El archivo `proxy_params`

`/etc/nginx/proxy_params` (Debian) envía al backend la información del cliente original:

```nginx
proxy_set_header Host              $http_host;
proxy_set_header X-Real-IP         $remote_addr;
proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

| Variable | Contenido |
|:--|:--|
| `$http_host` | `Host` que pidió el cliente |
| `$proxy_add_x_forwarded_for` | `X-Forwarded-For` recibido + IP del cliente (`$remote_addr`) |

A diferencia de Apache, Nginx **no** añade estas cabeceras si no se indican.

---

## Reescritura de redirecciones

Nginx **ya reescribe** la cabecera `Location` por defecto (`proxy_redirect default`): cambia la URL de `proxy_pass` por la de la `location` y la completa con el nombre que pidió el cliente.

```nginx
location / {
    proxy_pass     http://interno.example1.org/;
    # proxy_redirect default;   ← es el comportamiento por defecto
    # proxy_redirect off;       ← no reescribe: el cliente recibe el nombre interno
}
```

| Respuesta del backend | Lo que recibe el cliente |
|:--|:--|
| `Location: http://interno.example1.org/nuevodirectorio` | `Location: http://www.app1.org/nuevodirectorio` |

Equivale al `ProxyPassReverse` de Apache, que en cambio hay que escribir siempre.

---

## Proxy de subrutas en Nginx

```nginx
server {
    listen 80;
    server_name www.ejemplo.org;
    location /shop/ { proxy_pass http://192.168.100.10:3000/; }
    location /game/ { proxy_pass http://192.168.100.10:8080/; }
}
```

- **Barra final en `proxy_pass`**: con `/` al final se quita el prefijo (`/shop/a` → `/a`); sin barra, se reenvía la URL completa (`/shop/a`)
- Si la `location` acaba en `/`, Nginx redirige solo `/shop` a `/shop/`
- Para los enlaces absolutos: `sub_filter` (reescribe el texto del HTML)

---

## Reescribir los enlaces del HTML

Si el backend usa enlaces absolutos (`/css/…`), el proxy puede añadirles el prefijo:

<div class="cols-2" style="margin-top:0.6rem">

<div>

**Apache** (`a2enmod proxy_html xml2enc`)

```apache
<Location "/shop/">
  ProxyPass "http://192.168.100.10:3000/"
  ProxyPassReverse "http://192.168.100.10:3000/"
  ProxyHTMLEnable On
  ProxyHTMLURLMap "/" "/shop/"
</Location>
```

</div>

<div>

**Nginx** (`sub_filter`)

```nginx
location /shop/ {
  proxy_pass http://192.168.100.10:3000/;
  proxy_set_header Accept-Encoding "";
  sub_filter 'href="/' 'href="/shop/';
  sub_filter 'src="/'  'src="/shop/';
  sub_filter_once off;
}
```

</div>

</div>

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Solo cambia el <strong>HTML</strong>, no el JavaScript ni el CSS. Si la aplicación no funciona en una subruta, se publica con su <strong>propio nombre</strong> (<code>shop.ejemplo.org</code>).</div>
</div>

---

## Apache vs Nginx — proxy inverso

| Aspecto | Apache | Nginx |
|:--|:--|:--|
| Módulos | `a2enmod proxy proxy_http` | Nativo |
| Reenvío | `ProxyPass` | `proxy_pass` |
| Redirecciones | `ProxyPassReverse` (hay que ponerlo) | `proxy_redirect default` (activo) |
| `Host` por defecto | El de `ProxyPass` | El de `proxy_pass` (`$proxy_host`) |
| `Host` del cliente | `ProxyPreserveHost On` | `proxy_set_header Host $host` |
| `X-Forwarded-For` | Automática | `proxy_params` o `proxy_set_header` |
| Subruta sin barra | `RedirectMatch` | Redirección automática |
| Comprobar sintaxis | `apache2ctl configtest` | `nginx -t` |

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# Balanceo de carga

## Algoritmos y software

---

## Algoritmos de balanceo — estáticos

No tienen en cuenta el estado de los servidores: el reparto se decide **de antemano**.

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### Round Robin

Las peticiones se reparten **por turnos** entre los servidores. Requiere servidores equivalentes.

### Round Robin ponderado

Cada servidor tiene un **peso**: los más potentes reciben más peticiones.

</div>

<div class="card card-green">

### Sticky (afinidad)

Un mismo cliente va **siempre al mismo servidor** (por su IP o con una cookie).

### Hash

Una función *hash* sobre la IP, la URL… **determina** qué servidor atiende cada petición.

</div>

</div>

---

## Algoritmos de balanceo — dinámicos

Tienen en cuenta el **estado real** de los servidores en cada momento.

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### Conexiones mínimas

La nueva petición va al servidor con **menos conexiones activas**.

</div>

<div class="card card-green">

### Menor tiempo de respuesta

Se elige el servidor con el **tiempo de respuesta más bajo** según las últimas mediciones.

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Se adaptan mejor a peticiones de duración muy distinta, pero el balanceador tiene que <strong>medir</strong> el estado de cada servidor.</div>
</div>

---

## Software de balanceo de carga

| Software | Características |
|:--|:--|
| **HAProxy** | Especializado en balanceo, TCP y HTTP. *Health checks* activos y página de estadísticas |
| **Nginx** (`upstream`) | Servidor web que también balancea: bloque `upstream` + `proxy_pass` |
| **Apache** (`mod_proxy_balancer`) | `<Proxy "balancer://...">` con `BalancerMember` |
| **Traefik** | Proxy inverso para **contenedores** (Docker, Kubernetes): se configura solo a partir de etiquetas |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>También hay balanceadores <strong>hardware</strong> y los que ofrecen los proveedores de nube. En esta unidad usamos <strong>HAProxy</strong>.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">05</p>

# Balanceador de carga con HAProxy

## Distribución de tráfico entre backends

---

## ¿Qué es HAProxy?

> **HAProxy** (*High Availability Proxy*) es **software libre de alto rendimiento** especializado en **balanceo de carga** y **proxy inverso** para servicios TCP y HTTP.

- **Estabilidad** y rendimiento muy contrastados
- **Health checks** automáticos sobre los backends
- Varios **algoritmos** de balanceo
- Puede balancear **TCP genérico** o entender **HTTP** (cabeceras, cookies)
- Página de **estadísticas** para ver y operar el balanceador

---

## Escenario de laboratorio

| Máquina | Rol | Red externa | Red de datos |
|:--|:--|:--|:--|
| **balanceador** | HAProxy | `192.168.10.10` | `192.168.100.1` |
| **apache1** | Servidor web | `192.168.10.11` | `192.168.100.100` |
| **apache2** | Servidor web | `192.168.10.12` | `192.168.100.101` |
| **Tu equipo** | Cliente | `192.168.10.1` | — |

- El cliente accede a `www.example.org`, que apunta a la IP **externa del balanceador**
- HAProxy se conecta a los servidores web por la **red de datos**
- Cada servidor web muestra su nombre, para ver quién atiende cada petición

---

## Instalación

```bash
sudo apt install haproxy
systemctl status haproxy

# Validar la sintaxis antes de aplicar cambios
sudo haproxy -c -f /etc/haproxy/haproxy.cfg

# Aplicar cambios
sudo systemctl reload haproxy
```

Los mensajes de HAProxy van al journal: `journalctl -u haproxy`.

---

## Estructura del archivo de configuración

`/etc/haproxy/haproxy.cfg` se organiza en secciones:

| Sección | Para qué sirve |
|:--|:--|
| **`global`** | Parámetros del demonio (logs, usuario, límites) |
| **`defaults`** | Valores por defecto para `frontend`, `backend` y `listen` |
| **`frontend`** | Cómo se **reciben** las peticiones (IP, puerto) |
| **`backend`** | A qué **servidores** se reenvían y con qué algoritmo |
| **`listen`** | `frontend` + `backend` en un solo bloque |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El fichero que instala Debian ya trae <code>global</code> y <code>defaults</code>: solo hay que añadir el <code>frontend</code> y el <code>backend</code>.</div>
</div>

---

## Secciones `global` y `defaults`

```haproxy
global
    log /dev/log local0
    user haproxy
    group haproxy

defaults
    log     global
    mode    http
    option  httplog
    timeout connect 5s
    timeout client  50s
    timeout server  50s
```

- **`mode http`** — entiende HTTP (alternativa: `mode tcp` para TCP genérico)
- **`timeout`** — tiempos de espera de conexión, cliente y servidor

---

## Secciones `frontend` y `backend`

```haproxy
frontend servidores_web
    bind *:80
    default_backend servidores_web_backend

backend servidores_web_backend
    balance roundrobin
    option  httpchk GET /
    server  apache1 192.168.100.100:80 check
    server  apache2 192.168.100.101:80 check
```

- **`bind *:80`** — escucha en el puerto 80 · **`default_backend`** — adónde van las peticiones
- **`balance`** — algoritmo de balanceo · **`server ... check`** — con *health check* activo

---

## Algoritmos en HAProxy

| `balance` | Algoritmo | Tipo |
|:--|:--|:--|
| `roundrobin` | Round Robin (ponderado con `weight`) | Estático |
| `source` | Hash de la IP del cliente: afinidad por IP | Estático |
| `uri` | Hash de la URL: la misma URL, al mismo servidor | Estático |
| `leastconn` | Conexiones mínimas | Dinámico |

```haproxy
server apache1 192.168.100.100:80 check weight 2
server apache2 192.168.100.101:80 check weight 1
```

---

## Health checks

HAProxy pide `GET /` a cada servidor cada cierto tiempo (vale un 2xx o 3xx):

```haproxy
backend servidores_web_backend
    option httpchk GET /
    server apache1 192.168.100.100:80 check inter 2s fall 3 rise 2
    server apache2 192.168.100.101:80 check inter 2s fall 3 rise 2
```

| Parámetro | Significado |
|:--|:--|
| `inter` | Intervalo entre comprobaciones |
| `fall` | Fallos seguidos para marcarlo **DOWN** |
| `rise` | Aciertos seguidos para volver a marcarlo **UP** |

Si uno falla, **deja de enviarle tráfico**. Si caen todos: **503**.

---

## Acceso desde el cliente

En el `/etc/hosts` del cliente: `192.168.10.10   www.example.org`

Varias peticiones seguidas:

```bash
$ for i in 1 2 3; do curl -s http://www.example.org/ | grep Servidor; done
<h1>Servidor: apache1</h1>
<h1>Servidor: apache2</h1>
<h1>Servidor: apache1</h1>
```

Con `roundrobin`, las respuestas **alternan** entre los dos servidores.

---

## La IP del cliente en los servidores web

Para el servidor web, el cliente es el **balanceador**:

```
192.168.100.1 - - [03/Oct/2026:10:15:02 +0200] "GET / HTTP/1.1" 200 312 "-" "curl/8.14.1"
```

Con **`option forwardfor`**, HAProxy añade la cabecera `X-Forwarded-For` con la IP real del cliente:

```haproxy
backend servidores_web_backend
    option forwardfor
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>La cabecera llega al servidor web, pero el log <strong>no la registra</strong> hasta que se cambia su formato.</div>
</div>

---

## Registrar la IP real en el log

```apache
# Apache
LogFormat "%{X-Forwarded-For}i %l %u %t \"%r\" %>s %O" balanceado
CustomLog ${APACHE_LOG_DIR}/access.log balanceado
```

```nginx
# Nginx
log_format balanceado '$http_x_forwarded_for - [$time_local] "$request" $status';
access_log /var/log/nginx/access.log balanceado;
```

Resultado:

```
192.168.10.1 - - [03/Oct/2026:10:16:40 +0200] "GET / HTTP/1.1" 200 312
```

---

## Afinidad de sesión: el problema

Una **sesión** de PHP se guarda **en el servidor** (`/var/lib/php/sessions`); el navegador solo guarda la cookie con el identificador (`PHPSESSID`).

<div style="display:flex;align-items:center;justify-content:center;gap:0.8rem;margin:0.8rem 0">
<div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem 0.9rem;background:#f1f5f9;font-weight:600;text-align:center">Navegador<br><small>PHPSESSID=abc</small></div>
<div style="font-size:1.6rem;color:var(--teal-600)">→</div>
<div style="border:2px solid var(--teal-600);border-radius:8px;padding:0.4rem 0.9rem;background:var(--teal-50);font-weight:600;text-align:center">Balanceador</div>
<div style="font-size:1.6rem;color:var(--teal-600)">→</div>
<div style="display:flex;flex-direction:column;gap:0.4rem">
<div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem 0.9rem;background:#f1f5f9;font-weight:600;text-align:center">apache1<br><small>sesión abc: visitas = 3</small></div>
<div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem 0.9rem;background:#f1f5f9;font-weight:600;text-align:center">apache2<br><small>no conoce la sesión abc</small></div>
</div>
</div>

Con `roundrobin`, la siguiente petición va a `apache2`, que **no tiene** esa sesión: crea una nueva y el contador vuelve a empezar.

---

## Afinidad de sesión con una cookie

HAProxy añade su **propia cookie** con el servidor que ha atendido al cliente:

```haproxy
backend servidores_web_backend
    balance roundrobin
    cookie  SERVERID insert indirect nocache
    server  apache1 192.168.100.100:80 check cookie s1
    server  apache2 192.168.100.101:80 check cookie s2
```

- **`insert`** — HAProxy envía `Set-Cookie: SERVERID=s1`; el navegador la devuelve en cada petición y HAProxy lo manda a `apache1`
- **`indirect`** — la cookie no llega al servidor web · **`nocache`** — que no la guarden las cachés

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Alternativas: <code>balance source</code> (por IP) o sesiones <strong>compartidas</strong> entre servidores (base de datos, Redis).</div>
</div>

---

## Comprobar la afinidad

`curl` **no guarda cookies**: cada petición es un cliente nuevo y el contador siempre vale 1.

```bash
$ curl -sI http://www.example.org/sesion.php | grep -i set-cookie
set-cookie: PHPSESSID=k3j9q2...; path=/
set-cookie: SERVERID=s1; path=/
```

Para que `curl` guarde las cookies y las envíe en las siguientes peticiones:

```bash
$ curl -s -c cookies.txt -b cookies.txt http://www.example.org/sesion.php
```

Repetido varias veces: **siempre el mismo servidor** y el contador sube. El navegador lo hace solo.

---

## Página de estadísticas

Bloque `listen` propio, en otro puerto (`http://192.168.10.10:8404/stats`):

```haproxy
listen estadisticas
    bind *:8404
    stats enable
    stats uri   /stats
    stats auth  admin:asir
    stats admin if TRUE
```

- Estado de cada servidor (**UP** / **DOWN** / **MAINT**), sesiones y errores
- Con **`stats admin`**, poner un servidor en **mantenimiento** desde la web

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Siempre con autenticación y nunca accesible desde redes públicas.</div>
</div>

---

## Proxy inverso y balanceador juntos

<div style="display:flex;align-items:center;justify-content:center;gap:0.6rem;margin:0.8rem 0">
<div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem 0.8rem;background:#f1f5f9;font-weight:600;text-align:center">Cliente</div>
<div style="font-size:1.6rem;color:var(--teal-600)">→</div>
<div style="border:2px solid var(--teal-600);border-radius:8px;padding:0.4rem 0.8rem;background:var(--teal-50);font-weight:600;text-align:center">Proxy inverso<br><small>app.ejemplo.org</small></div>
<div style="font-size:1.6rem;color:var(--teal-600)">→</div>
<div style="border:2px solid var(--teal-600);border-radius:8px;padding:0.4rem 0.8rem;background:var(--teal-50);font-weight:600;text-align:center">HAProxy</div>
<div style="font-size:1.6rem;color:var(--teal-600)">→</div>
<div style="display:flex;flex-direction:column;gap:0.4rem">
<div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem 0.8rem;background:#f1f5f9;font-weight:600;text-align:center">backend1</div>
<div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem 0.8rem;background:#f1f5f9;font-weight:600;text-align:center">backend2</div>
</div>
</div>

- **Proxy inverso**: único punto de entrada; elige el destino por el **nombre** o la **ruta**
- **Balanceador**: reparte entre servidores **que sirven lo mismo** y retira los que fallan
- Para el proxy, HAProxy es **un backend más**; para HAProxy, el cliente es el proxy
- `X-Forwarded-For` **acumula** las IP: `X-Forwarded-For: <cliente>, <proxy>`

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">06</p>

# Resumen

## Problemas frecuentes

---

## Problemas frecuentes: proxy inverso

| Síntoma | Causas habituales |
|:--|:--|
| **502** Bad Gateway | El backend no responde: servicio parado, IP o puerto mal |
| **504** Gateway Timeout | El backend tarda más que el `timeout` del proxy |
| **Sale otro sitio** | El `Host` que recibe el backend no coincide con su `ServerName` |
| **Redirige a un nombre interno** | Falta `ProxyPassReverse` o hay `proxy_redirect off` |
| **Página sin estilos** en una subruta | Enlaces absolutos del backend · falta la barra final |
| **Nginx no arranca** | No resuelve el nombre de `proxy_pass` |

Probar sin tocar `/etc/hosts`: `curl -H "Host: www.app1.org" http://<ip del proxy>/`

Antes de recargar: `apache2ctl configtest` o `nginx -t`. Ante un error: los logs del **proxy** y del **backend**.

---

## Problemas frecuentes: balanceador

| Síntoma | Causas habituales |
|:--|:--|
| **503** Service Unavailable | Todos los servidores **DOWN**: servicios parados o el *health check* pide una página que no existe |
| En el log del servidor web sale la **IP del balanceador** | Falta `option forwardfor` o cambiar el formato del log |
| El contador de la sesión **vuelve a 1** | Sin afinidad · o se prueba con `curl` sin guardar cookies |
| Siempre responde **el mismo servidor** | El otro está DOWN · afinidad activa · `balance source` |

Antes de recargar: `haproxy -c -f /etc/haproxy/haproxy.cfg`. El estado de cada servidor, en la página de **estadísticas**.

---

## Para profundizar

- [Documentación oficial de HAProxy](https://docs.haproxy.org/)
- [Apache mod_proxy](https://httpd.apache.org/docs/current/mod/mod_proxy.html)
- [Nginx — proxy_pass](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Proxy inverso y balanceo

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI</span>
</div>
