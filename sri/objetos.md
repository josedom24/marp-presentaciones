---
marp: true
title: Almacenamiento de objetos
theme: profesional
paginate: true
header: 'SRI · Unidad 2 — Almacenamiento de objetos'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Almacenamiento de **objetos**

## Datos accesibles por HTTP a través de una API REST

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI · Servicios de Red e Internet</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# ¿Por qué otro tipo de almacenamiento?

## El problema que resuelve

---

## Los datos de hoy

Las aplicaciones actuales generan y sirven enormes cantidades de **datos no estructurados**:

- **Fotos y vídeos** que suben los usuarios de una red social o una tienda *online*
- **Copias de seguridad** de servidores y equipos
- **Logs**, métricas y ficheros de auditoría
- **Documentos**, adjuntos, paquetes de *software*, imágenes de contenedores…

Y además:

- Hay que acceder a ellos **desde cualquier sitio**: navegadores, móviles, otras aplicaciones
- El volumen **no cabe en un servidor** y no para de crecer
- No se pueden **perder**, aunque falle un disco o un servidor entero

---

## ¿Nos sirven NFS o iSCSI?

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### Lo que ofrecen

- Un sistema operativo **monta** el recurso
- Se trabaja con **ficheros** (NFS) o **bloques** (iSCSI)
- Pensados para una **red local** y unos pocos clientes
- Un servidor (o una cabina) con una **capacidad limitada**

</div>

<div class="card card-red">

### Lo que necesitamos

- Que el cliente sea un **navegador** o una **aplicación**, no un sistema que monta discos
- Acceso **por Internet**, atravesando cortafuegos y *proxies*
- **Millones** de clientes a la vez
- Crecer **añadiendo servidores**, sin límite práctico

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El almacenamiento de objetos no sustituye a NAS ni a SAN: resuelve <strong>otro problema</strong>.</div>
</div>

---

## La idea en una frase

<div class="alerta alerta-ok" style="margin-top:1.2rem; font-size:1.15rem">
<span>💡</span><div>El almacenamiento de objetos es almacenamiento <strong>al que se accede como a una web</strong>: no se monta, se le hacen <strong>peticiones HTTP</strong> a través de una <strong>API</strong>.</div>
</div>

<div style="margin-top:1.2rem">

- Cada dato guardado tiene su propia **URL**
- Para guardarlo, leerlo o borrarlo se usan los **métodos HTTP** que ya conocemos
- Detrás de esa URL puede haber **cientos de servidores**: el cliente no lo sabe ni le importa

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Conceptos

## Objetos, contenedores y metadatos

---

## ¿Qué es un objeto?

<div style="display:flex;align-items:stretch;justify-content:center;gap:0.8rem;margin:1rem 0">
<div style="border:2px solid var(--teal-600);border-radius:8px;padding:0.6rem 1rem;background:var(--teal-50);text-align:center;width:14rem"><strong>Identificador</strong><br><small>el nombre (o <em>clave</em>) del objeto<br><code>fotos/playa.jpg</code></small></div>
<div style="border:2px solid #64748b;border-radius:8px;padding:0.6rem 1rem;background:#f1f5f9;text-align:center;width:14rem"><strong>Datos</strong><br><small>el contenido: una imagen, un vídeo, un <em>backup</em>…</small></div>
<div style="border:2px solid var(--teal-600);border-radius:8px;padding:0.6rem 1rem;background:var(--teal-50);text-align:center;width:14rem"><strong>Metadatos</strong><br><small>información sobre los datos<br>tipo, tamaño, fecha, <em>hash</em>, propios…</small></div>
</div>

- Un **objeto** es una unidad de información: sus datos más sus metadatos, con un identificador único
- Los metadatos los pone el sistema (tamaño, fecha, `Content-Type`…) y también **el usuario** (`autor`, `proyecto`…)
- Un objeto **no se modifica por partes**: se sube entero y, si cambia, se **sustituye** entero

---

## Contenedores

> Los objetos se guardan en **contenedores** (en S3 se llaman ***buckets***).

- Un contenedor agrupa objetos, como un directorio… pero **no se pueden anidar**
- Dentro de un contenedor el espacio de nombres es **plano**: no hay carpetas de verdad
- Un nombre como `fotos/2026/playa.jpg` es **un único nombre** que contiene barras
- Las interfaces muestran «carpetas» agrupando por ese **prefijo**, pero es solo una forma de verlo
- Los **permisos** se suelen dar a nivel de contenedor

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>En OpenStack Swift la jerarquía es <strong>cuenta → contenedor → objeto</strong>. La cuenta corresponde a un <strong>proyecto</strong> de OpenStack.</div>
</div>

---

## Ficheros, bloques y objetos

