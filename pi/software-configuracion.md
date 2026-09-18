---
marp: true
title: Software de Gestión de la Configuración
theme: profesional
paginate: true
header: 'PI · Proyecto Interdisciplinar'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Software de Gestión de la **Configuración**

## CMS · Ansible

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

## Orquestación, Configuración y DevOps

---

## Infraestructura como Código

> Gestionar y aprovisionar infraestructura mediante **ficheros de código**, en lugar de configurarla manualmente.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### 🏗️ Software de Orquestación

Crea escenarios completos: máquinas, redes, almacenamiento…

**Terraform / OpenTofu**

</div>

<div class="card card-green">

### ⚙️ Software de Gestión de la Configuración (CMS)

Configura el **software** de las máquinas ya creadas.

**Puppet, Chef, Salt, Ansible**

</div>

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

Sí: la IaC es una de las piedras angulares de la cultura DevOps. Orquestación (OpenTofu) y configuración (Ansible) son las dos herramientas que la hacen posible.

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Gestión de la Configuración

## Configurar el software de las máquinas

---

## ¿Qué hace un software de gestión de la configuración?

Instala y configura el software de las máquinas de forma **automática, declarativa e idempotente**.

<div class="cols-3" style="margin-top:0.8rem">

<div class="card card-blue">

### Automatización

Instala y configura software **sin intervención manual** en cada máquina

</div>

<div class="card card-green">

### Declarativo

Se declara **qué estado** debe tener la máquina, no los pasos para llegar a él

</div>

<div class="card card-purple">

### Idempotencia

Ejecutar la misma configuración **varias veces** da siempre el mismo resultado

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>💡</span><div><strong>Ejemplos:</strong> Ansible, Puppet, Chef, Salt.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# Introducción a Ansible

## Qué es, instalación y primeros pasos

---

## ¿Qué es Ansible?

> **Ansible** es una herramienta de **gestión de configuración y automatización** que permite definir, en ficheros de texto YAML, el estado deseado de servidores y aplicaciones, y aplicarlo **sin instalar agentes** en los nodos (usa SSH).

<div class="card card-blue" style="margin-top:0.8rem">

### Características

- Desarrollado principalmente por **Red Hat**
- Escrito en **Python** · disponible en PyPI
- Arquitectura **push** sin agentes — solo SSH
- Configuración declarativa en **YAML**
- Primera versión: **2012**

</div>

---

## ¿Para qué sirve Ansible?

<div class="card card-green" style="margin-top:0.8rem">

### ¿Para qué sirve?

- Configurar servidores de forma automática
- Instalar y gestionar paquetes y servicios
- Desplegar aplicaciones
- Orquestar tareas en **múltiples máquinas** a la vez
- Gestionar actualizaciones del sistema

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

<div class="card card-blue" style="margin-top:0.8rem">

### ping

Comprueba la conectividad con los nodos. No es un ping ICMP, sino una verificación de conexión SSH y Python.

```bash
ansible all -m ping
```

</div>

---

## Módulos esenciales (II)

<div class="card card-blue" style="margin-top:0.8rem">

### command / shell

Ejecuta comandos en el nodo remoto. `shell` permite pipes, redirecciones y variables de entorno.

```bash
ansible all -m shell -a "echo $HOME | wc -c"
```

</div>

---

## Módulos esenciales (III)

<div class="card card-blue" style="margin-top:0.8rem">

### copy

Copia ficheros desde el nodo de control al nodo remoto.

- `src` — fichero origen (local)
- `dest` — ruta de destino (remoto)
- `mode` — permisos (opcional)

```bash
ansible all -m copy -a "src=./index.html dest=/tmp/index.html mode=0644"
```

</div>

---

## Módulos esenciales (IV)

<div class="card card-blue" style="margin-top:0.8rem">

### apt

Instala, actualiza o elimina paquetes en sistemas Debian/Ubuntu.

- `name` — paquete
- `state` — `present`, `absent`, `latest`

```bash
ansible nodo1 -m apt -a "name=apache2 state=present" --become
```

</div>

---

## Módulos esenciales (V)

<div class="card card-blue" style="margin-top:0.8rem">

### service

Gestiona servicios del sistema.

- `name` — servicio
- `state` — `started`, `stopped`, `restarted`
- `enabled` — arranque automático

```bash
ansible nodo1 -m service -a "name=apache2 state=started enabled=yes" --become
```

</div>

---

## Módulos esenciales (VI)

<div class="card card-blue" style="margin-top:0.8rem">

### file

