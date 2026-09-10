---
marp: true
title: Virtualización en Linux
theme: profesional
paginate: true
header: 'IV · Unidad 1 — Virtualización en Linux'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# **Virtualización** en Linux

## Conceptos, tipos y herramientas

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 IV · Infraestructura Virtual</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Introducción a la virtualización

## Concepto, usos, ventajas y conceptos básicos

---

## ¿Qué es la virtualización?

> La **virtualización** utiliza el software para imitar las características del hardware y crear un sistema informático virtual.

### Idea fundamental

- Permite ejecutar **varios sistemas virtuales** sobre un mismo equipo físico
- Múltiples **sistemas operativos** y aplicaciones funcionando en paralelo
- Aumenta el **rendimiento** del hardware disponible
- Aprovecha el tiempo de procesamiento que normalmente se desperdicia

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div>La virtualización es la base sobre la que se construyen las modernas plataformas <strong>cloud</strong> y los sistemas de despliegue automatizado.</div>
</div>

---

## ¿Para qué se utiliza?

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Producción y servicios

- **Aislamiento e independencia** de servicios y contenidos
- **Migración en vivo** entre servidores
- Creación de **clústeres** y sistemas distribuidos

</div>

<div class="card card-green">

### Desarrollo y formación

- **Laboratorio de pruebas** sin riesgo
- Virtualización de **arquitecturas** de las que no se dispone
- **Herramientas de aprendizaje** y experimentación

</div>

</div>

---

## Ventajas y desventajas

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-green">

### Ventajas

- Importante **ahorro económico**
- Mayor **seguridad** y aislamiento
- Mejor **aprovechamiento** de recursos
- **Migración en vivo** entre hosts
- Notable **ahorro energético**

</div>

<div class="card card-red">

### Desventajas

- Muchos sistemas dependen de un **único equipo físico**
- Ligera **penalización** en rendimiento
- Mayor **complejidad** de gestión

</div>

</div>

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>La <strong>alta disponibilidad</strong> es imprescindible cuando concentramos varios servicios virtualizados en un mismo host.</div>
</div>

---

## Conceptos básicos

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Anfitrión (*host*)

Sistema operativo que ejecuta el software de virtualización. **Controla el hardware real**.

### Invitado / huésped (*guest*)

Sistema operativo **virtualizado** que se ejecuta sobre el anfitrión.

</div>

<div class="card card-green">

### Hipervisor

Software de virtualización que **gestiona los invitados** y reparte los recursos del host.

### Soporte hardware

Desde 2005, **Intel VT** y **AMD-V** añaden extensiones de virtualización al procesador para mejorar el rendimiento.

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Tipos de virtualización

## Emulación, virtualización completa (tipo 1 y 2) y contenedores

---

## Emulación

<div class="cols-40-60" style="margin-top:0.8rem">

<div>

![w:100%](img/emulacion.png)

</div>

<div>

- El hipervisor **imita por software** una arquitectura completa: procesador, memoria, instrucciones, comunicaciones…
- Los programas se ejecutan creyendo que están sobre una **arquitectura concreta**
- Útil para correr software de **otra plataforma** (ej: ARM sobre x86)
- **Rendimiento bastante bajo**

### Ejemplos

`QEMU` · `Microsoft Virtual PC` · `Wine`

</div>

</div>

---

## Virtualización completa

<div class="cols-40-60" style="margin-top:0.8rem">

<div>

![w:100%](img/virt_completa.png)

</div>

<div>

- El hipervisor simula **suficiente hardware** para que un sistema operativo **no adaptado** se ejecute aislado, sin darse cuenta de que está virtualizado
- La CPU **debe disponer** de las extensiones de virtualización (Intel VT / AMD-V)
- Se clasifica en **dos tipos**, según dónde se ejecuta el hipervisor

</div>

</div>

---

## Tipo 1 (nativo) y Tipo 2 (alojado)

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Tipo 1 — nativo / *bare-metal*

- El hipervisor se ejecuta **directamente sobre el hardware**, sin un SO anfitrión de por medio
- **Mayor rendimiento**: es la opción habitual para servidores

#### Ejemplos

`KVM` · `Xen` · `VMware ESXi` · `Microsoft Hyper-V`

</div>

<div class="card card-green">

### Tipo 2 — alojado / *hosted*

- El hipervisor se instala **como una aplicación más** sobre el SO del host
- **Menor rendimiento**: añade una capa extra entre el hardware y la VM

#### Ejemplos

`VirtualBox` · `VMware Workstation` · `Parallels Desktop`

</div>

</div>

---

## Virtualización ligera o en contenedores

<div class="cols-40-60" style="margin-top:0.8rem">

<div>

![w:100%](img/virt_ligera.png)

</div>

<div>

- También llamada **virtualización a nivel de SO**
- Sobre el **núcleo** del SO se ejecuta una capa que permite múltiples **espacios de usuario aislados** (contenedores)
- Un contenedor es un **conjunto de procesos** aislados con:
  - **Sistema de ficheros** propio
  - **Configuración de red** propia
  - Acceso a recursos del host (CPU, memoria)

</div>

</div>

---

## Tipos de contenedores

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Contenedores de **sistema**

Su uso es similar al de una máquina virtual tradicional:

- Se accede por **SSH**
- Se **instalan servicios**, se actualizan
- Ejecutan un **conjunto de procesos**

#### Ejemplo

`LXC` *(Linux Container)*

</div>

<div class="card card-green">

### Contenedores de **aplicación**

Pensados para el **despliegue** de aplicaciones (especialmente web):

- Una aplicación = un contenedor
- Imágenes versionadas y portables
- Base del *cloud-native*

#### Ejemplos

`Docker` · `Podman`

</div>

</div>

---

## Resumen comparativo

| Tipo | Hipervisor | SO modificado | Rendimiento | Ejemplos |
|:--|:--|:--:|:--|:--|
| **Emulación** | — | No | Muy bajo | QEMU, Wine |
| **Virtualización completa** | Tipo 1 (nativo) | No | Alto | KVM, Xen, ESXi |
| **Virtualización completa** | Tipo 2 (alojado) | No | Medio | VirtualBox, VMware |
| **Contenedores** | — | — | Casi nativo | LXC, Docker |

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

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

- En virtualización completa, los dispositivos (discos, red…) están **emulados por software**
- La VM interactúa con ellos como si fuesen físicos → **poco rendimiento**
- KVM ofrece una alternativa: los **dispositivos paravirtualizados**, agrupados como **`virtIO`**

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-red">

### Dispositivos emulados

- Compatibilidad universal
- **Rendimiento bajo**
- Cada operación atraviesa la capa de emulación

</div>

<div class="card card-green">

### Dispositivos `virtIO`

- Drivers específicos en el invitado
- **Rendimiento muy cercano al real**
- Recomendado para discos y tarjetas de red

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

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Virtualización en Linux

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 IV</span>
</div>
