---
marp: true
title: Servidores web Apache y Nginx
theme: profesional
paginate: true
header: 'SRI · Unidad 3 — Servidores web: Apache y Nginx'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Servidores web **Apache** y **Nginx**

## Instalación y configuración en Debian

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI · Servicios de Red e Internet</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Apache HTTP Server

## Instalación y estructura de configuración

---

## Instalación de Apache

```bash
sudo apt update
sudo apt install apache2
```

### Tras la instalación

- El servicio queda **levantado y habilitado** automáticamente
- Se gestiona con **`systemctl`** (`apache2.service`)
- Escucha por defecto en el puerto **80** (HTTP) y **443** (HTTPS)
- Sirve contenido desde `/var/www/html/` con la página de bienvenida de Debian

```bash
systemctl status apache2
systemctl reload apache2     # recargar configuración sin cortar conexiones
systemctl restart apache2    # reiniciar el servicio
```

---

## Estructura de `/etc/apache2/`

| Ruta | Para qué sirve |
|:--|:--|
| `apache2.conf` | Archivo principal de configuración global |
| `ports.conf` | Define los puertos en los que escucha el servidor |
| `sites-available/` | Configuraciones de virtual hosts **disponibles** |
| `sites-enabled/` | Virtual hosts **activos** (enlaces simbólicos) |
| `mods-available/` | Módulos disponibles para activar |
| `mods-enabled/` | Módulos **cargados** (enlaces simbólicos) |
| `conf-available/` | Fragmentos de configuración disponibles |
| `conf-enabled/` | Fragmentos **activos** (enlaces simbólicos) |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El patrón <code>*-available</code> + <code>*-enabled</code> permite mantener configuraciones <strong>preparadas pero apagadas</strong> y activarlas con un comando.</div>
</div>

---

## Comandos de gestión

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Sitios y configuraciones

```bash
a2ensite  ejemplo.conf   # activar virtual host
a2dissite ejemplo.conf   # desactivar virtual host
a2enconf  fragmento      # activar fragmento
a2disconf fragmento      # desactivar fragmento
```

</div>

<div class="card card-green">

### Módulos

```bash
a2enmod  rewrite         # activar módulo
a2dismod rewrite         # desactivar módulo
```

Tras cualquier cambio:

```bash
systemctl reload apache2
```

</div>

</div>

---

## Virtual Hosts en Apache

Cada sitio se define en un archivo de `sites-available/`:

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@ejemplo.com
    ServerName  ejemplo.com
    ServerAlias www.ejemplo.com
    DocumentRoot /var/www/ejemplo

    ErrorLog  ${APACHE_LOG_DIR}/ejemplo_error.log
    CustomLog ${APACHE_LOG_DIR}/ejemplo_access.log combined
</VirtualHost>
```

### Activación

```bash
sudo a2ensite ejemplo.conf
sudo systemctl reload apache2
```

---

## Directivas en bloques `<Directory>`

Permiten configurar **opciones por directorio** del sistema de ficheros.

```apache
<Directory /var/www/ejemplo>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

- **`Options`** — comportamiento (`Indexes` para listar, `FollowSymLinks` para enlaces…)
- **`AllowOverride`** — si se permiten archivos `.htaccess`
- **`Require`** — quién puede acceder (`all granted`, `ip 192.168.1`, `valid-user`…)

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Funcionalidades en Apache

## Alias, autenticación y redirecciones

---

## Alias y redirecciones

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### `Alias`

Asocia una URL a una **ruta del sistema**:

```apache
Alias "/imagenes" "/srv/recursos/img"

<Directory "/srv/recursos/img">
    Require all granted
</Directory>
```

</div>

<div class="card card-green">

### `Redirect`

Redirige peticiones a otra URL:

```apache
Redirect "/old" "/new"

Redirect permanent "/uno" \
    "http://www.pagina2.com/dos"
```

`permanent` envía un **301**; sin él, un **302**.

</div>

</div>

---

## Autenticación básica

Restringe el acceso a un directorio mediante usuario y contraseña.

```apache
<Directory "/var/www/miweb/privado">
    AuthType Basic
    AuthName "Zona restringida"
    AuthUserFile "/etc/apache2/claves/passwd.txt"
    Require valid-user
</Directory>
```

### Crear el archivo de contraseñas

```bash
# Primer usuario: crea el archivo (-c)
sudo htpasswd -c /etc/apache2/claves/passwd.txt usuario1

# Usuarios siguientes: sin -c (no sobreescribe)
sudo htpasswd /etc/apache2/claves/passwd.txt usuario2
```

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>La autenticación básica envía las credenciales en <strong>Base64</strong>: usar <strong>siempre</strong> sobre HTTPS.</div>
</div>

