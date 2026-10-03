---
marp: true
title: Servidores web Apache y Nginx
theme: profesional
paginate: true
header: 'SRI · Unidad 2 — Servidores web: Apache y Nginx'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Servidores web **Apache** y **Nginx**

## Instalación y configuración

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI · Servicios de Red e Internet</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Servidores web

## Conceptos comunes a Apache y Nginx

---

## ¿Qué es un servidor web?

- Programa que **implementa el protocolo HTTP**
- Sirve **páginas estáticas** (HTML, CSS, JS, imágenes…)
- Sirve **páginas dinámicas** generadas por lenguajes como **PHP**, **Python**, **Java**…
- Suele apoyarse en un **servidor de aplicaciones** para ejecutar el código
- Implementa funcionalidades del protocolo: **virtual hosts, redirecciones, autenticación, control de acceso…**

---

## Virtual hosts

Un **virtual host** permite que un mismo servidor atienda **varios sitios web** desde una **única dirección IP**.

- Cada sitio tiene su propio **contenido**, **configuración** y **logs**
- El servidor elige el sitio por la cabecera **`Host`** de la petición (*virtual hosts basados en nombre*)
- Si el `Host` no coincide con ningún sitio (por ejemplo, al entrar por la IP), responde el **sitio por defecto**

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Sin la cabecera <code>Host</code> el servidor no sabría qué sitio servir: por eso es <strong>obligatoria</strong> en HTTP/1.1.</div>
</div>

---

## De la URL al fichero

Cada sitio tiene un **directorio raíz** (`DocumentRoot` en Apache, `root` en Nginx): la ruta de la URL se busca a partir de él. Con `www.sitio1.org` → `/var/www/sitio1` y `www.sitio2.org` → `/var/www/sitio2`:

| URL pedida | Fichero | Respuesta |
|:--|:--|:--|
| `http://www.sitio1.org/docs/a.html` | `/var/www/sitio1/docs/a.html` | **200** con el fichero |
| `http://www.sitio2.org/docs/a.html` | `/var/www/sitio2/docs/a.html` | Mismo servidor, **otro sitio** |
| `http://www.sitio1.org/` | `/var/www/sitio1/index.html` | **200** con el fichero índice |
| `http://www.sitio1.org/docs/` | `/var/www/sitio1/docs/index.html` | Si no hay índice: **listado** (si se permite) o **403** |
| `http://www.sitio1.org/docs` | Es un directorio | **301** a `/docs/` |
| `http://www.sitio1.org/info.php` | `/var/www/sitio1/info.php` | Se **ejecuta** y se devuelve el HTML generado |
| `http://www.sitio1.org/nada.html` | No existe | **404** |

---

## El usuario `www-data`

En Debian, Apache y Nginx atienden las peticiones con el usuario sin privilegios **`www-data`**:

- El proceso principal arranca como **`root`** (hace falta para abrir el puerto 80) y los procesos que atienden las peticiones son de **`www-data`**
- Si alguien aprovecha un fallo de la web, actúa como `www-data`, **no como `root`**
- Necesita **lectura** (`r`) en los ficheros y **paso** (`x`) en **todos** los directorios de la ruta; escritura solo si la aplicación guarda algo

```bash
ps aux | grep -E 'apache2|nginx'      # usuario de cada proceso
namei -l /home/usuario/doc/a.pdf      # permisos de cada directorio de la ruta
```

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>En Debian los directorios personales se crean con permisos <code>700</code>: lo que esté en <code>/home/usuario</code> da <strong>403</strong> hasta que <code>www-data</code> pueda atravesarlo.</div>
</div>

---

## Cómo probar los virtual hosts

Sin DNS, el cliente necesita **resolver los nombres** del sitio a la IP del servidor:

```
# /etc/hosts del cliente
10.0.0.1   www.sitio1.org www.sitio2.org
```

Con `curl` también se puede **indicar el `Host`** sin tocar la resolución:

```bash
curl http://www.sitio1.org                    # resuelve con /etc/hosts
curl -H "Host: www.sitio2.org" http://10.0.0.1 # mismo servidor, otro sitio
curl http://10.0.0.1                           # sitio por defecto
```

---

## Alias

Un **alias** asocia una URL a una **ruta del sistema de ficheros** distinta del directorio raíz del sitio, sin mover los ficheros.

`http://servidor/imagenes/logo.png` → `/srv/recursos/img/logo.png`

- Útil para publicar directorios que están **fuera del directorio raíz**
- El usuario del servidor web (`www-data`) necesita **permisos** sobre la ruta real

