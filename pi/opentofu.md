---
marp: true
title: OpenTofu — Orquestación de infraestructura con libvirt
theme: profesional
paginate: true
header: 'PI · OpenTofu + libvirt'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# **OpenTofu** + libvirt

## Orquestación de infraestructura con KVM

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 PI · Puesta en Producción de Aplicaciones</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Terraform y OpenTofu

## Historia, licencia y por qué usamos OpenTofu

---

## ¿Qué es Terraform?

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Terraform

- Herramienta de **orquestación de infraestructura** creada por **HashiCorp** en 2014
- Lenguaje declarativo propio: **HCL** (*HashiCorp Configuration Language*)
- Permite definir, crear y administrar infraestructura como **código**
- Compatible con múltiples proveedores: AWS, Azure, GCP, OpenStack, libvirt…

</div>

<div class="card card-green">

### Modelo declarativo

En lugar de describir *cómo* crear la infraestructura, **declaramos el estado deseado** y la herramienta se encarga de alcanzarlo.

```hcl
resource "libvirt_domain" "server1" {
  name   = "servidor1"
  memory = 1024
  vcpu   = 1
}
```

</div>

</div>

---

## El problema con Terraform: cambio de licencia

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-red">

### Agosto 2023 — HashiCorp cambia la licencia

- Terraform pasa de **MPL 2.0** (libre) a **BUSL 1.1** (*Business Source License*)
- BUSL **no es software libre**: prohíbe usar Terraform para ofrecer servicios que compitan con HashiCorp
- La comunidad reacciona con rechazo

</div>

<div class="card card-purple">

### Consecuencias

- Los usuarios no pueden garantizar el uso sin restricciones en el futuro
- Proyectos que dependen de Terraform quedan en una zona gris legal
- Se hace necesaria una alternativa **100 % libre y compatible**

</div>

</div>

---

## ¿Qué es OpenTofu?

> **OpenTofu** es un *fork* de **Terraform 1.5.x**, creado por la comunidad (liderado por la *Linux Foundation*) como respuesta al cambio de licencia de Terraform en 2023.

<div class="cols-3" style="margin-top:0.8rem">

<div class="card card-blue">

### Licencia libre

Licenciado bajo **MPL 2.0** (*Mozilla Public License*), reconocida como licencia libre y open source

</div>

<div class="card card-green">

### Gobernanza abierta

Gobernado por la **Linux Foundation**. Todo el desarrollo es abierto y las decisiones se discuten públicamente

</div>

<div class="card card-purple">

### Compatibilidad total

Compatible con los ficheros `.tf` y los *providers* existentes de Terraform. Sin restricciones de uso

</div>

</div>

---

## ¿Por qué usamos OpenTofu y no Terraform?

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### OpenTofu ✅

- Licencia **MPL 2.0** — 100 % libre
- Sin restricciones de uso en entornos educativos ni comerciales
- Gobernado por la **Linux Foundation**
- Desarrollo transparente y comunitario
- Compatible con todo el ecosistema de Terraform

</div>

<div class="card card-red">

### Terraform ❌

- Licencia **BUSL 1.1** — no libre
- Restricciones de uso para productos o servicios competidores
- Controlado únicamente por **HashiCorp** (ahora IBM)
- Puede cambiar las condiciones en cualquier momento

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>💡</span><div>OpenTofu conserva el <strong>modelo declarativo de IaC</strong> y es totalmente compatible con los ficheros <code>.tf</code> y los <em>providers</em> existentes.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Conceptos básicos de OpenTofu

## Providers, resources, variables y outputs

---

## ¿Qué hace OpenTofu?

> Permite **definir, crear y administrar infraestructura** (máquinas virtuales, redes, contenedores, etc.) usando archivos de texto declarativos (`.tf`).

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Qué describes tú

```hcl
resource "libvirt_domain" "server1" {
  name   = "maquina1"
  memory = 1024
  vcpu   = 1
}
```

**Qué infraestructura quieres** — sin detallar los pasos

</div>

<div class="card card-green">

### Qué hace OpenTofu

- **Construye** la infraestructura si no existe
- **Modifica** los recursos que hayan cambiado
- **Destruye** los recursos que ya no estén declarados

De forma **reproducible y automatizada**

</div>

</div>

---

## ¿Qué es un *provider*?

> Un **provider** (proveedor) es un **plugin** que permite a OpenTofu interactuar con una plataforma o tecnología concreta. Cada provider sabe cómo crear, leer, actualizar y eliminar recursos.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Ejemplos de providers

| Provider | Plataforma |
|:--|:--|
| `aws` | Amazon Web Services |
| `azurerm` | Microsoft Azure |
| `google` | Google Cloud Platform |
| `libvirt` | Máquinas virtuales con KVM |
| `docker` | Contenedores Docker |

</div>

<div class="card card-green">

### Provider libvirt

El provider **`libvirt`** permite a OpenTofu **crear y gestionar VMs en tu sistema KVM local**.

