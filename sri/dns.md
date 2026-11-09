---
marp: true
title: Protocolo DNS
theme: profesional
paginate: true
header: 'SRI · Unidad 3 — Protocolo DNS'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Protocolo **DNS**

## Resolución de nombres en Internet

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI · Servicios de Red e Internet</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Conceptos sobre DNS

## Nombres, zonas y registros

---

## ¿Qué es DNS?

> **Domain Name System** es el protocolo que permite **guardar y consultar información** asociada a un nombre de dominio: dirección IP de un equipo, nombre asociado a una IP, servidor de correo de un dominio…

- Las redes funcionan con **direcciones IP**, las personas con **nombres**
- DNS es el **directorio jerárquico** que traduce de uno a otro a escala global
- Es **distribuido**: ningún servidor conoce todos los nombres
- Su funcionamiento se basa en **delegación** entre dominios

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El servicio escucha por defecto en el puerto <strong>53</strong>, sobre <strong>UDP</strong> (consultas) y <strong>TCP</strong> (transferencias y respuestas grandes).</div>
</div>

---

## Nombres de equipo

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### Hostname

- **Una sola palabra** que identifica al equipo
- Ejemplo: `macaco`
- Se guarda en `/etc/hostname`

</div>

<div class="card card-green">

### FQDN — *Fully Qualified Domain Name*

- Hostname **+ dominio completo**
- Ejemplo: `macaco.gonzalonazareno.org.`
- Identifica al equipo de forma **inequívoca**
- Se comprueba con `hostname -f`

</div>

</div>

---

## Nombres de dominio

Sucesión de nombres separados por **puntos**, que se lee de **derecha a izquierda** (de lo general a lo concreto):

```
     macaco . gonzalonazareno . org .         www . gonzalonazareno . org .
     ──┬───   ───────┬───────   ─┬─   ┬       ─┬─   ───────┬───────   ─┬─   ┬
    equipo        dominio       TLD  raíz   servicio      dominio       TLD  raíz
```

Tienen nombre tanto los **equipos** (`macaco`) como los **servicios** que ofrecen (`www`, `correo`, `ftp`…).

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### Dominio raíz

- El **punto final** (`.`): se suele omitir, pero **siempre está**

</div>

<div class="card card-green">

### TLD — *Top Level Domain*

- Bajo la raíz: genéricos (`com`, `org`…) y de país (`es`, `fr`…)

</div>

</div>

---

## Zonas DNS

> Una **zona** es la parte del espacio de nombres que gestiona su **servidor con autoridad**.

- Se guarda en un **fichero de zona**: un texto con **registros** (`A`, `MX`, `PTR`…), uno por dato
- Un dominio suele tener **dos tipos de zonas**:

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### Zona de resolución directa

- **Nombre → IP** (y otros datos)
- Se organiza por **nombres de dominio**

</div>

<div class="card card-green">

### Zona de resolución inversa

- **IP → nombre**
- Se organiza por **direcciones IP**

</div>

</div>

---

## Resolución directa

Contiene **los nombres bajo un dominio** y su información. Ejemplo de la zona `ejemplo.org`:

| Nombre | Tipo | Valor |
|:--|:--|:--|
| `ejemplo.org.` | `NS` | `dns1.ejemplo.org.` |
| `ejemplo.org.` | `MX` | `10 correo.ejemplo.org.` |
| `dns1.ejemplo.org.` | `A` | `192.168.1.2` |
| `servidor.ejemplo.org.` | `A` | `192.168.1.10` |
| `www.ejemplo.org.` | `CNAME` | `servidor.ejemplo.org.` |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Las <strong>máquinas</strong> se definen con <code>A</code> y los <strong>servicios</strong> (<code>www</code>, <code>ftp</code>…) con <code>CNAME</code> hacia la máquina que los aloja.</div>
</div>

---

## Tipos de registros más usados

| Registro | Para qué sirve |
|:--|:--|
| **SOA** | *Start of Authority*: datos de la zona (maestro, nº de serie, tiempos). **Obligatorio** |
| **NS** | Servidor con autoridad sobre la zona (y delegaciones). **Al menos uno** |
| **A** | Dirección **IPv4** de un nombre |
| **AAAA** | Dirección **IPv6** de un nombre |
| **CNAME** | **Alias** — otro nombre que apunta al «real» |
| **MX** | Servidor de **correo** del dominio, con **prioridad** (menor = preferente) |
| **PTR** | Resolución **inversa** (IP → nombre) |
| **TXT** | Texto libre (SPF, DKIM, verificación…) |