| | **Bloques** (SAN) | **Ficheros** (NAS) | **Objetos** |
|:--|:--|:--|:--|
| El cliente ve… | Un disco | Carpetas y ficheros | Contenedores y objetos |
| ¿Quién es el cliente? | Un sistema operativo | Un sistema operativo | Una aplicación, un navegador |
| Cómo se accede | iSCSI, FC | NFS, SMB | **HTTP** (API REST) |
| Organización | Bloques | Árbol de directorios | Espacio **plano** + metadatos |
| Modificar | Cualquier bloque | Cualquier parte del fichero | Objeto **entero** |
| Escala | Una cabina | Un servidor | Muchos servidores |

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# HTTP y la API REST

## El corazón del almacenamiento de objetos

---

## ¿Por qué HTTP?

- **Todos lo hablan**: navegadores, `curl`, cualquier lenguaje de programación, cualquier móvil
- **Atraviesa la red**: pasa por cortafuegos, *proxies* y balanceadores sin configuración especial
- **Cada objeto es una URL**: se puede enlazar desde una página web o compartir
- Los **metadatos** viajan en las **cabeceras** (`Content-Type`, `Content-Length`, `ETag`…)
- Los **códigos de estado** indican el resultado de cada operación
- Con **HTTPS**, la comunicación va cifrada

<div class="alerta alerta-ok" style="margin-top:0.6rem">
<span>✅</span><div>No hay que inventar un protocolo nuevo ni instalar nada especial en el cliente: <strong>HTTP ya lo tiene todo</strong>.</div>
</div>

---

## HTTP es *stateless*: por eso escala

<div style="display:flex;align-items:center;justify-content:center;gap:0.8rem;margin:0.8rem 0">
<div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem 0.9rem;background:#f1f5f9;font-weight:600;text-align:center;width:9rem">Clientes</div>
<div style="color:var(--teal-600);font-weight:600">⟶</div>
<div style="border:2px solid var(--teal-600);border-radius:8px;padding:0.4rem 0.9rem;background:var(--teal-50);font-weight:600;text-align:center;width:11rem">Balanceador<br><small>/ proxy</small></div>
<div style="color:var(--teal-600);font-weight:600">⟶</div>
<div style="display:flex;flex-direction:column;gap:0.3rem">
<div style="border:2px solid #64748b;border-radius:8px;padding:0.2rem 0.9rem;background:#f1f5f9;text-align:center;width:11rem"><small>servidor de almacenamiento 1</small></div>
<div style="border:2px solid #64748b;border-radius:8px;padding:0.2rem 0.9rem;background:#f1f5f9;text-align:center;width:11rem"><small>servidor de almacenamiento 2</small></div>
<div style="border:2px solid #64748b;border-radius:8px;padding:0.2rem 0.9rem;background:#f1f5f9;text-align:center;width:11rem"><small>… servidor N</small></div>
</div>
</div>

- Cada petición lleva **todo lo necesario** (qué objeto, qué operación, quién la pide)
- Cualquier servidor puede atender cualquier petición: basta con **añadir servidores**
- Cada objeto se guarda **varias veces** en servidores distintos: si uno falla, no se pierde nada
- Es la misma idea del **balanceador de carga** que vimos con los servidores web

---

## ¿Qué es una API?

> Una **API** (*Application Programming Interface*) es la forma que ofrece un servicio para que **otros programas** lo usen.

- Define **qué operaciones** se pueden hacer y **cómo pedirlas**
- La aplicación no necesita saber cómo funciona el servicio por dentro
- En el almacenamiento de objetos, la API es **la única forma de entrar**: no hay otra

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Una <strong>API web</strong> es una API a la que se accede con peticiones HTTP. Las respuestas no son páginas para personas, sino datos para programas (texto, JSON, cabeceras).</div>
</div>

---

## API REST

**REST** es una forma de diseñar APIs web aprovechando HTTP tal cual:

- Todo lo que se gestiona es un **recurso**: una cuenta, un contenedor, un objeto
- Cada recurso se identifica con una **URL**
- La operación la indica el **método HTTP**
- El resultado lo indica el **código de estado**
- Cada petición es **independiente**: no hay sesión en el servidor (*stateless*)

```text
https://openstack.gonzalonazareno.org/swift/v1/AUTH_<proyecto>/fotos/playa.jpg
└──────────────── servicio ────────────────┘└─ cuenta ─┘└contenedor┘└─objeto─┘
```

---

## Métodos HTTP = operaciones

| Método | Sobre un contenedor | Sobre un objeto |
|:--|:--|:--|
| `PUT` | Crearlo | **Subirlo** (o sustituirlo) |
| `GET` | **Listar** sus objetos | **Descargarlo** |
| `HEAD` | Ver sus metadatos | Ver sus **metadatos** sin descargarlo |
| `POST` | Cambiar metadatos y permisos | Cambiar sus metadatos |
| `DELETE` | Borrarlo (si está vacío) | **Borrarlo** |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Son los métodos que vimos en el tema de HTTP. Aquí <code>PUT</code> y <code>DELETE</code>, que casi no se usan en las webs, son <strong>imprescindibles</strong>.</div>
</div>

