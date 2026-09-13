---
marp: true
title: IA y educación — Qué debe saber un alumno de ASIR de IA
theme: profesional
paginate: true
header: 'IA y educación'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# **IA y educación**

## Qué debe saber un alumno de ASIR de IA

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 Formación Profesional · ASIR</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# ¿Qué es la Inteligencia Artificial?

## De las reglas escritas a mano al aprendizaje por ejemplos

---

## ¿Qué es la IA?

Conjunto de técnicas informáticas que permiten a una máquina realizar tareas que, hechas por una persona, diríamos que requieren inteligencia: reconocer una cara, traducir un texto, mantener una conversación, conducir un coche.

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Definición clásica de John McCarthy (1956): <strong>«la ciencia e ingeniería de hacer máquinas inteligentes»</strong>.</div>
</div>

La IA **no** es lo mismo que la automatización (un script de Bash automatiza, pero no aprende), ni es magia: es **estadística aplicada a gran escala** sobre hardware muy potente.

---

## La idea clave: reglas frente a ejemplos

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-red">

### Programación clásica

El programador escribe las reglas paso a paso:

`si pasa X, haz Y`

Funciona bien en dominios cerrados y predecibles.

</div>

<div class="card card-blue">

### IA moderna

No se escriben reglas: se le muestran al sistema **muchísimos ejemplos** y aprende a descubrir los patrones por sí mismo.

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>💡</span><div>Es como enseñar a un niño a reconocer un gato: no le damos una lista de reglas (orejas puntiagudas, bigotes, cola), le enseñamos muchos gatos y aprende solo.</div>
</div>

---

## IA débil frente a IA general

| | IA débil (estrecha) | IA general (AGI) |
|:--|:--|:--|
| Qué es | Resuelve **una** tarea concreta | Capacidad cognitiva equivalente a la humana en **cualquier** dominio |
| Ejemplos | Jugar al ajedrez, traducir, clasificar spam, ChatGPT, Claude | No existe todavía |
| Estado | Es **todo** lo que existe hoy en producción | Objetivo de investigación, no un producto |

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Aunque un chatbot "parezca" hablar con inteligencia general, sigue siendo IA débil: no le atribuyas propiedades que no tiene.</div>
</div>

---

## Dos enfoques históricos

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-yellow">

### IA simbólica (años 50–80)

Codifica el conocimiento como **reglas explícitas**: *"si el paciente tiene fiebre y dolor de garganta, sospechar amigdalitis"*.

Son los **sistemas expertos**. Funcionan bien en dominios cerrados (el motor de reglas de un firewall), pero no escalan ante la incertidumbre.

</div>

<div class="card card-blue">

### IA conexionista (desde 2010)

Abandona las reglas explícitas y **aprende de ejemplos** con redes inspiradas vagamente en el cerebro (neuronas conectadas).

Es donde está hoy **prácticamente toda la IA** que usamos: el <strong>machine learning</strong>.

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Cómo aprende una máquina

## Machine learning y deep learning

---

## Aprendizaje automático (Machine Learning)

El sistema mejora su rendimiento a partir de datos, sin ser programado explícitamente para la tarea. Tres modalidades:

<div class="cols-3" style="margin-top:0.8rem">

<div class="card card-blue">

### Supervisado

Ejemplos ya **etiquetados** ("este correo es spam"). Aprende a clasificar casos nuevos, como con un profesor que corrige cada respuesta.

</div>

<div class="card card-green">

### No supervisado

Datos **sin etiquetar**. El sistema descubre patrones o anomalías por sí mismo. Útil cuando no sabemos qué buscamos (ej.: tráfico de red anómalo).

</div>

<div class="card card-purple">

### Por refuerzo

Aprende por **ensayo y error**, con recompensas al acertar. Así aprenden a jugar al ajedrez o a caminar los robots.

</div>

</div>

---

## Aprendizaje profundo (Deep Learning)

Subcampo del ML basado en **redes neuronales artificiales** con muchas capas ("profundo"). Cada capa transforma la información un poco más hasta llegar al resultado final.

```
Píxeles → bordes → formas → partes (ojos, hocico) → concepto (gato)
```

- Los **parámetros** (o pesos) son los números que la red ajusta durante el entrenamiento
- Un modelo moderno puede tener desde millones hasta cientos de miles de millones de parámetros
- Es la técnica detrás de casi todos los avances recientes: visión artificial, voz, chatbots

<div class="alerta alerta-info" style="margin-top:0.5rem">
<span>ℹ️</span><div>Su salto en eficacia se explica por tres factores que coincidieron hacia 2010: muchísimos datos (internet), potencia de cálculo barata (GPUs) y mejores algoritmos.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">03</p>

# IA generativa y modelos de lenguaje

