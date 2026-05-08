---
marp: true
title: Servidor de correo electrónico
theme: profesional
paginate: true
header: 'SRI · Unidad 6 — Servidor de correo electrónico'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Servidor de **correo electrónico**

## SMTP, IMAP y Postfix

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI · Servicios de Red e Internet</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Cómo funciona el correo electrónico

## Direcciones, agentes y protocolos

---

## Direcciones de correo

> Toda dirección de correo tiene la forma **`usuario@nombre_servicio_correo`**.

### ¿Cómo se localiza el servidor?

- Habitualmente, lo que hay tras la `@` es un **nombre de dominio**
- El sistema necesita conocer la **IP del servidor de correo** asociado a ese dominio
- Para ello consulta el DNS:
  1. Si el nombre tras la `@` tiene un **registro A**, esa IP puede valer
  2. Si es un dominio (sin IP propia) o el correo se sirve en otro equipo, hace falta un **registro MX** que apunte al servidor de correo

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El registro <strong>MX</strong> es la pieza clave que conecta el dominio con el servidor que recibe su correo.</div>
</div>

---

## Agentes de correo: MUA, MTA y MDA

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### MUA — *Mail User Agent*

- **Cliente** de correo del usuario
- Permite redactar, enviar y leer mensajes
- Ejemplos: Thunderbird, Outlook, Roundcube

### MTA — *Mail Transfer Agent*

- **Servidor** de correo
- Transfiere mensajes entre máquinas usando **SMTP**
- Ejemplos: Postfix, Sendmail, Exim

</div>

<div class="card card-green">

### MDA — *Mail Delivery Agent*

- Entrega el correo al **buzón** del usuario
- Si el usuario está en el propio servidor, lo hace un **LDA** (Local Delivery Agent)
- Si el usuario lee desde fuera, se accede mediante **POP3** o **IMAP**

</div>

</div>

---

## Protocolos del correo

| Protocolo | Función | Puerto típico |
|:--|:--|:--|
| **SMTP** | Transferencia de correo entre servidores (envío) | 25/tcp |
| **ESMTP** + STARTTLS | Envío del cliente al servidor (autenticado y cifrado) | 587/tcp |
| **SMTPS** | Variante "implícita" del SMTP cifrado | 465/tcp |
| **POP3** | Recogida de correo: **descarga** los mensajes | 110 / 995/tcp |
| **IMAP** | Recogida de correo: **sincroniza** estado | 143 / 993/tcp |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>SMTP <strong>envía</strong>; POP3 e IMAP <strong>reciben</strong>. Todo cliente de correo necesita los dos lados configurados.</div>
</div>

---

## Recorrido completo de un correo

```
   Remitente                                                   Destinatario
   ────────                                                    ────────────

   MUA ──SMTP──► MTA ──SMTP──► MTA ──LDA──► buzón ──IMAP/POP3──► MUA
  (Thunderbird) (Postfix    (Postfix       (maildir)            (Thunderbird)
                 origen)     destino)
```

### Lo que pasa por debajo

- El **MUA** redacta el mensaje y lo entrega al MTA local
- El **MTA origen** consulta el **registro MX** del dominio destino y abre una conexión SMTP
- El **MTA destino** acepta el mensaje y se lo pasa al **LDA**
- El destinatario recoge el correo con **IMAP** o **POP3**

---

## Formato MIME

> El estándar **MIME** (*Multipurpose Internet Mail Extensions*) define cómo se codifican los mensajes actuales. Antes sólo era posible texto **US-ASCII**.

### MIME permite

- Texto en **otros conjuntos de caracteres** (UTF-8, ISO-8859-1…)
- **Adjuntos** binarios (imágenes, PDF, vídeo…)
- Cuerpos de mensaje **multi-parte** (texto + HTML + adjuntos)

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Cabeceras como <code>MIME-Version</code> y <code>Content-Type</code> indican al cliente cómo interpretar cada parte del mensaje.</div>
</div>

---

## Cabeceras importantes

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Información del mensaje

- `From` — remitente
- `To`, `Cc`, `Bcc` — destinatarios
- `Subject` — asunto
- `Date` — fecha de envío

</div>

<div class="card card-green">

### Trazabilidad y seguridad

