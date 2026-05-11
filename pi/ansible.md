---
marp: true
title: Ansible
theme: profesional
paginate: true
header: 'PI · Ansible'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# **Ansible**

## Gestión de la configuración y automatización

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 PI · Puesta en Producción de Aplicaciones</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Introducción a Ansible

## Qué es, instalación y primeros pasos

---

## ¿Qué es Ansible?

> **Ansible** es una herramienta de **gestión de configuración y automatización** que permite definir, en ficheros de texto YAML, el estado deseado de servidores y aplicaciones, y aplicarlo **sin instalar agentes** en los nodos (usa SSH).

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Características

- Desarrollado principalmente por **Red Hat**
- Escrito en **Python** · disponible en PyPI
- Arquitectura **push** sin agentes — solo SSH
- Configuración declarativa en **YAML**
- Primera versión: **2012**

</div>

<div class="card card-green">

### ¿Para qué sirve?

- Configurar servidores de forma automática
- Instalar y gestionar paquetes y servicios
- Desplegar aplicaciones
- Orquestar tareas en **múltiples máquinas** a la vez
- Gestionar actualizaciones del sistema

</div>

</div>

---

## Ansible en el contexto de IaC

La **Infraestructura como Código (IaC)** define y gestiona la infraestructura mediante ficheros de configuración en lugar de hacerlo manualmente. Existen dos tipos de herramientas:

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Software de Orquestación

Programa la **creación de escenarios**: máquinas virtuales, redes, almacenamiento…

**Ejemplo: OpenTofu**

</div>

<div class="card card-green">

### Software de Gestión de la Configuración (CMS)

Programa la **configuración que van a tener los servidores** una vez creados.

**Ejemplo: Ansible**

</div>

</div>

---

## Instalación de Ansible

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Con apt (Debian/Ubuntu)

```bash
apt install ansible
```

Instala la versión empaquetada por la distribución.

</div>

<div class="card card-green">

### Con pip en entorno virtual (recomendado)

```bash
python3 -m venv ansible-env
source ansible-env/bin/activate
pip install ansible
```

Permite tener la **versión más reciente** e independiente del sistema.

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Ansible solo necesita instalarse en el <strong>nodo de control</strong> (tu máquina). Los nodos gestionados únicamente requieren Python y acceso SSH.</div>
</div>

---

## El inventario

El **inventario** es el fichero donde definimos los equipos que Ansible va a gestionar. Los equipos se organizan en **grupos**.

```yaml
all:
  children:
    servidores:
      hosts:
        nodo1:
          ansible_ssh_host: 192.168.1.10
          ansible_ssh_user: usuario
          ansible_ssh_private_key_file: ~/.ssh/id_rsa
```

- `all` — grupo raíz que engloba todos los equipos
- `servidores` — grupo personalizado con nuestros hosts
- Cada host puede tener variables propias: IP, usuario SSH, clave privada…

---

## Fichero de configuración

Junto al inventario se crea un fichero `ansible.cfg` que configura el comportamiento de Ansible para el proyecto:

```ini
[defaults]
inventory = hosts
host_key_checking = False
```

- **`inventory`** — ruta al fichero de inventario
- **`host_key_checking = False`** — evita la verificación de la clave del host en cada conexión SSH (útil en entornos de pruebas)

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Ansible busca este fichero en el directorio de trabajo. Cada proyecto puede tener su propia configuración.</div>
</div>

---

## Módulos y comandos ad-hoc

Un **módulo** permite ejecutar una acción concreta en uno o varios servidores remotos.

Los **comandos ad-hoc** ejecutan un módulo directamente desde la línea de comandos, sin necesidad de un playbook:

```bash
ansible <hosts> -m <módulo> -a "<parámetros>"
```

| Parte | Descripción |
|:--|:--|
| `<hosts>` | `all`, nombre de grupo o nombre de host del inventario |
| `-m <módulo>` | módulo a usar |
| `-a "<parámetros>"` | argumentos del módulo |
| `--become` | ejecutar como `root` en el nodo remoto |

---

## Módulos esenciales (I)

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### ping

Comprueba la conectividad con los nodos. No es un ping ICMP, sino una verificación de conexión SSH y Python.

```bash
ansible all -m ping
ansible servidores -m ping
```

### command / shell

Ejecuta comandos en el nodo remoto. `shell` permite pipes, redirecciones y variables de entorno.

```bash
ansible all -m command -a "uptime"
ansible all -m shell -a "echo $HOME | wc -c"
```

</div>

<div class="card card-green">

### copy

Copia ficheros desde el nodo de control al nodo remoto.

- `src` — fichero origen (local)
- `dest` — ruta de destino (remoto)
- `mode` — permisos (opcional)

```bash
ansible all -m copy \
  -a "src=./index.html dest=/tmp/index.html mode=0644"
```

</div>

</div>

---

## Módulos esenciales (II)

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### apt

Instala, actualiza o elimina paquetes en sistemas Debian/Ubuntu.

- `name` — paquete
- `state` — `present`, `absent`, `latest`

```bash
ansible nodo1 -m apt \
  -a "name=apache2 state=present" --become
```

### service

Gestiona servicios del sistema.

- `name` — servicio
- `state` — `started`, `stopped`, `restarted`
- `enabled` — arranque automático

```bash
ansible nodo1 -m service \
  -a "name=apache2 state=started enabled=yes" --become
```

</div>

<div class="card card-green">

### file

Gestiona ficheros, directorios y permisos.

- `path` — ruta en el nodo remoto
- `state` — `file`, `directory`, `absent`, `link`
- `mode` — permisos

```bash
ansible all -m file \
  -a "path=/tmp/demo state=directory mode=0755"
```

### user

Crea, modifica o elimina usuarios.

- `name` — nombre del usuario
- `state` — `present`, `absent`
- `shell`, `groups`

```bash
ansible all -m user \
  -a "name=demo shell=/bin/bash state=present" --become
```

</div>

</div>

---

## Declarativo e idempotencia

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Enfoque declarativo

Ansible **no** usa un esquema imperativo (*"instala apache"*).

Declaramos el **estado deseado** del servidor:

> *"Quiero que el servidor tenga apache2 instalado"*

Ansible hará todas las operaciones necesarias para que ese estado se cumpla.

</div>

<div class="card card-green">

### Idempotencia

Si el estado declarado **ya se ha alcanzado**, Ansible no ejecuta ninguna operación adicional.

- Primera ejecución: instala apache → salida en **amarillo** (`changed`)
- Segunda ejecución: ya está instalado → salida en **verde** (`ok`)

Podemos ejecutar el mismo playbook **múltiples veces** con total seguridad.

</div>

</div>

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Ansible · Introducción

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 PI</span>
</div>
