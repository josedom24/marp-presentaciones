---
marp: true
title: Software de Orquestación
theme: profesional
paginate: true
header: 'PI · Software de Orquestación'
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
  <span>📚 PI · Puesta en Producción de Aplicaciones</span>
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
- **Auditoría** completa de los cambios
- Recuperación ante desastres mucho más rápida

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

*Fork* libre de Terraform 1.5.x creado por la comunidad y gobernado por la **Linux Foundation**. Licencia **MPL 2.0** (libre), 100% compatible con los ficheros `.tf` y providers existentes.

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
resource "aws_instance" "web" {
  ami           = var.imagen
  instance_type = "t2.micro"
}

output "ip_publica" {
  value = aws_instance.web.public_ip
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

## Ficheros más importantes de un proyecto

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Los escribimos nosotros

- **`provider.tf`** — configura el/los provider a usar
- **`variables.tf`** — declara las variables reutilizables
- **`main.tf`** — define los recursos: VMs, redes, discos…
- **`output.tf`** — muestra información al finalizar (IPs, IDs…)

</div>

<div class="card card-yellow">

### Lo genera OpenTofu

- **`terraform.tfstate`** — el **estado** de la infraestructura
- **`.terraform/`** — providers y módulos descargados por `init`

No se editan a mano.

</div>

</div>

---

## El estado de la infraestructura

> El **estado** (`terraform.tfstate`) es un fichero donde OpenTofu guarda **qué recursos ha creado realmente** y con qué identificadores (IDs, IPs, dependencias…).

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### ¿Por qué se guarda?

- Es la única forma de saber **qué existe ya** sin preguntar a la plataforma recurso a recurso
- Permite **comparar** lo declarado en el código con lo que hay realmente creado
- Sin él, OpenTofu no sabría qué modificar o eliminar

</div>

<div class="card card-red">

### Cuidado con el estado

- **No se edita a mano**
- Si varios usuarios trabajan a la vez, hace falta un **backend remoto** compartido para evitar conflictos
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

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Software de Orquestación · OpenTofu

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 PI</span>
</div>