---

## Redirección y reescritura

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Redirección

- El servidor responde con un **3xx** y la cabecera **`Location`**
- El cliente hace una **nueva petición** a esa URL
- La URL **cambia** en el navegador
- **301** permanente · **302** temporal

</div>

<div class="card card-green">

### Reescritura

- El servidor cambia la ruta **internamente**
- Responde directamente con el recurso (**200**)
- La URL **no cambia** en el navegador
- Sirve para **URL amigables**: `/producto/5` → `/producto.php?id=5`

</div>

</div>

---

## Autenticación básica

Mecanismo para que el cliente se identifique con **usuario y contraseña** antes de obtener un recurso.

- Sin credenciales, el servidor responde **401** y el navegador pide usuario y contraseña
- El navegador las envía **en cada petición**, codificadas en **Base64**:

```
Authorization: Basic dXNlcjpwYXNz
$ echo dXNlcjpwYXNz | base64 -d
user:pass
```

- Existe también la autenticación *Digest*, que envía un *hash*, pero apenas se usa

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Usa <strong>siempre HTTPS</strong> con autenticación: Base64 es una <strong>codificación</strong>, no un cifrado. Cualquiera que capture el tráfico obtiene la contraseña. HTTPS cifra la conexión completa, cabeceras incluidas.</div>
</div>

---

## Control de acceso

Restringe quién puede acceder a determinados recursos o directorios. Puede combinar varios criterios:

- **Dirección IP** o red de origen
- **Nombre de usuario y contraseña**
- **Nombres de dominio** del cliente

Si se deniega el acceso, el servidor responde **403 Forbidden**.

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Los criterios se pueden <strong>combinar</strong>: que se cumplan todos o que baste con uno (por ejemplo, entrar sin contraseña desde la red interna y con contraseña desde fuera).</div>
</div>

---

## Logs

Los dos servidores guardan dos registros principales:

- **`access.log`** — una línea por **petición** atendida
- **`error.log`** — errores de configuración, permisos, backends…

```
10.0.0.2 - pepe [06/Oct/2026:10:15:32 +0200] "GET /secreto/ HTTP/1.1" 200 512 "-" "curl/8.14.1"
```

**IP del cliente** · usuario autenticado · fecha · **línea de petición** · **código de estado** · tamaño · *referer* (página desde la que se llegó) · *user agent* (cliente)

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Ante un error, mira el <strong>código de estado</strong> en el <code>access.log</code> y el motivo en el <code>error.log</code>.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Apache2

## Instalación, configuración y funcionalidades

---

## Apache HTTP Server

- Servidor web **HTTP de código abierto**, multiplataforma
- Desarrollado por la **Apache Software Foundation** (proyecto *httpd*)
- Surgió en 1995 a partir del servidor **NCSA HTTPd**
- **Servidor web más usado en Internet entre 1996 y 2020**
- Destaca por su **estabilidad**, **modularidad** y comunidad
- Integración sencilla con **PHP, Python, Perl, CGI**…
- Atiende las conexiones con **procesos e hilos**, según el **MPM** que se use
- Versión estable actual: **2.4**

---

## Módulos de multiprocesamiento (MPM)

El **MPM** decide cómo reparte Apache las conexiones entre procesos e hilos. Solo puede haber **uno** activo.

| MPM | Cómo funciona | Cuándo se usa |
|:--|:--|:--|
| **prefork** | Un **proceso** por conexión, sin hilos | Con módulos que no admiten hilos, como **`mod_php`**. Consume más memoria |
| **worker** | Varios procesos, cada uno con varios **hilos**; un hilo por conexión | Menos memoria que prefork |
| **event** | Como worker, pero las conexiones *keep-alive* en espera no ocupan un hilo | **Por defecto** en Debian. El más eficiente |

```bash
apache2ctl -V | grep MPM                      # MPM activo
sudo a2dismod mpm_event && sudo a2enmod mpm_prefork
```

---

## Instalación de Apache

```bash
sudo apt update
sudo apt install apache2
```

- El servicio queda **levantado y habilitado** (`apache2.service`)
- Escucha por defecto en el puerto **80**
- Sirve contenido desde `/var/www/html/`

```bash
systemctl reload apache2     # recargar configuración sin cortar conexiones
systemctl restart apache2    # reiniciar el servicio
```

---

<!-- _class: destacado -->

## Ejercicio rápido 1: Instalar Apache

<span class="badge badge-purple">≈ 5 minutos</span>