---

## Resolución inversa

> La zona de resolución inversa permite obtener el **nombre** a partir de la **IP**. Se usa en servidores de correo, *logs* y comprobaciones de seguridad.

Como DNS busca por **nombres**, la IP se convierte en un nombre bajo el dominio especial **`in-addr.arpa`**, escribiendo los **octetos al revés**:

```
172.22.0.1   →   1.0.22.172.in-addr.arpa.   →   PTR   →   macaco.gonzalonazareno.org.
```

- Se invierte porque en un nombre lo **general va a la derecha** y en una IP, a la **izquierda**
- La jerarquía sigue a las direcciones: `in-addr.arpa` → `172` → `22`…

---

## Zonas de resolución inversa

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### Nombre de la zona

Es la **parte de red** invertida, según la máscara:

| Red | Zona inversa |
|:--|:--|
| `172.22.0.0/16` | `22.172.in-addr.arpa` |
| `192.168.1.0/24` | `1.168.192.in-addr.arpa` |

</div>

<div class="card card-green">

### Contenido de la zona

- Registros **`PTR`**: uno por cada `A` de la zona directa
- El nombre de cada equipo es la **parte de host** invertida
- Los **alias** (`CNAME`) no aparecen

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Las zonas inversas de IPs públicas las gestiona quien tiene <strong>asignado el bloque</strong> (el proveedor), no el dueño del dominio.</div>
</div>

---

## Servidores maestro y esclavo

Una zona debe tener **varios servidores con autoridad** (`NS`): si sólo hay uno y cae, no se resuelve.

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### Servidor maestro (primario)

- Es donde se **edita** el fichero de zona
- Es la **fuente de verdad** de la zona

</div>

<div class="card card-green">

### Servidor esclavo (secundario)

- Tiene una **copia** de la zona y responde con **autoridad**
- La obtiene del maestro por **transferencia de zona**

</div>

</div>

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Cuando la zona cambia, el maestro <strong>avisa</strong> al esclavo, que compara el <strong>número de serie</strong> del <code>SOA</code> para saber si su copia está desfasada: <strong>hay que incrementarlo en cada cambio</strong>.</div>
</div>

---

## Delegación de dominios

DNS es una **jerarquía de zonas**: cada zona puede **delegar** una parte de su espacio de nombres en otros servidores.

```
                     .  (raíz)          → root servers (13 conjuntos, A–M, con anycast)
           ┌─────────┼─────────┐
          org        es       com       → TLD
           │
   gonzalonazareno.org                  → servidores de CDMON
           │
   tunombre.gonzalonazareno.org         → tu servidor DNS
```

- La zona **padre** guarda los **NS** del subdominio: *«para `tunombre…`, pregunta a este servidor»*
- Quien recibe la delegación tiene **autoridad** sobre esa parte y puede delegar a su vez

---

## *Glue records*

¿Qué pasa si el servidor con autoridad está **dentro** del propio subdominio delegado?

```
tunombre.gonzalonazareno.org.   NS   dns.tunombre.gonzalonazareno.org.
```

- Para preguntar a `dns.tunombre.gonzalonazareno.org` necesito su **IP**…
- …pero su IP está en la zona `tunombre.gonzalonazareno.org`, que gestiona **él mismo**

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Para romper el círculo, la zona <strong>padre</strong> guarda también la <strong>IP</strong> del servidor delegado (un registro <code>A</code>). Ese registro se llama <strong>glue record</strong> («pegamento») y se envía en la sección <code>ADDITIONAL</code> de las respuestas.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Resolución y recursión

## Cómo se resuelve un nombre

---

## ¿Quién pregunta a quién?

1. El equipo comprueba la **resolución estática** (`/etc/hosts`, según `nsswitch.conf`)
2. Si no, pregunta al **servidor DNS** de `/etc/resolv.conf`, que **busca por él** en la jerarquía

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### Consulta recursiva

