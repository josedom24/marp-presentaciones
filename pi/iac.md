---
marp: true
title: Infraestructura como Código
theme: profesional
paginate: true
header: 'PI · Infraestructura como Código'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Infraestructura como **Código**

## OpenTofu, Terraform y Ansible

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 PI · Puesta en Producción de Aplicaciones</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Evolución de la gestión de infraestructura

## Del despliegue tradicional al cloud

---

## Despliegue tradicional

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### El proceso clásico

- Comprar el servidor o crear la máquina virtual
- Instalar y configurar el sistema operativo
- Instalar y configurar los servicios
- Configurar la seguridad
- Desplegar la aplicación
- **Documentar... ¿o no?**

</div>

<div class="card card-red">

### Problemas del enfoque tradicional

- **Configuración artesanal** — difícil de reproducir
- La documentación **se desactualiza** o directamente no existe
- **Escalado vertical**: paradas de servicio
- Misma configuración usada durante años sin revisión
- *"Funciona en mi máquina"*

</div>

</div>

---

## Despliegue moderno con Cloud Computing

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-green">

### Nuevas posibilidades

- Aparición del **Cloud Computing (IaaS)**: AWS, Azure, GCP, OpenStack…
- Aprovisionamiento de VMs **desde una imagen o plantilla**
- Uso de herramientas de **gestión de la configuración**
- Despliegue desde un entorno de pruebas **idéntico al de producción**
- **Escalado horizontal** sin paradas

</div>

<div class="card card-purple">

### El cambio de mentalidad

- Los servidores no tienen por qué mantenerse con la misma configuración durante años
- La infraestructura es **desechable y reproducible**
- Los recursos se crean y destruyen según la demanda
- **Todo el proceso puede automatizarse**

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Infraestructura como Código

## IaC: orquestación, configuración y DevOps

---

## ¿Podemos programar la infraestructura?

> Los servicios de virtualización y Cloud Computing son **software**, por lo tanto los podemos **programar mediante APIs**.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Software de Orquestación

Permite **crear y gestionar escenarios** completos: máquinas virtuales, redes, almacenamiento…

**Ejemplos:** Terraform, OpenTofu, Pulumi, AWS CloudFormation, Heat (OpenStack)

</div>

<div class="card card-green">

### Software de Gestión de la Configuración (CMS)

Permite **configurar el software** de las máquinas que hemos creado.

**Ejemplos:** Ansible, Puppet, Chef, Salt

</div>

</div>

---

## Software de Orquestación

<div class="cols-2" style="margin-top:0.8rem">

<div>

**Utilizado para crear escenarios completos con múltiples servidores o contenedores** (aprovisionamiento de recursos).

- Muy útil con **demanda variable de recursos**
- Muy útil cuando la configuración **cambia continuamente**
- Puede incluir funcionalidad de **autoescalado**
- Puede incluir respuestas a **eventos**

</div>

<div class="card card-blue">

### Enfoque declarativo

En lugar de describir *cómo* crear la infraestructura, **declaramos el estado deseado** y la herramienta se encarga de alcanzarlo.

```hcl
resource "openstack_compute_instance_v2" "web" {
  name      = "servidor-web"
  image_id  = "ubuntu-22.04"
  flavor_id = "m1.small"
}
```

</div>

</div>

---

## Software de Gestión de la Configuración (CMS)

Proporciona la gestión e instalación del software de forma **declarativa**:

<div class="cols-3" style="margin-top:0.8rem">

<div class="card card-blue">

### Automatización

Instala y configura software de forma **automática** en las máquinas de la infraestructura

</div>

<div class="card card-green">

### Declarativo

Se usan **ficheros de texto** donde se declara la configuración deseada, no los pasos para llegar a ella

</div>

<div class="card card-purple">

### Idempotencia

Se puede ejecutar una declaración **múltiples veces** y el resultado siempre es el mismo

</div>

</div>

---

## Cambio de paradigma: Infraestructura como Código

> Trata tu infraestructura como el **software** que es.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Buenas prácticas de desarrollo

- **Control de versiones** (Git) para toda la infraestructura
- Código **legible y documentado**
- **Revisión de código** antes de aplicar cambios
- **Tests** para validar la configuración

</div>

<div class="card card-green">

### Beneficios operativos

- Escenarios **replicables** y predecibles
- **Auditoría** completa de cambios
- **Recuperación ante desastres** mucho más rápida
- Entornos de desarrollo **idénticos a producción**

</div>

</div>

---

## DevOps

<div class="cols-60-40" style="margin-top:0.8rem">

<div>

**Conflicto histórico:** los equipos de Desarrollo (DEV) y Sistemas (OPS) han tenido objetivos y responsabilidades diferentes.

### ¿Cómo solucionarlo?

- **Mismas herramientas** para dev y ops
- Extender las buenas prácticas de desarrollo a sistemas
- De integración continua → **entrega continua** → **despliegue continuo**
- **Infraestructura como código**
- **Escenarios replicables y automatización** de la configuración