En una máquina virtual o una instancia de OpenStack (el **servidor**):

1. Instala `apache2` y comprueba que el servicio está activo.
2. Desde tu equipo: `curl -I http://<ip de acceso>`. ¿Qué dice la cabecera `Server`?
3. `ps aux | grep apache2`: ¿qué usuario tiene cada proceso? ¿Por qué?
4. ¿Qué MPM está usando? (`apache2ctl -V`)

---

## Estructura de `/etc/apache2/`

| Ruta | Para qué sirve |
|:--|:--|
| `apache2.conf` | Archivo principal de configuración global |
| `ports.conf` | Puertos en los que escucha el servidor |
| `sites-available/` · `sites-enabled/` | Virtual hosts **disponibles** · **activos** |
| `mods-available/` · `mods-enabled/` | Módulos disponibles · **cargados** |
| `conf-available/` · `conf-enabled/` | Fragmentos de configuración disponibles · **activos** |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Los directorios <code>*-enabled</code> contienen <strong>enlaces simbólicos</strong> a los <code>*-available</code>: la configuración se prepara y se activa con un comando.</div>
</div>

---

## Comandos de gestión

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Activar y desactivar

```bash
a2ensite  sitio1     # virtual host
a2dissite sitio1
a2enmod   rewrite    # módulo
a2dismod  rewrite
a2enconf  security   # fragmento
a2disconf security
```

</div>

<div class="card card-green">

### Comprobar antes de recargar

```bash
apache2ctl configtest   # sintaxis
apache2ctl -S           # virtual hosts
apache2ctl -M           # módulos cargados
systemctl reload apache2
```

`apache2ctl -S` muestra qué virtual hosts hay y **cuál es el de por defecto**; `apache2ctl -M`, si un módulo (`rewrite`, `proxy`…) está cargado.

</div>

</div>

---

<!-- _class: destacado -->

## Ejercicio rápido 2: Explorar la configuración

<span class="badge badge-purple">≈ 5 minutos</span>

1. `ls -l /etc/apache2/sites-enabled/`: ¿qué tipo de fichero hay y a dónde apunta?
2. `apache2ctl -S`: ¿cuál es el virtual host por defecto?
3. `apache2ctl -M | grep rewrite`: ¿está cargado? Actívalo y vuelve a comprobarlo.
4. Abre `ports.conf`: ¿en qué puerto escucha?

---

## Virtual hosts en Apache

Cada sitio se define en un fichero de `sites-available/`:

```apache
<VirtualHost *:80>
    ServerName  www.ejemplo.org
    ServerAlias ejemplo.org
    DocumentRoot /var/www/ejemplo
    ErrorLog  ${APACHE_LOG_DIR}/ejemplo_error.log
    CustomLog ${APACHE_LOG_DIR}/ejemplo_access.log combined
</VirtualHost>
```

- **`ServerName`** / **`ServerAlias`**: nombres del sitio · **`DocumentRoot`**: directorio raíz
- Logs propios en `${APACHE_LOG_DIR}` (`/var/log/apache2`), con el formato `combined`
- Se activa con `a2ensite ejemplo` y `systemctl reload apache2`

---

## ¿Qué virtual host responde?

Apache compara la cabecera `Host` con el **`ServerName`** y los **`ServerAlias`** de cada virtual host.

- Si coincide con alguno, responde ese sitio
- Si **no coincide con ninguno**, responde el **primero que se ha cargado**
- Los ficheros se cargan por **orden alfabético**: por eso el sitio por defecto de Debian se llama `000-default.conf`

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Si desactivas <code>000-default</code>, el sitio por defecto pasa a ser el primero de los tuyos por orden alfabético.</div>
</div>

---

<!-- _class: destacado -->

## Ejercicio rápido 3: Tu primer virtual host

<span class="badge badge-purple">≈ 5 minutos</span>

1. Crea `www.ejemplo.org` con `DocumentRoot /var/www/ejemplo` y un `index.html` propio. Actívalo.
2. En tu equipo, añade el nombre a `/etc/hosts` con la IP del servidor.
3. `curl http://www.ejemplo.org` y `curl http://<ip de acceso>`: ¿qué sitio responde en cada caso?
4. Desactiva `000-default` y repite: ¿qué ha cambiado y por qué?

---

## Cambiar el puerto

Útil para tener **varios servicios en una máquina** (por ejemplo, Apache y Nginx a la vez).

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Apache

```apache
# /etc/apache2/ports.conf
Listen 8080
```