- La hace el **cliente** (*stub resolver*) a su servidor DNS
- Significa: *«no me des pistas, **devuélveme la respuesta final** o un error»*
- El servidor asume **todo el trabajo**

</div>

<div class="card card-green">

### Consulta iterativa

- La hace el **servidor recursivo** a los servidores de la jerarquía
- Cada uno responde *«no lo sé, **pregunta a este otro**»* (una **referencia**)
- El servidor recursivo **sigue las pistas** hasta la autoridad

</div>

</div>

---

## Resolución paso a paso

```
 Cliente          Recursivo           Root         TLD .org      Autoridad
   │ ¿www.example.org? │                │              │             │
   │ ─────────────────►│  ¿www…?        │              │             │
   │   (recursiva)     │ ──────────────►│              │             │
   │                   │ ◄── «pregunta a los NS de org»│             │
   │                   │  ¿www…?                       │             │
   │                   │ ─────────────────────────────►│             │
   │                   │ ◄── «pregunta a los NS de example.org»      │
   │                   │  ¿www…?                                     │
   │                   │ ───────────────────────────────────────────►│
   │                   │ ◄─────────────────────── IP (respuesta, aa) │
   │ ◄── IP            │   guarda en caché                           │
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Con la caché vacía, el cliente hace <strong>una sola pregunta</strong> y el recursivo hace todo el recorrido. <code>dig +trace</code> enseña cada salto.</div>
</div>

---

## Qué responde cada servidor

Los servidores intermedios **no dan la respuesta**: dan una **referencia**.

| Sección | En una referencia | En la respuesta final |
|:--|:--|:--|
| `ANSWER` | **vacía** | el registro pedido (`A`, `MX`…) |
| `AUTHORITY` | registros **NS** del siguiente nivel | normalmente vacía |
| `ADDITIONAL` | **IPs** de esos NS (*glue*) | datos extra (EDNS) |

- El **root** responde con los NS del **TLD** (`org`)
- El **TLD** responde con los NS del **dominio** (`example.org`)
- El servidor **con autoridad** responde con la IP y marca la respuesta como autoritativa (`aa`)

---

## Caché y TTL

- Cada registro lleva un **TTL**: segundos que se puede **guardar en caché**
- Mientras no caduque, el recursivo responde **sin preguntar a nadie**; el TTL va bajando:

```
$ dig www.debian.org
www.debian.org.   162   IN   A   194.177.211.216     ← quedan 162 s en caché
```

- **Caché positiva**: la respuesta, durante el **TTL** del registro
- **Caché negativa**: que un nombre **no existe** (`NXDOMAIN`), durante el *negative TTL* del **SOA**

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Un cambio en un registro <strong>no se ve al instante</strong>: hay que esperar a que caduque el TTL en las cachés.</div>
</div>

---

## Tipos de servidores DNS

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Servidor recursivo

- Resuelve **cualquier consulta** preguntando él mismo a la jerarquía
- Mantiene **caché** de respuestas
- El de proveedores y redes locales

### Servidor forward

- **Reenvía** las consultas a otro servidor recursivo
- No resuelve por sí mismo desde la raíz

</div>

<div class="card card-green">

### Servidor con autoridad

- Responde **sólo** sobre las zonas que aloja
- **No hace recursión**: es la fuente «oficial» del dominio

### Servidor caché

- Sólo guarda respuestas previas
- Útil como capa intermedia para acelerar consultas

</div>

</div>

---

## DNS en una red local

¿Y si el servidor DNS configurado en `resolv.conf` está en **nuestra propia red**?

- Las resoluciones se hacen **más rápidas**: lo que esté en caché no se vuelve a preguntar a Internet
- Si tenemos **servicios internos** con nombres propios, podemos crear una **zona local**: el servidor DNS tendrá autoridad sobre ella y resolverá esos nombres a **IPs privadas**
- El servidor **DHCP** de la red repartirá ese DNS local a los clientes
- Un mismo servidor puede ser **autoridad** de nuestras zonas y **recursivo** para el resto de nombres

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Esto permite tener nombres como <code>impresora.lan</code> o <code>servidor.lan</code> sin depender de DNS externos ni tocar <code>/etc/hosts</code> en cada equipo.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# Anatomía de una respuesta

## `dig`, cabecera y bits de control

---

## La herramienta `dig`

`dig` hace **consultas DNS** desde la línea de comandos y muestra **toda** la respuesta del servidor, no sólo la IP.

```
dig [tipo de registro] [@servidor DNS] consulta_DNS
```

- **Tipo de registro** por defecto: `A`. **Servidor** por defecto: el de `/etc/resolv.conf`
- `-x <IP>` para la resolución **inversa**
- `+short` respuesta resumida · `+trace` sigue la jerarquía desde la raíz · `+norec` consulta **sin recursión**
- Paquete Debian: **`dnsutils`** (`sudo apt install dnsutils`)

---

## Salida completa de `dig`

```
; <<>> DiG 9.20.30 <<>> mx debian.org                       ← ① comando
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16139   ← ② cabecera
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1
;; OPT PSEUDOSECTION:                                       ← ③ EDNS
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:                                        ← ④ pregunta
;debian.org.			IN	MX
;; ANSWER SECTION:                                          ← ⑤ respuesta
debian.org.		28800	IN	MX	0 mailly.debian.org.
debian.org.		28800	IN	MX	0 muffat.debian.org.
debian.org.		28800	IN	MX	0 mitropoulos.debian.org.
;; Query time: 27 msec                                      ← ⑥ pie
;; SERVER: 100.100.100.100#53(100.100.100.100) (UDP)
;; WHEN: Mon Nov 09 14:07:21 CET 2026
;; MSG SIZE  rcvd: 113
```

---

## ② La cabecera (`HEADER`)

```
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16139
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1
```

| Campo | Significado |
|:--|:--|
| `opcode` | Tipo de operación: `QUERY` es la consulta normal |
| `status` | **Resultado**: `NOERROR`, `NXDOMAIN`, `SERVFAIL`, `REFUSED` |
| `id` | Número que **empareja** cada respuesta con su consulta (también dificulta la suplantación) |
| `flags` | **Bits de control** del mensaje: `qr rd ra ad`… (se detallan más adelante) |
| `QUERY` | Nº de preguntas (siempre **1**) |
| `ANSWER` / `AUTHORITY` / `ADDITIONAL` | Nº de **registros** que hay en cada sección |

---

## ③ y ④ `OPT` y `QUESTION`

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### `OPT PSEUDOSECTION`

```
; EDNS: version: 0, flags:; udp: 1232
```

- **EDNS**: extensión del protocolo
- `udp: 1232`: tamaño máximo de respuesta UDP que acepta el cliente
- `flags: do` aparece al usar `+dnssec` (*DNSSEC OK*: «quiero las firmas»)

</div>

<div class="card card-green">

### `QUESTION SECTION`

```
;debian.org.     IN    MX
```

- **Qué se ha preguntado**: nombre, clase y tipo
- `IN` es la clase *Internet* (la única que se usa)
- Va con `;` al principio: es un **comentario**, no un registro

</div>

</div>

---

## ⑤ `ANSWER SECTION`

Es **la respuesta**: una línea por **registro** (*resource record*), con cinco campos:

```
debian.org.      28800    IN    MX    0 mailly.debian.org.
   ①               ②       ③     ④            ⑤