---

## Control de acceso

```apache
<Directory "/var/www/miweb/admin">
    # Por dirección IP
    Require ip 192.168.1
    Require ip 10.0.0.5

    # O por usuario autenticado
    # Require valid-user
</Directory>
```

### Combinaciones habituales

- **`Require all granted`** — acceso libre
- **`Require all denied`** — bloquear todo
- **`Require ip <red>`** — sólo desde una red
- **`Require valid-user`** — cualquier usuario autenticado

---

## Módulos habituales en Apache

| Módulo | Función |
|:--|:--|
| `mod_rewrite` | Reescritura de URLs y reglas avanzadas |
| `mod_proxy` | Proxy inverso y balanceo de carga |
| `mod_headers` | Manipulación de cabeceras HTTP |
| `mod_deflate` | Compresión de respuestas |
| `mod_expires` | Cabeceras de expiración para caché |
| `mod_ssl` | Soporte de HTTPS / TLS |

```bash
sudo a2enmod rewrite
sudo systemctl reload apache2
```

---

## Logs en Apache

Los registros se guardan por defecto en `/var/log/apache2/`.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### `access.log`

- Registra **todas las peticiones** atendidas
- Incluye IP cliente, fecha, método, URL, código de estado y tamaño

</div>

<div class="card card-green">

### `error.log`

- Errores del servidor y de configuración
- Problemas de permisos y de módulos
- Mensajes de inicio y parada del servicio

</div>

</div>

```bash
sudo tail -f /var/log/apache2/error.log
sudo journalctl -u apache2
```

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# Nginx

## Instalación y estructura de configuración

---

## Instalación de Nginx

```bash
sudo apt update
sudo apt install nginx
```

### Tras la instalación

- El servicio queda activado automáticamente (`nginx.service`)
- Escucha en los puertos **80** y **443**
- Sirve por defecto desde `/var/www/html/`

```bash
systemctl status nginx
systemctl reload nginx
systemctl restart nginx
nginx -t                  # comprobar la sintaxis antes de recargar
```

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Nginx y Apache no pueden escuchar en el mismo puerto a la vez: instalar los dos en una misma máquina requiere ajustar <code>ports.conf</code> o el bloque <code>listen</code>.</div>
</div>

---

## Estructura de `/etc/nginx/`

| Ruta | Para qué sirve |
|:--|:--|
| `nginx.conf` | Archivo principal de configuración |
| `conf.d/` | Fragmentos de configuración cargados por defecto |
| `sites-available/` | Configuraciones de virtual hosts disponibles |
| `sites-enabled/` | Sitios **activos** (enlaces simbólicos) |
| `snippets/` | Fragmentos **reutilizables** (incluibles desde otros archivos) |
| `/var/www/` | Directorio raíz por defecto del contenido |
| `/var/log/nginx/` | Logs de acceso y errores |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Nginx en Debian usa el mismo patrón <code>sites-available/</code> + <code>sites-enabled/</code> que Apache, pero <strong>no</strong> dispone de comandos tipo <code>a2ensite</code>: la activación se hace creando el enlace simbólico a mano.</div>
</div>

---

## Server Blocks (Virtual Hosts)

```nginx
server {
    listen 80;
    server_name ejemplo.com www.ejemplo.com;

    root  /var/www/ejemplo;
    index index.html index.htm;

    access_log /var/log/nginx/ejemplo_access.log;
    error_log  /var/log/nginx/ejemplo_error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Activación

```bash
sudo ln -s /etc/nginx/sites-available/ejemplo \
           /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

---

## Directivas principales

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Identificación del sitio

- **`listen`** — puerto y protocolo
- **`server_name`** — nombres atendidos por el bloque
- **`root`** — directorio raíz del contenido
- **`index`** — archivos por defecto

</div>

<div class="card card-green">

### Resolución de peticiones

- **`location`** — mapea URLs a recursos
- **`try_files`** — orden de búsqueda de archivos
- **`alias`** — ruta alternativa para una URL
- **`proxy_pass`** — reenvío al backend

</div>

</div>

---

## Servidor por defecto

Cuando una petición llega con un `Host` que no coincide con ningún `server_name`, Nginx la atiende con el bloque marcado como `default_server`:

```nginx
server {
    listen 80 default_server;
    server_name _;
    root /var/www/html;
}
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El nombre <code>_</code> es un convenio para indicar "cualquier nombre"; lo que de verdad importa es la opción <code>default_server</code> en <code>listen</code>.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# Funcionalidades en Nginx

## location, autenticación, redirecciones y snippets

---

## Bloques `location`

Mapean **URLs** a recursos del sistema. Pueden anidarse y combinar con otras directivas:

```nginx
location /imagenes/ {
    alias /srv/recursos/imagenes/;
    autoindex on;
}
```

- **`alias`** sustituye la parte coincidente de la URL por la ruta indicada
- **`autoindex on`** muestra el listado del directorio si no hay `index`

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Nginx <strong>sigue enlaces simbólicos</strong> por defecto siempre que el usuario del proceso tenga permisos sobre el destino.</div>
</div>

---

## Autenticación básica

```nginx
location /privado/ {
    auth_basic "Zona restringida";
    auth_basic_user_file /etc/nginx/.htpasswd;
}
```

Crear el archivo de contraseñas (requiere el paquete **`apache2-utils`**):

```bash
sudo apt install apache2-utils
sudo htpasswd -c /etc/nginx/.htpasswd usuario1
sudo htpasswd    /etc/nginx/.htpasswd usuario2
```

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>El formato del archivo es el mismo que en Apache: una herramienta sirve para los dos servidores.</div>
</div>

---

## Control de acceso por IP

```nginx
location /admin/ {
    allow 192.168.1.0/24;
    allow 10.0.0.5;
    deny  all;
}
```

### Combinar IP y autenticación

```nginx
location /panel/ {
    satisfy any;          # basta con cumplir UNA de las dos

    allow 192.168.1.0/24;
    deny  all;

    auth_basic           "Acceso restringido";
    auth_basic_user_file /etc/nginx/.htpasswd;
}
```

`satisfy all` (por defecto) exigiría cumplir ambas; `satisfy any` permite el acceso si se cumple **cualquiera**.

---

## Redirecciones

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### `return`

```nginx
# Permanente (301)
return 301 http://nuevo.ejemplo.com$request_uri;

# Temporal (302)
return 302 /aviso.html;
```

Es la forma **recomendada** y más eficiente de redirigir.

</div>

<div class="card card-green">

### `rewrite`

```nginx
rewrite ^/blog/(.*)$ /articulos/$1 permanent;
```

Permite **reescribir** URLs aplicando expresiones regulares. `permanent` genera un **301**.

</div>

</div>

---

## Snippets reutilizables

Fragmentos guardados en `/etc/nginx/snippets/` que pueden incluirse desde varios `server`:

```nginx
# /etc/nginx/snippets/seguridad.conf
add_header X-Frame-Options       "SAMEORIGIN";
add_header X-Content-Type-Options "nosniff";
```

Inclusión:

```nginx
server {
    listen 80;
    server_name ejemplo.com;
    include snippets/seguridad.conf;
    ...
}
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Idea equivalente al patrón <code>conf-available/</code> de Apache: configuración modular que se reutiliza en varios sitios.</div>
</div>

---

## Módulos habituales en Nginx

| Módulo | Función |
|:--|:--|
| `ngx_http_rewrite_module` | Reescritura de URLs y redirecciones |
| `ngx_http_proxy_module` | Proxy inverso y balanceo de carga |
| `ngx_http_headers_module` | Manipulación de cabeceras HTTP |
| `ngx_http_ssl_module` | Soporte HTTPS / TLS |
| `ngx_http_auth_basic_module` | Autenticación básica |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>A diferencia de Apache, los módulos en Nginx se eligen al <strong>compilar</strong>: en Debian vienen los más habituales ya incluidos en el paquete <code>nginx</code>.</div>
</div>

---

## Logs en Nginx

Los registros se guardan por defecto en `/var/log/nginx/`.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### `access.log`

- Todas las peticiones atendidas
- IP, método, URL, código, tamaño y tiempo de respuesta

</div>

<div class="card card-green">

### `error.log`

- Errores en la ejecución del servidor
- Errores de configuración
- Problemas con los backends (PHP-FPM, proxy_pass…)

</div>

</div>

```bash
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
sudo journalctl -u nginx
```

---

## Apache vs Nginx — resumen

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Apache

- Modelo basado en **procesos / hilos**
- Configuración por directorio con **`.htaccess`**
- Gran catálogo de **módulos cargables**
- Comandos `a2ensite` / `a2enmod`
- Ideal con **PHP** y configuraciones heterogéneas

</div>

<div class="card card-green">

### Nginx

- Modelo **asíncrono** orientado a eventos
- Configuración **centralizada**, sin `.htaccess`
- Mejor rendimiento sirviendo **estáticos**
- Activación de sitios **manual** con enlaces
- Excelente como **proxy inverso** y balanceador

</div>

</div>

---

## Para profundizar

- [Documentación oficial de Apache](https://httpd.apache.org/docs/)
- [Documentación oficial de Nginx](https://nginx.org/en/docs/)

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Apache y Nginx → Aplicaciones web dinámicas

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI</span>
</div>