```hcl
terraform {
  required_providers {
    libvirt = {
      source = "dmacvicar/libvirt"
    }
  }
}
```

</div>

</div>

---

## Ficheros de configuración `.tf`

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Ficheros habituales en un proyecto

- **`provider.tf`** — configura el provider a usar. Se ejecuta `tofu init` una sola vez
- **`variables.tf`** — declara variables globales reutilizables
- **`main.tf`** — define los recursos: VMs, redes, discos…
- **`output.tf`** — muestra información al finalizar (IPs, nombres…)
- **`networks.tf`** — define las redes (si se gestionan con OpenTofu)

</div>

<div class="card card-green">

### Ejemplo: variable y output

```hcl
variable "libvirt_pool_name" {
  default = "default"
}

output "server1_ip" {
  value = libvirt_domain.server1
          .network_interface[0].addresses[0]
}
```

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# Comandos de OpenTofu

## El flujo de trabajo básico

---

## Comandos más importantes

| Comando | Descripción |
|:--|:--|
| `tofu init` | Inicializa el proyecto, descarga los *providers* y prepara el entorno |
| `tofu plan` | Muestra qué acciones realizará OpenTofu (sin aplicar cambios) |
| `tofu apply` | Aplica los cambios: crea, modifica o elimina recursos según los `.tf` |
| `tofu destroy` | Elimina todos los recursos gestionados por el proyecto |
| `tofu validate` | Verifica que la sintaxis de los archivos `.tf` sea correcta |
| `tofu show` | Muestra el estado actual de los recursos creados |
| `tofu output` | Muestra los valores definidos en bloques `output` |

---

## Flujo de trabajo habitual

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Primera vez

```bash
# 1. Inicializar — solo una vez
tofu init

# 2. Revisar los cambios
tofu plan

# 3. Aplicar el escenario
tofu apply
```

Aparecerá la confirmación de las acciones. Escribe `yes` para continuar.

</div>

<div class="card card-green">

### Gestión del escenario

```bash
# Ver la información del output
tofu output

# Ver el estado de los recursos
tofu show

# Destruir todo el escenario
tofu destroy
```

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Los comandos se ejecutan siempre <strong>desde el directorio del proyecto</strong> donde están los ficheros <code>.tf</code>.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# OpenTofu + libvirt

## Creación de VMs con cloud-init

---

## Imágenes cloud y cloud-init

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Imágenes cloud

- Plantillas de sistemas operativos preparadas para aprovisionamiento automático
- Tamaño muy reducido (sin escritorio, configuración mínima)
- Formato `qcow2`

```bash
# Descargar imágenes base
sudo wget https://cloud.debian.org/images/cloud/...
       -O debian13-base.qcow2
sudo wget https://cloud-images.ubuntu.com/...
       -O ubuntu2404-base.qcow2
```

</div>

<div class="card card-green">

### cloud-init

Herramienta que configura automáticamente la máquina en su **primer arranque**:

- Hostname
- Usuario y contraseña / clave SSH
- Paquetes a instalar
- Configuración de red (netplan)

OpenTofu genera un **disco ISO** con la configuración cloud-init.

</div>

</div>

---

## Recursos típicos en `main.tf`

```hcl
# Disco principal — clonación enlazada desde la imagen base
resource "libvirt_volume" "server1-disk" {
  name           = "server1.qcow2"
  pool           = var.libvirt_pool_name
  base_volume_id = "/var/lib/libvirt/images/debian13-base.qcow2"
  format         = "qcow2"
}

# Disco ISO con la configuración cloud-init
resource "libvirt_cloudinit_disk" "server1-cloudinit" {
  name      = "server1-cloudinit.iso"
  user_data = data.template_file.user_data.rendered
}

# Máquina virtual
resource "libvirt_domain" "server1" {
  name   = "server1"
  memory = 1024
  vcpu   = 1
  disk   = [{ volume_id = libvirt_volume.server1-disk.id }, ...]
  network_interface { network_name = "default"; wait_for_lease = true }
}
```

---

## Tipos de red con OpenTofu + libvirt

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Redes no gestionadas por OpenTofu

Se referencia por nombre:

```hcl
network_interface {
  network_name    = "default"
  wait_for_lease  = true
}
```

</div>

<div class="card card-green">

### Redes gestionadas por OpenTofu

Se crea y se referencia por ID:

```hcl
resource "libvirt_network" "nat_dhcp" {
  name      = "red-nat"
  mode      = "nat"
  addresses = ["192.168.100.0/24"]
}

network_interface {
  network_id     = libvirt_network.nat_dhcp.id
  wait_for_lease = true
}
```

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Cuando conectamos una VM a dos redes, debemos configurar <strong>ambas interfaces en cloud-init</strong> con un fichero <code>network-config.yaml</code>.</div>
</div>

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## OpenTofu + libvirt

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 PI</span>
</div>
