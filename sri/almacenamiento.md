---
marp: true
title: Servicios de almacenamiento
theme: profesional
paginate: true
header: 'SRI · Unidad 4 — Servicios de almacenamiento'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Servicios de **almacenamiento**

## NAS con NFS · SAN con iSCSI

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI · Servicios de Red e Internet</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Introducción al almacenamiento

## DAS, NAS, SAN y Cloud

---

## ¿Por qué necesitamos servicios de almacenamiento?

Los servidores y aplicaciones generan y consumen volúmenes de datos cada vez mayores. El **almacenamiento local** se queda corto en cuanto necesitamos:

- **Compartir datos** entre varios equipos o servicios
- **Centralizar copias de seguridad** y políticas de retención
- **Ampliar capacidad** sin tocar el hardware del servidor
- **Tolerar fallos** y caídas de un nodo individual
- **Separar cómputo y datos** para escalar de forma independiente

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>De ahí surgen distintas arquitecturas: cada una resuelve un problema diferente.</div>
</div>

---

## Las cuatro grandes categorías

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### DAS — *Direct Attached Storage*

Disco conectado **directamente** al servidor (SATA, SAS, USB).

### NAS — *Network Attached Storage*

Almacenamiento compartido **por red** a nivel de **sistema de ficheros**.

</div>

<div class="card card-green">

### SAN — *Storage Area Network*

Red **dedicada** que ofrece a los servidores **dispositivos de bloques**.

### Cloud — *Object Storage*

Almacenamiento en la **nube** orientado a objetos (S3, Swift…).

</div>

</div>

---

## DAS — Direct Attached Storage

> El disco está **físicamente conectado** al servidor que lo usa.

### Características

- Conexión **directa**: SATA, SAS, NVMe, USB…
- **Máximo rendimiento** y la menor latencia posible
- **Económico** y simple de instalar
- Sólo accesible desde **un único servidor**
- No es trivial **compartir** los datos con otros equipos
- Si el servidor cae, los datos quedan **inaccesibles**

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Es lo que tiene un PC o un servidor "estándar": discos internos formateados con un sistema de ficheros local.</div>
</div>

---

## NAS — Network Attached Storage

> Un servidor comparte **sistemas de ficheros completos** por la red.

### Características

- Trabaja a nivel de **archivo**: el cliente ve carpetas y ficheros
- Habitualmente sobre **TCP/IP**, en redes de uso general
- Protocolos típicos: **NFS** (Unix/Linux), **SMB/CIFS** (Windows)
- Sencillo de **administrar** y de compartir entre varios clientes
- Ideal para **copias de seguridad**, perfiles de usuario, contenidos web

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>El cliente <strong>no</strong> formatea el almacenamiento: lo monta como una carpeta y trabaja con ficheros.</div>
</div>

---

## SAN — Storage Area Network

> Una **red dedicada** ofrece a los servidores **dispositivos de bloques**.

### Características

- Trabaja a nivel de **bloque**: el cliente ve un disco "como si fuera local"
- Red **dedicada**, normalmente de **alta velocidad** (10 Gbps, fibra…)
- El cliente **formatea** el disco con su sistema de ficheros preferido
- Protocolos típicos: **iSCSI** (sobre TCP/IP) y **Fibre Channel**
- Habitual en virtualización, bases de datos y entornos exigentes

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Un dispositivo SAN <strong>no</strong> debe montarse simultáneamente en dos clientes con un sistema de ficheros tradicional: corrompería los datos.</div>
</div>

---

## Cloud Storage

> Almacenamiento ofrecido como servicio en la nube, generalmente **orientado a objetos**.

### Características

- **API HTTP** (REST) para subir, descargar y gestionar objetos
- Estándar de facto: **S3** de Amazon (y compatibles: MinIO, Ceph RGW…)
- **Escalado** prácticamente ilimitado
- Modelo de pago **por uso**
- Pensado para datos **inmutables** o de poca modificación: imágenes, vídeos, *backups*

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>No reemplaza a NAS o SAN: cubre <strong>otro caso de uso</strong>, donde la latencia importa menos que la escala y la durabilidad.</div>
</div>

