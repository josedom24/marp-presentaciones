---
marp: true
title: Software de Orquestación
theme: profesional
paginate: true
header: 'PI · Unidad 1 — Software de Orquestación'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Software de **Orquestación**

## Terraform / OpenTofu

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 PI · Proyecto Intermodular</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Infraestructura como Código

## Qué es y por qué la usamos

---

## Del despliegue manual al código

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-red">

### Despliegue tradicional

- Configuración manual, servidor a servidor
- **Difícil de reproducir** y de documentar
- La documentación se desactualiza o no existe
- *"Funciona en mi máquina"*

</div>

<div class="card card-green">

### Con Infraestructura como Código

- La infraestructura se **describe en ficheros de texto**
- Se crea y se modifica de forma **automática**
- Es **reproducible**: el mismo código da siempre el mismo resultado
- Se gestiona con **Git**, igual que el software

</div>

</div>

---

## ¿Qué es la Infraestructura como Código (IaC)?

> Consiste en **gestionar y aprovisionar infraestructura** (servidores, redes, almacenamiento…) mediante **ficheros de código**, en lugar de configurarla manualmente.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Lo hace posible

Los servicios de virtualización y Cloud Computing son **software**, y todo software se puede **programar mediante una API**.

</div>

<div class="card card-purple">

### Beneficios

- Escenarios **replicables** y predecibles
- **Control de versiones** de toda la infraestructura
- **Auditoría** de los cambios y recuperación rápida ante desastres

</div>

</div>

---

## Dos tipos de herramientas IaC

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### 🏗️ Software de Orquestación

Crea y gestiona **escenarios completos**: máquinas virtuales, redes, almacenamiento…

**Ejemplos:** Terraform, OpenTofu, Pulumi, CloudFormation, Heat

</div>

<div class="card card-green">

### ⚙️ Software de Gestión de la Configuración (CMS)

Configura el **software** de las máquinas ya creadas.

**Ejemplos:** Ansible, Puppet, Chef, Salt

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>💡</span><div>En este módulo usamos <strong>OpenTofu</strong> para orquestar y <strong>Ansible</strong> para configurar: primero se crea la infraestructura, después se configura.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Software de orquestación: OpenTofu

## Orquestación de infraestructura declarativa

---

## De Terraform a OpenTofu

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Terraform

Herramienta de orquestación creada por **HashiCorp** (2014), lenguaje declarativo propio **HCL**. En agosto de 2023 cambió su licencia a **BUSL 1.1**, que ya no es software libre.

</div>

<div class="card card-green">

### OpenTofu

*Fork* libre de Terraform 1.5.x creado por la comunidad y gobernado por la **Linux Foundation**. Licencia **MPL 2.0** (libre), compatible con los ficheros `.tf` y con los providers de Terraform.

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>💡</span><div>Usamos <strong>OpenTofu</strong> en lugar de Terraform precisamente por ser software libre, sin restricciones de uso.</div>
</div>

---

## Elementos de configuración

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Los cuatro bloques básicos

- **Provider**: plugin que conecta con una API concreta
- **Resource**: elemento de infraestructura a crear (VM, red, volumen…)
- **Variable**: parámetro para reutilizar la configuración
- **Output**: valor de salida (IP, ID…) tras aplicar el plan

</div>

<div class="card card-green">

### Ejemplo

```hcl
variable "memoria" { default = 1024 }

resource "libvirt_domain" "web" {
  name   = "web"
  memory = var.memoria
}

output "nombre" {
  value = libvirt_domain.web.name
}
```

</div>

</div>

---

## Un mismo lenguaje, distintos providers

Un **provider** es un plugin que conecta OpenTofu con una plataforma concreta. Cada uno define sus propios *resources*, pero el lenguaje (HCL) es siempre el mismo — en este módulo usaremos `libvirt` y `openstack`.

| Provider | Plataforma |
|:--|:--|
| `aws` | Amazon Web Services |
| `azurerm` | Microsoft Azure |
| `google` | Google Cloud Platform |
| `openstack` | Nube privada OpenStack |
| `libvirt` | Máquinas virtuales con KVM |
| `docker` | Contenedores Docker |

---

## Ficheros de un proyecto

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Los escribimos nosotros

- **`provider.tf`** — el provider y su versión
- **`variables.tf`** — las variables reutilizables
- **`main.tf`** — los recursos: VMs, discos…
- **`network.tf`** — las redes
- **`output.tf`** — lo que se muestra al terminar (IPs…)
- **`cloud-init/`** — la configuración inicial de cada VM