</div>

<div class="card card-purple" style="align-self:center">

### El objetivo es común

Entregar software **de calidad**, de forma **rápida** y **fiable**.

La IaC es una de las piedras angulares de la cultura DevOps.

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# OpenTofu

## Orquestación de infraestructura declarativa

---

## OpenTofu

> **OpenTofu** es una herramienta de **orquestación de infraestructura** de código abierto, fork libre de Terraform, mantenida por la **Linux Foundation** (licencia MPL 2.0).

- Creado en **septiembre de 2023** cuando HashiCorp cambió la licencia de Terraform a BSL 1.1 (no libre)
- Lenguaje declarativo propio: **HCL** (*HashiCorp Configuration Language*)
- Funciona con **múltiples proveedores**: AWS, Azure, GCP, OpenStack, Kubernetes…
- Gestiona el **estado de la infraestructura** en un fichero `terraform.tfstate`
- Compatible con todos los **módulos y providers** de Terraform
- Sitio oficial: **opentofu.org**

---

## Conceptos sobre OpenTofu

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Elementos de configuración

- **Provider**: plugin que conecta con una API (OpenStack, AWS…)
- **Resource**: elemento de infraestructura a crear (VM, red, volumen…)
- **Variable**: parámetro para reutilizar la configuración en distintos entornos
- **Output**: valor de salida (IP, ID…) tras aplicar el plan
- **Module**: agrupación reutilizable de resources

</div>

<div class="card card-green">

### Flujo de trabajo

```bash
# Inicializar — descarga providers y módulos
tofu init

# Planificar — muestra los cambios a realizar
tofu plan

# Aplicar — crea/modifica la infraestructura
tofu apply

# Destruir — elimina todos los recursos
tofu destroy
```

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# Ansible

## Gestión de la configuración

---

## ¿Qué es Ansible?

> **Ansible** es un software de **gestión de la configuración (CMS)** que permite declarar y automatizar la configuración de máquinas.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Características

- Desarrollado principalmente por **Red Hat**
- Escrito en **Python** (disponible en PyPI)
- Arquitectura **push** — sin agentes, solo SSH
- Configuración declarativa en **YAML**
- Primera versión: **2012**

</div>

<div class="card card-green">

### ¿Por qué Ansible?

- Sintaxis **sencilla y conocida** (YAML)
- **Sin agentes** — solo necesita SSH
- **Fácil de instalar** y de empezar a usar
- **Comunidad muy activa**
- Repositorio de roles: **Ansible Galaxy**

</div>

</div>

---

## Conceptos fundamentales de Ansible

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Módulos

Funciones predefinidas que realizan una acción concreta: copiar un fichero, instalar paquetes, reiniciar un servicio…

Cada módulo acepta parámetros obligatorios y opcionales.

```yaml
- name: Instalar Apache
  ansible.builtin.apt:
    name: apache2
    state: present
```

</div>

<div class="card card-green">

### Inventario

Fichero donde se definen los **hosts** y **grupos de hosts** que Ansible va a gestionar.

```ini
[webservers]
servidor1.ejemplo.com
servidor2.ejemplo.com

[dbservers]
db.ejemplo.com
```

</div>

</div>

---

## Playbooks y Roles

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-purple">

### Playbooks

Ficheros YAML que agrupan **jugadas (plays)**: conjuntos de tareas para conseguir una configuración completa.

```yaml
- name: Configurar servidor web
  hosts: webservers
  become: true
  tasks:
    - name: Instalar Apache
      apt:
        name: apache2
        state: present
    - name: Copiar configuración
      copy:
        src: apache.conf
        dest: /etc/apache2/sites-enabled/
```

</div>

<div class="card card-blue">

### Roles

Permiten **estructurar y reutilizar** los playbooks dividiéndolos por servicio o funcionalidad.

```
roles/
└── apache2/
    ├── tasks/main.yml
    ├── handlers/main.yml
    ├── templates/
    ├── files/
    └── defaults/main.yml
```

Los roles se comparten en **Ansible Galaxy**: `ansible-galaxy install geerlingguy.apache`

</div>

</div>

---

## OpenTofu + Ansible: la combinación perfecta

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### OpenTofu — Orquestación

Crea y gestiona la **infraestructura**:

- Máquinas virtuales
- Redes y subredes
- Almacenamiento
- IPs flotantes
- Grupos de seguridad

</div>

<div class="card card-green">

### Ansible — Configuración

Configura el **software** de las máquinas creadas:

- Instalación de paquetes
- Configuración de servicios
- Despliegue de aplicaciones
- Gestión de usuarios
- Actualizaciones del sistema

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>💡</span><div>OpenTofu puede invocar Ansible automáticamente como <strong>provisioner</strong>, o bien generar el inventario dinámico para que Ansible lo consuma.</div>
</div>

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Infraestructura como Código

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 PI</span>
</div>