```

| | Campo | Significado |
|:--:|:--|:--|
| ① | **Nombre** | Nombre del registro (FQDN, acaba en `.`) |
| ② | **TTL** | Segundos que puede estar en **caché** |
| ③ | **Clase** | `IN` (Internet) |
| ④ | **Tipo** | `A`, `AAAA`, `NS`, `MX`, `CNAME`, `PTR`, `TXT`… |
| ⑤ | **Datos** | IP, nombre, prioridad + servidor… según el tipo |

---

## `AUTHORITY` y `ADDITIONAL`

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### `AUTHORITY SECTION`

- Registros **NS** de los servidores con autoridad
- En una **referencia**: a quién preguntar después
- Si el nombre **no existe**: el **SOA** de la zona (con el TTL de la caché negativa)
- Con `dig ns dominio` la respuesta está en `ANSWER`; `AUTHORITY` aparece cuando es información de apoyo

</div>

<div class="card card-green">

### `ADDITIONAL SECTION`

- Datos **extra** que ahorran otra consulta
- Típico: las **IPs** (registros `A`/`AAAA`) de los servidores de las secciones anteriores (*glue*)
- Ejemplo: al pedir los `MX`, el servidor puede añadir la IP de cada servidor de correo

</div>

</div>

---

## ⑥ El pie: servidor, tiempo y tamaño

```
;; Query time: 27 msec
;; SERVER: 100.100.100.100#53(100.100.100.100) (UDP)
;; WHEN: Mon Nov 09 14:07:21 CET 2026
;; MSG SIZE  rcvd: 113
```

| Línea | Significado |
|:--|:--|
| `Query time` | Tiempo de respuesta: alto la **primera** vez, ≈ 0 ms si viene de **caché** |
| `SERVER` | **Servidor que ha contestado**: IP, puerto `53` y protocolo. **Compruébalo siempre** |
| `WHEN` | Fecha y hora de la consulta |
| `MSG SIZE rcvd` | Tamaño en **bytes** de la respuesta |

---

## Los bits de la cabecera DNS

La cabecera de cada mensaje DNS lleva **bits de control** (*flags*). Los de `dig` se ven en la línea `flags:`.

| Bit | Nombre | Significado |
|:--:|:--|:--|
| `qr` | *Query/Response* | `0` = consulta, `1` = **respuesta** |
| `aa` | *Authoritative Answer* | la respuesta viene de un servidor **con autoridad** |
| `rd` | *Recursion Desired* | el cliente **pide** recursión |
| `ra` | *Recursion Available* | el servidor **ofrece** recursión |
| `tc` | *Truncated* | respuesta **truncada**: repetir por **TCP** |
| `ad` | *Authenticated Data* | datos **validados con DNSSEC** |
| `cd` | *Checking Disabled* | el cliente pide **no validar** DNSSEC |

---

## Recursión en los bits: `rd` y `ra`

La recursión se **negocia**: el cliente pide **`rd`** y el servidor contesta **`ra`** si está dispuesto a hacerla.

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### Servidor recursivo

```
;; flags: qr rd ra
```

- Recibe `rd`, responde con **`ra`**
- Sin `aa`, salvo que el nombre sea de una zona suya

</div>

<div class="card card-green">

### Servidor con autoridad

```
;; flags: qr aa rd
```

- Responde con **`aa`** y **sin `ra`**
- Si preguntas por un nombre ajeno: referencia o `REFUSED`

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div><code>dig +norec</code> pregunta <strong>sin</strong> <code>rd</code>, como hace un recursivo con la jerarquía: recibe <strong>referencias</strong>.</div>
</div>

---

## `aa` y `tc`: autoridad y tamaño

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### `aa` — respuesta autoritativa

- La respuesta viene **de la fuente**: el servidor tiene la zona
- Una respuesta **desde caché** nunca lleva `aa`
- Sirve para distinguir si el dato es **oficial** o puede estar **desfasado** por la caché

</div>

<div class="card card-green">

### `tc` — respuesta truncada

- En **UDP** una respuesta tiene un tamaño máximo (con EDNS suele ser ~1232 bytes)
- Si no cabe, el servidor la corta y activa **`tc`**
- El cliente **repite la consulta por TCP** (`dig` lo indica: *Truncated, retrying in TCP mode*)

</div>

</div>

---

## Códigos de estado (`status`)

| Estado | Significado |
|:--|:--|
| `NOERROR` | consulta correcta (puede venir con la sección `ANSWER` vacía si el nombre existe pero no tiene ese tipo de registro) |
| `NXDOMAIN` | el nombre **no existe** |
| `SERVFAIL` | el servidor **no ha podido resolver** (servidor autoritativo caído, o **fallo de validación DNSSEC**) |
| `REFUSED` | el servidor **se niega** a responder (p. ej. no permite recursión o consultas desde tu red) |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Son los errores que verás al montar tu propio servidor: <code>REFUSED</code> suele ser un control de acceso mal configurado y <code>SERVFAIL</code> una zona con errores.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# DNSSEC

## Autenticar las respuestas DNS

---

## El problema: DNS no se autentica

- Una respuesta DNS **no demuestra** que venga de quien dice
- Un atacante puede **suplantarla** y llevarte a otra IP: **envenenamiento de caché** (*cache poisoning*)
- La respuesta falsa queda guardada en el **recursivo** y la reciben **todos sus clientes** hasta que caduque el TTL

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>DNS no cifra ni firma. <strong>DNSSEC</strong> (*DNS Security Extensions*) añade <strong>autenticidad e integridad</strong>: permite comprobar que la respuesta la ha generado el dueño de la zona y que no se ha modificado. <strong>No cifra</strong> (no da confidencialidad).</div>
</div>

---

## Cómo funciona: firmas y claves

El dueño de la zona **firma** sus registros con **criptografía de clave pública**:

| Registro | Contenido |
|:--|:--|
| **DNSKEY** | La **clave pública** de la zona |
| **RRSIG** | La **firma** de cada conjunto de registros (por ejemplo, los `A` de un nombre) |
| **DS** | *Delegation Signer*: **huella** de la clave de la zona hija, guardada en la zona **padre** |
| **NSEC / NSEC3** | **Prueba de inexistencia** firmada: demuestra que un nombre *no existe* |

- Normalmente hay dos claves: **KSK** (firma las claves, se identifica en el DS) y **ZSK** (firma los registros)
- Un **resolver validador** recalcula la firma con la DNSKEY y comprueba que coincide

---

## La cadena de confianza

¿Por qué confiar en la clave de `debian.org`? Porque está **avalada por el padre**, y así hasta la raíz:

```
  .  (raíz)      clave pública conocida de antemano (trust anchor)
   │  DS de org     firmado por la raíz
   ▼
  org            DNSKEY de org, avalada por el DS
   │  DS de debian.org   firmado por org
   ▼
  debian.org     DNSKEY de debian.org  →  RRSIG de sus registros
