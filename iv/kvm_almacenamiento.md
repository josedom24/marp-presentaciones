---
marp: true
title: Almacenamiento en QEMU/KVM + libvirt
theme: profesional
paginate: true
header: 'IV · Almacenamiento en QEMU/KVM + libvirt'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Almacenamiento en **QEMU/KVM** + libvirt

## Pools, volúmenes, gestión y redimensión

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 IV · Infraestructura Virtual</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Conceptos y tipos de almacenamiento

## Pools, volúmenes y formatos de imagen

---

## Conceptos básicos

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Pool de almacenamiento

Recurso de almacenamiento gestionado por libvirt.

**Tipos principales:**
- `dir` — directorio del sistema de archivos
- `logical` — grupo de volúmenes LVM
- `netfs` — directorio NAS (NFS…)
- `iSCSI` — disco desde servidor iSCSI

</div>

<div class="card card-green">

### Volumen

Medio de almacenamiento que representa el **disco de una MV**.

Según el tipo de pool puede ser:
- Un **fichero de imagen** (`dir`, `netfs`)
- Un **volumen lógico LVM** (`logical`)
- Un **disco iSCSI** (`iscsi`)

</div>

</div>

---

## Tipo de pool: `dir`

- El pool controla un **directorio del host** (sistema de archivos)
- Los volúmenes son **ficheros de imagen de disco**:

| Formato | Características |
|:--|:--|
| **raw** | Imagen binaria de disco. Ocupa todo el espacio asignado. Acceso más eficiente. Sin snapshots ni aprovisionamiento dinámico |
| **qcow2** | Formato QEMU copy-on-write. Solo ocupa el espacio de los datos (aprovisionamiento ligero). Soporta snapshots. Algo menos eficiente |
| vdi, vmdk | Formatos de otros sistemas de virtualización (VirtualBox, VMware) |

<div class="alerta alerta-warning" style="margin-top:0.5rem">
<span>⚠️</span><div>El tipo <code>dir</code> <strong>no ofrece almacenamiento compartido</strong> entre hosts.</div>
</div>

---

## Tipos de pool: `logical`, `netfs`, `iSCSI`

<div class="cols-3" style="margin-top:0.8rem">

<div class="card card-blue">

### `logical`

- Controla un **Grupo de Volúmenes LVM**
- Los volúmenes son **LVs**
- Sin almacenamiento compartido
- Sin snapshots ni aprovisionamiento ligero

</div>

<div class="card card-green">

### `netfs`

- Monta un **directorio desde un servidor NAS** (NFS…)
- Los volúmenes son **ficheros de imagen**
- **Almacenamiento compartido** entre hosts

</div>

<div class="card card-purple">

### `iSCSI`

- Monta un **disco desde un servidor iSCSI**
- Los datos se guardan directamente en ese disco
- **Almacenamiento compartido** (con las consideraciones de acceso concurrente)

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Gestión de pools y volúmenes

## virsh, qemu-img y herramientas específicas

---

## Dos enfoques para gestionar volúmenes

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Con libvirt (`virsh` / `virt-manager`)

- Pool `dir` → crea una **imagen de disco** (qcow2, raw…)
- Pool `logical` → crea un **volumen lógico LVM**

Toda la gestión queda registrada en libvirt.

</div>

<div class="card card-green">

### Con herramientas específicas

- Pool `dir` → `qemu-img create …`  
  → después `pool-refresh`
- Pool `logical` → `lvcreate …`  
  → después `pool-refresh`

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Tras crear o modificar un volumen con herramientas externas, hay que ejecutar <code>virsh pool-refresh &lt;pool&gt;</code> para que libvirt lo detecte.</div>
</div>

---

## Gestión de pools de almacenamiento

```bash
virsh pool-list
virsh pool-info default
virsh pool-dumpxml default

# Crear, construir, arrancar y persistir un nuevo pool
virsh pool-define-as vm-images dir --target /srv/images
virsh pool-build      vm-images
virsh pool-start      vm-images
virsh pool-autostart  vm-images

# Detener y eliminar
virsh pool-destroy  vm-images
virsh pool-delete   vm-images
virsh pool-undefine vm-images
```

---

## Gestión de volúmenes con libvirt

```bash
virsh vol-list default
virsh vol-list default --details
virsh vol-info    prueba1.qcow2 default
virsh vol-dumpxml vol.qcow2     default

# Crear un volumen qcow2 de 10 GB en el pool default
virsh vol-create-as default vol1.qcow2 --format qcow2 10G

# Eliminar un volumen
virsh vol-delete vol1.qcow2 default
```

---