- `Return-Path`, `Delivered-To`
- `Received` — saltos por los MTA
- `MIME-Version`, `Content-Type`
- `Received-SPF`, `DKIM-Signature`

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Las cabeceras <code>Received</code> permiten reconstruir el <strong>recorrido completo</strong> del correo desde el remitente hasta el destinatario.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Servidor Postfix

## Instalación y configuración básica

---

## ¿Qué es Postfix?

> **Postfix** es un servidor de correo (MTA) de **software libre** creado como alternativa más rápida, sencilla de administrar y segura al veterano **Sendmail**.

### Datos clave

- Escrito por **Wietse Venema** en el Thomas J. Watson Research Center de IBM
- Es el MTA **por defecto** en muchas distribuciones Linux y en macOS
- Diseño **modular**: varios procesos pequeños con responsabilidades concretas
- En 2016 movía aproximadamente el **25%** del correo de Internet
- En desarrollo activo

---

## Instalación de Postfix

```bash
sudo apt update
sudo apt install postfix
```

### Opciones del instalador

- **Tipo de servidor**: `Internet Site` (vamos a enviar y recibir correo directamente)
- **System mail name**: el **dominio** que gestionará el servidor

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El "system mail name" se guarda en <code>/etc/mailname</code> y es el dominio con el que Postfix se identificará al enviar correo.</div>
</div>

---

## Estructura de configuración

| Archivo | Para qué sirve |
|:--|:--|
| `/etc/postfix/main.cf` | Archivo principal de configuración |
| `/etc/postfix/master.cf` | Define los procesos del servidor (ports, servicios) |
| `/etc/mailname` | Dominio con el que el servidor se identifica |
| `/etc/aliases` | Alias locales (mapeo de nombres a usuarios) |
| `/var/log/mail.log` | Registro de actividad del MTA |
| `/var/spool/postfix/` | Cola de mensajes pendientes |

```bash
systemctl status postfix
systemctl reload postfix
sudo journalctl -fu postfix@-
```

---

## Directivas principales en `main.cf`

| Directiva | Significado |
|:--|:--|
| **`mydestination`** | Dominios **propios**: el correo a usuarios de aquí se entrega localmente |
| **`relay_domains`** | Dominios para los que el servidor **reenvía** correo |
| **`mynetworks`** | IPs **autorizadas** a enviar mensajes a través del servidor |
| **`myorigin`** | Dominio con el que el servidor **firma** los envíos |
| **`relayhost`** | Servidor SMTP **intermedio** al que delegar el envío |
| **`inet_interfaces`** | Interfaces en las que escucha el servicio |

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Configurar <code>mynetworks</code> demasiado abierto convierte el servidor en un <strong>open relay</strong>: cualquiera podría enviar correo a través de él.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# Gestión del correo en el servidor

## Envío local, salida a Internet y recepción

---

## Casos que vamos a estudiar

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Desde el servidor

- **Caso 1** — Envío local entre usuarios del servidor
- **Caso 2** — Envío de correos a Internet
- **Caso 3** — Recepción de correos de Internet

</div>

<div class="card card-green">

### Desde un cliente remoto

- **Caso 4** — Recepción del correo del usuario
- **Caso 5** — Envío del correo del usuario

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Los tres primeros casos se resuelven con <strong>Postfix</strong>; los dos últimos requieren además un servidor <strong>IMAP/POP3</strong> y autenticación SMTP.</div>
</div>

---

## Caso 1 — envío local

Para mandar correos entre usuarios del propio servidor se usa el comando **`mail`** (paquete `bsd-mailx`):

```bash
debian@maquina:~$ mail root@localhost
```

El destinatario lo lee con la misma herramienta:

```
root@maquina:~# mail
"/var/mail/root": 1 message 1 unread
> U  1 debian@DOMINIO  Sat Jan 16 18:22  79/3597  Re: hola
& 1
```

### Comprobar el log

```bash
sudo journalctl -fu postfix@-
```

---

## Caso 2 — envío a Internet (desde el aula)

Desde el aula, los servidores no pueden enviar directamente: hay que **delegar** en un servidor de salida (relay).

```ini
# /etc/postfix/main.cf
relayhost = mail.gonzalonazareno.org
```