## El salto reciente que lo ha cambiado todo

---

## De clasificar a crear

Hasta hace poco, la IA principalmente **clasificaba o predecía**: ¿es spam?, ¿cuánto vale esta casa? La IA **generativa** crea contenido nuevo: texto, imagen, código, audio, vídeo.

<div class="alerta alerta-ok" style="margin-top:0.6rem">
<span>✅</span><div>No "copia" cosas que ya ha visto: ha aprendido patrones estadísticos a partir de miles de millones de ejemplos y sintetiza algo nuevo a partir de ellos — igual que una persona que aprende a escribir leyendo mucho, sin copiar.</div>
</div>

El modelo genera la salida **palabra a palabra** (o píxel a píxel), calculando en cada paso qué es lo estadísticamente más plausible según lo aprendido. No "consulta una base de datos" ni "busca en internet" por defecto.

---

## Tres familias de modelos generativos

<div class="cols-3" style="margin-top:0.8rem">

<div class="card card-purple">

### Modelos de difusión

Imagen y vídeo (Stable Diffusion, DALL·E, Midjourney). Parten de **ruido aleatorio** y lo van limpiando paso a paso hasta que emerge la imagen pedida.

</div>

<div class="card card-blue">

### LLM

Texto y código (ChatGPT, Claude, Gemini, Llama, Mistral). El código también es texto con reglas estrictas.

</div>

<div class="card card-green">

### Multimodales

Aceptan o generan varios tipos de contenido a la vez: le subes una foto y la explica, le hablas y responde con voz.

</div>

</div>

---

## Conceptos clave de los LLM (I)

**Token** — el modelo no procesa letras ni palabras enteras, sino "trozos" llamados tokens (≈ media palabra en español). Importa porque las APIs cobran por tokens, no por palabras.

```
"ordenador"  →  1 token
"ordenador"  →  "orden" + "ador"  (2 tokens, según el modelo)
```

**Embedding** — forma de convertir un texto en un vector de números que captura su **significado**. Las palabras parecidas quedan cerca en ese espacio numérico:

<div class="alerta alerta-info" style="margin-top:0.4rem">
<span>ℹ️</span><div>"perro" y "gato" tienen embeddings parecidos; "perro" y "termodinámica" no. Es la base de <strong>RAG</strong>.</div>
</div>

---

## Conceptos clave de los LLM (II)

**Transformer** — arquitectura de red neuronal que hizo posible el salto cualitativo (*"Attention is all you need"*, Google, 2017). Su idea clave es la **atención**: pondera qué partes del texto previo son relevantes para predecir lo siguiente.

**Ventana de contexto** — cantidad máxima de texto (en tokens) que el modelo puede "tener en mente" a la vez. Si se supera, empieza a "olvidar" lo más antiguo.

**Parámetros** — los pesos ajustados en el entrenamiento. Cifras como "Llama 70B" = 70 mil millones de parámetros. Da idea del tamaño, no equivale exactamente a calidad.

---

## Entrenamiento frente a inferencia

| | Entrenamiento | Inferencia |
|:--|:--|:--|
| Qué es | Ajustar los parámetros a partir de enormes corpus de texto | Usar el modelo ya entrenado para responder |
| Coste | Lento y **carísimo** | Rápido |
| Frecuencia | Se hace **una vez** | Cada vez que un usuario interactúa |

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Cuando "hablas con ChatGPT" estás haciendo <strong>inferencia</strong>, no entrenamiento: lo que escribas no modifica el modelo.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# Trabajar con IA

## Vocabulario imprescindible en el día a día

---

## Prompt y prompt engineering

**Prompt** — la instrucción que le damos al modelo. **Prompt engineering** es el oficio (más artesanal que científico) de formularla bien: dar contexto, poner ejemplos, descomponer la tarea en pasos, especificar el formato de salida.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-red">

### Prompt pobre

> "Cómo configuro KVM"

Respuesta genérica, poco útil para tu caso concreto.

</div>

<div class="card card-green">

### Prompt preciso

> "Crear una VM con KVM en Ubuntu 22.04, 4GB RAM, 2 vCPUs, disco virtio, conectada al bridge br0, con virt-install de forma desatendida"

</div>

</div>

---

## Fine-tuning y RAG

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-yellow">

### Fine-tuning

Tomar un modelo ya entrenado y **reentrenarlo** con datos específicos para especializarlo (ej.: Llama reentrenado con manuales internos de una empresa).

Menos común hoy: RAG suele dar mejores resultados con menos coste.

</div>

<div class="card card-blue">

### RAG (Retrieval-Augmented Generation)

En lugar de reentrenar, se le da al modelo acceso a una **fuente externa** (base documental, wiki interna) en el momento de la consulta.

