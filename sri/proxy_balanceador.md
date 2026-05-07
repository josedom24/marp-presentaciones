---
marp: true
title: Proxy inverso y balanceador de carga
theme: profesional
paginate: true
header: 'SRI · Unidad 3 — Proxy inverso y balanceador de carga'
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

## Concepto y casos de uso

---

## ¿Qué es un proxy inverso?

> Un **proxy inverso** es un servidor que **acepta peticiones de los clientes**, las **reenvía** a uno o varios servidores internos y devuelve sus respuestas como si las hubiera generado él mismo.

### Idea fundamental

- El cliente **no accede directamente** a los servidores backend
- El proxy es la **única cara visible** del servicio en Internet
- Los servidores internos pueden estar en una **red privada** sin acceso desde fuera
- Permite **centralizar** funcionalidades comunes (TLS, autenticación, caché…)

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Un <strong>proxy directo</strong> protege al cliente; un <strong>proxy inverso</strong> protege al servidor.</div>
</div>

---

## Casos de uso

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Protección y centralización

- **Ocultar** la infraestructura backend
- Consolidar varias aplicaciones internas bajo un **único dominio público**
- Aplicar **filtros** y **autenticación** centralizada
- Centralizar la gestión de **cabeceras** de seguridad

</div>

<div class="card card-green">

### Rendimiento y operación

- **Distribuir carga** entre varios backends
- **Terminación TLS**: HTTPS sólo en el proxy
- **Reescritura** de URLs y rutas internas
- **Caché** de respuestas estáticas

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
sudo systemctl reload apache2
```

### Otros módulos relacionados

- **`mod_headers`** — manipular cabeceras de la petición y la respuesta
- **`mod_proxy_balancer`** — balanceo de carga
- **`mod_ssl`** — terminación HTTPS en el propio proxy
- **`mod_proxy_http2`** — proxy a backends que hablan HTTP/2

---

## Configuración básica

VirtualHost que reenvía **todas** las peticiones a un backend interno:

```apache
<VirtualHost *:80>
    ServerName proxy.example.org

    ProxyPass        "/" "http://interno.example.org/"
    ProxyPassReverse "/" "http://interno.example.org/"

    ErrorLog  ${APACHE_LOG_DIR}/proxy_error.log
    CustomLog ${APACHE_LOG_DIR}/proxy_access.log combined
</VirtualHost>
```

- **`ProxyPass`** — reenvía las peticiones entrantes al backend
- **`ProxyPassReverse`** — reescribe en la respuesta cabeceras como `Location` para que apunten a la URL pública

---

## El problema de las redirecciones

Cuando el backend responde con una redirección (por ejemplo a `/login`), suele incluir su **URL interna** en la cabecera `Location`:

```
Location: http://interno.example.org/login
```

Si esa URL llega tal cual al cliente, **no podrá resolverla** desde Internet.

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div><code>ProxyPassReverse</code> reescribe automáticamente esa cabecera para que apunte al dominio público. Es <strong>imprescindible</strong> en cualquier proxy inverso.</div>
</div>

---

## Proxy de subrutas

Permite **publicar varias aplicaciones** internas bajo un mismo dominio público:

```apache
ProxyPass        "/web/" "http://interno.example.org/"
ProxyPassReverse "/web/" "http://interno.example.org/"

ProxyPass        "/api/" "http://api.example.org/"
ProxyPassReverse "/api/" "http://api.example.org/"
```

- `/web/` y `/api/` se redirigen a backends distintos
- El cliente sólo ve **un dominio**: `proxy.example.org`
- Cada subruta puede tener sus propias cabeceras y reglas

---

## Cabeceras hacia el backend

`mod_headers` permite que el backend conozca **datos del cliente original**:

```apache
ProxyPreserveHost On

RequestHeader set X-Real-IP         "%{REMOTE_ADDR}s"
RequestHeader add X-Forwarded-For   "%{REMOTE_ADDR}s"
RequestHeader set X-Forwarded-Proto "http"
RequestHeader set X-Forwarded-Host  "%{HTTP_HOST}s"
```

| Cabecera | Significado |
|:--|:--|
| `Host` | Dominio que pidió el cliente (con `ProxyPreserveHost On`) |
| `X-Real-IP` | IP real del cliente |
| `X-Forwarded-For` | Cadena de proxies intermedios |
| `X-Forwarded-Proto` | Protocolo del cliente (`http` / `https`) |

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# Proxy inverso con Nginx

## proxy_pass y proxy_params

---

## Configuración básica

Nginx incluye soporte de proxy inverso de forma **nativa**, sin módulos adicionales.

```nginx
server {
    listen 80;
    server_name proxy.example.org;

    location / {
        proxy_pass http://interno.example.org/;
        include    proxy_params;
    }

    access_log /var/log/nginx/proxy_access.log;
    error_log  /var/log/nginx/proxy_error.log;
}
```

- **`proxy_pass`** — destino al que reenviar la petición
- **`include proxy_params`** — fragmento estándar de Debian con cabeceras útiles

---

## El archivo `proxy_params`

`/etc/nginx/proxy_params` reenvía al backend la información del cliente original:

```nginx
proxy_set_header Host              $http_host;
proxy_set_header X-Real-IP         $remote_addr;
proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