```apache
<VirtualHost *:8080>
    ...
</VirtualHost>
```

</div>

<div class="card card-green">

### Nginx

```nginx
server {
    listen 8080;
    ...
}
```

</div>

</div>

Para probarlo, el puerto va en la URL: `curl http://www.ejemplo.org:8080`

---

## Directivas en bloques `<Directory>`

Configuran **opciones por directorio** del sistema de ficheros.

```apache
<Directory /var/www/ejemplo>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

- **`Options`** — `Indexes` lista el directorio si no hay `index.html`; `FollowSymLinks` sigue enlaces simbólicos
- **`AllowOverride`** — qué se puede cambiar con ficheros `.htaccess` (los vemos más adelante)
- **`Require`** — quién puede acceder


---

## Qué directorios puede servir Apache

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### `/etc/apache2/apache2.conf`

```apache
<Directory />
    Require all denied
</Directory>

<Directory /var/www/>
    Options Indexes FollowSymLinks
    Require all granted
</Directory>

#<Directory /srv/>
#    Options Indexes FollowSymLinks
#    Require all granted
#</Directory>
```

</div>

<div class="card card-green">

### Qué significa

- Se **deniega todo** el disco y solo se permite **`/var/www`** (y `/usr/share`)
- Un `DocumentRoot` o un alias **fuera** de ellos da **403**: necesita su `<Directory>` con `Require all granted`
- Para usar **`/srv`** como base de los sitios, basta con **descomentar** su bloque
- En `/var/www` hay `Indexes`: sin índice, **se lista** el directorio
- **Nginx** no tiene esta restricción: solo cuentan los permisos de `www-data`

</div>

</div>
---

<!-- _class: destacado -->

## Ejercicio rápido 4: De la URL al fichero

<span class="badge badge-purple">≈ 10 minutos</span>

En `www.ejemplo.org`, crea el directorio `docs/` con un fichero `a.html` y **sin** `index.html`.

1. Con `curl -I`, anota el código de `/docs/a.html`, `/docs` (sin barra), `/docs/` y `/nada.html`. Explica cada uno.
2. Desactiva el listado (`Options -Indexes` en un `<Directory>` del sitio): ¿qué devuelve ahora `/docs/`?
3. Cambia el `DocumentRoot` a `/srv/ejemplo`: ¿qué pasa? ¿Qué tienes que hacer para que funcione?

---

## Alias y redirecciones

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### `Alias`

```apache
Alias "/imagenes" "/srv/recursos/img"

<Directory "/srv/recursos/img">
    Require all granted
</Directory>
```

El directorio real necesita su bloque `<Directory>` con permiso de acceso.

</div>

<div class="card card-green">

### `Redirect`

```apache
Redirect "/antigua" "/nueva"

Redirect permanent "/uno" \
    "http://www.pagina2.com/dos"
```

`permanent` envía un **301**; sin él, un **302**.

</div>

</div>

---

## Redirigir solo la raíz

`Redirect` compara por **prefijo**: `Redirect "/"` afecta a **todas** las URL, también a la de destino.

```apache
# MAL: bucle, /principal también empieza por /
Redirect "/" "/principal/"

# BIEN: solo la raíz, con una expresión regular
RedirectMatch "^/$" "/principal/"
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div><code>RedirectMatch</code> usa expresiones regulares: <code>^</code> es el principio y <code>$</code> el final de la ruta.</div>
</div>

---

## Autenticación básica

```apache
<Directory "/var/www/ejemplo/privado">
    AuthType Basic
    AuthName "Zona restringida"
    AuthUserFile "/etc/apache2/.htpasswd"
    Require valid-user
</Directory>
```

### Crear el archivo de contraseñas

```bash
sudo htpasswd -c /etc/apache2/.htpasswd usuario1   # -c crea el fichero
sudo htpasswd    /etc/apache2/.htpasswd usuario2   # sin -c: añade
```

---

## Control de acceso con `Require`

```apache
<Directory "/var/www/ejemplo/admin">
    Require ip 192.168.1.0/24
</Directory>
```

| Directiva | Significado |
|:--|:--|
| `Require all granted` / `all denied` | Acceso libre / bloquear todo |
| `Require ip <red>` | Solo desde esa red o IP |
| `Require not ip <red>` | Todos menos esa red (dentro de `RequireAll`) |
| `Require valid-user` | Cualquier usuario autenticado |

---

## Combinar condiciones

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### `<RequireAny>`: basta con una

```apache
<RequireAny>
    Require ip 192.168.1.0/24
    Require valid-user
</RequireAny>
```

