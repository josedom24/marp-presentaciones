---
marp: true
title: Protocolo DNS
theme: profesional
paginate: true
header: 'SRI · Unidad 5 — Protocolo DNS'
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

### Ideas clave

- Las redes funcionan con **direcciones IP**, las personas con **nombres**
- DNS es el **directorio jerárquico** que traduce de uno a otro a escala global
- Es **distribuido**: ningún servidor conoce todos los nombres
- Su funcionamiento se basa en **delegación** entre dominios

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El servicio escucha por defecto en el puerto <strong>53</strong>, sobre <strong>UDP</strong> (consultas) y <strong>TCP</strong> (transferencias y respuestas grandes).</div>
</div>

---

## Tipos de nombres

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Hostname

- **Una sola palabra** que identifica al equipo
- Se guarda en `/etc/hostname`
- Ejemplo: `macaco`

### FQDN — *Fully Qualified Domain Name*

- Hostname **+ dominio completo**
- Identifica al equipo de forma **inequívoca**
- Ejemplo: `macaco.gonzalonazareno.org.`

</div>

<div class="card card-green">

### Domain Name

- Sucesión de nombres separados por **puntos**
- Ejemplo: `gonzalonazareno.org`

### Dominio raíz

- Es el **punto final**: todo nombre acaba en `.`
- Ejemplo completo: `macaco.gonzalonazareno.org.`

### TLD — *Top Level Domain*

- Dominios bajo el raíz: `com`, `org`, `es`…

</div>

</div>

---

## Zonas DNS

> Una **zona** agrupa la información de un nombre de dominio. El servidor que guarda una zona es su **autoridad**.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Resolución directa

- Conjunto de nombres bajo un dominio
- Permite obtener la **IP** a partir del **nombre**
- En `gonzalonazareno.org`: `macaco`, `www`, `dit`, `openstack`…

</div>

<div class="card card-green">

### Resolución inversa

- Permite obtener el **nombre** a partir de la **IP**
- Depende del **direccionamiento** de red
- Útil para registros de correo, *logs* y comprobaciones

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El <strong>fichero de zona</strong> es un archivo de texto donde se guardan los <strong>registros</strong> (cada tipo de información tiene su propio tipo de registro).</div>
</div>

---

## Autoridad sobre una zona

- El **servidor DNS** que guarda el **fichero de una zona** es su **servidor con autoridad**
- En el propio fichero de zona se declara con un **registro NS**
- Conviene tener **varios** servidores con autoridad por zona:
  - **Maestro** — servidor principal donde se edita la zona
  - **Esclavos** — sincronizan automáticamente con el maestro
- Los servidores con autoridad sobre el **dominio raíz** se llaman ***root servers***

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Hay <strong>13 conjuntos</strong> de root servers identificados con letras (A–M); cada uno se replica con <em>anycast</em> en cientos de ubicaciones por todo el mundo.</div>
</div>

---

## Tipos de registros más usados

| Registro | Para qué sirve |
|:--|:--|
| **SOA** | *Start of Authority* — metainformación de la zona |
| **NS** | Servidor con autoridad sobre la zona |
| **A** | Dirección **IPv4** de un nombre |
| **AAAA** | Dirección **IPv6** de un nombre |
| **CNAME** | **Alias** — otro nombre que apunta al "real" |
| **MX** | Servidor de **correo** del dominio |
| **PTR** | Resolución **inversa** (IP → nombre) |
| **TXT** | Texto libre (SPF, DKIM, verificación…) |

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Resolución de consultas

## Recursión, caché y tipos de servidor

---

## Consulta DNS por recursión

Cuando un equipo necesita resolver un nombre, sigue este flujo:

1. Comprueba la **resolución estática** (`/etc/hosts`, NSS)
2. Si no lo encuentra, consulta al **servidor DNS** configurado en `resolv.conf`
3. Pueden pasar **tres cosas**:

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### Si el servidor lo conoce

- **Tiene autoridad** sobre el dominio → responde directamente
- **Lo tiene en caché** de una consulta previa → responde directamente

</div>

<div class="card card-green">

### Si no lo conoce

- Pregunta a los **root servers**
- Estos derivan al **TLD** correspondiente
- El TLD deriva al servidor **con autoridad** del dominio
- Se devuelve la respuesta y se **guarda en caché**

</div>

</div>

---

## Esquema de la resolución