```

- El resolver **sólo necesita conocer la clave de la raíz**: el resto se valida **encadenando** DS → DNSKEY → RRSIG
- Si **toda la cadena** es válida, devuelve la respuesta con el bit **`ad`** activo
- Si una firma **falla**, devuelve **`SERVFAIL`**: es mejor no responder que dar un dato falso

---

## DNSSEC con `dig`

```
$ dig +dnssec debian.org
;; flags: qr rd ra ad;                   ← ad: validado
debian.org.   172  IN  A      151.101.2.132
debian.org.   172  IN  RRSIG A 8 2 300 20261211013633 20261101011742 29026 debian.org. NXTIX…

$ dig DS debian.org +short
40756 8 2 BBE42151B1A41EFC1F7E6D74D86E601D55051D2B9EB99B1CD0CEAA6E2A5618F5

$ dig DNSKEY debian.org +short
256 3 8 AwEAAaFKOvqD3ZPE/…        ← 256 = ZSK
257 3 8 AwEAAcOt7qF88Cx7…         ← 257 = KSK
```

- **`+dnssec`** pide los registros `RRSIG` junto con la respuesta
- El `RRSIG` indica algoritmo, validez (fechas de inicio y caducidad) y el *key tag* de la clave que firma

---

## Un fallo de validación

`dnssec-failed.org` es un dominio con la firma **rota a propósito** para probar los validadores:

```
$ dig dnssec-failed.org
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL
;; flags: qr rd ra;            ← sin ad
```

```
$ dig +cd dnssec-failed.org     ← cd: «no valides»
;; ->>HEADER<<- opcode: QUERY, status: NOERROR
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El nombre existe y el servidor tiene la IP, pero el <strong>resolver validador la descarta</strong> porque no puede comprobar la firma. Con <code>+cd</code> se desactiva la validación y sí responde.</div>
</div>