Desde la red indicada entra **sin contraseña**; desde fuera **se pide**.

</div>

<div class="card card-green">

### `<RequireAll>`: todas

```apache
<RequireAll>
    Require ip 192.168.1.0/24
    Require valid-user
</RequireAll>
```

Solo desde esa red **y** con usuario autenticado.

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Varios <code>Require</code> sueltos equivalen a <code>&lt;RequireAny&gt;</code>. Las directivas <code>AuthType</code>, <code>AuthName</code> y <code>AuthUserFile</code> van fuera del bloque.</div>
</div>

---

## Ficheros `.htaccess`

Permiten cambiar la configuración **de un directorio** sin tocar el virtual host ni recargar el servidor.

- Se colocan **en el propio directorio** del contenido
- Solo funcionan si **`AllowOverride`** lo permite en su `<Directory>`

| `AllowOverride` | Permite en `.htaccess` |
|:--|:--|
| `None` | Nada: el fichero se ignora (por defecto) |
| `AuthConfig` | Autenticación |
| `FileInfo` | Redirecciones y `mod_rewrite` |
| `All` | Todo |

---

## Reescritura con `mod_rewrite`

Se activa con `a2enmod rewrite` y reiniciando Apache.

```apache
RewriteEngine On
# /producto/5 → producto.php?id=5 (la URL no cambia)
RewriteRule "^producto/([0-9]+)$" "producto.php?id=$1" [L]
```

- Expresiones regulares: `^` principio · `$` final · `[0-9]+` uno o más dígitos · `.*` cualquier cosa · `( )` guarda lo que coincide en `$1`, `$2`…
- **`[L]`**: última regla, no se aplican las siguientes
- En un `.htaccess`, la ruta **no empieza por `/`**; en el virtual host, sí

---

## Redirecciones con `mod_rewrite`

Con la opción **`R`**, la regla deja de ser una reescritura y pasa a ser una **redirección**:

```apache
RewriteEngine On
# /antigua/... → /nueva/... (el cliente recibe un 301)
RewriteRule "^antigua/(.*)$" "/nueva/$1" [R=301,L]
```

- **`[R=301]`** redirección permanente · **`[R=302]`** temporal
- Para redirecciones sencillas basta con `Redirect` / `RedirectMatch`; `mod_rewrite` se usa cuando hacen falta **patrones o condiciones**

---

## PHP en Apache

```bash
sudo apt install libapache2-mod-php
```

- Instala PHP y el módulo **`mod_php`**, que se activa solo
- `mod_php` no admite hilos: Debian cambia el MPM a **prefork** automáticamente
- Los ficheros **`.php`** del sitio se ejecutan y se devuelve el HTML generado