| Variable | Contenido |
|:--|:--|
| `$http_host` | Dominio que pidió el cliente |
| `$remote_addr` | IP del cliente |
| `$proxy_add_x_forwarded_for` | Acumula la cadena de proxies |
| `$scheme` | Protocolo usado (`http` / `https`) |

---

## Reescritura de redirecciones

Nginx no reescribe automáticamente las cabeceras `Location`. Hay que indicarlo con **`proxy_redirect`**:

```nginx
location / {
    proxy_pass     http://interno.example.org/;
    proxy_redirect http://interno.example.org/ /;
    include        proxy_params;
}
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Equivale al <code>ProxyPassReverse</code> de Apache: reescribe las cabeceras <code>Location</code> y <code>Refresh</code> para que apunten al dominio público.</div>
</div>

---

## Proxy de subrutas

```nginx
server {
    listen 80;
    server_name proxy.example.org;

    location /web/ {
        proxy_pass http://interno.example.org/;
        include    proxy_params;
    }

    location /api/ {
        proxy_pass http://api.example.org/;
        include    proxy_params;
    }
}
```

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Cuidado con la <strong>barra final</strong> en <code>proxy_pass</code>: con <code>/</code> al final se elimina el prefijo coincidente; sin barra, se reenvía la URL completa.</div>
</div>

---

## Apache vs Nginx — proxy inverso

| Aspecto | Apache | Nginx |
|:--|:--|:--|
| Módulos requeridos | `mod_proxy`, `mod_proxy_http` | Nativo |
| Reenvío | `ProxyPass` | `proxy_pass` |
| Reescritura de redirecciones | `ProxyPassReverse` | `proxy_redirect` |
| Cabeceras estándar | `mod_headers` (manual) | `proxy_params` (predefinido) |
| Conservar `Host` | `ProxyPreserveHost On` | `proxy_set_header Host $http_host` |

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# Balanceador de carga con HAProxy

## Distribución de tráfico entre backends

---

## ¿Qué es HAProxy?

> **HAProxy** (*High Availability Proxy*) es **software libre de alto rendimiento** especializado en **balanceo de carga** y **proxy inverso** para servicios basados en TCP y HTTP(S).

### Por qué se usa tanto en producción

- **Estabilidad** y rendimiento muy contrastados
- **Capacidades de monitorización** muy completas
- **Health checks** automáticos sobre los backends
- Soporta múltiples **algoritmos** de balanceo
- Puede balancear **TCP genérico** o entender **HTTP** y enrutar por cabeceras

---

## Escenario de laboratorio

| Máquina | Rol | Servicio | IP |
|:--|:--|:--|:--|
| **frontend** | Balanceador | HAProxy | `10.10.10.10` |
| **backend1** | Servidor web | Apache / Nginx | `10.10.10.11` |
| **backend2** | Servidor web | Apache / Nginx | `10.10.10.12` |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El cliente accede a <code>www.example.org</code> (la IP del frontend); HAProxy reparte las peticiones entre los dos backends de forma transparente.</div>
</div>

---

## Instalación

Instalación estándar en Debian 13:

```bash
sudo apt update
sudo apt install haproxy
```

### Comprobaciones

```bash
systemctl status haproxy

# Validar la sintaxis antes de aplicar cambios
sudo haproxy -c -f /etc/haproxy/haproxy.cfg

# Aplicar cambios
sudo systemctl restart haproxy
```

---

## Estructura del archivo de configuración

`/etc/haproxy/haproxy.cfg` se organiza en secciones:

| Sección | Para qué sirve |
|:--|:--|
| **`global`** | Parámetros del demonio (logs, usuario, límites, socket admin) |
| **`defaults`** | Valores por defecto para `frontend` y `backend` |
| **`frontend`** | Cómo se **reciben** las peticiones (puertos, ACLs) |
| **`backend`** | A qué **servidores** se reenvían y con qué algoritmo |
| **`listen`** | Combina `frontend` + `backend` en un solo bloque (más simple) |

---

## Sección `global` y `defaults`

```haproxy
global
    log /dev/log local0
    user haproxy
    group haproxy
    maxconn 2048
    stats socket /run/haproxy/admin.sock mode 660 level admin