Gestiona ficheros, directorios y permisos.

- `path` — ruta en el nodo remoto
- `state` — `file`, `directory`, `absent`, `link`
- `mode` — permisos

```bash
ansible all -m file -a "path=/tmp/demo state=directory mode=0755"
```

</div>

---

## Módulos esenciales (VII)

<div class="card card-blue" style="margin-top:0.8rem">

### user

Crea, modifica o elimina usuarios.

- `name` — nombre del usuario
- `state` — `present`, `absent`
- `shell`, `groups`

```bash
ansible all -m user -a "name=demo shell=/bin/bash state=present" --become
```

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

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# Playbooks

## Automatización declarativa con Ansible

---

## Play y Playbook

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Play

Una **jugada** es la unidad mínima de trabajo: declara **en qué hosts** se ejecuta y **qué tareas** se realizan.

Cada tarea usa un módulo con sus parámetros para alcanzar un estado deseado.

</div>

<div class="card card-green">

### Playbook

Un **libro de jugadas** es un fichero YAML que agrupa uno o varios plays para conseguir una **configuración completa** de la infraestructura.

Se ejecuta con:

```bash
ansible-playbook site.yml
```

</div>

</div>

---

## Estructura de un playbook

```yaml
---
- name: Configurar servidor web          # Descripción del play
  hosts: servidores                      # A qué hosts se aplica
  become: true                           # Ejecutar tareas como root (sudo)

  tasks:

    - name: Actualizar caché de apt
      ansible.builtin.apt:
        update_cache: true

    # ... resto de la lista de tareas
```

---

## Variables en Ansible

Ansible puede trabajar con variables obtenidas de distintas fuentes:

- **Nivel de nodo** — definidas en el inventario para un host concreto
- **Nivel de grupo** — ficheros en `group_vars/` para todos los hosts de un grupo
- **Gathering Facts** — recopiladas automáticamente por Ansible de cada nodo

---

## Variables: nivel de nodo

<div class="card card-blue" style="margin-top:0.8rem">

### Nivel de nodo

Variables definidas directamente en el inventario para un host concreto.

```yaml
hosts:
  nodo1:
    ansible_ssh_host: 192.168.1.10
    ansible_ssh_user: jose
    http_port: 80
```

</div>

---

## Variables: nivel de grupo

<div class="card card-green" style="margin-top:0.8rem">

### Nivel de grupo

Variables en **`group_vars/`** que se aplican a todos los hosts de un grupo.

```
mi-proyecto/
└── group_vars/
    ├── all          # Globales
    └── servidores   # Del grupo
```

</div>

---

## group_vars/all

<div class="card card-blue" style="margin-top:0.8rem">

### Contenido

```yaml
---
usuario_app: deploy
puerto_web: 80
paquetes:
  - apache2
  - git
  - curl
```

</div>

---

## Uso de group_vars en el playbook

<div class="card card-green" style="margin-top:0.8rem">

### Uso en el playbook

Se referencian con dobles llaves `{{ }}`:

```yaml
- name: Instalar paquetes
  ansible.builtin.apt:
    name: "{{ item }}"
    state: present
  loop: "{{ paquetes }}"
```

</div>

---

## Gathering Facts

Al inicio de cada play, Ansible ejecuta automáticamente la tarea **Gather Facts** que recopila información del nodo remoto.

<div class="card card-blue" style="margin-top:0.8rem">

### Variables disponibles

| Variable | Valor de ejemplo |
|:--|:--|
| `ansible_hostname` | `servidor1` |
| `ansible_distribution` | `Debian` |
| `ansible_distribution_version` | `12` |
| `ansible_default_ipv4.address` | `192.168.1.10` |
| `ansible_memtotal_mb` | `2048` |

</div>

---

## Uso de las facts en plantillas y tareas

<div class="card card-green" style="margin-top:0.8rem">

### Uso en plantillas y tareas

```yaml
- name: Mensaje de bienvenida
  ansible.builtin.copy:
    content: >
      Servidor: {{ ansible_hostname }}
      Sistema: {{ ansible_distribution }}
      {{ ansible_distribution_version }}
    dest: /etc/motd
```

</div>

---

## Ver las facts de un nodo

Se pueden consultar todas las facts recopiladas de un nodo con el módulo `setup`:

```bash
ansible nodo1 -m setup
ansible nodo1 -m setup -a "filter=ansible_distribution*"
```

---

## El módulo template

El módulo **template** copia una plantilla **Jinja2** al nodo remoto sustituyendo las variables por sus valores.

