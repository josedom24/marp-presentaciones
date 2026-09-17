---
marp: true
title: QEMU/KVM y libvirt — Introducción
theme: profesional
paginate: true
header: 'IV · QEMU/KVM y libvirt'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# **QEMU/KVM** y libvirt

## Conceptos, instalación y gestión de máquinas virtuales

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 IV · Infraestructura Virtual</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# QEMU/KVM y libvirt

## Virtualización completa en Linux

---

## QEMU

> **QEMU** es un emulador genérico y de código abierto de máquinas virtuales.

### Dos modos de funcionamiento

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### Modo **emulador**

- Permite ejecutar SO de una **arquitectura distinta** (ej. ARM sobre x86)
- Útil para **desarrollo cruzado**
- Rendimiento bajo

</div>

<div class="card card-green">

### Modo **virtualización**

- Apoyado en hipervisores como **KVM**
- Aprovecha las extensiones del procesador
- **Alto rendimiento**

</div>

</div>

---

## KVM

> **Kernel-based Virtual Machine** es un hipervisor de **tipo 1** integrado al kernel de Linux.

### Características

- Solución de **virtualización completa** para Linux
- Necesita CPU con extensiones **Intel VT** o **AMD-V**
- Se compone de varios **módulos del kernel**:
  - `kvm.ko` — infraestructura base de virtualización
  - `kvm-intel.ko` / `kvm-amd.ko` — módulo específico del procesador

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div><strong>QEMU + KVM</strong> es la combinación habitual: QEMU emula los dispositivos y KVM acelera la ejecución del invitado.</div>
</div>

---

## Dispositivos paravirtualizados (`virtIO`)

- En virtualización completa, los dispositivos (discos, red…) están **emulados por software**: el invitado cree que habla con hardware real, y cada operación se traduce mediante una capa de emulación
- Un dispositivo **paravirtualizado** es distinto: el invitado *sabe* que está virtualizado y usa un **driver específico** que habla directamente con el hipervisor mediante una interfaz simple y eficiente, sin fingir ser hardware real
- KVM agrupa estos dispositivos bajo el estándar **`virtIO`**

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-red">

### Dispositivos emulados

- Compatibilidad universal
- **Rendimiento bajo**
- Cada operación atraviesa la capa de emulación

</div>

<div class="card card-green">

### Dispositivos `virtIO`

- **`virtio-net`** — tarjeta de red
- **`virtio-blk` / `virtio-scsi`** — disco
- **Rendimiento muy cercano al real**

</div>

</div>

---

## libvirt

> **libvirt** es la API y conjunto de herramientas que facilita la **gestión** de los recursos virtualizados.

### ¿Por qué libvirt?

- Trabajar directamente con QEMU/KVM es **complejo**
- libvirt ofrece una **API genérica** y un **demonio** comunes
- Soporta varios sistemas: **KVM**, **LXC**, **Xen**…
- Permite usar las **mismas herramientas** independientemente del hipervisor

---

## Mecanismos de conexión a libvirt

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Local sin privilegios

`qemu:///session`

- Acceso a las VM **del usuario actual**
- Sin permisos para crear redes
- Útil para usuarios de escritorio

### Local privilegiado

`qemu:///system`

- Acceso a las VM **del sistema**
- Permisos completos sobre red y almacenamiento

</div>

<div class="card card-green">

### Remoto privilegiado por SSH

`qemu+ssh:///system`

- Conexión **a un servidor remoto** que ejecuta libvirt
- Autenticación a través de **SSH**
- Base para administrar **clústeres** de hipervisores

</div>

</div>

---

## Aplicaciones del ecosistema libvirt