---

## Comparativa rápida

| | **DAS** | **NAS** | **SAN** | **Cloud** |
|:--|:--|:--|:--|:--|
| Acceso | Local | Red TCP/IP | Red dedicada | Internet (HTTP) |
| Granularidad | Bloque | Archivo | Bloque | Objeto |
| Compartir entre clientes | ❌ | ✅ | ⚠️ con cuidado | ✅ |
| Rendimiento | Muy alto | Medio | Alto | Variable |
| Coste | Bajo | Medio | Alto | Pago por uso |
| Caso típico | Servidor único | Compartir ficheros | Virtualización, BBDD | Backups, *media* |

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# NAS con NFS

## Compartir directorios entre máquinas Linux

---

## ¿Qué es NFS?

> **Network File System** es un protocolo para **compartir ficheros y directorios** por red. Originalmente desarrollado por **Sun Microsystems**, es el estándar de NAS en entornos Unix y Linux.

### Datos clave

- El servidor **exporta** uno o varios directorios
- El cliente los **monta** como si fueran locales
- Las operaciones de lectura y escritura se realizan **remotamente** sobre TCP/IP
- Puerto **2049/tcp** (también admite UDP)

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Es transparente para las aplicaciones: leen y escriben en una ruta como cualquier otra.</div>
</div>

---

## Instalación del servidor NFS

```bash
sudo apt update
sudo apt install nfs-kernel-server
```

### Tras la instalación

- El servicio queda activo (`nfs-server.service`)
- Aún no se exporta nada: hay que configurar `/etc/exports`

```bash
systemctl status nfs-server
systemctl restart nfs-server
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>NFS se apoya en <strong>RPC</strong> (<code>rpcbind</code>): si hay cortafuegos por medio, conviene revisar también los puertos auxiliares.</div>
</div>

---

## Configuración: `/etc/exports`

Cada línea declara un **directorio exportado** y la lista de clientes con sus opciones:

```
/srv/nfs/compartido   192.168.1.0/24(rw,sync,no_subtree_check)
/srv/nfs/lectura      *(ro,sync,no_subtree_check)
/srv/nfs/web          web1.lan(rw,sync) web2.lan(rw,sync)
```

### Aplicar los cambios

```bash
sudo exportfs -ra        # recarga la configuración
sudo exportfs -v         # muestra lo exportado actualmente
```

---

## Opciones más usadas en exports

| Opción | Para qué sirve |
|:--|:--|
| `rw` / `ro` | Lectura y escritura / sólo lectura |
| `sync` | Confirma la escritura en disco antes de responder |
| `async` | Más rápido, pero menos seguro ante caídas |
| `no_subtree_check` | Desactiva la verificación de subdirectorio (recomendado) |
| `root_squash` | El `root` del cliente se mapea a un usuario sin privilegios |
| `no_root_squash` | Permite que el `root` remoto sea `root` real (peligroso) |
| `all_squash` | Todos los usuarios remotos se mapean a `nobody` |

---

## Cliente NFS — instalación

```bash
sudo apt install nfs-common
```

### Montaje manual

```bash
sudo mkdir -p /mnt/nfs/compartido

sudo mount 192.168.1.10:/srv/nfs/compartido \
           /mnt/nfs/compartido
```

A partir de ahí, `/mnt/nfs/compartido` se usa como cualquier directorio local.

---

## Montaje persistente con `/etc/fstab`

Para que el directorio se monte **automáticamente** en cada arranque, añadir una línea al `fstab`:

```
192.168.1.10:/srv/nfs/compartido  /mnt/nfs/compartido  nfs  defaults  0  0
```

### Aplicar sin reiniciar

```bash
sudo systemctl daemon-reload
sudo mount -a
```

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Si el servidor NFS no está accesible, un <code>defaults</code> puede bloquear el arranque. En servidores conviene revisar opciones como <code>_netdev</code>, <code>nofail</code> o <code>x-systemd.automount</code>.</div>
</div>

---

## Comprobaciones útiles

```bash
# Listar lo exportado por un servidor
showmount -e 192.168.1.10