defaults
    log     global
    mode    http
    option  httplog
    timeout connect 5s
    timeout client  30s
    timeout server  30s
    retries 3
    maxconn 1024
```

- **`mode http`** — entiende HTTP (alternativa: `mode tcp` para TCP genérico)
- **`timeout`** — tiempos de espera de conexión, cliente y servidor

---

## Sección `frontend`

Define **cómo se reciben** las peticiones:

```haproxy
frontend servidores_web
    bind *:80
    default_backend servidores_web_backend

    # Página de estadísticas
    stats enable
    stats uri    /ha_stats
    stats refresh 10s
    stats auth   admin:admin123
```

- **`bind *:80`** — escucha en todas las interfaces, puerto 80
- **`default_backend`** — backend al que se mandan las peticiones por defecto
- **`stats`** — habilita la página de monitorización en `/ha_stats`

---

## Sección `backend`

Define **a qué servidores** se reparten las peticiones:

```haproxy
backend servidores_web_backend
    balance roundrobin

    option      httpchk GET /
    http-check  expect status 200

    server backend1 10.10.10.11:80 check
    server backend2 10.10.10.12:80 check
```

- **`balance`** — algoritmo de balanceo
- **`option httpchk`** + **`http-check expect`** — comprueba periódicamente que el backend responde con 200
- **`server ... check`** — servidor con *health check* activo

---

## Algoritmos de balanceo

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Estáticos

- **`roundrobin`** — alterna las peticiones entre los servidores
- **`static-rr`** — round-robin sin tener en cuenta los pesos dinámicos
- **`source`** — el mismo cliente (IP) va siempre al mismo servidor (afinidad)
- **`uri`** — distribuye según un *hash* de la URI

</div>

<div class="card card-green">

### Dinámicos

- **`leastconn`** — al servidor con **menos conexiones** activas
- **`first`** — al primero disponible (escalado bajo demanda)
- **`random`** — elección aleatoria con distribución uniforme

</div>

</div>

---

## Health checks

HAProxy comprueba periódicamente que cada backend está **operativo** y deja de enviarle tráfico si falla.

```haproxy
backend servidores_web_backend
    option     httpchk GET /healthz
    http-check expect status 200

    server backend1 10.10.10.11:80 check inter 2s fall 3 rise 2
    server backend2 10.10.10.12:80 check inter 2s fall 3 rise 2
```

| Parámetro | Significado |
|:--|:--|
| `inter` | Intervalo entre comprobaciones |
| `fall` | Comprobaciones fallidas seguidas para marcarlo **caído** |
| `rise` | Comprobaciones correctas seguidas para volver a marcarlo **activo** |

---

## Página de estadísticas

Una vez aplicada la configuración, basta acceder a:

```
http://www.example.org/ha_stats
```

con las credenciales `admin` / `admin123`.

### Información que muestra

- Estado de cada **backend** (UP / DOWN / MAINT)
- **Conexiones activas** y totales por servidor
- Tiempos de respuesta y métricas de error
- Permite **deshabilitar manualmente** un nodo (mantenimiento)

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>La página de estadísticas <strong>nunca</strong> debe quedar accesible sin autenticación ni en redes públicas: muestra detalles internos del clúster.</div>
</div>

---

## Acceso desde el cliente

Para probar el balanceo, añadir la resolución estática en `/etc/hosts`:

```
10.10.10.10   www.example.org
```

A continuación, recargar varias veces:

```
http://www.example.org/
```

Las respuestas alternarán entre `backend1` y `backend2` (con `roundrobin`).

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Si los backends sirven páginas distintas (con su nombre), se ve a simple vista cómo HAProxy reparte las peticiones.</div>
</div>

---

## Monitorización con HATop

**HATop** es una herramienta de **terminal** que muestra en tiempo real el estado de HAProxy.

```bash
sudo apt install hatop
sudo hatop -s /run/haproxy/admin.sock
```

### Atajos útiles

- **F9** — habilitar un servidor
- **F10** — deshabilitar un servidor (mantenimiento)
- Navegación entre frontends y backends con cursores

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Útil para operar el balanceador desde la consola sin abrir un navegador en la red interna.</div>
</div>

---

## Para profundizar

- [Documentación oficial de HAProxy](https://docs.haproxy.org/)
- [Apache mod_proxy](https://httpd.apache.org/docs/current/mod/mod_proxy.html)
- [Nginx — proxy_pass](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Proxy inverso y balanceo → Alta disponibilidad

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI</span>
</div>
