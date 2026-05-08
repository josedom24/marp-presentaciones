---
marp: true
title: Introducción a Kubernetes
theme: profesional
paginate: true
header: 'IV · Introducción a Kubernetes'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Introducción a **Kubernetes**

## Orquestación de contenedores en producción

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 IV · Implantación de Aplicaciones Web</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# Orquestadores de contenedores

## Limitaciones de Docker y soluciones de orquestación

---

## Limitaciones de Docker Engine

Cuando desplegamos aplicaciones en producción con Docker Engine surgen preguntas sin respuesta:

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-red">

### Disponibilidad y escalado

- ¿Cómo se **balancea la carga** entre múltiples contenedores iguales?
- ¿Se puede **variar a demanda** el número de réplicas de un contenedor?
- ¿Es posible **mover la carga** entre diferentes nodos?
- ¿Se puede actualizar una aplicación **sin interrupción**?

</div>

<div class="card card-yellow">

### Gestión y operación

- ¿Cómo se gestionan los **cambios entre versiones**?
- ¿Cómo se realizan los **cambios en producción**?
- ¿Cómo se conectan contenedores que se ejecutan en **diferentes demonios** de Docker?

</div>

</div>

---

## Orquestadores de contenedores

> Surge la necesidad de un software que gestione de forma coordinada **múltiples nodos** con contenedores y proporcione la funcionalidad necesaria para **poner aplicaciones en producción**.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Principales orquestadores

- **Docker Swarm** — solución nativa de Docker
- **Apache Mesos** — gestión de recursos en clúster
- **HashiCorp Nomad** — orquestador multi-workload
- **Kubernetes** — estándar de facto actual

</div>

<div class="card card-green">

### ¿Qué aportan?

- Despliegue y actualización **automatizados**
- **Alta disponibilidad** y tolerancia a fallos
- **Escalado dinámico** de réplicas
- **Balanceo de carga** integrado
- Comunicación entre nodos del clúster

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# El proyecto Kubernetes

## Historia, características y ecosistema

---

## Origen del proyecto

- Google inicia el proyecto **Kubernetes en 2014** como software libre para orquestar contenedores
- No parte de cero: surge de una herramienta interna de Google llamada **Borg**, por lo que la primera versión ya era muy funcional
- Google cede el control del proyecto a la **Cloud Native Computing Foundation (CNCF)**

<div class="alerta alerta-info" style="margin-top:0.8rem">
<span>ℹ️</span><div><strong>Kubernetes</strong> proviene del griego antiguo y significa <em>timonel</em>. Se abrevia habitualmente como <strong>k8s</strong> (8 letras entre la k y la s).</div>
</div>

---

## ¿Qué es Kubernetes?

> **k8s** es un software pensado para gestionar completamente el **despliegue de aplicaciones sobre contenedores**.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Capacidades principales

- Despliega aplicaciones **rápidamente**
- **Escala** las aplicaciones al vuelo
- Integra **cambios sin interrupciones**
- Permite **limitar los recursos** a utilizar

</div>

<div class="card card-green">

### Datos técnicos

- Desarrollado en **Go**
- Licencia **Apache 2.0** (software libre permisivo)
- Código en **GitHub** (`kubernetes/kubernetes`)
- Enorme **ecosistema** de herramientas complementarias: `landscape.cncf.io`

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# Arquitectura de Kubernetes

## Clúster, nodos master, workers y addons

---

## El clúster de Kubernetes

- k8s se instala en varios nodos que se gestionan de forma **coordinada** formando un **clúster**
- Los nodos pueden ser máquinas físicas, virtuales o habitualmente **instancias IaaS** (AWS, GCP, OpenStack…)

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Nodos master (*control plane*)

- Ejecutan los **servicios principales** de k8s
- **Ordenan** a los nodos worker qué contenedores ejecutar
- Son el cerebro del clúster

</div>

<div class="card card-green">

### Nodos worker

- **Reciben las órdenes** de los controladores
- En ellos se ejecutan los **contenedores de las aplicaciones**
- Son la fuerza de trabajo del clúster

</div>

</div>

---

## Componentes del nodo master