```
            Cliente                 DNS local                  Internet
              │                         │                          │
              │  ¿www.example.org?      │                          │
              │ ───────────────────────►│                          │
              │                         │  ¿quién resuelve org?    │
              │                         │ ────────────────────────►│  Root
              │                         │ ◄────────────────────────│
              │                         │  ¿quién resuelve         │
              │                         │   example.org?           │
              │                         │ ────────────────────────►│  TLD .org
              │                         │ ◄────────────────────────│
              │                         │  ¿IP de www.example.org? │
              │                         │ ────────────────────────►│  Autoridad
              │  IP                     │ ◄────────────────────────│
              │ ◄───────────────────────│                          │
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El cliente hace <strong>una sola pregunta</strong>; toda la cadena de consultas la realiza el servidor DNS local.</div>
</div>

---

## Tipos de servidores DNS

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Servidor recursivo

- Resuelve **cualquier consulta** preguntando él mismo a la jerarquía
- Mantiene **caché** de respuestas
- Es el típico de los proveedores y de las redes locales

### Servidor forward

- **Reenvía** las consultas a otro servidor
- No resuelve por sí mismo desde la raíz

</div>

<div class="card card-green">

### Servidor con autoridad

- Responde **sólo** sobre las zonas que aloja
- No hace recursión
- Es la fuente "oficial" de los registros del dominio

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

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Esto permite tener nombres como <code>impresora.lan</code> o <code>servidor.lan</code> sin depender de DNS externos ni tocar <code>/etc/hosts</code> en cada equipo.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

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

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# Ejercicio práctico

## Consultas DNS con `dig`

---

## La herramienta `dig`

`dig` permite hacer **consultas DNS** desde la línea de comandos y ver toda la respuesta del servidor.

### Sintaxis

```
dig [tipo de registro] [@servidor DNS] consulta_DNS
```

- El **tipo de registro** por defecto es `A`
- El **servidor DNS** por defecto es el de `/etc/resolv.conf`
- La respuesta aparece en la sección **`ANSWER SECTION`**

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Si no tienes <code>dig</code> instalado, añade el paquete <strong><code>dnsutils</code></strong>: <code>sudo apt install dnsutils</code>.</div>
</div>

---

## Tipos de registro habituales con `dig`

| Tipo | Información que devuelve |
|:--|:--|
| `A` | Dirección IPv4 (valor por defecto) |
| `AAAA` | Dirección IPv6 |
| `NS` | Servidores con autoridad sobre la zona |
| `MX` | Servidores de correo del dominio |
| `CNAME` | Alias del nombre consultado |
| `SOA` | Metainformación de la zona |
| `TXT` | Registros de texto (SPF, DKIM, verificaciones…) |
| `-x <IP>` | Resolución **inversa** de la dirección IP |

---

## Ejercicio — direcciones IP

Obtén la dirección **IP** de los siguientes dominios:

- `www.gonzalonazareno.org`
- `www.debian.org`

### Pista

```bash
dig www.gonzalonazareno.org
dig www.debian.org
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Fíjate en la sección <code>ANSWER SECTION</code>: puede haber un <code>CNAME</code> intermedio antes de llegar al registro <code>A</code> con la IP final.</div>
</div>

---

## Ejercicio — servidores con autoridad

Obtén los **servidores con autoridad** de los dominios:

- Dominio **raíz** (`.`)
- `es`
- `gonzalonazareno.org`

### Pista

```bash
dig ns .
dig ns es
dig ns gonzalonazareno.org
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El dominio raíz se representa con un punto: <code>.</code>. Devolverá los <strong>13 root servers</strong> (de <code>a.root-servers.net</code> a <code>m.root-servers.net</code>).</div>
</div>

---

## Ejercicio — servidores de correo

Obtén los **servidores de correo (MX)** de los dominios:

- `gonzalonazareno.org`
- `us.es`

### Pista

```bash
dig mx gonzalonazareno.org
dig mx us.es
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Cada registro <code>MX</code> incluye una <strong>prioridad</strong> (número entero más bajo = más prioritario). Si el principal no responde, el correo se intenta entregar al siguiente.</div>
</div>

---

## Ejercicio — tipo de registro

¿Qué **tipo de registro** resuelve los siguientes dominios?

- `www.josedomingo.org`
- `informatica.gonzalonazareno.org`

### Pista

```bash
dig www.josedomingo.org
dig informatica.gonzalonazareno.org
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Mira la <code>ANSWER SECTION</code>: si aparece un <code>CNAME</code>, el nombre es un <strong>alias</strong> de otro; si aparece un <code>A</code> directamente, hay una IP asociada al propio nombre.</div>
</div>

---

## Ejercicio — resolución inversa

En clase: ¿a qué **nombre** corresponde la dirección IP `172.22.0.1`?

### Pista

```bash
dig -x 172.22.0.1
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>La resolución inversa se hace con la opción <code>-x</code>. La respuesta llega como un registro <strong><code>PTR</code></strong> en la <code>ANSWER SECTION</code>.</div>
</div>

---

## Trucos útiles con `dig`

```bash
# Consulta a un servidor DNS concreto
dig @8.8.8.8 www.debian.org

# Respuesta corta (sólo la IP / dato relevante)
dig +short www.debian.org

# Sin las secciones de cabecera y estadísticas
dig +noall +answer www.debian.org

# Seguir la cadena desde los root servers
dig +trace www.gonzalonazareno.org

# Consulta inversa
dig -x 172.22.0.1
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div><code>+trace</code> es muy útil para entender cómo se navega la jerarquía DNS desde el dominio raíz hasta el servidor con autoridad.</div>
</div>

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## DNS → Servidor BIND en la práctica

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI</span>
</div>