## Gestión de volúmenes con `qemu-img`

Trabajamos con un pool de tipo **`dir`**:

```bash
cd /var/lib/libvirt/images

# Crear imagen qcow2 de 2 GB
qemu-img create -f qcow2 vol2.qcow2 2G

# Ver información de la imagen
qemu-img info vol2.qcow2

# Hacer que libvirt detecte el nuevo fichero
virsh pool-refresh vm-images
```

---

## Creación de MV usando volúmenes existentes

```bash
virt-install \
             --virt-type kvm \
             --name prueba4 \
             --cdrom ~/iso/debian-11.3.0-amd64-netinst.iso \
             --os-variant debian10 \
             --disk vol=default/vol1.qcow2 \
             --memory 1024 \
             --vcpus 1
```

Otras formas de indicar el disco:

```bash
--disk path=/var/lib/libvirt/images/vol1.qcow2
--disk pool=vm-images,size=10
```

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# Añadir y redimensionar discos

## Attach, resize y virt-resize

---

## Añadir nuevos discos a una MV

```bash
# Añadir disco (funciona también en caliente)
virsh attach-disk prueba4 \
        /srv/images/vol2.qcow2 vdb \
        --driver=qemu --type disk  \
        --subdriver qcow2          \
        --persistent

# Eliminar disco
virsh detach-disk prueba4 vdb --persistent
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div><code>--persistent</code> guarda el cambio en la configuración XML de la MV para que persista tras reinicios.</div>
</div>

---

## Redimensión de discos

Con la MV **parada**:

```bash
# Con libvirt
virsh vol-resize vol2.qcow2 3G --pool vm-images

# Con qemu-img
sudo qemu-img resize /srv/images/vol2.qcow2 3G
```

Con la MV **en ejecución** (en caliente):

```bash
virsh domblklist prueba4
virsh blockresize prueba4 /srv/images/vol2.qcow2 3G
```

Después, dentro de la MV, redimensionar el sistema de ficheros:

```bash
resize2fs /dev/vdb
```

---

## Redimensión del SF sin entrar en la MV

Usamos **`virt-resize`** para expandir el sistema de ficheros desde el host:

```bash
# 1. Ampliar el fichero de imagen
qemu-img resize vol1.qcow2 10G

# 2. Copiar la imagen (virt-resize trabaja origen → destino)
cp vol1.qcow2 newvol1.qcow2

# 3. Expandir la partición dentro de la imagen
virt-resize --expand /dev/sda1 vol1.qcow2 newvol1.qcow2

# 4. Reemplazar la imagen original
mv newvol1.qcow2 vol1.qcow2
```

<div class="alerta alerta-warning" style="margin-top:0.5rem">
<span>⚠️</span><div><code>virt-resize</code> requiere el paquete <code>libguestfs-tools</code> y <strong>siempre trabaja sobre una copia</strong> del fichero original.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# Clonación, imágenes cloud y snapshots

## Plantillas, cloud-init y instantáneas de MV

---

## Métodos de clonación

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### A partir de una MV

- **`virt-clone`** o `virt-manager`
- La MV clonada es **idéntica** a la original
- Problema: misma configuración, hostname, claves SSH…

</div>

<div class="card card-green">

### A partir de una plantilla

Imagen preconfigurada y generalizada (**copia maestra**):

- **Clonación completa (Full)**: copia completa de la plantilla. Requiere el mismo espacio en disco.
- **Clonación enlazada (Linked)**: la plantilla actúa como imagen base de solo lectura (*backing store*). Requiere mucho menos espacio.

</div>

</div>

---

## Clonación desde una MV con `virt-clone`

```bash
# Clonación automática (nombre e imagen generados automáticamente)
virt-clone \
           --original prueba4 --auto-clone

# Especificando nombre e imagen destino
virt-clone \
           --original prueba4 \
           --name prueba5 \
           --file /var/lib/libvirt/images/prueba5.qcow2 \
           --auto-clone