```php
<?php
echo "<h1>Servidor: " . gethostname() . "</h1>";
?>
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Alternativa: <strong>PHP-FPM</strong> (<code>php-fpm</code>), un servidor de aplicaciones aparte. Es lo que se usa con Nginx.</div>
</div>

---

## Módulos y seguridad básica

| Módulo | Función |
|:--|:--|
| `mod_rewrite` | Reescritura de URLs |
| `mod_proxy` | Proxy inverso y balanceo de carga |
| `mod_headers` | Manipulación de cabeceras HTTP |

### Ocultar la versión del servidor

En `/etc/apache2/conf-available/security.conf`:

```apache
ServerTokens Prod      # cabecera Server: Apache (sin versión)
ServerSignature Off    # sin versión en las páginas de error
```

---

## Logs en Apache

Por defecto en `/var/log/apache2/` (`access.log`, `error.log`), o en los ficheros que indique cada virtual host.

```apache
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined
```

| Campo | Contenido |
|:--|:--|
| `%h` | IP del cliente |
| `%u` | Usuario autenticado |
| `%r` · `%>s` | Línea de petición · código de estado |
| `%{Cabecera}i` | Valor de una cabecera de la petición |

---

<!-- _class: destacado -->

## Ejercicio rápido 5: Leer los logs

<span class="badge badge-purple">≈ 5 minutos</span>

1. Deja abierto `tail -f` sobre el log de acceso de `www.ejemplo.org`.
2. Desde tu equipo, repite peticiones de los ejercicios anteriores: localiza en el log la IP, la URL y el código de cada una.
3. Haz una petición con `curl -A "soy-yo"` y búscala en el log.
4. Quita a `www-data` el permiso de lectura de un fichero (`chmod 600`) y pídelo: ¿qué código obtienes? ¿Qué dice el `error.log`?

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# Nginx

## Instalación, configuración y funcionalidades

---

## Nginx

- Servidor **web y proxy inverso** ligero, de **alto rendimiento**
- **Software libre** (BSD simplificada); existe versión comercial **Nginx Plus**
- Creado por **Igor Sysoev** en 2004 para el portal ruso *Rambler*
- Modelo **asíncrono orientado a eventos**: pocos procesos (*workers*) y cada uno atiende **miles de conexiones** a la vez, sin un proceso o hilo por conexión
- Usa **menos memoria** que Apache y es más rápido, sobre todo con **contenido estático**
- **Menos flexible** en configuraciones dinámicas (no hay `.htaccess`)

---

## Instalación de Nginx

```bash
sudo apt update
sudo apt install nginx
```

- El servicio queda activado (`nginx.service`), en el puerto **80**
- Sirve por defecto desde `/var/www/html/`

```bash
sudo nginx -t                  # comprobar la sintaxis antes de recargar
sudo systemctl reload nginx
```

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Nginx y Apache no pueden escuchar en el mismo puerto a la vez: en la misma máquina hay que cambiar el puerto de uno de los dos.</div>
</div>

---

<!-- _class: destacado -->

## Ejercicio rápido 6: Instalar Nginx

<span class="badge badge-purple">≈ 5 minutos</span>

1. Con Apache en marcha, instala `nginx`. ¿Arranca? Busca el motivo con `systemctl status nginx`.
2. Para y deshabilita Apache, y arranca Nginx.
3. `curl -I` desde tu equipo: ¿qué cabecera `Server` ves ahora?
4. `ps aux | grep nginx`: ¿qué usuario tienen el proceso principal y los *workers*?

---

## Estructura de `/etc/nginx/`

| Ruta | Para qué sirve |
|:--|:--|
| `nginx.conf` | Archivo principal de configuración |
| `conf.d/` | Fragmentos de configuración cargados por defecto |
| `sites-available/` · `sites-enabled/` | Virtual hosts disponibles · **activos** |
| `snippets/` | Fragmentos **reutilizables** (con `include`) |
| `/var/log/nginx/` | Logs de acceso y errores |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Mismo patrón <code>sites-available/</code> + <code>sites-enabled/</code> que Apache, pero <strong>sin</strong> comandos tipo <code>a2ensite</code>: el enlace simbólico se crea a mano.</div>
</div>

---

<!-- _class: destacado -->

## Ejercicio rápido 7: Explorar Nginx

<span class="badge badge-purple">≈ 5 minutos</span>

1. `ls -l /etc/nginx/sites-enabled/`: ¿qué sitio hay activo y a dónde apunta?
2. `sudo nginx -T | less`: muestra **toda** la configuración que se carga. Busca el `server` con `default_server`.
3. En `nginx.conf`, ¿con qué usuario se ejecutan los *workers* (`user`)? ¿Cuántos hay (`worker_processes`)? Compáralo con el número de CPU (`nproc`) y con `ps aux | grep nginx`.

---

## Server blocks (virtual hosts)

Cada sitio se define en un fichero de `sites-available/`:

```nginx
server {
    listen 80;
    server_name www.ejemplo.org ejemplo.org;
    root  /var/www/ejemplo;
    index index.html;
    access_log /var/log/nginx/ejemplo_access.log;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

- **`try_files`**: busca el fichero, luego el directorio y, si no existe ninguno, devuelve **404**
- Se activa con `ln -s /etc/nginx/sites-available/ejemplo /etc/nginx/sites-enabled/`, `nginx -t` y `systemctl reload nginx`

---

## Servidor por defecto

Si el `Host` no coincide con ningún `server_name`, responde el bloque marcado con **`default_server`** (si no hay ninguno, el **primero** que se carga):

```nginx
server {
    listen 80 default_server;
    server_name _;
    root /var/www/html;
}
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El nombre <code>_</code> es un convenio para "ningún nombre concreto"; lo que de verdad importa es <code>default_server</code> en <code>listen</code>. En Debian lo tiene el sitio <code>default</code>.</div>
</div>

---

<!-- _class: destacado -->

## Ejercicio rápido 8: Server blocks

<span class="badge badge-purple">≈ 5 minutos</span>

1. Crea en Nginx el sitio `www.ejemplo.org` con el mismo contenido (`/var/www/ejemplo`). Actívalo.
2. Repite las pruebas del ejercicio 4 (`/docs/a.html`, `/docs`, `/docs/`, `/nada.html`): ¿cambia algún código? ¿Por qué?
3. ¿Qué sitio responde al entrar por la IP?

---

## Bloques `location`

Asocian **URL** a una configuración. Nginx elige el bloque así:

| Tipo | Ejemplo | Cuándo se usa |
|:--|:--|:--|
| Exacta | `location = / { }` | Solo esa URL; tiene **prioridad** |
| Prefijo | `location /docs/ { }` | Se elige el prefijo **más largo** |
| Prefijo preferente | `location ^~ /img/ { }` | Como prefijo, pero **no** mira las regex |
| Expresión regular | `location ~ \.php$ { }` | La **primera** que coincide (`~*` sin distinguir mayúsculas) |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Orden: exacta → prefijo más largo (si es <code>^~</code>, se queda) → regex en orden → prefijo más largo.</div>
</div>

---

<!-- _class: destacado -->

## Ejercicio rápido 9: ¿Qué `location` responde?

<span class="badge badge-purple">≈ 10 minutos</span>

En el sitio `www.ejemplo.org`, sustituye la `location /` por estos bloques (`return 200 "texto"` responde directamente con ese texto):

```nginx
location = /      { return 200 "exacta /\n"; }
location /        { return 200 "prefijo /\n"; }
location /docs/   { return 200 "prefijo /docs/\n"; }
location ^~ /img/ { return 200 "preferente /img/\n"; }
location ~ \.png$ { return 200 "regex .png\n"; }
```

**Antes de probarlo**, apunta qué bloque crees que responde a `/`, `/index.html`, `/docs/a.html`, `/docs/a.png` e `/img/a.png`. Después compruébalo con `curl`.

---

## `root` y `alias`

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### `root`: añade la URL completa

```nginx
location /img/ {
    root /srv;
}
```

`/img/logo.png` → `/srv/img/logo.png`

</div>

<div class="card card-green">

### `alias`: sustituye el prefijo

```nginx
location /imagenes/ {
    alias /srv/recursos/img/;
    autoindex on;
}
```

`/imagenes/logo.png` → `/srv/recursos/img/logo.png`

</div>

</div>

- **`autoindex on`** muestra el listado del directorio si no hay `index`
- Con `alias`, pon la **barra final** en la `location` y en la ruta

---

## Autenticación básica

```nginx
location /privado/ {
    auth_basic           "Zona restringida";
    auth_basic_user_file /etc/nginx/.htpasswd;
}
```

El fichero de contraseñas se crea con **`htpasswd`** (paquete **`apache2-utils`**):

```bash
sudo apt install apache2-utils
sudo htpasswd -c /etc/nginx/.htpasswd usuario1
sudo htpasswd    /etc/nginx/.htpasswd usuario2
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El formato del fichero es el mismo que en Apache.</div>
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

- Las reglas se comprueban **en orden**: se aplica la **primera** que coincide
- Si se deniega, responde **403**

---

## Combinar IP y autenticación

```nginx
location /panel/ {
    satisfy any;          # basta con cumplir UNA de las dos

    allow 192.168.1.0/24;
    deny  all;

    auth_basic           "Acceso restringido";
    auth_basic_user_file /etc/nginx/.htpasswd;
}
```

- **`satisfy any`**: desde la red indicada entra **sin contraseña**; desde fuera **se pide**
- **`satisfy all`** (por defecto): exige las **dos** condiciones

---

## Redirecciones con `return`

```nginx
# Toda la web, permanente (301)
return 301 http://nuevo.ejemplo.org$request_uri;

# Solo la raíz: location exacta
location = / {
    return 302 /principal/;
}
```

- **`return`** es la forma **recomendada** de redirigir
- `$request_uri` conserva la ruta y la consulta originales
- Con `location = /` solo se redirige la raíz: el resto de URL no entra en ese bloque

---

## Reescritura con `rewrite`

```nginx
# Reescritura: /producto/5 → /producto.php?id=5 (la URL no cambia)
rewrite ^/producto/([0-9]+)$ /producto.php?id=$1 last;

# Redirección: el cliente recibe un 301
rewrite ^/noticias/(.*)$ /novedades/$1 permanent;
```

| Opción | Efecto |
|:--|:--|
| `last` | Reescritura interna; vuelve a buscar la `location` |
| `break` | Reescritura interna dentro de la misma `location` |
| `redirect` · `permanent` | Redirección **302** · **301** |

---

## Snippets y seguridad básica

Fragmentos en `/etc/nginx/snippets/` que se incluyen desde varios `server`:

```nginx
# /etc/nginx/snippets/privado.conf
auth_basic           "Zona restringida";
auth_basic_user_file /etc/nginx/.htpasswd;
```

```nginx
location /admin/   { include snippets/privado.conf; }
location /informes/ { include snippets/privado.conf; }
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Para ocultar la versión en la cabecera <code>Server</code>: <code>server_tokens off;</code> en <code>nginx.conf</code>.</div>
</div>

---

## PHP con PHP-FPM

Nginx no ejecuta PHP: pasa las peticiones de ficheros `.php` a **PHP-FPM**, un servidor de aplicaciones, con el protocolo **FastCGI**. Se comunican por un **socket**: un fichero especial del sistema.

```bash
sudo apt install php-fpm
```

```nginx
index index.php index.html;

location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/run/php/php8.4-fpm.sock;
}
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El nombre del socket depende de la versión de PHP: compruébalo con <code>ls /run/php/</code>.</div>
</div>

