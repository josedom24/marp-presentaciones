---
marp: true
title: Contenedores LXC
theme: profesional
paginate: true
header: 'IV · Contenedores LXC'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Contenedores **LXC**

## Virtualización ligera en Linux

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 IV · Infraestructura Virtual</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Introducción a LXC

## Concepto, instalación y ciclo de vida

---

## ¿Qué es LXC?

> **LinuX Containers (LXC)** es una tecnología de **virtualización ligera**: sobre el núcleo del sistema operativo se ejecuta una capa de virtualización que permite que existan **múltiples instancias aisladas de espacios de usuario**. A estas instancias las llamamos **contenedores**.

### Características principales

- Pertenece a los **contenedores de sistema**: su gestión y ciclo de vida son similares a los de una máquina virtual tradicional
- Se accede a ellos por SSH, se instalan servicios, se actualizan…
- Lo desarrolla la comunidad del proyecto **Linux Containers**; página oficial: `linuxcontainers.org`

---

## Instalación

```bash
apt install lxc
```

Junto a `lxc` se instalan sus paquetes **recomendados**; dos de ellos hacen falta en el curso:

- **`apparmor`**: confina los contenedores con el perfil que genera LXC (`lxc.apparmor.profile = generated`)
- **`lxcfs`**: hace que `free` o `/proc/cpuinfo` muestren, dentro del contenedor, los recursos limitados

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Sin <code>apparmor</code>, el contenedor no arranca: <code>lxc-start</code> da <code>Failed to receive the container state</code> y, en primer plano (<code>lxc-start -n contenedor1 -F</code>), <code>Cannot use generated profile: apparmor_parser not available</code>. Se soluciona con <code>apt install apparmor</code>.</div>
</div>

---

## Tecnologías del kernel que lo hacen posible

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Grupos de control (*cgroups*)

- En Debian se usa **cgroups v2**
- **Limitan el uso de recursos** de un proceso y sus hijos: memoria, CPU, E/S de disco…

</div>

<div class="card card-green">

### Espacios de nombres (*namespaces*)

- Proporcionan una **vista aislada** del sistema a cada contenedor
- Aíslan: **interfaces de red**, procesos, usuarios, sistema de ficheros…

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>A diferencia de las MV, todos los contenedores <strong>comparten el kernel del host</strong>. No hay hipervisor: el aislamiento lo provee el propio kernel.</div>
</div>

---

## Contenedores privilegiados y no privilegiados

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Privilegiados

- Se crean y se ejecutan como **`root`**
- Configuración en `/var/lib/lxc/`
- Son los que **usamos en el curso**

</div>

<div class="card card-green">

### No privilegiados

- Se crean y se ejecutan con un **usuario normal**
- Más seguros: el `root` del contenedor no es el `root` del host

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Todos los comandos <code>lxc-*</code> de esta presentación se ejecutan como <code>root</code>.</div>
</div>

---

## Creación de contenedores — plantilla `download`

```bash
lxc-create -n contenedor1 -t download
```

`-n`: nombre del contenedor · `-t download`: descarga la imagen del servidor de imágenes

1. Muestra la **lista de distribuciones** disponibles
2. Pide la **distribución**, la **versión** y la **arquitectura**
3. La primera vez descarga el sistema de ficheros comprimido (**plantilla**) y lo guarda en caché en **`/var/cache/lxc`**
4. Se descomprime la plantilla y se crea el sistema de ficheros del contenedor en **`/var/lib/lxc/contenedor1/rootfs/`**
5. Los siguientes contenedores con la misma distribución, versión y arquitectura se crean **más rápido**: no se vuelve a descargar

---

## Creación no interactiva

Indicamos la distribución, la versión y la arquitectura después de `--`:

```bash
lxc-create -n contenedor2 -t download -- -d debian -r trixie -a amd64
```