</div>

<div class="card card-yellow">

### Lo genera OpenTofu

- **`terraform.tfstate`** — el **estado** de la infraestructura
- **`.terraform/`** — providers descargados por `init`
- **`.terraform.lock.hcl`** — versiones exactas de los providers

No se editan a mano ni se suben a Git (`.gitignore`).

</div>

</div>

---

## Dependencias entre recursos

```hcl
resource "libvirt_volume" "disco" {
  name = "web.qcow2"
  ...
}

resource "libvirt_domain" "web" {
  disk { volume_id = libvirt_volume.disco.id }   # referencia a otro recurso
}
```

- Al usar `libvirt_volume.disco.id`, OpenTofu sabe que **el disco va antes** que la máquina
- Con esas referencias construye un **grafo de dependencias**: crea en orden, lo independiente a la vez, y **destruye al revés**
- No importa en qué orden se escriban los recursos ni en qué fichero: se leen todos los `.tf` del directorio

---

## El estado de la infraestructura

> El **estado** (`terraform.tfstate`) es un fichero donde OpenTofu guarda **qué recursos ha creado realmente** y con qué identificadores.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### ¿Por qué se guarda?

- Es la forma de saber **qué existe ya** sin preguntar a la plataforma recurso a recurso
- Permite **comparar** lo declarado en el código con lo que hay realmente creado

</div>

<div class="card card-red">

### Cuidado con el estado

- **No se edita a mano**
- Si trabajan varios a la vez, hace falta un **backend remoto** compartido
- Si el estado se pierde, OpenTofu "olvida" lo que había creado

</div>

</div>

---

## Comandos más útiles

| Comando | Descripción |
|:--|:--|
| `tofu init` | Inicializa el proyecto, descarga los *providers* |
| `tofu plan` | Muestra qué acciones se realizarán (sin aplicar) |
| `tofu apply` | Aplica los cambios: crea, modifica o elimina recursos |
| `tofu destroy` | Elimina todos los recursos gestionados por el proyecto |
| `tofu validate` | Verifica que la sintaxis de los ficheros `.tf` sea correcta |
| `tofu show` | Muestra el estado actual de los recursos creados |
| `tofu output` | Muestra los valores definidos en los bloques `output` |
| `tofu refresh` | Actualiza el estado con lo que hay realmente en la plataforma |

---

## Flujo de trabajo habitual

```bash
tofu init       # 1. Una sola vez: descarga el provider
tofu plan       # 2. Revisar qué se va a crear, cambiar o borrar
tofu apply      # 3. Crear el escenario (pide confirmación)
tofu output     # 4. Ver la información del output (IPs…)
tofu destroy    # 5. Eliminar todo el escenario
```

- Si cambias los ficheros, vuelve a `plan` y `apply`: OpenTofu solo aplica **la diferencia**
- Todos los comandos se ejecutan **en el directorio del proyecto**

---

## ¿Qué hace realmente `tofu plan`?

> `plan` **compara tres cosas** y muestra la diferencia, sin modificar nada todavía.

<div class="cols-3" style="margin-top:0.8rem">

<div class="card card-blue">

### 1. El código

Lo que **declaras** en los ficheros `.tf` (el estado deseado)

</div>

<div class="card card-green">

### 2. El estado

Lo último que OpenTofu **sabe** que existe (`terraform.tfstate`)

</div>

<div class="card card-purple">

### 3. La realidad

Lo que **hay de verdad** en la plataforma (puede haber cambiado a mano: *drift*)

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>💡</span><div><code>plan</code> es la red de seguridad antes de aplicar: revisa siempre su salida antes de ejecutar <code>apply</code>.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# OpenTofu + libvirt

## Máquinas virtuales con KVM y cloud-init

---

## Los ejemplos del repositorio `ejercicios_pi`

Cada ejemplo está en su directorio **`opentofu/ejemploN`** y sus recursos llevan el prefijo **`ejN-`**:

| Ejemplo | Escenario |
|:--|:--|
| `ejemplo1` | 1 máquina Debian en la red `default` |
| `ejemplo2` | Igual, con un **disco adicional** de 1 GB |
| `ejemplo3` | 1 máquina conectada a **dos redes con DHCP**: una NAT creada por OpenTofu y `default` |
| `ejemplo4` | Red NAT con DHCP + red **aislada sin DHCP** (IP estática) |
| `ejemplo5` | 2 máquinas (Debian y Ubuntu), red **muy aislada** e **inventario de Ansible** |