| Componente | Función |
|:--|:--|
| **kube-apiserver** | Gestiona la API de k8s — punto de entrada a todo el clúster |
| **etcd** | Almacén clave-valor con la **configuración del clúster** |
| **kube-scheduler** | Selecciona el nodo donde **ejecutar los contenedores** |
| **kube-controller-manager** | Ejecuta los **controladores** de k8s |
| **cloud-controller-manager** | Controladores que interactúan con el **proveedor de nube**: nodos, enrutamiento, balanceadores, volúmenes |
| **container runtime** | Ejecuta los contenedores necesarios (`containerd`, `docker`…) |

---

## Componentes de un nodo worker

| Componente | Función |
|:--|:--|
| **kubelet** | Controla los **Pods asignados** a su nodo |
| **kube-proxy** | Permite la **conexión a través de la red** |
| **container runtime** | Ejecuta los contenedores (`containerd`, `docker`…) |
| **supervisord** | Monitoriza y controla kubelet y el runtime |

---

## Complementos (*addons*)

Los componentes anteriores forman la estructura básica de k8s. Es habitual añadir funcionalidad adicional:

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Cluster DNS

Proporciona registros DNS para los servicios de k8s. Normalmente mediante **CoreDNS**.

### Web UI

Interfaz web para el manejo y monitorización del clúster.

</div>

<div class="card card-green">

### Container Resource Monitoring

Recoge métricas de forma centralizada: **Prometheus**, Sysdig…

### Cluster-level Logging

Almacena y gestiona los **logs de los contenedores** de forma centralizada.

</div>

</div>

---

## Alternativas para instalación de k8s

| Herramienta | Descripción | Uso recomendado |
|:--|:--|:--|
| **Minikube** | Clúster de un solo nodo en local. Proyecto oficial de k8s | **Aprendizaje** — la opción más sencilla |
| **kubeadm** | Clúster multi-nodo más realista. Instalación menos automatizada | **Laboratorio** con varios nodos |
| **kind** | Clúster multi-nodo sobre Docker. Proyecto oficial más reciente | **Desarrollo** y CI/CD |
| **k3s** | Versión ligera para producción. Pensada para IoT, edge, ARM | **Producción** en recursos limitados |

<div class="alerta alerta-info" style="margin-top:0.5rem">
<span>ℹ️</span><div><strong>k3s</strong> lo desarrolló Rancher y hoy lo mantiene la CNCF. Es una opción real para entornos con recursos limitados o arquitecturas ARM.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# Despliegues con Kubernetes

## Recursos: Pods, ReplicaSets, Deployments y Services

---

## Despliegue tradicional

![w:900px](img/esquema-tradicional.png)

---

## Despliegue con Kubernetes

![w:900px](img/esquema-k8s.png)

---

## Recursos principales de k8s

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Pod

Unidad mínima de k8s. **Ejecuta uno o varios contenedores** que comparten red y almacenamiento.

### ReplicaSet

- Garantiza que no haya **caída del servicio**
- Gestiona la **tolerancia a fallos**
- Proporciona **escalabilidad dinámica**

</div>

<div class="card card-green">

### Deployment

- Gestiona las **actualizaciones continuas**
- Realiza **despliegues automáticos**
- Controla el historial de versiones y los *rollbacks*

### Service

- Gestiona el **acceso a los Pods**
- **Balancea la carga** entre los Pods disponibles
- Proporciona una IP y DNS estables, independientemente de los Pods

</div>

</div>

---

## Ingress

> Un **Ingress** gestiona el acceso desde el exterior al clúster a través de **nombre de dominio**, actuando como punto de entrada HTTP/HTTPS.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Sin Ingress

- Cada Service de tipo `LoadBalancer` necesita una IP externa propia
- Costoso en entornos cloud (una IP pública por servicio)

</div>

<div class="card card-green">

### Con Ingress

- Una sola IP de entrada para **múltiples servicios**
- Enrutamiento por **nombre de dominio** o ruta URL
- Gestión centralizada de **TLS/HTTPS**

</div>

</div>

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## Introducción a Kubernetes

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 IV</span>
</div>