<div class="cols-2" style="margin-top:0.8rem">

<div>

Las plantillas se guardan en el directorio **`templates/`** con extensión `.j2`:

**`templates/index.j2`:**

```html
<!DOCTYPE html>
<html>
<body>
  <h1>{{ ansible_hostname }}</h1>
  <p>Sistema: {{ ansible_distribution }}
             {{ ansible_distribution_version }}</p>
  <p>Gestionado por: {{ usuario_app }}</p>
</body>
</html>
```

</div>

<div class="card card-blue">

### Tarea en el playbook

```yaml
- name: Desplegar página de inicio
  ansible.builtin.template:
    src: templates/index.j2
    dest: /var/www/html/index.html
    mode: "0644"
```

</div>

</div>

---

## Condicionales y bucles en Jinja2

Jinja2 permite también **condicionales** y **bucles** dentro de las plantillas:

```jinja2
{% if ansible_distribution == "Debian" %}
Sistema compatible.
{% endif %}

{% for pkg in paquetes %}
- {{ pkg }}
{% endfor %}
```

---

## Estructura de un proyecto Ansible

```
mi-proyecto/
├── ansible.cfg          # Configuración de Ansible
├── hosts                # Inventario
├── site.yml             # Playbook principal
├── files/               # Ficheros estáticos para copiar con copy
│   └── foo.txt
├── templates/           # Plantillas Jinja2 para el módulo template
│   └── index.j2
└── group_vars/          # Variables por grupo
    ├── all              # Variables globales
    └── servidores       # Variables del grupo servidores
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Ansible localiza automáticamente los directorios <strong>files/</strong>, <strong>templates/</strong> y <strong>group_vars/</strong> si están junto al playbook o en el directorio de trabajo.</div>
</div>

---

## Ejecutar un playbook e interpretar la salida

```bash
ansible-playbook site.yml
```

<div class="card card-blue" style="margin-top:0.8rem">

### Colores de la salida

| Color | Estado | Significado |
|:--|:--|:--|
| 🟢 Verde | `ok` | Ya estaba en el estado deseado |
| 🟡 Amarillo | `changed` | Se ha realizado un cambio |
| 🔴 Rojo | `failed` | La tarea ha fallado |
| 🔵 Azul | `skipping` | Tarea omitida (condición) |

</div>

---

## Resumen final (PLAY RECAP)

```
PLAY RECAP **********************
nodo1 : ok=4  changed=2  unreachable=0
        failed=0  skipped=0
```

- `ok` + `changed` — tareas ejecutadas
- Si `changed=0` en la segunda ejecución → **idempotencia** funcionando
- `unreachable` o `failed` → hay que revisar errores

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">05</p>

# Roles

## Organización y reutilización en Ansible

---

## ¿Qué es un rol?

> Un **rol** es una unidad de configuración reutilizable que agrupa todas las tareas, ficheros, plantillas y variables necesarias para configurar **un servicio concreto**.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Motivación

En un playbook con muchas tareas todo acaba en un único fichero. Los roles permiten:

- **Separar** la configuración de cada servicio
- **Reutilizar** el mismo rol en distintos proyectos
- **Asignar roles** a grupos de hosts diferentes
- Mantener el código **ordenado y mantenible**

</div>

<div class="card card-green">

### Ejemplo típico

Un proyecto con dos servidores:

- **`commons`** → tareas comunes a todas las máquinas
- **`apache2`** → instala y configura el servidor web
- **`mariadb`** → instala y configura la base de datos

</div>

</div>

---

## Estructura de un rol

Cada rol es un directorio dentro de `roles/` con subdirectorios predefinidos:

```
roles/
└── apache2/
    ├── tasks/
    │   └── main.yml       # Lista de tareas del rol
    ├── handlers/
    │   └── main.yml       # Handlers (reinicio de servicios, etc.)
    ├── templates/
    │   └── vhost.conf.j2  # Plantillas Jinja2
    ├── files/
    │   └── index.html     # Ficheros estáticos para copiar
    └── defaults/
        └── main.yml       # Variables por defecto del rol
```

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Solo es necesario crear los subdirectorios que se vayan a usar. Ansible los detecta automáticamente por nombre.</div>
</div>

---

## Usando roles en el playbook

El fichero `site.yml` asigna cada rol al grupo de hosts que corresponde:

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

```yaml
---
# Tareas comunes
- name: Configuración común
  hosts: all
  become: true
  roles:
    - commons

# Servidor web
- name: Configurar servidor web
  hosts: servidores_web
  become: true
  roles:
    - apache2