# Ver montajes NFS activos
mount | grep nfs
findmnt -t nfs,nfs4

# Estadísticas y errores
nfsstat
journalctl -u nfs-server
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div><code>showmount</code> es muy útil desde el cliente para confirmar que un servidor exporta el recurso esperado.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# SAN con iSCSI

## Compartir dispositivos de bloques sobre TCP/IP

---

## ¿Qué es iSCSI?

> **Internet Small Computer Systems Interface** transporta **comandos SCSI** sobre **TCP/IP**, permitiendo acceder remotamente a discos como si fueran **dispositivos de bloques locales**.

### Características

- Implementa **SAN** sobre redes Ethernet **estándar**, sin hardware específico
- Alternativa **económica** a Fibre Channel
- Habitual en redes de **1 Gbps** y **10 Gbps**
- El cliente ve un **disco nuevo** (`/dev/sdb`, `/dev/sdc`…) que formatea y monta a su gusto
- Puerto **3260/tcp** entre *initiator* y *target*

---

## Elementos de iSCSI

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Lado servidor

- **Target** — recurso publicado por el servidor; agrupa una o varias LUN
- **LUN** (*Logical Unit Number*) — unidad concreta de almacenamiento (un disco, una partición, un volumen lógico…)

</div>

<div class="card card-green">

### Lado cliente

- **Initiator** — el cliente iSCSI; descubre targets y se conecta a ellos
- **IQN** (*iSCSI Qualified Name*) — identificador único del recurso

</div>

</div>

### Formato IQN

```
iqn.2021-11.org.example:target1
iqn.2020-01.org.gonzalonazareno:sdb4
```

---

## Implementaciones en Linux

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Servidores (target)

- **`tgt`** — sencillo, configuración por archivos
- **Linux-IO (LIO)** — actual, gestionado con `targetcli`
- `scst`, `istgt` — alternativas menos extendidas

</div>

<div class="card card-green">

### Cliente (initiator)

- **`open-iscsi`** — implementación estándar en Linux
- Gestionado con la herramienta **`iscsiadm`**

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>En estos ejemplos usaremos <strong>tgt</strong> en el servidor y <strong>open-iscsi</strong> en el cliente.</div>
</div>

---

## Servidor — instalación de `tgt`

```bash
sudo apt update
sudo apt install tgt
```

### Comprobación

```bash
systemctl status tgt
```

Por defecto **no exporta nada**: hay que crear targets y LUNs.

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Antes de exportar, prepara el bloque que vas a compartir: puede ser un <strong>disco entero</strong> (<code>/dev/vdb</code>), una <strong>partición</strong> o un <strong>volumen LVM</strong>.</div>
</div>

---

## Crear un target con `tgtadm`

```bash
# 1 · Crear un target con su IQN
sudo tgtadm --lld iscsi --op new --mode target \
            --tid 1 -T iqn.2021-11.org.example:target1

# 2 · Asociar una LUN con un dispositivo de bloques
sudo tgtadm --lld iscsi --op new --mode logicalunit \
            --tid 1 --lun 1 -b /dev/vdb

# 3 · Permitir el acceso desde cualquier red (luego se restringe)
sudo tgtadm --lld iscsi --op bind --mode target \
            --tid 1 -I ALL

# 4 · Comprobar la configuración
sudo tgtadm --lld iscsi --op show --mode target
```

---

## Hacer la configuración persistente

Las órdenes de `tgtadm` se pierden al reiniciar. Para que sobrevivan:

```bash
sudo tgt-admin --dump > /etc/tgt/conf.d/example.conf
```

A partir de ese momento, `tgt` reaplica el contenido del archivo en cada arranque.

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>El parámetro <code>-I ALL</code> permite el acceso desde cualquier dirección. En producción conviene <strong>restringirlo</strong> a las IPs concretas que deben usar el target.</div>
</div>