```

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>La MV clonada es <strong>igual a la original</strong>: mismo hostname, mismas claves SSH, misma configuración. Hay que entrar y modificarla manualmente.</div>
</div>

---

## Despliegue automatizado con imágenes cloud

Aunque podemos crear plantillas manualmente con `virt-sysprep` (ver capítulo 5 del curso *Profundización a la virtualización con KVM/libvirt*), en esta asignatura usaremos un enfoque más moderno y estándar:

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Imágenes cloud

Discos base **ya generalizados** y listos para usar como plantilla.

Las distribuyen directamente Ubuntu, Debian, Fedora, CentOS, AlmaLinux…

</div>

<div class="card card-green">

### cloud-init

Herramienta que **personaliza la MV en su primer arranque**: hostname, usuarios, claves SSH, paquetes, red…

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>Una imagen cloud <strong>es una plantilla</strong>: generalizada, sin configuración específica, lista para clonar y personalizar.</div>
</div>

---

## Imágenes cloud: plantillas listas para usar

Las imágenes cloud son discos base preconfigurados y optimizados para entornos virtualizados:

- **Generalizadas**: sin hostname, sin usuarios locales fijos, sin claves SSH
- **Aprovisionamiento ligero**: arranque rápido, tamaño mínimo
- **Personalizables** en el primer arranque mediante `cloud-init`

Distribuciones que ofrecen imágenes cloud:

| Distribución | URL |
|:--|:--|
| **Ubuntu** | https://cloud-images.ubuntu.com |
| **Debian** | https://cloud.debian.org/images/cloud/ |
| **Fedora** | https://alt.fedoraproject.org/cloud/ |
| **AlmaLinux** | https://repo.almalinux.org/cloud/ |
| **Rocky Linux** | https://dl.rockylinux.org/pub/rocky/ |

---

## ¿Qué es cloud-init?

`cloud-init` es el estándar para la **inicialización automática** de MV e instancias en la nube (OpenStack, AWS, Azure…) y en entornos KVM/libvirt.

Lee una **configuración en formato YAML** (`cloud-config`) y la aplica en el primer arranque:

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Configuración del sistema

- Hostname y FQDN
- Red (IP, DNS, rutas)
- Particiones y montajes

</div>

<div class="card card-green">

### Usuarios y software

- Creación de usuarios y grupos
- Inyección de claves SSH
- Instalación de paquetes
- Ejecución de comandos o scripts

</div>

</div>

---

## Configuración: fichero cloud-config

Creamos un fichero `cloud.yaml` con la configuración deseada:

```yaml
#cloud-config

hostname: ubuntu-vm

package_update: true
package_upgrade: true

chpasswd:
  expire: False
  users:
    - name: root
      password: newpassword
      type: text
    - name: ubuntu
      password: asdasd
      type: text
```

Con este fichero se cambia el nombre de la MV, se actualizan los paquetes y se establecen contraseñas para los usuarios `root` y `ubuntu`.

---

## Preparar la imagen: descarga y clonación enlazada

Descargamos la imagen cloud y la guardamos en el pool:

```bash
wget https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img
sudo mv noble-server-cloudimg-amd64.img /var/lib/libvirt/images
```

Creamos una **clonación enlazada** usando la imagen cloud como *backing store*:

```bash
cd /var/lib/libvirt/images
sudo qemu-img create -f qcow2 \
     -b noble-server-cloudimg-amd64.img \
     -F qcow2 ubuntu2404.qcow2
```

Opcionalmente, ampliamos el tamaño del disco (cloud-init lo redimensionará en el primer arranque):

```bash
sudo qemu-img resize ubuntu2404.qcow2 20G
```

---

## Crear la MV con cloud-init

```bash
virt-install --connect qemu:///system \
             --virt-type kvm \
             --name ubuntu-vm \
             --memory 2048 \
             --vcpus 2 \
             --os-variant ubuntu24.04 \
             --disk path=/var/lib/libvirt/images/ubuntu2404.qcow2,format=qcow2,bus=virtio \
             --import \
             --cloud-init user-data=/var/lib/libvirt/images/cloud.yaml \
             --noautoconsole
```

El parámetro `--import` indica que el disco ya tiene SO instalado. El parámetro `--cloud-init` pasa el fichero de configuración a la MV para que `cloud-init` lo aplique en el primer arranque.

Una vez arrancada, accedemos por consola serie:

```bash
virsh console ubuntu-vm
```

---

## Instantáneas de MV (*snapshots*)

- Guardan el **estado del disco y de la memoria** en un momento dado
- Permiten **volver a un estado anterior**
- Requieren imagen de disco en formato **`qcow2`**
- Se pueden hacer con la MV **apagada o encendida**

```bash
# Crear instantánea
virsh snapshot-create-as prueba2 \
      --name "instantánea1" \
      --description "Creada carpeta importante" \
      --atomic

# Listar y obtener info
virsh snapshot-list prueba2
sudo qemu-img info /var/lib/libvirt/images/prueba2.qcow2

# Restaurar
virsh snapshot-revert prueba2 instantánea1
```

Otros subcomandos: `snapshot-dumpxml`, `snapshot-info`, `snapshot-delete`.

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Almacenamiento en QEMU/KVM + libvirt

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 IV</span>
</div>
