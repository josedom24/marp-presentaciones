---
marp: true
theme: profesional
paginate: true
header: 'SRI · Unidad 1 — Configuración inicial de un servidor Linux'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Configuración inicial de un **servidor Linux**

## Acceso, identidad, red y resolución de nombres

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI · Servicios de Red e Internet</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Acceso seguro al servidor

## SSH con autenticación por clave pública

---

## ¿Por qué clave pública en lugar de contraseña?

La autenticación por contraseña es vulnerable a **ataques de fuerza bruta** y depende de que el usuario elija y custodie un secreto fuerte. La clave pública sustituye ese secreto por un **par criptográfico**: el servidor verifica la identidad sin que ninguna contraseña viaje por la red.

### Ventajas frente a la contraseña

- **Mayor seguridad**: no se transmite ningún secreto en la conexión
- **Resistencia a fuerza bruta** y a ataques de *sniffing*
- **Automatización**: scripts y tareas remotas sin intervención humana
- **Comodidad**: un único par de claves vale para muchos servidores

---

## Criptografía asimétrica: la idea fundamental

> Un par de claves matemáticamente vinculadas: lo cifrado con una sólo puede verificarse con la otra.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Clave privada

- **Permanece en el cliente**, nunca se comparte
- Se protege con permisos estrictos del sistema de ficheros
- Puede ir cifrada con una *passphrase* adicional
- Es la prueba de identidad del usuario

</div>

<div class="card card-green">

### Clave pública

- **Se instala en el servidor**, en cada cuenta autorizada
- Puede distribuirse libremente: no compromete la privada
- El servidor la usa para **verificar** al cliente
- Sólo autentica a quien posea la privada correspondiente

</div>

</div>

---

## ¿Cómo se establece la conexión?

1. El cliente solicita conectarse con un usuario del servidor
2. El servidor consulta las **claves públicas autorizadas** para esa cuenta
3. Lanza un **reto criptográfico** que sólo puede resolver quien tenga la clave privada
4. El cliente responde firmando el reto con su clave privada
5. El servidor verifica la firma con la clave pública y **autoriza el acceso**

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>El secreto (la clave privada) <strong>nunca abandona el cliente</strong>. El servidor sólo necesita la pública para verificar.</div>
</div>

---

## Herramientas en la práctica

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### `ssh-keygen`

- Genera el **par de claves** (privada y pública) en el cliente
- Se guarda por defecto en `~/.ssh/` (`id_rsa`, `id_ed25519`…)
- Permite proteger la clave privada con una **passphrase**

</div>

<div class="card card-green">

### `ssh-copy-id`

- Copia automáticamente la **clave pública** del cliente al servidor
- La añade al archivo `~/.ssh/authorized_keys` del usuario remoto
- Alternativa manual: copiar la clave pública a `~/.ssh/authorized_keys` en el servidor

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>El archivo <code>~/.ssh/authorized_keys</code> contiene <strong>todas las claves públicas</strong> que pueden iniciar sesión en esa cuenta.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Administración delegada

## Privilegios controlados con sudo

---

## El problema del usuario root

Trabajar directamente como **root** rompe el principio de **mínimo privilegio**: un error tipográfico puede dañar el sistema, y todas las acciones quedan bajo un único identificador, dificultando la auditoría.

### ¿Qué aporta `sudo`?

- Permite que **usuarios concretos** ejecuten tareas administrativas con sus propias credenciales
- **No es necesario compartir** la contraseña de root
- Cada acción privilegiada queda **registrada con el usuario real** que la ejecuta
- Se puede limitar **qué comandos** puede ejecutar cada usuario

---

## Ventajas frente a iniciar sesión como root

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Auditoría y trazabilidad

- Cada comando privilegiado queda registrado con el usuario que lo lanzó
- Permite reconstruir quién hizo qué y cuándo
- Útil en equipos con varios administradores

</div>

<div class="card card-green">

### Control granular

- Se puede autorizar a un usuario sólo a comandos concretos
- Se decide si requiere o no contraseña
- Se asignan permisos por **usuario** o por **grupo**
- Aplica el principio de **mínimo privilegio**

</div>

</div>

---

## Configuración: sudoers y sudoers.d

<div class="cols-2" style="margin-top:0.8rem">

<div>

### `/etc/sudoers`

- Archivo principal de configuración
- Define qué usuarios o grupos pueden ejecutar qué comandos con privilegios
- Se edita con una herramienta específica que **valida la sintaxis** antes de guardar
- Un error en este archivo puede dejar el sistema sin acceso administrativo

</div>

<div>

### `/etc/sudoers.d/`

- Directorio con archivos **modulares** de configuración
- Cada archivo añade reglas sin tocar el principal
- Facilita la gestión por **automatización** (Ansible, paquetes, scripts)
- Permite separar reglas por usuario, servicio o despliegue

</div>

</div>

<div class="alerta alerta-warning" style="margin-top:0.8rem">
<span>⚠️</span><div>Editar siempre con la herramienta específica para evitar dejar el archivo corrupto.</div>
</div>

---

