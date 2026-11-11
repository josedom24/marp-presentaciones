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

## Lo esencial de `dig`

---

## La herramienta `dig`

`dig` hace **consultas DNS** desde la línea de comandos y muestra **toda** la respuesta del servidor, no sólo la IP.

```
dig [tipo de registro] [@servidor DNS] consulta_DNS
```

- **Tipo** por defecto: `A`. **Servidor** por defecto: el de `/etc/resolv.conf`
- `-x <IP>` resolución **inversa** · `+short` respuesta resumida
- `+trace` sigue la jerarquía desde la raíz · `+norec` consulta **sin recursión**
- Paquete Debian: **`dnsutils`**

---

## Salida de `dig`

```
; <<>> DiG 9.20.30 <<>> mx debian.org
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16139   ← ① cabecera
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1
;; QUESTION SECTION:                                        ← ② pregunta
;debian.org.			IN	MX
;; ANSWER SECTION:                                          ← ③ respuesta
debian.org.		28800	IN	MX	0 mailly.debian.org.
debian.org.		28800	IN	MX	0 muffat.debian.org.
debian.org.		28800	IN	MX	0 mitropoulos.debian.org.
;; Query time: 27 msec                                      ← ④ pie
;; SERVER: 100.100.100.100#53(100.100.100.100) (UDP)
```

- ① **Resultado** (`status`) y **bits de control** (`flags`)
- ② Lo que se ha **preguntado** · ③ la **respuesta**
- ④ **Qué servidor** ha contestado y cuánto ha tardado: **compruébalo siempre**

---

## Secciones de la respuesta

Cada línea de `ANSWER` es un **registro** con cinco campos:

```
debian.org.      28800    IN    MX    0 mailly.debian.org.
 nombre           TTL    clase  tipo        datos
```

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### `AUTHORITY`

- Los **NS** con autoridad: a quién preguntar después (**referencia**)
- Si el nombre **no existe**: el **SOA** de la zona

</div>

<div class="card card-green">

### `ADDITIONAL`

- Datos **extra** que ahorran otra consulta
- Típico: las **IPs** de los servidores NS (*glue*)

</div>

</div>

---

## Bits (`flags`) y estado (`status`)

<div class="cols-2" style="margin-top:0.6rem">

<div>

| Bit | Significado |
|:--:|:--|
| `qr` | es una **respuesta** |
| `rd` | el cliente **pide** recursión |
| `ra` | el servidor **ofrece** recursión |
| `aa` | respuesta **autoritativa** (nunca desde caché) |
| `tc` | **truncada**: repetir por **TCP** |

</div>

<div>

| `status` | Significado |
|:--|:--|
| `NOERROR` | consulta correcta |
| `NXDOMAIN` | el nombre **no existe** |
| `SERVFAIL` | el servidor **no ha podido resolver** |
| `REFUSED` | el servidor **se niega** a responder |

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Un <strong>recursivo</strong> responde con <code>ra</code>; un servidor con <strong>autoridad</strong> con <code>aa</code> y sin <code>ra</code>. <code>REFUSED</code> suele ser un control de acceso mal configurado y <code>SERVFAIL</code>, una zona con errores.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

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
