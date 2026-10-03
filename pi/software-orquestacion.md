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

## DevOps

<div class="cols-60-40" style="margin-top:0.8rem">

<div>

**Conflicto histórico:** los equipos de Desarrollo (Dev) y de Sistemas (Ops) han tenido objetivos y herramientas distintas.

### ¿Cómo lo soluciona DevOps?

- Mismas herramientas para dev y ops
- Extiende las buenas prácticas de desarrollo (Git, revisión de código, tests) a los sistemas
- Integración → entrega → despliegue **continuos**

</div>

<div class="card card-purple" style="align-self:center">

### ¿Tiene relación con IaC?

Sí, es una de sus piedras angulares. **Sin IaC no se puede automatizar** el aprovisionamiento de infraestructura, y sin automatización no hay DevOps.

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Software de Orquestación

## Crear infraestructura como código

---

## ¿Qué hace un software de orquestación?

<div class="cols-2" style="margin-top:0.8rem">

<div>

**Crea escenarios completos** con múltiples servidores, redes o contenedores (aprovisionamiento de recursos).

- Útil con demanda **variable** de recursos
- Útil cuando la configuración **cambia continuamente**
- Puede incluir **autoescalado** y respuesta a eventos

</div>

<div class="card card-blue">

### Enfoque declarativo

En lugar de describir *cómo* crear la infraestructura, **declaramos el estado deseado** y la herramienta se encarga de alcanzarlo.

```hcl
resource "libvirt_domain" "web" {
  name   = "servidor-web"
  memory = 1024
  vcpu   = 1
}
```

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# OpenTofu

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

<p class="numero">04</p>

# OpenTofu + libvirt

## Máquinas virtuales con KVM y cloud-init

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
- Versión **fijada** a la `0.8.3`: desde la `0.9` se reescribió con otra sintaxis

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

---

## Los discos: `libvirt_volume`

**Clon ligero** de la imagen base (*backing store*: solo guarda los cambios) y disco extra vacío de 1 GB:

```hcl
resource "libvirt_volume" "server1-disk" {
  name             = "server1.qcow2"
  pool             = var.libvirt_pool_name
  base_volume_name = var.base_image
  base_volume_pool = var.libvirt_pool_name
  format           = "qcow2"
}

resource "libvirt_volume" "server1-extra" {
  name   = "server1-extra.qcow2"
  pool   = var.libvirt_pool_name
  format = "qcow2"
  size   = 1 * 1024 * 1024 * 1024
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
hostname: server1
users:
  - name: debian
    sudo: ALL=(ALL) NOPASSWD:ALL
    ssh-authorized-keys:
      - ssh-ed25519 AAAA... tu clave
packages:
  - qemu-guest-agent
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
      addresses: ["10.0.0.1/24"]
```

</div>

</div>

---

## La máquina virtual: `libvirt_domain`

```hcl
# Disco ISO con los ficheros de cloud-init
resource "libvirt_cloudinit_disk" "server1-cloudinit" {
  name           = "server1-cloudinit.iso"
  pool           = var.libvirt_pool_name
  user_data      = file("${path.module}/cloud-init/user-data1.yaml")
  network_config = file("${path.module}/cloud-init/network-config1.yaml")
}

resource "libvirt_domain" "server1" {
  name       = "server1"
  memory     = 1024
  vcpu       = 2
  qemu_agent = true
  network_interface { network_name = "default" }
  disk { volume_id = libvirt_volume.server1-disk.id }
  cloudinit = libvirt_cloudinit_disk.server1-cloudinit.id
}
```

---

## Redes de libvirt

| Tipo | Definición | ¿El anfitrión tiene IP? | ¿Salida al exterior? |
|:--|:--|:--|:--|
| **NAT** | `mode = "nat"` + `addresses` | Sí (la `.1`) | Sí |
| **Aislada** | `mode = "none"` + `addresses` | Sí (la `.1`) | No |
| **Muy aislada** | `mode = "none"`, sin `addresses` | No | No |

```hcl
resource "libvirt_network" "nat-dhcp" {
  name      = "nat-dhcp"
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
  network_id     = libvirt_network.nat-dhcp.id
  wait_for_lease = true
}
```

</div>

</div>

- **`wait_for_lease = true`** — esperar a que el DHCP le dé IP: **solo** en redes con DHCP
- Cada `network_interface` es una interfaz más (`ens3`, `ens4`…), que hay que configurar en el **`network-config`**

---

## Las IP en el `output`

```hcl
output "server1" {
  value = {
    ip1 = try(libvirt_domain.server1.network_interface[0].addresses[0], "No disponible")
  }
}
```

- Sin agente, OpenTofu solo conoce las IP que reparte el **DHCP** de libvirt
- Con **`qemu_agent = true`**, se las pregunta al agente **`qemu-guest-agent`** de la máquina (lo instala cloud-init): también las **estáticas**
- **`try(…, "No disponible")`** — si aún no hay IP, el `output` no da error
- Si sale «No disponible», el agente aún no funcionaba: `tofu refresh` y `tofu output`

---

## Del `output` al inventario de Ansible

<div class="cols-2" style="margin-top:0.6rem">

<div>

**`inventario.tf`**

```hcl
resource "local_file" "inventario" {
  filename = "${path.module}/hosts"
  content = templatefile(
    "${path.module}/inventario.tftpl", {
      ip_web = try(libvirt_domain.web
        .network_interface[0].addresses[0], "")
  })
}
```

</div>

<div>

**`inventario.tftpl`** (la plantilla)

```ini
[servidores_web]
web ansible_host=${ip_web} ansible_user=debian
```

</div>

</div>

- **`templatefile`** rellena la plantilla con las IP · **`local_file`** la escribe en tu equipo (provider `hashicorp/local`: repite `tofu init`)
- Cada `apply` genera el inventario con las IP correctas: **primero OpenTofu, después Ansible**

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">05</p>

# Resumen

## Problemas frecuentes

---

## Problemas frecuentes

| Síntoma | Causas habituales |
|:--|:--|
| `storage volume not found` | Falta `virsh pool-refresh default` · el nombre de la imagen no coincide con `variables.tf` |
| La red ya existe o se solapa | No se ha destruido el escenario anterior (mismo nombre o rango) |
| `Permission denied (publickey)` | No has puesto tu clave pública en el `user-data` |
| El `apply` se queda esperando | `wait_for_lease` en una red sin DHCP |
| Una interfaz sin IP | Falta en el `network-config` · nombre de interfaz equivocado |
| cloud-init termina con error | La máquina no tiene salida al exterior y se instalan paquetes |
| Errores de sintaxis del provider | Se ha cambiado la versión fijada (`0.8.3`) · documentación de la `0.9` |

Antes de aplicar: `tofu validate` y `tofu plan`. Dentro de la máquina: `cloud-init status --long`.

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