---

## Cliente — instalación de `open-iscsi`

```bash
sudo apt update
sudo apt install open-iscsi
```

### Identificador del initiator

El instalador genera automáticamente el **IQN** del cliente en:

```
/etc/iscsi/initiatorname.iscsi
```

Cada cliente tiene un IQN único: si más adelante se restringe el acceso por initiator, hay que tenerlo a mano.

---

## Descubrir y conectar al target

```bash
# 1 · Descubrir los targets que publica un portal (servidor)
sudo iscsiadm --mode discovery --type sendtargets \
              --portal 10.0.0.1

# 2 · Iniciar sesión contra un target concreto
sudo iscsiadm --mode node \
              -T iqn.2021-11.org.example:target1 \
              --portal 10.0.0.1 --login
```

Tras el `--login`, el kernel detecta un **disco nuevo** (`/dev/sdb`, por ejemplo) que se trata como cualquier otro:

```bash
lsblk
sudo mkfs.ext4 /dev/sdb
sudo mount /dev/sdb /mnt/iscsi
```

---

## Sesiones y desconexión

```bash
# Sesiones activas
sudo iscsiadm -m session

# Desconectar de un target concreto
sudo iscsiadm -m node \
              -T iqn.2021-11.org.example:target1 \
              -p 10.0.0.1 -u

# Desconectar de todos los targets de un portal
sudo iscsiadm -m node -p 10.0.0.1 --logout
```

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Antes de cerrar la sesión, <strong>desmonta</strong> el dispositivo: si hay datos en uso, el sistema puede quedar en un estado inconsistente.</div>
</div>

---

## Reconexión automática al arrancar

```bash
sudo iscsiadm -m node \
              -T iqn.2021-11.org.example:target1 \
              -p 10.0.0.1 --op update \
              -n node.startup -v automatic
```

A partir de ese momento, el cliente **vuelve a conectarse** al target en cada arranque.

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Para que el dispositivo se monte automáticamente conviene añadirlo a <code>/etc/fstab</code> con la opción <strong><code>_netdev</code></strong>, que retrasa el montaje hasta que la red esté disponible.</div>
</div>

---

## Autenticación CHAP

iSCSI puede autenticar al cliente con **CHAP** (usuario y contraseña) para que sólo los initiator autorizados puedan conectarse.

### En el cliente

```bash
sudo iscsiadm --mode node \
    -T iqn.2021-11.org.example:target1 --portal 10.0.0.1 \
    -o update -n node.session.auth.username -v usuario

sudo iscsiadm --mode node \
    -T iqn.2021-11.org.example:target1 --portal 10.0.0.1 \
    -o update -n node.session.auth.password -v contraseña
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>En el servidor hay que declarar el usuario y la contraseña dentro de la definición del target, normalmente en su archivo de <code>/etc/tgt/conf.d/</code>.</div>
</div>

---

## NFS vs iSCSI — cuándo elegir cada uno

| | **NFS (NAS)** | **iSCSI (SAN)** |
|:--|:--|:--|
| Granularidad | Archivos y directorios | Dispositivo de bloques |
| Quién formatea | El servidor | El cliente |
| Acceso simultáneo | ✅ pensado para varios clientes | ⚠️ un cliente a la vez (sin FS clusterizado) |
| Rendimiento | Bueno para ficheros | Mejor para bases de datos y máquinas virtuales |
| Caso típico | Compartir documentos, *home* de usuarios, backups | Discos para VMs, BBDD, almacenamiento dedicado |

---

## Para profundizar

- [Documentación oficial de NFS (kernel.org)](https://www.kernel.org/doc/Documentation/filesystems/nfs/)
- [Wiki de Open-iSCSI](https://github.com/open-iscsi/open-iscsi)
- [Manual de tgt — Linux SCSI target framework](https://stgt.sourceforge.net/)

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Almacenamiento → Alta disponibilidad y respaldo

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 SRI</span>
</div>
