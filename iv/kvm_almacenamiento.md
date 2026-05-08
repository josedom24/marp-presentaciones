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
  <span>📚 IV · Implantación de Aplicaciones Web</span>
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
virsh -c qemu:///system pool-list
virsh -c qemu:///system pool-info default
virsh -c qemu:///system pool-dumpxml default

# Crear, construir, arrancar y persistir un nuevo pool
virsh -c qemu:///system pool-define-as vm-images dir --target /srv/images
virsh -c qemu:///system pool-build      vm-images
virsh -c qemu:///system pool-start      vm-images
virsh -c qemu:///system pool-autostart  vm-images

# Detener y eliminar
virsh -c qemu:///system pool-destroy  vm-images
virsh -c qemu:///system pool-delete   vm-images
virsh -c qemu:///system pool-undefine vm-images
```

---

## Gestión de volúmenes con libvirt

```bash
virsh -c qemu:///system vol-list default
virsh -c qemu:///system vol-list default --details
virsh -c qemu:///system vol-info    prueba1.qcow2 default
virsh -c qemu:///system vol-dumpxml vol.qcow2     default

# Crear un volumen qcow2 de 10 GB en el pool default
virsh -c qemu:///system vol-create-as default vol1.qcow2 --format qcow2 10G

# Eliminar un volumen
virsh -c qemu:///system vol-delete vol1.qcow2 default
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
virsh -c qemu:///system pool-refresh vm-images
```

---

## Creación de MV usando volúmenes existentes

```bash
virt-install --connect qemu:///system \
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
virsh -c qemu:///system attach-disk prueba4 \
        /srv/images/vol2.qcow2 vdb \
        --driver=qemu --type disk  \
        --subdriver qcow2          \
        --persistent

# Eliminar disco
virsh -c qemu:///system detach-disk prueba4 vdb --persistent
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div><code>--persistent</code> guarda el cambio en la configuración XML de la MV para que persista tras reinicios.</div>
</div>

---

## Redimensión de discos

Con la MV **parada**:

```bash
# Con libvirt
virsh -c qemu:///system vol-resize vol2.qcow2 3G --pool vm-images

# Con qemu-img
sudo qemu-img resize /srv/images/vol2.qcow2 3G
```

Con la MV **en ejecución** (en caliente):

```bash
virsh -c qemu:///system domblklist prueba4
virsh -c qemu:///system blockresize prueba4 /srv/images/vol2.qcow2 3G
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

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Almacenamiento en QEMU/KVM + libvirt

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 IV</span>
</div>