---

## Códigos de estado = resultado

| Código | Significado en el almacenamiento de objetos |
|:--|:--|
| `200 OK` | Descarga o listado correctos |
| `201 Created` | Objeto o contenedor creado |
| `202 Accepted` | Metadatos de un objeto actualizados (`POST`) |
| `204 No Content` | Operación correcta sin cuerpo: borrado, metadatos de un contenedor… |
| `401 Unauthorized` | Falta el token o no es válido |
| `403 Forbidden` | El token es válido, pero no hay permiso |
| `404 Not Found` | El contenedor o el objeto no existe |
| `409 Conflict` | Se intenta borrar un contenedor que no está vacío |

---

## Una petición real sin autenticar

```text
$ curl -i https://openstack.gonzalonazareno.org/swift/v1/AUTH_prueba
HTTP/1.1 401 Unauthorized
Server: nginx/1.22.1
Content-Type: text/html; charset=UTF-8
Www-Authenticate: Swift realm="AUTH_prueba"
WWW-Authenticate: Keystone uri="http://172.22.123.100/identity"
X-Trans-Id: tx28d1aefad2d44697a661a-006aeae3aa
```

- Es una **petición HTTP normal**, hecha con `curl`
- El servidor responde con un **código de estado** (401) y unas **cabeceras**
- La cabecera `WWW-Authenticate` nos dice **dónde** conseguir credenciales: Keystone
- Delante de Swift hay un **proxy inverso** (nginx)

---

## Autenticación con *token*

<div class="steps" style="margin-top:0.8rem">
<div class="step">1. Usuario y contraseña al servicio de identidad (Keystone)</div>
<div class="step">2. Keystone devuelve un <strong>token</strong> y la URL del almacenamiento</div>
<div class="step">3. Cada petición lleva el token en una cabecera</div>
</div>

```text
GET /swift/v1/AUTH_<proyecto>/fotos HTTP/1.1
Host: openstack.gonzalonazareno.org
X-Auth-Token: gAAAAABp...
```

- El token **caduca** (por defecto, a la hora): después hay que pedir otro
- No hay cookies ni sesión: el token va **en cada petición** (*stateless*)
- Se usa HTTPS para que nadie pueda **capturar** el token

---

## Privado o público

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### Por defecto: privado

- Todo es **privado**: hace falta un token
- Solo los usuarios del **proyecto** pueden ver y modificar

</div>

<div class="card card-green">

### Contenedor público

- Se le da una **ACL de lectura pública** (`.r:*`)
- Cualquiera puede **descargar** sus objetos **sin token**
- Ya se puede poner la URL en un `<img src="…">` de una página web
- Sigue haciendo falta token para **subir o borrar**

</div>

</div>

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Muchas fugas de datos famosas han sido <strong>buckets públicos por error</strong>. Hazlo público solo cuando lo necesites.</div>
</div>

---

## Muchos clientes, una misma API

<div style="display:flex;align-items:center;justify-content:center;gap:0.8rem;margin:0.8rem 0">
<div style="display:flex;flex-direction:column;gap:0.3rem">
<div style="border:2px solid #64748b;border-radius:8px;padding:0.2rem 0.9rem;background:#f1f5f9;text-align:center;width:13rem"><small>Horizon (navegador)</small></div>
<div style="border:2px solid #64748b;border-radius:8px;padding:0.2rem 0.9rem;background:#f1f5f9;text-align:center;width:13rem"><small>cliente <code>openstack</code></small></div>
<div style="border:2px solid #64748b;border-radius:8px;padding:0.2rem 0.9rem;background:#f1f5f9;text-align:center;width:13rem"><small><code>curl</code></small></div>
<div style="border:2px solid #64748b;border-radius:8px;padding:0.2rem 0.9rem;background:#f1f5f9;text-align:center;width:13rem"><small>una aplicación (Python, PHP…)</small></div>
</div>
<div style="text-align:center;color:var(--teal-600);font-weight:600">⟶<br><small>peticiones HTTP</small></div>
<div style="border:2px solid var(--teal-600);border-radius:8px;padding:0.6rem 1rem;background:var(--teal-50);font-weight:600;text-align:center;width:12rem">API REST<br><small>del almacenamiento</small></div>
</div>

- La **interfaz web**, el **cliente de línea de comandos** y `curl` hacen **exactamente lo mismo**
- Todos acaban enviando las **mismas peticiones HTTP** a la API
- Lo que cambia es la **comodidad**: Horizon y `openstack` construyen las peticiones por nosotros

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# El almacenamiento de objetos hoy