- **Destruye siempre el escenario** antes de pasar al siguiente: varios usan el mismo rango (`192.168.100.0/24`)

---

## El provider `libvirt`

```hcl
terraform {
  required_providers {
    libvirt = {
      source  = "dmacvicar/libvirt"
      version = "0.8.3"
    }
  }
}

provider "libvirt" {
  uri = "qemu:///system"
}
```

- **`uri`** — la conexión a libvirt, la misma que `virsh -c qemu:///system`
- Versión **fijada** a la `0.8.3` (la `0.9` usa otra sintaxis): consulta **su** documentación
- `provider.tf` y `output.tf` **no hay que modificarlos**

---

## Imágenes cloud y el pool

Las **imágenes cloud** son sistemas ya instalados, mínimos, en formato `qcow2` y preparados para configurarse con cloud-init en el primer arranque.

```bash
cd /var/lib/libvirt/images                  # directorio del pool "default"
sudo wget <url de la imagen> -O debian13-base.qcow2
sudo qemu-img resize debian13-base.qcow2 10G
sudo virsh pool-refresh default             # que libvirt vea el fichero nuevo
```

- Se descargan **una vez** y se usan como **imagen base** de todas las máquinas
- En los ejemplos: `debian13-base.qcow2` (Debian 13) y `ubuntu2604-base.qcow2` (Ubuntu 26.04)
- Se eligen con variables de `variables.tf`: **`var.base_image`** y **`var.libvirt_pool_name`**

---

## Los discos: `libvirt_volume`

**Clon ligero** de la imagen base (*backing store*: solo guarda los cambios) y disco extra vacío de 1 GB:

```hcl
resource "libvirt_volume" "ej2-server1-disk" {
  name             = "ej2-server1.qcow2"
  pool             = var.libvirt_pool_name
  base_volume_name = var.base_image
  base_volume_pool = var.libvirt_pool_name
  format           = "qcow2"
}

resource "libvirt_volume" "ej2-server1-disk-extra1" {
  name   = "ej2-server1-disk-extra1.qcow2"
  pool   = var.libvirt_pool_name
  format = "qcow2"
  size   = 1 * 1024 * 1024 * 1024 # 1 GB en bytes
}
```

---

## cloud-init

Configura la máquina en su **primer arranque**, a partir de dos ficheros:

<div class="cols-2" style="margin-top:0.6rem">

<div>

**`user-data`**: usuarios, claves, paquetes…

```yaml
#cloud-config
hostname: ej4-server1
timezone: Europe/Madrid
users:
  - name: debian
    sudo: ALL=(ALL) NOPASSWD:ALL
    ssh_authorized_keys:
      - ssh-ed25519 AAAA... tu clave
package_update: true
package_upgrade: true
```

</div>

<div>

**`network-config`**: la red (netplan)

```yaml
network:
  version: 2
  ethernets:
    ens3:
      dhcp4: true
    ens4:
      dhcp4: false
      addresses: ["192.168.130.10/24"]
```

</div>

</div>

- Se empaquetan en una **ISO** con el recurso **`libvirt_cloudinit_disk`** (parámetros `user_data` y `network_config`)

---

## La máquina virtual: `libvirt_domain`

```hcl
resource "libvirt_domain" "ej1-server1" {
  name   = "ej1-server1"
  memory = 1024
  vcpu   = 2
  network_interface {
    network_name   = "default"
    wait_for_lease = true
  }
  disk { volume_id = libvirt_volume.ej1-server1-disk.id }
  cloudinit = libvirt_cloudinit_disk.ej1-server1-cloudinit.id
  console {
    type        = "pty"
    target_port = "0"
    target_type = "serial"
  }
}
```

- **`console`**: consola serie, la esperan las imágenes cloud y permite `virsh console` aunque falle la red

---

## Redes de libvirt

| Tipo | Definición | ¿El anfitrión tiene IP? | ¿Salida al exterior? |
|:--|:--|:--|:--|
| **NAT** | `mode = "nat"` + `addresses` | Sí (la `.1`) | Sí |
| **Aislada** | `mode = "none"` + `addresses` | Sí (la `.1`) | No |
| **Muy aislada** | `mode = "none"`, sin `addresses` | No | No |