Es el patrón **dominante hoy** en entornos profesionales.

</div>

</div>

---

## Agentes de IA

Un **agente** es un sistema que usa un LLM como "cerebro" para tomar decisiones y ejecutar **acciones encadenadas**: consultar una API, leer y escribir ficheros, navegar por la web, lanzar comandos en una terminal.

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>🤖</span><div>Es lo que distingue a un <strong>chatbot</strong> (solo responde) de algo como <strong>Claude Code</strong> (escribe código y modifica ficheros directamente) o un asistente que reserva vuelos.</div>
</div>

---

## Modelos abiertos frente a cerrados

| | Modelos abiertos | Modelos cerrados |
|:--|:--|:--|
| Ejemplos | Llama, Mistral, DeepSeek, Qwen | GPT, Claude, Gemini |
| Uso | Se descargan y ejecutan en **infraestructura propia** | Solo vía **API** del fabricante |
| Importa por | Privacidad, control, coste, soberanía del dato | Rendimiento de vanguardia, sin mantenimiento propio |

<div class="alerta alerta-ok" style="margin-top:0.6rem">
<span>✅</span><div>Para un perfil <strong>sysadmin</strong> es la frontera interesante: un alumno de ASIR puede levantar un Llama en un servidor con GPU y experimentar en serio.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">05</p>

# Límites y marco legal

## Lo que hay que tener interiorizado antes de confiar en la IA

---

## Limitaciones que debes conocer

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-red">

### Alucinaciones

Puede generar información falsa con total aplomo. No es un fallo puntual: genera lo **plausible**, no lo verdadero.

### Sesgo

Refleja los sesgos presentes en sus datos de entrenamiento.

</div>

<div class="card card-yellow">

### Corte de conocimiento

Solo "sabe" hasta la fecha en que se entrenó, salvo que tenga búsqueda web conectada.

### No determinismo

La misma pregunta puede dar respuestas distintas cada vez.

</div>

</div>

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Entrenamiento e inferencia consumen mucha <strong>energía</strong>: un dato relevante para decidir entre modelo local pequeño o API en la nube.</div>
</div>

---

## Marco ético y legal

- **Reglamento Europeo de IA (AI Act)** — aprobado en 2024, aplicación escalonada hasta 2027. Clasifica los sistemas por **nivel de riesgo** (inaceptable, alto, limitado, mínimo). Los sistemas de IA en selección de personal o evaluación educativa se consideran de **alto riesgo**
- **RGPD** — cualquier sistema de IA que trate datos personales sigue plenamente sujeto a él
- **Propiedad intelectual** — zona gris en formación: ¿se puede entrenar con contenido protegido?, ¿quién es autor de lo generado? Hay litigios en curso

---

<!-- _class: destacado -->

# Hasta aquí, el **qué**

## Ahora vamos con el **cómo**: usar la IA para aprender de verdad

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">06</p>

# Usar la IA para aprender

## Un cambio de mentalidad, no solo una herramienta nueva

---

## El cambio de mentalidad previo

El objetivo de un alumno de ASIR **ya no es** aprender a ejecutar tareas de memoria, sino **aprender a tomar decisiones técnicas fundamentadas**. Ese cambio de enfoque lo condiciona todo lo demás.

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>💡</span><div>Memorizar comandos, rutas de configuración o flags pierde sentido casi por completo. Lo que hay que desarrollar es entender <strong>qué</strong> hay que hacer y <strong>por qué</strong>, y usar la IA para ejecutarlo.</div>
</div>

---

## La IA como compañera de trabajo

No como tema opcional o curiosidad, sino como parte del flujo de trabajo habitual desde el primer día: generar scripts, analizar logs, documentar configuraciones.

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-green">

### Lo que la IA aporta

- Respuestas y borradores en segundos
- Explicaciones bajo demanda
- Automatización de lo repetitivo

</div>

<div class="card card-blue">

### Lo que sigue siendo tuyo

- **Validar** lo que produce
- **Corregir** lo que falla
- **Cuestionar** por qué lo hace así

</div>

</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">07</p>

# Metodología de trabajo con IA

## Las 6 etapas para cualquier tarea de administración de sistemas

---

## Etapa 1 — Comprensión del problema

Antes de preguntarle nada a la IA, tienes que poder responder tú solo:

- ¿Qué tengo que conseguir exactamente?
- ¿Qué restricciones tengo (recursos, sistema operativo, red, seguridad)?
- ¿Qué conceptos técnicos están implicados?

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Si no puedes responder esto, no estás listo para usar la IA de forma útil: pedirás cosas vagas y obtendrás respuestas genéricas. <strong>Esta etapa mide tu base técnica real.</strong></div>
</div>

---

## Etapa 2 — Formulación precisa de la consulta

