---
marp: true
title: Configuración inicial de un servidor Linux
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

- Archivo principal de configuración de sudo
- Define qué usuarios/grupos pueden ejecutar qué comandos con privilegios
- Se edita con **`visudo`**, que valida la sintaxis antes de guardar

</div>

<div>

### `/etc/sudoers.d/`

- Directorio con archivos **modulares** de configuración
- Cada archivo añade reglas sin tocar el principal
- Facilita la gestión por **automatización** (Ansible, paquetes, scripts)

</div>

</div>

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Usar siempre <strong>visudo</strong>: un error en sudoers puede dejar el sistema sin acceso administrativo.</div>
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

## ifupdown

- Herramienta **tradicional** de Debian
- Archivos de configuración en `/etc/network/interfaces` y `/etc/network/interfaces.d/`
- Integrada con `systemd` mediante el servicio `networking.service`
- Sencilla y predecible: ideal para **servidores con configuración estable**

### Comandos principales

```bash
ifup eth0          # Levanta una interfaz según la configuración
ifdown eth0        # La desactiva
ip a               # Muestra interfaces y direcciones IP
ip route           # Muestra la tabla de rutas
systemctl restart networking   # Aplica cambios en interfaces
```

---

## NetworkManager

- Pensada para **gestión dinámica** de la red
- Soporta **Wi-Fi, VPN, redes móviles** y conexiones cambiantes
- Habitual en **escritorios** y en distribuciones tipo Fedora
- Almacena las conexiones como *perfiles* en `/etc/NetworkManager/system-connections/`

### Comandos principales

```bash
nmcli device status            # Estado de cada interfaz
nmcli connection show          # Lista de conexiones definidas
nmcli connection up <perfil>   # Activa una conexión
nmtui                          # Interfaz interactiva en modo texto
nm-applet                      # Applet gráfico de escritorio
```

---

## systemd-networkd

- **Forma parte de systemd**: integrada en el sistema base
- Configuración **declarativa** en archivos `.network` y `.link` dentro de `/etc/systemd/network/`
- Ligera y eficiente: pensada para **servidores** y entornos automatizados
- Combinada con `systemd-resolved` cubre red y DNS

### Comandos principales

```bash
systemctl enable --now systemd-networkd   # Activar el servicio
systemctl restart systemd-networkd        # Aplicar cambios
networkctl                                # Estado de las interfaces
networkctl status eth0                    # Detalle de una interfaz
```

---

## Netplan

- Herramienta de **Ubuntu** (también disponible en otras distribuciones)
- Configuración declarativa en archivos **YAML** en `/etc/netplan/`
- No gestiona la red directamente: **traduce** la configuración a NetworkManager o systemd-networkd
- Unifica la sintaxis entre escritorio y servidor

### Comandos principales

```bash
netplan generate   # Genera la configuración del backend
netplan apply      # Aplica los cambios
netplan try        # Prueba la configuración con rollback automático
```

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

## Para profundizar — Configuración de red

Artículos del blog con explicaciones detalladas y ejemplos de cada herramienta:

- [Configuración de red en sistemas Linux — **ifupdown**](https://www.josedomingo.org/pledin/2025/01/configuracion-red-linux-ifupdown/)
- [Configuración de red en sistemas Linux — **NetworkManager**](https://www.josedomingo.org/pledin/2025/01/configuracion-red-linux-networkmanager/)
- [Configuración de red en sistemas Linux — **systemd-networkd**](https://www.josedomingo.org/pledin/2025/03/configuracion-red-linux-systemd-networkd/)
- [Configuración de red en sistemas Linux — **Netplan**](https://www.josedomingo.org/pledin/2025/04/configuracion-red-linux-netplan/)

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
- Servidor DNS local de reenvío en `127.0.0.53`
- Integración con **NSS** mediante módulos propios (`resolve`, `myhostname`, `mymachines`)
- Combina resolución estática (`/etc/hosts`), DNS remoto y nombres de contenedores

---

## Comandos de `resolvectl`

```bash
resolvectl status                  # Estado del servicio y DNS configurados
resolvectl query www.ejemplo.com   # Consulta usando systemd-resolved
resolvectl dns eth0 1.1.1.1        # Establece servidores DNS para una interfaz
resolvectl flush-caches            # Vacía la caché de resolución
```

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div><code>resolvectl</code> usa el flujo del propio sistema (caché incluida), por lo que sus respuestas reflejan lo que verá una aplicación real.</div>
</div>

---

## Herramientas de consulta: cuidado con lo que prueban

No todas las herramientas siguen el mismo camino que el sistema operativo cuando una aplicación normal resuelve un nombre.

- **Consultas directas al DNS**: `dig`, `host`, `nslookup`
   *No respetan* el orden de NSS — diagnostican el DNS en sí mismo
- **Consultas que respetan NSS**: `getent hosts`, `getent ahosts`
   Reproducen el camino real de una aplicación

<div class="alerta alerta-warning" style="margin-top:0.8rem">
<span>⚠️</span><div>Si <code>dig</code> resuelve un nombre pero <code>ping</code> no, lo más probable es que el problema esté en NSS o en <code>/etc/hosts</code>, no en el DNS.</div>
</div>

---

## Comandos de consulta DNS

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Directos al DNS

```bash
dig ejemplo.com
host ejemplo.com
nslookup ejemplo.com
```

</div>

<div class="card card-green">

### Respetando NSS

```bash
getent hosts ejemplo.com
getent ahosts ejemplo.com
```

</div>

</div>

---

## Para profundizar — Resolución de nombres

- [Resolución de nombres de dominios en sistemas Linux](https://www.josedomingo.org/pledin/2024/02/resolucion-nombres-linux/)

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>El artículo desarrolla en detalle el funcionamiento de NSS, <code>/etc/hosts</code>, <code>/etc/resolv.conf</code> y <code>systemd-resolved</code>, con ejemplos prácticos.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">06</p>

# Router Linux, SNAT y DNAT

## Enrutamiento y traducción de direcciones con iptables

---

## ¿Para qué se usa un router Linux?

Un equipo Linux puede actuar como **router** para conectar dos o más redes y permitir el enrutamiento de paquetes entre ellas. Es habitual en redes domésticas, laboratorios docentes o firewalls personalizados.

### Funciones que puede realizar

- **NAT** — traducción de direcciones de red
- **Filtrado de paquetes** — control del tráfico permitido
- **Redirección de puertos** — exponer servicios internos al exterior
- **Compartir conexión a Internet** — dar salida a una red privada

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Linux dispone de todo lo necesario en el propio kernel: sólo hay que <strong>activar el reenvío</strong> y definir las reglas adecuadas con <code>iptables</code>.</div>
</div>

---

## Habilitar el reenvío de IPs (IP Forwarding)

Por defecto el kernel **no reenvía** paquetes entre interfaces. Para que un equipo actúe como router hay que activarlo.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Activación temporal

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Se pierde tras reiniciar el sistema.

</div>

<div class="card card-green">

### Activación persistente (Debian 13)

En **Debian 13** ya no se proporciona `/etc/sysctl.conf`: la configuración se organiza de forma modular en `etc/sysctl.d/`. 

```bash
echo "net.ipv4.ip_forward = 1" > /etc/sysctl.d/99-router.conf
```

Aplicar los cambios: 

```bash
sysctl --system`
```

</div>

</div>

---

## SNAT — Source NAT

Permite que una red local con **direcciones privadas** salga a Internet usando la **IP pública** del router. Cambia la **IP de origen** de los paquetes salientes.

### Regla con `iptables`

```bash
iptables -t nat -A POSTROUTING -o eth0 -s 192.168.0.0/24 -j SNAT --to-source 192.0.2.1
```

- `-o eth0` — interfaz de salida (hacia Internet)
- `-s 192.168.0.0/24` — red de origen que tendrá acceso a Internet
- `--to-source 192.0.2.1` — IP pública del router

---

## SNAT con IP dinámica: MASQUERADE

Cuando la IP pública del router **no es fija** (por ejemplo, asignada por DHCP del proveedor), conviene usar `MASQUERADE` en lugar de `SNAT`.

### Regla equivalente

```bash
iptables -t nat -A POSTROUTING -o eth0 -s 192.168.0.0/24 -j MASQUERADE
```

- Toma **automáticamente** la IP de la interfaz de salida
- No hay que especificar la IP pública
- Más cómodo cuando la dirección puede cambiar

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div><code>MASQUERADE</code> es una variante de <code>SNAT</code> pensada para enlaces con IP dinámica: paga un pequeño coste extra por consultar la IP cada vez.</div>
</div>

---

## DNAT — Destination NAT

Se usa para **redirigir tráfico entrante** desde el exterior hacia una máquina de la red interna. Cambia la **IP de destino** de los paquetes.

### Regla con `iptables`

```bash
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:80
```

- `-i eth0` — interfaz por la que entra el tráfico
- `-p tcp --dport 80` — protocolo y puerto de destino
- `--to-destination 192.168.1.100:80` — máquina interna a la que se redirige

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Se aplica en la cadena <code>PREROUTING</code> porque la decisión de redirigir se toma <strong>antes</strong> de enrutar el paquete.</div>
</div>

---

## SNAT vs DNAT: comparación

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### SNAT — salida

- Cambia la **IP de origen**
- Cadena `POSTROUTING`
- Sentido: **red interna → Internet**
- Comparte una IP pública entre varios equipos
- Caso típico: dar Internet a una LAN

</div>

<div class="card card-green">

### DNAT — entrada

- Cambia la **IP de destino**
- Cadena `PREROUTING`
- Sentido: **Internet → red interna**
- Publica un servicio interno hacia el exterior
- Caso típico: redirección de puertos a un servidor

</div>

</div>

---

## Hacer las reglas de iptables persistentes

Las reglas de `iptables` **no sobreviven a un reinicio**. Hay que guardarlas para que se restauren automáticamente al arrancar.

### Con el paquete `iptables-persistent` (Debian / Ubuntu)

```bash
# Guardar las reglas actuales
iptables-save > /etc/iptables/rules.v4
```

- Las reglas se restauran **automáticamente** al iniciar el sistema
- Existe el equivalente para IPv6 en `/etc/iptables/rules.v6`

<div class="alerta alerta-warning" style="margin-top:0.8rem">
<span>⚠️</span><div>Después de modificar reglas, no olvides <strong>volver a guardarlas</strong>: si no, los cambios se perderán en el próximo reinicio.</div>
</div>

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Configuración inicial → Servicios de red

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI</span>
</div>