```hcl
# network.tf: la red se crea con "tofu apply" y se elimina con "tofu destroy"
resource "libvirt_network" "ej3-nat-dhcp" {
  name      = "ej3-nat-dhcp"
  mode      = "nat"
  addresses = ["192.168.100.0/24"]
  dhcp { enabled = true }
  autostart = true
}
```

---

## Conectar la máquina a las redes

<div class="cols-2" style="margin-top:0.6rem">

<div class="card card-blue">

### Red que no gestiona OpenTofu

Por su **nombre** (por ejemplo, `default`):

```hcl
network_interface {
  network_name   = "default"
  wait_for_lease = true
}
```

</div>

<div class="card card-green">

### Red creada por OpenTofu

Por su **id**:

```hcl
network_interface {
  network_id = libvirt_network
    .ej4-aislada-static.id
}
```

</div>

</div>

- **`wait_for_lease = true`** — esperar a que el DHCP le dé IP: **solo** en redes con DHCP (sin DHCP, el `apply` se queda esperando)
- Cada `network_interface` es una interfaz más (`ens3`, `ens4`…), que **hay que configurar** en el **`network-config`**

---

## Las IP en el `output`

```hcl
output "ej4-server1" {
  value = {
    nombre = "ej4-server1"
    ip1    = try(libvirt_domain.ej4-server1.network_interface[0].addresses[0], "No disponible")
    ip2    = "192.168.130.10" # estática: está en cloud-init/network-config1.yaml
  }
}
```

- Con **`wait_for_lease`**, OpenTofu lee la IP de las **concesiones** (*leases*) del DHCP de libvirt
- Las IP **estáticas** OpenTofu no las conoce: se escriben en el `output`, las mismas que en el `network-config`
- **`try(…, "No disponible")`** — si no hay IP, el `output` no da error
- Sin `wait_for_lease`, `apply` termina en cuanto arranca la máquina, todavía sin IP: «No disponible»

---

## Ciclo de vida de los recursos

Si cambiamos un escenario que ya existe, `tofu plan` indica qué hará con cada recurso:

| Símbolo | Acción | Ejemplo |
|:--|:--|:--|
| `~` | **update in-place**: se modifica sin destruirlo | Añadir `autostart = true` a la máquina |
| `-/+` | **destroy and then create replacement**: se recrea (`forces replacement`) | Cambiar la memoria · cambiar `var.base_image` |
| `+` | **create**: se crea | Una máquina borrada a mano con `virsh` |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>💡</span><div>Recrear una máquina es empezar de cero: se pierde lo que había en su disco y puede cambiar su IP. Los cambios se <strong>propagan</strong>: si se recrea el disco, también se recrea la máquina que lo usa.</div>
</div>

---

## Del `output` al inventario de Ansible

**`inventario.tf`**: genera el fichero `hosts` con las IP del escenario

```hcl
resource "local_file" "inventario" {
  filename        = "${path.module}/hosts"
  file_permission = "0644"
  content = templatefile("${path.module}/inventario.tftpl", {
    ip_server1 = try(libvirt_domain.ej5-server1.network_interface[0].addresses[0], "")
    ip_server2 = "10.0.0.2" # estática: está en cloud-init/network-config2.yaml
  })
}
```

- **`templatefile`** rellena la plantilla con las IP · **`local_file`** la escribe en tu equipo
- Usa el provider **`hashicorp/local`**: hay que añadirlo en `provider.tf` y repetir `tofu init`
- Cada `apply` genera el inventario con las IP correctas: **primero OpenTofu, después Ansible**

---

## La plantilla del inventario

**`inventario.tftpl`**: cada `${…}` se sustituye por el valor que le pasa `templatefile`

<div style="font-size:0.78em">

```ini
# Fichero generado por OpenTofu (inventario.tf): no lo edites a mano
[servidores]
ej5-server1 ansible_host=${ip_server1} ansible_user=debian
# server2 solo es accesible a través de server1 (ProxyJump)
ej5-server2 ansible_host=${ip_server2} ansible_user=ubuntu ansible_ssh_common_args='-o ProxyJump=debian@${ip_server1}'
```

</div>

- **`ej5-server1`** (Debian): IP por DHCP en la red NAT y `10.0.0.1` en la red muy aislada
- **`ej5-server2`** (Ubuntu): solo está en la red muy aislada (`10.0.0.2`), así que Ansible llega a ella **a través de `ej5-server1`** (*ProxyJump*)
- `ej5-server2` **no tiene salida al exterior**: en su `user-data` está comentada la actualización de paquetes

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Software de Orquestación · OpenTofu

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 PI</span>
</div>