Aquí se entrena una habilidad nueva: **saber hablar con la IA con precisión técnica**. El nivel de detalle de la pregunta determina la calidad de la respuesta.

> *"Necesito crear una VM con KVM en Ubuntu 22.04 con 4GB de RAM, 2 vCPUs y disco virtio, conectada a un bridge br0 ya existente. Quiero hacerlo desde línea de comandos con virt-install de forma desatendida."*

Esto se entrena y se evalúa, no se da por supuesto.

---

## Etapa 3 — Análisis crítico de la respuesta

La IA responde. **No ejecutas nada todavía.** Primero te preguntas:

- ¿Entiendo lo que me ha generado?
- ¿Hay algún parámetro que no reconozco?
- ¿Tiene sentido para mi entorno concreto o es una respuesta genérica?
- ¿Podría haber algún problema de seguridad o compatibilidad?

<div class="alerta alerta-ok" style="margin-top:0.6rem">
<span>✅</span><div>Si hay algo que no entiendes, pregúntaselo a la propia IA antes de continuar. Esta etapa es la que más diferencia a quien <strong>aprende</strong> de quien <strong>copia</strong>.</div>
</div>

---

## Etapa 4 — Experimentación controlada

Ahora sí ejecutas, pero de forma **progresiva** y en un entorno de pruebas. No aplicas todo de golpe: vas paso a paso, comprobando el resultado esperado tras cada acción.

Si algo falla, **el error es el material de aprendizaje**, y la IA se convierte en herramienta de diagnóstico:

> *"Esperaba este resultado y he obtenido este otro. Aquí está el error. ¿Qué puede estar pasando?"*

---

## Etapa 5 — Consolidación y automatización

Una vez que la tarea funciona, das un paso más. Le pides a la IA que te ayude a:

- **Automatizar** lo hecho manualmente (script, playbook de Ansible…)
- Hacerlo **reproducible** y documentado
- Pensar **qué pasaría si algo falla** y cómo prevenirlo

<div class="alerta alerta-info" style="margin-top:0.5rem">
<span>ℹ️</span><div>Esta etapa transforma una tarea puntual en una solución profesional, y obliga a entender el conjunto, no solo los pasos sueltos.</div>
</div>

---

## Etapa 6 — Reflexión y documentación propia

La etapa que más se suele saltar y **más valor tiene**. Documenta con tus propias palabras:

- Qué problema tenías y qué solución encontraste
- Qué le preguntaste a la IA y por qué
- Qué tuviste que corregir o adaptar de lo que te dio
- Qué aprendiste que no sabías antes

<div class="alerta alerta-ok" style="margin-top:0.6rem">
<span>✅</span><div>Consolida el aprendizaje real y genera un <strong>portfolio técnico</strong> que podrás llevar a una entrevista de trabajo.</div>
</div>

---

## Las 6 etapas, de un vistazo

| Etapa | Lo que haces | Lo que demuestras |
|:--|:--|:--|
| 1. Comprensión | Analizas el problema solo | Base técnica |
| 2. Formulación | Construyes una consulta precisa | Claridad conceptual |
| 3. Análisis crítico | Entiendes y cuestionas la respuesta | Criterio técnico |
| 4. Experimentación | Ejecutas y diagnosticas errores | Capacidad práctica |
| 5. Automatización | Generalizas y haces reproducible | Visión profesional |
| 6. Reflexión | Documentas con tus palabras | Aprendizaje real |

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">08</p>

# Buenas prácticas

## Qué hacer y qué evitar al usar IA en tus prácticas

---

## Qué sí, qué no

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-green">

### Sí

- Entender antes de ejecutar
- Adaptar el resultado a tu entorno concreto
- Preguntar "por qué" cuando algo no está claro
- Documentar tus consultas y tus decisiones

</div>

<div class="card card-red">

### No

- Copiar y pegar sin leer
- Ejecutar comandos que no entiendes
- Dar por buena la primera respuesta sin verificarla
- Presentar el resultado de la IA como si fuera tuyo sin más

</div>

</div>

---

## Por qué importa en tu evaluación

El profesor no evalúa solo si llegaste al resultado correcto, sino **cómo** llegaste, qué entendiste por el camino y qué serías capaz de hacer ante un problema distinto.

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>💡</span><div>Un escenario que funciona pero que no sabes explicar vale menos que uno con algún problema que sabes diagnosticar y razonar correctamente.</div>
</div>

Eso es exactamente lo que el mercado laboral te va a exigir: **saber qué pedirle a la IA, entender lo que devuelve, y tomar buenas decisiones cuando la IA no llega.**

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## IA y educación — Qué debe saber un alumno de ASIR de IA

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
  <span>📚 ASIR</span>
</div>