## Comandos habituales con sudo

<div class="cols-2" style="margin-top:0.8rem">

<div>

### Habilitar a un usuario

```bash
# Añadir al grupo sudo
sudo usermod -aG sudo nombre_usuario

# Editar la configuración con validación
sudo visudo
```

`visudo` abre `/etc/sudoers` en un editor y **comprueba la sintaxis** antes de guardar.

</div>

<div>

### Ejemplos dentro de `sudoers`

```
# Usuario concreto, sin contraseña
nombre_usuario ALL=(ALL) NOPASSWD:ALL

# Cualquier miembro del grupo admin
%admin ALL=(ALL) ALL
```

El símbolo `%` indica un **grupo**. `NOPASSWD` evita pedir la contraseña al ejecutar `sudo`.

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# Identidad del sistema

## Hostname y FQDN

---

## Nombre del equipo en la red

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Hostname

- **Nombre corto** del equipo
- Identifica al sistema dentro de su red local
- Ejemplo: `sauron`

</div>

<div class="card card-green">

### FQDN — *Fully Qualified Domain Name*

- **Nombre completo**, incluyendo el dominio
- Identifica al equipo de forma **inequívoca** en Internet
- Ejemplo: `sauron.mordor.com`
- Necesario para servicios como correo, web o certificados TLS

</div>

</div>

---

## Archivos implicados

<div class="cols-2" style="margin-top:0.8rem">

<div>

### `/etc/hostname`

- Contiene **únicamente el nombre corto** del equipo
- Es la fuente persistente del hostname tras un reinicio
- Una sola línea, sin dominio ni IP

</div>

<div>

### `/etc/hosts`

- Tabla de **resolución estática** local
- Mapea direcciones IP con nombres (corto y FQDN)
- Permite que el propio equipo se resuelva sin depender del DNS
- La entrada para el equipo debe incluir **FQDN y hostname**:

```
127.0.1.1   sauron.mordor.com sauron
```

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>La identidad del sistema requiere coherencia entre <strong>ambos archivos</strong>: si discrepan, distintos servicios verán nombres distintos.</div>
</div>

---

## `hostnamectl`: gestión centralizada del nombre

`hostnamectl` es la herramienta moderna (parte de **systemd**) para consultar y modificar la identidad del sistema **sin editar archivos a mano**.

### Para qué sirve

- Cambia el hostname **de forma persistente**: actualiza `/etc/hostname` y aplica el cambio al sistema en ejecución, sin reiniciar
- Distingue entre varios tipos de nombre: *static*, *transient* y *pretty*
- Muestra información útil del equipo: kernel, arquitectura, ID de máquina, virtualización…
- Comandos típicos: `hostnamectl` (consulta) y `hostnamectl set-hostname sauron` (modificación)

<div class="alerta alerta-warning" style="margin-top:0.8rem">
<span>⚠️</span><div>Aunque <code>hostnamectl</code> actualiza el hostname, <strong>no modifica</strong> <code>/etc/hosts</code>: ese archivo hay que ajustarlo aparte.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# Configuración de red

## Interfaces, direccionamiento y mecanismos de gestión

---

## Conceptos básicos

Cada equipo conectado a una red necesita un conjunto mínimo de parámetros para comunicarse:

- **Interfaz de red**: dispositivo físico o virtual por el que circula el tráfico
- **Dirección IP**: identificador único dentro de la red
- **Máscara de red**: define el rango de direcciones de la subred
- **Puerta de enlace** (*gateway*): equipo al que se envía el tráfico hacia otras redes
- **Servidores DNS**: traducen nombres a direcciones IP

---

## Direccionamiento estático vs dinámico

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Estático

- La IP se **asigna manualmente** y se mantiene fija
- Apropiado para **servidores** y equipos que ofrecen servicios
- Garantiza que la dirección no cambia tras reinicios
- Requiere planificación para evitar conflictos

</div>

<div class="card card-green">

### Dinámico (DHCP)

- Un servidor **DHCP asigna** la configuración al arrancar
- La dirección puede **cambiar** entre concesiones
- Apropiado para **clientes** y equipos móviles
- Centraliza la administración de la red

</div>

</div>

---

## Mecanismos de configuración en Linux

Linux ofrece **varias herramientas** para gestionar la red. La que se utiliza depende de la distribución y del rol del equipo (servidor, escritorio, portátil…).

| Herramienta | Estilo | Uso típico |
|:------------|:-------|:-----------|
| **ifupdown** | Tradicional, archivos planos | Debian clásico, servidores simples |
| **NetworkManager** | Dinámico, GUI + CLI | Escritorios, portátiles, Wi-Fi, VPN |
| **systemd-networkd** | Declarativo, integrado en systemd | Servidores modernos, contenedores |
| **Netplan** | Declarativo en YAML | Ubuntu (traduce a NM o systemd-networkd) |

---

## ifupdown y NetworkManager

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### ifupdown

- Herramienta **tradicional** de Debian
- Archivos de configuración en `/etc/network/`
- Integrada con `systemd` mediante un servicio que la activa al arrancar
- Sencilla y predecible: ideal para **servidores con configuración estable**