| Opción | Significado |
|:--|:--|
| `-d` | Distribución (*distribution*) |
| `-r` | Versión (*release*) |
| `-a` | Arquitectura (*arch*) |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Las imágenes disponibles están en el servidor <code>images.linuxcontainers.org</code>.</div>
</div>

---

## Gestión básica de contenedores

```bash
lxc-ls [-f]                   # Listar contenedores (-f: con detalles)

lxc-start  contenedor1        # Arrancar
lxc-stop   contenedor1        # Parar
lxc-stop -r contenedor1       # Reiniciar

lxc-attach contenedor1             # Consola interactiva
lxc-attach contenedor1 -- ls -al   # Ejecutar un comando (contenedor arrancado)
lxc-execute contenedor1 -- ls -al  # Ejecutar un comando (contenedor parado)

lxc-destroy [-f] contenedor1  # Eliminar contenedor
```

```bash
$ lxc-ls -f
NAME        STATE   AUTOSTART GROUPS IPV4       IPV6 UNPRIVILEGED
contenedor1 RUNNING 0         -      10.0.3.180 -    false
```

---

## Información de un contenedor

```bash
lxc-info contenedor1
```

| Opción | Muestra |
|:--|:--|
| `-i` | Solo la **dirección IP** |
| `-S` | **Estadísticas** de uso: CPU, E/S de disco, memoria y datos enviados y recibidos por la red |
| `-s` | El **estado** del contenedor |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>La columna <code>UNPRIVILEGED</code> de <code>lxc-ls -f</code> indica si el contenedor es no privilegiado: en nuestro caso, <code>false</code>.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Configuración de LXC

## Fichero de configuración y límite de recursos

---

## Ficheros de configuración

| Fichero | Alcance |
|:--|:--|
| `/etc/lxc/default.conf` | **Plantilla** de configuración: la usan los contenedores **nuevos** |
| `/var/lib/lxc/<nombre>/config` | Configuración **de cada contenedor** (se genera a partir de la plantilla) |

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Si cambiamos <code>/etc/lxc/default.conf</code>, los <strong>nuevos contenedores</strong> tendrán la configuración indicada; los que ya existen no cambian.</div>
</div>

---

## Fichero de configuración de `contenedor1`

`/var/lib/lxc/contenedor1/config`:

```
# Distribution configuration
lxc.include = /usr/share/lxc/config/common.conf
lxc.arch = linux64

# Container specific configuration
lxc.apparmor.profile = generated
lxc.apparmor.allow_nesting = 1
lxc.rootfs.path = dir:/var/lib/lxc/contenedor1/rootfs
lxc.uts.name = contenedor1

# Network configuration
lxc.net.0.type = veth
lxc.net.0.link = lxcbr0
lxc.net.0.flags = up
```

---

## Parámetros importantes

| Parámetro | Significado |
|:--|:--|
| `lxc.rootfs.path` | Directorio con el **sistema de ficheros** del contenedor (cada contenedor tiene uno completo) |
| `lxc.uts.name` | **Nombre de host** (*hostname*) del contenedor |
| `lxc.net.0.*` | Configuración de la **interfaz de red**: conectada al Linux Bridge `lxcbr0` (`lxc.net.0.link`) |
| `lxc.start.auto = 1` | El contenedor se **inicia automáticamente** al iniciar el host |

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Para que una modificación tenga efecto <strong>hay que reiniciar el contenedor</strong>.</div>
</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Todos los parámetros: <code>linuxcontainers.org/lxc/manpages/man5/lxc.container.conf.5.html</code></div>
</div>

---

## Limitar los recursos — *cgroups*

Por defecto, un contenedor puede usar **toda la CPU, la RAM y el disco** del host. Los **grupos de control** (*cgroups*) permiten limitarlos.

Limitamos la memoria a **512 MB** y el uso a **una CPU** (la CPU 0) en `/var/lib/lxc/contenedor1/config`:

```
lxc.cgroup2.memory.max = 512M
lxc.cgroup2.cpuset.cpus = 0
```

Reiniciamos el contenedor:

```bash
lxc-stop contenedor1
lxc-start contenedor1
```

---

## Limitar los recursos — comprobación

```bash
$ lxc-attach contenedor1 -- free -h
               total        used        free      shared  buff/cache   available
Mem:           512Mi       6.0Mi       505Mi       0.0Ki       0.0Ki       505Mi

$ lxc-attach contenedor1 -- cat /proc/cpuinfo
processor	: 0
...
```

- **Memoria**: `free -h` muestra un total de **512Mi**, no la memoria del host
- **CPU**: en `/proc/cpuinfo` aparece **un solo procesador**

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Dentro del contenedor, <code>free</code> y <code>/proc/cpuinfo</code> muestran los límites gracias a <strong>LXCFS</strong> (paquete <code>lxcfs</code>, recomendado al instalar <code>lxc</code>). Sin él, verían los recursos del host aunque el límite se aplique.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# Redes en LXC

## Bridges, red por defecto y varias interfaces

---

## Red por defecto — `lxcbr0`

LXC ofrece varios mecanismos de red; en el curso usamos conexiones de tipo **bridge**.

Por defecto el contenedor está conectado (`lxc.net.0.link = lxcbr0`) al bridge **`lxcbr0`**, que crea el servicio **`lxc-net`**:

- Red de tipo **NAT**, con direccionamiento **`10.0.3.0/24`**
- El host ofrece los servicios de **DHCP** y **DNS** (`10.0.3.1`)
- Los contenedores obtienen IP automáticamente y tienen salida a Internet

```bash
$ lxc-ls -f
NAME        STATE   AUTOSTART GROUPS IPV4       IPV6 UNPRIVILEGED
contenedor1 RUNNING 1         -      10.0.3.180 -    false
```

---

## Red por defecto — comprobaciones en el contenedor

```bash
root@contenedor1:~# ip a
2: eth0@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP ...
    inet 10.0.3.180/24 brd 10.0.3.255 scope global dynamic eth0
root@contenedor1:~# ip r
default via 10.0.3.1 dev eth0
10.0.3.0/24 dev eth0 proto kernel scope link src 10.0.3.180
root@contenedor1:~# cat /etc/resolv.conf
nameserver 10.0.3.1
root@contenedor1:~# ping www.josedomingo.org
64 bytes from ns330309.ip-37-187-119.eu (37.187.119.60): icmp_seq=1 ttl=49 time=28.7 ms
```

1. IP **`10.0.3.180`**, obtenida por DHCP
2. Puerta de enlace **`10.0.3.1`**: la máquina física
3. La máquina física es también el **servidor DNS**
4. Hay **resolución** y **acceso al exterior** (instalando antes `iputils-ping`)

---

## Conectar a un bridge existente

Podemos conectar el contenedor a cualquier bridge, **creado por nosotros o por libvirt**. Por ejemplo, a la red `default` de libvirt (bridge **`virbr0`**):

1. Paramos el contenedor
2. Cambiamos en su configuración: `lxc.net.0.link = virbr0`
3. Lo iniciamos y comprobamos:

```bash
$ lxc-ls -f
NAME        STATE   AUTOSTART GROUPS IPV4            IPV6 UNPRIVILEGED
contenedor1 RUNNING 1         -      192.168.122.228 -    false
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Si el cambio se hace en <code>/etc/lxc/default.conf</code>, <strong>todos los contenedores que creemos a partir de entonces</strong> se conectarán a la red <code>default</code>.</div>
</div>

---

## Conectar un contenedor a varias redes

Cada interfaz se declara con **`lxc.net.X`**: `0` la primera, `1` la segunda…

```
lxc.net.0.type = veth
lxc.net.0.link = lxcbr0
lxc.net.0.flags = up

