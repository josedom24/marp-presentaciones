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

> **LinuX Containers (LXC)** es una tecnología de **virtualización ligera** que permite crear múltiples instancias aisladas de espacios de usuario sobre un mismo núcleo Linux.

### Características principales

- Pertenece a los **contenedores de sistema**: su gestión y ciclo de vida son similares a los de una máquina virtual tradicional
- Se accede a ellos por SSH, se instalan servicios, se actualizan…
- Mantenido por **Canonical**; página oficial: `linuxcontainers.org`

```bash
apt install lxc
```

---

## Tecnologías del kernel que lo hacen posible

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Grupos de control (*cgroups*)

- En Debian 11 se usa **cgroups v2**
- **Limitan el uso de recursos** de un proceso y sus hijos: memoria, CPU, I/O, red

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

## Creación de contenedores

Al crear un contenedor se descarga la **plantilla** (sistema de archivos base). Se guarda en caché y solo se descarga una vez:

```
/var/cache/lxc/debian/rootfs-<distribución>-amd64/
```

Como `root`, crear contenedores con distintas distribuciones:

```bash
lxc-create -n contenedor1 -t debian -- -r bookworm
lxc-create -n contenedor2 -t ubuntu -- -r jammy
```

- `-t` — nombre de la plantilla
- `-r` — nombre de la versión (*release*)

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Lista de plantillas disponibles: <code>ls /usr/share/lxc/templates/</code> o en el servidor de imágenes <code>images.linuxcontainers.org</code>.</div>
</div>

---

## Gestión básica de contenedores

```bash
lxc-ls [-f]                   # Listar contenedores (con detalles)

lxc-start  contenedor1        # Arrancar
lxc-stop   contenedor1        # Parar
lxc-stop -r contenedor1       # Reiniciar

lxc-attach contenedor1        # Consola interactiva
lxc-attach contenedor1 -- ls -al   # Ejecutar un comando

lxc-info    contenedor1       # Información del contenedor
lxc-execute contenedor1 -- ls -al  # Ejecutar sin entrar al entorno

lxc-destroy [-f] contenedor1  # Eliminar contenedor
```

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Configuración de LXC

## Ficheros de configuración, red y almacenamiento

---

## Ficheros de configuración

| Fichero | Alcance |
|:--|:--|
| `/etc/lxc/default.conf` | Configuración **global** aplicada a todos los contenedores nuevos |
| `/var/lib/lxc/<nombre>/config` | Configuración **individual** de cada contenedor (copia de la global al crear) |

Opciones más habituales:

```
lxc.net.0.link = lxcbr0      # Bridge al que conectar la interfaz
lxc.start.auto = 1            # Arrancar automáticamente con el host
lxc.cgroup2.memory.max = 512M # Límite de memoria
lxc.cgroup2.cpuset.cpus = 0   # CPUs disponibles para el contenedor
```

---

## Redes en LXC — `lxc-net`

La forma más sencilla: usar **`lxc-net`**, que crea automáticamente el bridge `lxcbr0`:

- El host actúa como **servidor DHCP, DNS y SNAT** para los contenedores
- Rango de red por defecto: **`10.0.3.0/24`**
- Los contenedores obtienen IP automáticamente y tienen salida a Internet

---

## Redes en LXC — conectar a un bridge existente

Para conectar un contenedor a un bridge ya creado (por libvirt u otro medio), editamos `/var/lib/lxc/contenedor1/config`:

```
lxc.net.0.type  = veth
lxc.net.0.hwaddr = 00:16:3e:cf:8f:c3
lxc.net.0.link  = lxcbr0
lxc.net.0.flags = up

lxc.net.1.type  = veth
lxc.net.1.hwaddr = 00:16:3e:cf:8f:d3
lxc.net.1.link  = virbr0
lxc.net.1.flags = up
```

Reiniciar el contenedor y configurar la nueva interfaz dentro de él.

---

## Redes en LXC — verificación

```bash
$ lxc-ls -f
NAME        STATE   AUTOSTART GROUPS IPV4                        IPV6 UNPRIVILEGED
contenedor1 RUNNING 1         -      10.0.3.10, 192.168.122.196  -    false
```

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>El contenedor tiene aquí <strong>dos interfaces</strong>: una en <code>lxcbr0</code> (<code>10.0.3.10</code>) y otra en el bridge de libvirt <code>virbr0</code> (<code>192.168.122.196</code>).</div>
</div>

---

## Almacenamiento en LXC — montar directorios del host

Para compartir un directorio del host con un contenedor:

1. Crear el punto de montaje **dentro del contenedor**:

```bash
lxc-attach contenedor1
root@contenedor1:/# mkdir /srv/www
```

2. Añadir la entrada en `/var/lib/lxc/contenedor1/config`:

```
lxc.mount.entry=/opt/contenedor1 srv/www none bind 0 0
```

3. Reiniciar el contenedor y verificar el montaje.

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>La ruta de destino en <code>lxc.mount.entry</code> es <strong>relativa</strong> al sistema de ficheros del contenedor (sin la barra inicial).</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

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
- Sus comandos pueden causar confusión con los de LXC:

```bash
lxc list
lxc launch
```

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
<span>ℹ️</span><div>Ambos ofrecen la misma filosofía: una única herramienta para gestionar <strong>contenedores de sistema y máquinas virtuales</strong>.</div>
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