</div>

<div class="card card-green">

### NetworkManager

- Pensada para **gestión dinámica** de la red
- Soporta **Wi-Fi, VPN, redes móviles** y conexiones cambiantes
- Ofrece interfaz gráfica, *applet* de escritorio y herramientas en línea de comandos
- Habitual en **escritorios** y en distribuciones tipo Fedora

</div>

</div>

---

## systemd-networkd y Netplan

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-purple">

### systemd-networkd

- **Forma parte de systemd**: integrada en el sistema base
- Configuración **declarativa** en `/etc/systemd/network/`
- Ligera y eficiente: pensada para **servidores** y entornos automatizados
- Combinada con `systemd-resolved` cubre red y DNS

</div>

<div class="card card-yellow">

### Netplan

- Herramienta de **Ubuntu** (también disponible en otras distribuciones)
- Configuración declarativa en archivos **YAML**
- No gestiona la red directamente: **traduce** la configuración a NetworkManager o systemd-networkd
- Unifica la sintaxis entre escritorio y servidor

</div>

</div>

---

## ¿Qué usa cada distribución?

| Distribución | Mecanismo principal | Alternativa habitual |
|:-------------|:--------------------|:---------------------|
| **Debian** (servidor) | ifupdown | NetworkManager |
| **Debian** (escritorio) | NetworkManager | ifupdown |
| **Ubuntu** | Netplan → systemd-networkd | NetworkManager |
| **Fedora / RHEL** | NetworkManager | systemd-networkd |

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Conviene <strong>no mezclar</strong> mecanismos en el mismo equipo: dos gestores intentando configurar la misma interfaz provocan conflictos imprevisibles.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">05</p>

# Resolución de nombres

## DNS, NSS y systemd-resolved

---

## ¿Qué es la resolución de nombres?

> Traducir un nombre legible (`www.ejemplo.com`) en la **dirección IP** que el equipo necesita para comunicarse.

### Ideas clave

- Las redes funcionan con direcciones IP, las personas con nombres
- El **DNS** (*Domain Name System*) es el servicio jerárquico que realiza esa traducción a escala global
- En Linux la resolución no se delega únicamente al DNS: existen **fuentes locales** y un mecanismo que decide en qué orden consultarlas

---

## NSS — Name Service Switch

**NSS** es la biblioteca del sistema que decide **dónde y en qué orden** se buscan distintos tipos de información: usuarios, grupos, hosts…

### `/etc/nsswitch.conf`

- Para cada categoría especifica las **fuentes consultadas** y el orden
- En la línea `hosts:` define cómo se resuelven los nombres
- Las fuentes habituales son: archivos locales, DNS, mDNS, resolved…

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>El orden típico es: <strong>archivos locales primero, DNS después</strong>. Permite sobreescribir un nombre puntualmente sin tocar el DNS global.</div>
</div>

---

## Archivos clásicos: hosts y resolv.conf

<div class="cols-2" style="margin-top:0.8rem">

<div>

### `/etc/hosts`

- Tabla **estática** de nombres y direcciones IP
- Resolución **inmediata**, sin red
- Útil para nombres locales, entornos de prueba o sobreescrituras puntuales
- Se consulta antes que el DNS si así lo indica NSS

</div>

<div>

### `/etc/resolv.conf`

- Lista de **servidores DNS** que se consultarán
- Puede incluir **dominios de búsqueda** y opciones avanzadas
- En sistemas modernos suele estar **gestionado automáticamente** por NetworkManager, systemd-resolved o el cliente DHCP
- Editarlo a mano puede ser **inútil** si lo regenera otro servicio

</div>

</div>

---

## systemd-resolved

Servicio moderno que **centraliza** la resolución de nombres en el sistema.

### Qué aporta

- **Caché local** de respuestas → mejor rendimiento
- Servidor DNS local de reenvío en una dirección de *loopback*
- Integración con **NSS** mediante módulos propios
- Combina resolución estática (`/etc/hosts`), DNS remoto y nombres de contenedores
- Herramientas propias para consultar el estado, vaciar la caché y diagnosticar

---

## Herramientas de consulta: cuidado con lo que prueban

No todas las herramientas siguen el mismo camino que el sistema operativo cuando una aplicación normal resuelve un nombre.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Consultan **directamente** al DNS

- `dig`, `host`, `nslookup`
- **No respetan** el orden de NSS
- Útiles para **diagnosticar el DNS** en sí mismo

</div>

<div class="card card-green">

### Respetan el orden de NSS

- `getent hosts` / `getent ahosts`
- Reproducen el camino real de una aplicación
- Útiles para verificar la **resolución completa** del sistema

</div>

</div>

<div class="alerta alerta-warning" style="margin-top:0.8rem">
<span>⚠️</span><div>Si <code>dig</code> resuelve un nombre pero <code>ping</code> no, lo más probable es que el problema esté en NSS o en <code>/etc/hosts</code>, no en el DNS.</div>
</div>

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Configuración inicial → Servicios de red

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI 25/26</span>
</div>