| Aplicación | Función |
|:--|:--|
| **virsh** | Cliente oficial de **línea de comandos**. Shell completa para la API |
| **virt-manager** | Aplicación **gráfica** con la mayor parte de las funcionalidades |
| **virt-install** | Creación de MV desde la **línea de comandos** (`virt-install`, `virt-clone`, `virt-xml`) |
| **virt-viewer** | Acceso a la **consola gráfica** de una VM |
| **gnome-boxes** | Aplicación gráfica **simple** para usuarios de escritorio |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div><code>virsh</code> es la herramienta de referencia para automatizar y administrar libvirt en servidores sin entorno gráfico.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Instalación y primeros pasos

## Instalación, red y almacenamiento por defecto

---

## Instalación de QEMU/KVM + libvirt

```bash
root@kvm:~# apt install qemu-system libvirt-clients libvirt-daemon-system

root@kvm:~# adduser usuario libvirt

usuario@kvm:~$ virsh -c qemu:///system list
```

Para no tener que indicar la conexión en cada comando, podemos definir una variable de entorno:

```bash
export LIBVIRT_DEFAULT_URI='qemu:///system'
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Con la variable definida, todos los comandos <code>virsh</code> usarán <code>qemu:///system</code> sin especificarlo.</div>
</div>

---

## Red disponible por defecto

```bash
usuario@kvm:~$ virsh net-list --all
 Nombre    Estado     Inicio automático   Persistente
-------------------------------------------------------
 default   inactivo   no                  si

usuario@kvm:~$ virsh net-start default
La red default se ha iniciado

usuario@kvm:~$ virsh net-autostart default
La red default ha sido marcada para iniciarse automáticamente
```

---

## Características de la red `default`

La red `default` es de tipo **NAT**. Al crear una MV se conectará a ella por defecto:

1. El host tiene un **servidor DHCP** (rango: `192.168.122.2 – 192.168.122.254`)
2. La **puerta de enlace** de la MV es `192.168.122.1` (el propio host)
3. El **servidor DNS** de la MV también es el host
4. La MV se conecta a un **Linux Bridge** llamado `virbr0`
5. El host se conecta al bridge `virbr0` con la IP `192.168.122.1`
6. El host realiza **SNAT** para dar conectividad al exterior

---

## Red `default` — esquema

![w:900px](img/red1.png)

---

## Almacenamiento disponible por defecto

- Los discos (**volumen**) de las MV se guardan por defecto en **ficheros con formato `qcow2`**
- El directorio (**pool de almacenamiento**) es **`/var/lib/libvirt/images`**

![w:900px](img/almacenamiento1.png)

---

## virt-install

Crea una MV desde la línea de comandos (`apt install virtinst`):

```bash
virt-install \
             --virt-type kvm \
             --name prueba1 \
             --cdrom /var/lib/libvirt/images/debian-11.3.0-amd64-netinst.iso \
             --os-variant debian10 \
             --disk size=10 \
             --memory 1024 \
             --vcpus 1
```

Para acceder a la consola gráfica: `virt-viewer prueba1`

---

## Gestión de MV con virsh

```bash
virsh --help
virsh list --help
```

Subcomandos más usados:

```
list --all                       dominfo <máquina>
shutdown <máquina>               domifaddr <máquina>
start <máquina>                  domblklist <máquina>
autostart <máquina>
reboot <máquina>
destroy <máquina>
suspend <máquina>
resume <máquina>
undefine --remove-all-storage <máquina>
```

---

## Definición XML de una máquina

```bash
virsh dumpxml <máquina>
```

```xml
<domain type='kvm' id='6'>
  <name>prueba1</name>
  <uuid>a88eebdc-8a00-4b9d-bf48-cbed7bb448d3</uuid>
  ...
  <memory unit='KiB'>1048576</memory>
  <currentMemory unit='KiB'>1048576</currentMemory>
  <vcpu placement='static'>1</vcpu>
  ...
  <os>
    <type arch='x86_64' machine='pc-q35-5.2'>hvm</type>
    <boot dev='hd'/>
  </os>