---

## La firma de la zona

Firmar una zona consiste en **generar las claves** y **publicar** DNSKEY, RRSIG y NSEC en ella, **renovando las firmas** antes de que caduquen.

1. Generar las claves **KSK** y **ZSK**
2. Firmar todos los conjuntos de registros → aparecen los **RRSIG**
3. Publicar el **DS** (huella de la KSK) en la **zona padre**: sin DS, **no hay cadena de confianza**
4. Mantener la zona: las firmas **caducan** y las claves se **rotan**

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>En <code>bind9</code> moderno la firma y la rotación son <strong>automáticas</strong>: la zona se asocia a una <em>política de firma</em> (<code>dnssec-policy</code>) y bind gestiona claves y firmas. Se verá en los ejercicios.</div>
</div>

---

## ¿Quién firma? Nuestro escenario

Tenemos **dos servidores DNS** con papeles distintos:

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### DNS externo (proveedor)

- Aloja la **zona pública** del dominio
- El proveedor suele ofrecer **firmar con un clic**
- Hay que publicar el **DS** en el **registrador** del dominio, que lo sube a la zona padre
- Si el **registrador y el proveedor** son distintos, el DS se copia a mano

</div>

<div class="card card-green">

### Nuestro bind (local)

- Autoridad de las zonas **internas** (`.lan`, vista interna…)
- Como **recursivo**, **valida** las respuestas de Internet (`dnssec-validation`)
- Firmar nuestra zona sólo tiene sentido si hay **DS en el padre**: en zonas internas o delegadas a mano **no hay cadena de confianza**