## Usos y servicios

---

## ¿Para qué se usa?

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### Contenido de webs y aplicaciones

- Fotos y vídeos que suben los usuarios
- Imágenes, CSS y JavaScript de una web
- **Webs estáticas** completas
- **Origen** de una CDN

### Copias de seguridad y archivo

- Copias **fuera** de la organización
- Datos que se guardan años y casi no se leen

</div>

<div class="card card-green">

### Datos a gran escala

- **Logs** y métricas
- *Data lakes*: datos para análisis
- Conjuntos de datos para entrenar modelos de **IA**

### Infraestructura

- **Imágenes** de máquinas virtuales y de contenedores
- Distribución de **paquetes** de *software*
- Almacenamiento de aplicaciones como Nextcloud

</div>

</div>

---

## Un ejemplo: una web con imágenes

<div style="display:flex;align-items:center;justify-content:center;gap:0.8rem;margin:0.8rem 0">
<div style="border:2px solid #64748b;border-radius:8px;padding:0.4rem 0.9rem;background:#f1f5f9;font-weight:600;text-align:center;width:9rem">Navegador</div>
<div style="display:flex;flex-direction:column;gap:0.8rem">
<div style="display:flex;align-items:center;gap:0.6rem"><span style="color:var(--teal-600);font-weight:600">⟶ <small>HTML</small></span><div style="border:2px solid var(--teal-600);border-radius:8px;padding:0.3rem 0.9rem;background:var(--teal-50);text-align:center;width:13rem"><small>Servidor web (Apache / Nginx)</small></div></div>
<div style="display:flex;align-items:center;gap:0.6rem"><span style="color:var(--teal-600);font-weight:600">⟶ <small>imágenes</small></span><div style="border:2px solid var(--teal-600);border-radius:8px;padding:0.3rem 0.9rem;background:var(--teal-50);text-align:center;width:13rem"><small>Almacenamiento de objetos</small></div></div>
</div>
</div>

```html
<img src="https://openstack.gonzalonazareno.org/swift/v1/AUTH_<proyecto>/web/logo.png">
```

- El servidor web solo entrega el **HTML**: las imágenes las pide el **navegador** directamente al almacenamiento
- El servidor web no guarda ni sirve los ficheros pesados
- Si hay varios servidores web, todos usan las **mismas** imágenes

---

## Servicios y *software*

| | Producto | API |
|:--|:--|:--|
| **Nube pública** | Amazon S3 | S3 |
| | Azure Blob Storage | Propia |
| | Google Cloud Storage | Propia (y compatible con S3) |
| **Instalación propia** | **OpenStack Swift** | Swift (opcional, compatible con S3) |
| | Ceph (RADOS Gateway) | S3 y Swift |
| | Garage, SeaweedFS… | S3 |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>La API de <strong>S3</strong> se ha convertido en el estándar de facto. Las APIs cambian en los detalles, pero la idea es siempre la misma: <strong>recursos, URLs y métodos HTTP</strong>.</div>
</div>

---

## Nuestro escenario: OpenStack Swift

- Swift es el servicio de **almacenamiento de objetos** de OpenStack
- Lo tenemos en la nube del instituto: **`openstack.gonzalonazareno.org`**
- Cada **proyecto** de OpenStack tiene su **cuenta** de Swift
- Lo usaremos de tres formas:

<div class="cols-3" style="margin-top:0.6rem">

<div class="card card-blue">

### Horizon

La interfaz web de OpenStack

</div>

<div class="card card-green">

### `openstack`

El cliente de línea de comandos

</div>

<div class="card card-purple">

### `curl`

Peticiones HTTP a mano a la API

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">05</p>

# Resumen

---

## Lo que tienes que saber

- El almacenamiento de objetos guarda **objetos**: datos + metadatos + identificador
- Los objetos se organizan en **contenedores** con un espacio de nombres **plano**
- No se monta: se accede por **HTTP** a través de una **API REST**
- En la API, cada recurso es una **URL**, cada operación un **método** y cada resultado un **código de estado**
- La autenticación va en un **token** que se envía en cada petición
- Por defecto es **privado**; un contenedor público se puede leer sin token
- Se usa para contenido web, *backups*, archivo, datos masivos e infraestructura
- **S3** es el estándar de facto; nosotros usaremos **OpenStack Swift**

---

## Para profundizar

- [Documentación de OpenStack Swift](https://docs.openstack.org/swift/latest/)
- [API de Object Storage de OpenStack](https://docs.openstack.org/api-ref/object-store/)
- [Cliente `openstack`: comandos de *object store*](https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/object.html)
- [Amazon S3: ¿qué es el almacenamiento de objetos?](https://aws.amazon.com/what-is/object-storage/)

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Almacenamiento de objetos

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI</span>
</div>