El relay (`macaco.gonzalonazareno.org`) se accede a través de la VPN:

```
$ dig mail.gonzalonazareno.org

mail.gonzalonazareno.org.      86400 IN  CNAME  macacovpn.gonzalonazareno.org.
macacovpn.gonzalonazareno.org. 86400 IN  A      172.200.0.1
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Sin <code>relayhost</code>, los correos saldrían (o intentarían salir) con la IP del aula, que está en redes filtradas y nunca se aceptarían en destino.</div>
</div>

---

## Caso 2 — envío a Internet (desde un VPS)

Con un servidor en la nube, **no necesitamos relay**: nuestra propia máquina envía el correo directamente.

### Para que funcione bien

- **IP estática y limpia**: que no esté en listas negras (RBL)
- Configurar en el DNS un **registro MX** apuntando a nuestro servidor
- Configurar un **registro SPF** (TXT) — hoy en día es prácticamente obligatorio

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Sin SPF (y a menudo también DKIM/DMARC), grandes proveedores como Gmail u Outlook <strong>rechazan o marcan como spam</strong> el correo entrante.</div>
</div>

---

## Que el correo "llegue a buen puerto"

| Mecanismo | Cómo funciona |
|:--|:--|
| **IP limpia** | La IP del servidor no debe estar en listas negras de spam |
| **SPF** (*Sender Policy Framework*) | Un registro **TXT** del dominio enumera qué IPs pueden enviar correo en su nombre |
| **DKIM** (*DomainKeys Identified Mail*) | El servidor firma cada mensaje con una clave privada; la pública se publica en DNS |
| **DMARC** | Política unificada que indica si se usa SPF, DKIM o ambos, y qué hacer si fallan |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Los tres trabajan juntos: <strong>SPF</strong> autentica la IP, <strong>DKIM</strong> autentica el contenido y <strong>DMARC</strong> dicta la política de aceptación o rechazo.</div>
</div>

---

## Caso 3 — recepción desde Internet

### Desde el aula

- El proceso completo se trabaja en el curso de correo electrónico
- Asegurarse de que el **registro MX** del dominio está bien configurado

### Desde un VPS

- Configurar Postfix con el dominio en `/etc/mailname`
- En el DNS:
  - **Registro A** que asocie el nombre del servidor a su IP pública
  - **Registro MX** del dominio apuntando a ese nombre

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El MX <strong>no</strong> debe apuntar directamente a una IP: por estándar tiene que apuntar a un nombre que después tenga su propio registro A.</div>
</div>

---

## Alias y redirecciones

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Alias

Redirige el correo de un usuario a **otro usuario del mismo servidor**.

```
# /etc/aliases
postmaster: root
webmaster:  debian
```

</div>

<div class="card card-green">

### Redirección

Reenvía el correo a una **cuenta externa**.

```
ventas: contacto@otrodominio.com
```

Requiere ejecutar `newaliases` tras modificar `/etc/aliases`.

</div>

</div>

---

## Usuarios virtuales

> Permite usar nombres de usuario que **no se corresponden** con cuentas del sistema.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### ¿Para qué sirve?

- Alojar **múltiples dominios** en un mismo servidor
- Cada dominio tiene sus propios buzones
- Separa cuentas de correo de cuentas del sistema

</div>

<div class="card card-green">

### Implementación

- Almacenamiento en **base de datos** (MySQL/PostgreSQL)
- O en **archivos planos** de texto
- Gestión habitual con **Postfix + Dovecot + base de datos**

</div>

</div>

---

## Lucha contra el spam

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### En el propio Postfix

- **`smtpd_*_restrictions`** — listas de comprobaciones a la entrada
- Filtrar **dominios** y **remitentes** sospechosos
- Restringir el **relay** a clientes autenticados
- Comprobar **listas negras** (RBL)

</div>

<div class="card card-green">

### Comprobaciones de autenticidad

- Verificar **SPF** del remitente
- Verificar **DKIM** del mensaje
- Aplicar la política **DMARC**

### Filtros adicionales

- **Antispam** (SpamAssassin, Rspamd)
- **Antivirus** (ClamAV)

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# Acceso desde clientes remotos

## Recepción y envío desde un MUA

---

## Caso 4 — recepción remota

Un cliente lejano (Thunderbird, móvil…) recoge el correo de su buzón mediante uno de estos protocolos:

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### POP3

- **Descarga** todos los correos al cliente
- Pensado para una sola máquina
- Por defecto borra los mensajes del servidor
- Puerto **110/tcp** (995/tcp con TLS)

</div>

<div class="card card-green">

### IMAP

- **Sincroniza** el estado entre cliente y servidor
- Permite consultar el correo desde varios dispositivos
- Soporta carpetas, etiquetas y banderas
- Puerto **143/tcp** (993/tcp con TLS)

</div>

</div>

---

## Tipos de buzón

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Buzón **mbox**

- **Todos los mensajes** en un único fichero
- Es el formato clásico
- Válido para **POP3** (donde se descargan todos los correos)

```
/var/mail/usuario
```

</div>

<div class="card card-green">

### Buzón **maildir**

- Cada mensaje en un **fichero independiente**, dentro de una carpeta
- Mucho mejor en concurrencia y rendimiento
- **Imprescindible para IMAP**

```
/home/usuario/Maildir/
    cur/  new/  tmp/