```

</div>

<div class="card card-green">

```yaml
# Servidor de base de datos
- name: Configurar servidor de base de datos
  hosts: servidores_bd
  become: true
  roles:
    - mariadb
```

</div>

</div>

---

## Handlers

> Un **handler** es una tarea especial que **solo se ejecuta cuando es notificada** por otra tarea. Se ejecuta **una sola vez**, al finalizar todas las tareas del play.

<div class="cols-2" style="margin-top:0.8rem">

<div>

**`roles/apache2/tasks/main.yml`**

```yaml
- name: Copiar configuración de Apache
  ansible.builtin.template:
    src: vhost.conf.j2
    dest: /etc/apache2/sites-available/000-default.conf
  notify: Reiniciar Apache
```

El parámetro `notify` indica qué handler activar si la tarea produce un cambio (`changed`). Si el fichero ya era igual, el handler **no se ejecuta**.

</div>

<div class="card card-blue">

**`roles/apache2/handlers/main.yml`**

```yaml
- name: Reiniciar Apache
  ansible.builtin.service:
    name: apache2
    state: restarted
```

Una tarea normal se ejecuta siempre; el handler **solo se dispara si hubo un cambio real**.

</div>

</div>

---

## Bucles con loop

Para ejecutar una tarea sobre una lista de elementos se usa `loop`. El valor de cada iteración se referencia con `{{ item }}`.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-red">

### ⚠️ Forma antigua (obsoleta)

```yaml
- name: Instalar paquetes
  ansible.builtin.apt:
    name: "{{ item }}"
    state: present
  with_items:
    - apache2
    - git
    - curl
```

`with_items` está **deprecado** desde Ansible 2.5.

</div>

<div class="card card-green">

### Forma actual

```yaml
- name: Instalar paquetes
  ansible.builtin.apt:
    name: "{{ item }}"
    state: present
  loop:
    - apache2
    - git
    - curl
```

O más idiomático para `apt`, pasando la lista directamente:

```yaml
- name: Instalar paquetes
  ansible.builtin.apt:
    name:
      - apache2
      - git
      - curl
    state: present
```

</div>

</div>

---

## El módulo lineinfile

Permite **modificar una línea concreta** de un fichero remoto sin sobreescribir el fichero completo. Muy útil para ajustar ficheros de configuración.

```yaml
- name: Configurar bind-address en MariaDB
  ansible.builtin.lineinfile:
    path: /etc/mysql/mariadb.conf.d/50-server.cnf
    regexp: '^bind-address'          # Línea a buscar (expresión regular)
    line: 'bind-address = 0.0.0.0'  # Valor que debe quedar
  notify: Reiniciar MariaDB
```

---

## Parámetros de lineinfile

| Parámetro | Descripción |
|:--|:--|
| `path` | Fichero a modificar en el nodo remoto |
| `regexp` | Expresión regular para localizar la línea |
| `line` | Contenido que debe tener esa línea |
| `state` | `present` (asegurar que existe) / `absent` (eliminar) |

---

## Gestión de bases de datos

Los módulos para MariaDB/MySQL pertenecen a la colección **`community.mysql`**, que hay que instalar antes de usarlos:

```bash
ansible-galaxy collection install community.mysql
```

<div class="card card-blue" style="margin-top:0.8rem">

### community.mysql.mysql_db

Crea, elimina o importa bases de datos.

```yaml
- name: Crear base de datos
  community.mysql.mysql_db:
    name: "{{ db_name }}"
    state: present
    login_unix_socket: /var/run/mysqld/mysqld.sock
```

</div>

---

## community.mysql.mysql_user

<div class="card card-green" style="margin-top:0.8rem">

Gestiona usuarios y sus privilegios.

```yaml
- name: Crear usuario de la BD
  community.mysql.mysql_user:
    name: "{{ db_user }}"
    password: "{{ db_password }}"
    priv: "{{ db_name }}.*:ALL"
    host: "%"
    state: present
    login_unix_socket: /var/run/mysqld/mysqld.sock
```

</div>

---

## Ansible Galaxy

> **Ansible Galaxy** es el repositorio oficial de roles y colecciones creados y compartidos por la comunidad.

```bash
# Buscar un rol
ansible-galaxy search apache

# Instalar un rol
ansible-galaxy install geerlingguy.apache

# Instalar una colección
ansible-galaxy collection install community.mysql

# Ver roles instalados
ansible-galaxy list
```

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Software de Gestión de la Configuración · Ansible

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 PI</span>
</div>