</div>

</div>

---

## DNSSEC en nuestras vistas

| | **Vista pública** (Internet) | **Vista interna** (intranet) |
|:--|:--|:--|
| Autoridad | **Proveedor** (CDMON) | **Nuestro bind** |
| Firma | La puede firmar el proveedor + **DS** en el registrador | Normalmente **sin firmar** |
| Validación | La hacen los **recursivos** de los clientes | La hace **nuestro recursivo** |

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Si el nombre <strong>interno</strong> existe también en Internet pero con otra IP (<em>split-horizon</em>), un validador que lo comprobara con DNSSEC público <strong>fallaría</strong>. Por eso nuestro recursivo no valida las zonas sobre las que tiene autoridad.</div>
</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Hoy <code>gonzalonazareno.org</code> <strong>no está firmado</strong>: no hay registro <code>DS</code> en <code>org</code>. Es lo habitual en la mayoría de dominios.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">05</p>

# DNS en el Gonzalo Nazareno

## Un mismo dominio, dos vistas

---

## Dos zonas para `gonzalonazareno.org`

El dominio **`gonzalonazareno.org`** tiene **dos zonas de resolución directa** distintas:

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Vista pública (Internet)

- Servidores DNS con autoridad alojados en la empresa **CDMON**
- `openstack.gonzalonazareno.org` resuelve a la **IP pública**: `5.196.224.198`

</div>

<div class="card card-green">

### Vista interna (intranet)

- Servidor DNS con autoridad: **`macaco`** (`172.22.0.1`)
- `openstack.gonzalonazareno.org` resuelve a `172.22.0.3` (**simio**, proxy inverso)

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Es un caso clásico de <strong>split-horizon DNS</strong>: la respuesta depende de desde dónde se pregunte.</div>
</div>

---

## Desde casa (vista pública)

```
$ dig ns gonzalonazareno.org

;; ANSWER SECTION:
gonzalonazareno.org.    21599    IN    NS    ns2.cdmon.net.
gonzalonazareno.org.    21599    IN    NS    ns4.cdmondns-01.org.
gonzalonazareno.org.    21599    IN    NS    ns1.cdmon.net.
gonzalonazareno.org.    21599    IN    NS    ns5.cdmondns-01.com.
gonzalonazareno.org.    21599    IN    NS    ns3.cdmon.net.

$ dig openstack.gonzalonazareno.org

;; ANSWER SECTION:
openstack.gonzalonazareno.org. 0 IN  CNAME  macaco.gonzalonazareno.org.
macaco.gonzalonazareno.org.    0 IN  A      5.196.224.198
```

---

## Desde el aula (vista interna)

```
$ dig ns gonzalonazareno.org

;; ANSWER SECTION:
gonzalonazareno.org.    86400    IN    NS    dns.gonzalonazareno.org.

$ dig openstack.gonzalonazareno.org

;; ANSWER SECTION:
openstack.gonzalonazareno.org. 86400 IN  CNAME  simio.gonzalonazareno.org.
simio.gonzalonazareno.org.     86400 IN  A      172.22.0.3
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El mismo nombre devuelve <strong>respuestas distintas</strong> según el servidor DNS al que se pregunte: el del aula tiene su propia zona con autoridad.</div>
</div>

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## DNS

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI</span>
</div>