```

</div>

</div>

---

## IMAP cifrado

Las contraseñas y el contenido de los correos viajan por la red: hay que **cifrar** la conexión.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### IMAP + STARTTLS

- Comienza en **143/tcp** (texto plano)
- El cliente solicita **`STARTTLS`** y se negocia el cifrado
- A partir de ahí, todo va cifrado por la **misma conexión**

</div>

<div class="card card-green">

### IMAPS

- Cifrado **desde el inicio**
- Puerto **993/tcp**
- Conceptualmente equivalente a HTTPS

</div>

</div>

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Aunque hoy ambos cifran igual, conviene desactivar el modo no cifrado: dejar 143 sin TLS puede usarse para <strong>capturar credenciales</strong>.</div>
</div>

---

## Caso 5 — envío remoto

El cliente del usuario también necesita **enviar correos** a través de nuestro servidor. SMTP estándar (puerto 25) **no sirve** desde fuera: hay que usar las extensiones del envío autenticado.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### ESMTP + STARTTLS

- Puerto **587/tcp** (envío)
- Soporta extensiones: `STARTTLS` y `SMTP-AUTH`
- Autenticación con **SASL** (*Simple Authentication and Security Layer*)
- Es el método **recomendado** hoy

</div>

<div class="card card-green">

### SMTPS

- Puerto **465/tcp**
- Cifrado **desde el inicio**
- No es una extensión de SMTP: es una variante con TLS implícito
- Conceptualmente equivalente a HTTPS

</div>

</div>

---

## Esquema cliente — servidor

```
                                   Servidor de correo
                                ┌────────────────────────┐
   Cliente (MUA)                │                        │
   ─────────────                │  Postfix (MTA / LDA)   │
                                │       │                │
   ──── ESMTP/STARTTLS ────────►│       ▼                │
        587/tcp                 │   /var/mail/* o        │
                                │   ~/Maildir/*          │
                                │       ▲                │
   ◄──── IMAP / IMAPS ──────────│       │  Dovecot       │
        143/tcp · 993/tcp       │                        │
                                └────────────────────────┘
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Lo habitual es combinar <strong>Postfix</strong> (envío y entrega local) con <strong>Dovecot</strong> (acceso IMAP/POP3 y autenticación SASL para Postfix).</div>
</div>

---

## Resumen — puertos del correo

| Servicio | Puerto | Cifrado | Uso |
|:--|:--|:--|:--|
| SMTP entre servidores | 25/tcp | STARTTLS oportunista | MTA → MTA |
| Envío del cliente (submission) | **587/tcp** | STARTTLS + SASL | MUA → MTA |
| SMTPS | 465/tcp | TLS implícito | MUA → MTA (alternativo) |
| POP3 | 110/tcp | STARTTLS | Recepción |
| POP3S | 995/tcp | TLS implícito | Recepción |
| IMAP | 143/tcp | STARTTLS | Recepción |
| **IMAPS** | **993/tcp** | TLS implícito | Recepción |

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Correo electrónico → Postfix y Dovecot en la práctica

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI</span>
</div>