---

## Logs en Nginx

Por defecto en `/var/log/nginx/` (`access.log`, `error.log`), o en los que indique cada `server`.

- `access.log` usa el formato **`combined`**, el mismo que Apache
- `error.log` recoge errores de configuración, permisos y backends (PHP-FPM, `proxy_pass`…)

```nginx
log_format personalizado '$remote_addr "$request" $status "$http_user_agent"';
access_log /var/log/nginx/ejemplo_access.log personalizado;
```

`$http_<cabecera>` es el valor de una cabecera de la petición (en minúsculas y con `_`).

---

<!-- _class: destacado -->

## Ejercicio rápido 10: Apache y Nginx a la vez

<span class="badge badge-purple">≈ 5 minutos</span>

1. Cambia Nginx para que escuche en el puerto **8080** y vuelve a arrancar Apache en el 80.
2. Comprueba con `sudo ss -tlnp` qué proceso escucha en cada puerto.
3. Desde tu equipo, pide `http://www.ejemplo.org` y `http://www.ejemplo.org:8080`: ¿qué cabecera `Server` devuelve cada uno?
4. ¿En qué log aparece cada petición?

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# Resumen

## Equivalencias y problemas frecuentes

---

## Equivalencias: configuración

| | Apache | Nginx |
|:--|:--|:--|
| Sitio | `<VirtualHost *:80>` | `server { listen 80; }` |
| Nombre | `ServerName` / `ServerAlias` | `server_name` |
| Directorio raíz | `DocumentRoot` | `root` |
| Puerto | `Listen` en `ports.conf` + `<VirtualHost *:8080>` | `listen 8080` |
| Sitio por defecto | El primero cargado (`000-default`) | `listen 80 default_server` |
| Activar sitio | `a2ensite` | `ln -s` a `sites-enabled/` |
| Comprobar | `apache2ctl configtest` | `nginx -t` |
| Por directorio | `.htaccess` | — |