```

---

## Definición XML — disco e interfaz de red

```xml
  <disk type='file' device='disk'>
    <driver name='qemu' type='qcow2'/>
    <source file='/var/lib/libvirt/images/prueba1.qcow2'/>
    <target dev='vda' bus='virtio'/>
    <address type='pci' domain='0x0000' bus='0x04' slot='0x00' function='0x0'/>
  </disk>
  ...
  <interface type='network'>
    <mac address='52:54:00:8a:50:d1'/>
    <source network='default'/>
    <model type='virtio'/>
    <address type='pci' domain='0x0000' bus='0x01' slot='0x00' function='0x0'/>
  </interface>
```

<div class="alerta alerta-info" style="margin-top:0.5rem">
<span>ℹ️</span><div>El disco y la tarjeta de red usan el bus <strong>virtio</strong> para mayor rendimiento (dispositivos paravirtualizados).</div>
</div>

---

## Modificación de una máquina virtual

Con `virsh edit <máquina>` se puede editar directamente el XML en `$EDITOR`, pero **solo conviene hacerlo si no existe un subcomando** para el cambio que se necesita: los subcomandos validan lo que se escribe y evitan dejar el XML en un estado inconsistente.

```bash
# Renombrar (MV parada)
virsh domrename prueba2 prueba1

# Cambiar vCPUs (MV parada)
virsh setvcpus prueba1 2 --config

# Arranque automático
virsh autostart prueba1

# Cambiar memoria asignada (MV parada, con --config; en caliente, sin --config)
virsh setmem prueba1 2G --config
```

<div class="alerta alerta-warning" style="margin-top:0.5rem">
<span>⚠️</span><div>Algunos cambios requieren la MV <strong>parada</strong>, otros admiten <strong>cambio en caliente</strong> y otros necesitan <strong>reinicio</strong>.</div>
</div>

---

## virt-manager

Aplicación gráfica para gestionar libvirt:

![w:900px](img/virt-manager1.png)

---

## Creación de MV Windows — el problema

- Por defecto, `virt-install` puede crear discos y tarjetas de red **sin especificar el bus**: usa dispositivos **emulados** (ej. disco IDE, red `e1000`), que funcionan sin drivers pero con **peor rendimiento**
- Si se piden dispositivos **VirtIO** (`bus=virtio`, `model=virtio`) para mejor rendimiento, Windows **no los reconoce de forma nativa**: el instalador ni siquiera detecta el disco
- Hace falta instalar los **drivers VirtIO para Windows** durante la propia instalación

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Con <code>bus=virtio</code> y sin proporcionar los drivers, el instalador de Windows no encuentra ningún disco donde instalar el sistema.</div>
</div>

---

## Creación de MV Windows — drivers VirtIO

1. Descarga la ISO de drivers del proyecto oficial **virtio-win**: `https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/`
2. Añádela como un **segundo CD-ROM** en la propia creación de la MV
3. Durante la instalación, cuando no aparezca ningún disco, usa **"Cargar controlador"** y navega hasta ese CD-ROM

```bash
virt-install \
             --virt-type kvm \
             --name prueba4 \
             --cdrom /var/lib/libvirt/images/Win10_21H2_Spanish_x64.iso \
             --os-variant win10 \
             --disk size=40,bus=virtio \
             --disk /var/lib/libvirt/images/virtio-win-0.1.217.iso,device=cdrom \
             --network=default,model=virtio \
             --memory 2048 \
             --vcpus 2
```

---

## Para profundizar

- **Curso 1**: Introducción a la virtualización con KVM/libvirt usando virt-manager
  [github.com/josedom24/curso_kvm_ow/curso1](https://github.com/josedom24/curso_kvm_ow/blob/main/curso1)
- **Curso 2**: Profundización en la virtualización con KVM/libvirt
  [github.com/josedom24/curso_kvm_ow/curso2](https://github.com/josedom24/curso_kvm_ow/blob/main/curso2)

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## QEMU/KVM y libvirt

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 IV</span>
</div>
