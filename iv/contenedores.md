---
marp: true
title: Virtualización en contenedores
theme: profesional
paginate: true
header: 'IV · Virtualización en contenedores'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Virtualización en **contenedores**

## Concepto y tipos de contenedores

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 IV · Infraestructura Virtual</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Virtualización ligera en contenedores

## Concepto y tipos

---

## ¿Qué es la virtualización en contenedores?

<div class="cols-40-60" style="margin-top:0.8rem">

<div>

![w:100%](img/virt_ligera.png)

</div>

<div>

- También llamada **virtualización a nivel de sistema operativo** o **virtualización basada en contenedores**
- Sobre el núcleo del SO se ejecuta una capa de virtualización que permite múltiples **instancias aisladas** de espacios de usuario
- Un **contenedor** es un conjunto de procesos aislados con:
  - **Sistema de ficheros** propio
  - **Configuración de red** propia
  - Acceso a los recursos del host (memoria y CPU)

</div>

</div>

---

## Tipos de contenedores

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Contenedores de sistema

Su uso es similar al de una **máquina virtual tradicional**:

- Se accede por **SSH**
- Se instalan servicios, se actualizan
- Ejecutan un conjunto de procesos en paralelo

**Ejemplo:** `LXC` *(Linux Container)*

</div>

<div class="card card-green">

### Contenedores de aplicación

Pensados para el **despliegue de aplicaciones** web:

- Una aplicación por contenedor
- Paradigma *build, ship and run*
- El contenedor se para cuando termina su proceso principal

**Ejemplos:** `Docker`, `Podman`

</div>

</div>

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Virtualización en contenedores

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 IV</span>
</div>