---

## Equivalencias: funcionalidades

| | Apache | Nginx |
|:--|:--|:--|
| Alias | `Alias` + `<Directory>` | `location` + `alias` |
| Listado | `Options Indexes` | `autoindex on` |
| Redirección | `Redirect` / `RedirectMatch` | `return 301` |
| Reescritura | `RewriteRule … [L]` | `rewrite … last` |
| Acceso por IP | `Require ip` | `allow` / `deny` |
| Autenticación | `AuthType Basic` + `Require valid-user` | `auth_basic` |
| IP **o** usuario | `<RequireAny>` | `satisfy any` |
| PHP | `mod_php` | PHP-FPM |

---

## Problemas frecuentes

| Síntoma | Causas habituales |
|:--|:--|
| **403** | Permisos de `www-data` · control de acceso · sin índice ni listado |
| **404** | `DocumentRoot`, `root` o `alias` mal · el fichero no está |
| **Sale otro sitio** | El nombre no resuelve · no coincide con `ServerName` / `server_name` · sitio sin activar o sin recargar |
| **500** | Error de configuración (`.htaccess`, módulo sin activar) o de la aplicación |
| **502** | Nginx no llega a PHP-FPM (socket mal o servicio parado) |
| **No arranca** | Error de sintaxis · puerto ocupado |

Antes de recargar: `apache2ctl configtest` o `nginx -t`. Ante un error: el **`error.log`**.

---

## Para profundizar

- [Documentación oficial de Apache](https://httpd.apache.org/docs/2.4/)
- [Documentación oficial de Nginx](https://nginx.org/en/docs/)

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Servidores web

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI</span>
</div>
