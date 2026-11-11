---
marp: true
title: DNSSEC
theme: profesional
paginate: true
header: 'SRI · Unidad 3 — DNSSEC'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# **DNSSEC**

## Autenticar las respuestas DNS

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI · Servicios de Red e Internet</span>
</div>

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

## Bits de DNSSEC en `dig`

| Bit / opción | Significado |
|:--|:--|
| `ad` | *Authenticated Data*: datos **validados con DNSSEC** |
| `cd` | *Checking Disabled*: el cliente pide **no validar** (`dig +cd`) |
| `do` / `+dnssec` | *DNSSEC OK*: el cliente pide los **RRSIG** junto con la respuesta |

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

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## DNSSEC

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI</span>
</div>