lxc.net.1.type = veth
lxc.net.1.link = virbr0
lxc.net.1.flags = up
```

Paramos y arrancamos el contenedor (`lxc-stop` y `lxc-start`).

<div class="alerta alerta-warning" style="margin-top:0.5rem">
<span>⚠️</span><div>La nueva interfaz hay que configurarla <strong>dentro del contenedor</strong>, según el mecanismo de configuración de red que tenga instalado (<em>ifupdown</em>, <em>systemd-networkd</em>, <em>netplan</em>…).</div>
</div>

---

## Instalación de netplan en lxc

En los contenedores **Debian** la red la gestiona **`systemd-networkd`**; podemos configurarla con **netplan** (en los contenedores **Ubuntu** ya viene instalado):

```bash
apt install netplan.io
# Creamos /etc/netplan/10-lxc.yaml; debe tener permisos restrictivos:
chmod 600 /etc/netplan/10-lxc.yaml
```

```yaml
# /etc/netplan/10-lxc.yaml
network:
  version: 2
  ethernets:
    eth1:
      addresses: [10.0.0.10/24]
```

Se aplica con `netplan apply`.

---

## `netplan apply` en el contenedor

`netplan apply` da error porque en el contenedor **no está `udev`**. Se puede simular que está:

```bash
mkdir -p /usr/local/bin
printf '#!/bin/sh\nexit 0\n' > /usr/local/bin/udevadm
chmod +x /usr/local/bin/udevadm
```

Después, `netplan apply` ya funciona.

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div><code>/usr/local/bin</code> va antes que <code>/usr/bin</code> en el <code>PATH</code>: netplan encuentra este <code>udevadm</code>, que no hace nada y termina sin error.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# Almacenamiento en LXC

## Montar directorios del host

---

## Montar un directorio del host

Montamos `/opt/contenedor1` del host en `/srv/www` del contenedor:

1. Crear el punto de montaje **dentro del contenedor**:

```bash
root@contenedor1:~# mkdir /srv/www
```

2. Añadir la entrada en `/var/lib/lxc/contenedor1/config`:

```
lxc.mount.entry=/opt/contenedor1 srv/www none bind 0 0
```

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>La ruta de destino es <strong>relativa</strong> (sin la barra inicial) al sistema de ficheros del contenedor: <code>/var/lib/lxc/contenedor1/rootfs/</code>.</div>
</div>

---

## Montar un directorio del host — comprobación

3. Reiniciar el contenedor y comprobar el montaje:

```bash
$ lxc-stop contenedor1
$ lxc-start contenedor1
$ lxc-attach contenedor1
root@contenedor1:~# cd /srv/www
root@contenedor1:/srv/www# ls
index.html
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El fichero <code>index.html</code> está en <code>/opt/contenedor1</code> del host: los cambios que se hagan en el host se ven en el contenedor y al revés.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">05</p>

# LXD e Incus

## Herramientas de gestión de contenedores y MV

---

## LXD

> **LXD** *(Linux Container Daemon)* es una herramienta de **gestión** de contenedores y máquinas virtuales para Linux, desarrollada por Canonical.

### Características

- Ofrece una **REST API** accesible desde línea de comandos o herramientas de terceros
- Gestiona dos tipos de **instancias**:
  - **Contenedores** — usando LXC internamente
  - **Máquinas virtuales** — usando QEMU/KVM internamente
- Sus comandos (`lxc list`, `lxc launch`…) pueden causar confusión con los de LXC (`lxc-ls`, `lxc-start`…)

---

## Incus

> **Incus** es un fork de LXD que también gestiona contenedores y máquinas virtuales. Forma parte del proyecto **Linux Containers**.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### LXD

- Desarrollado por **Canonical**
- Integrado en el ecosistema Ubuntu
- Proyecto original

</div>

<div class="card card-green">

### Incus

- Fork de la comunidad
- Proyecto **Linux Containers** (`linuxcontainers.org/incus/`)
- Alternativa independiente de Canonical

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Ambos: una única herramienta para gestionar <strong>contenedores de sistema y MV</strong>.</div>
</div>

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Contenedores LXC

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 IV</span>
</div>
