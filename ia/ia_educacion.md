---
marp: true
title: IA y educación — Qué debe saber un alumno sobre IA
theme: profesional
paginate: true
header: 'IA y educación'
footer: ''
---

<!-- _class: portada -->
<!-- _paginate: false -->
<!-- _header: '' -->

# **IA y educación**

## Qué debe saber un alumno sobre IA

<div style="margin-top:2rem; display:flex; flex-direction:column; gap:0.5rem; justify-content:center; font-size:0.85rem; color:white">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">01</p>

# ¿Qué es la Inteligencia Artificial?

## De las reglas escritas a mano al aprendizaje por ejemplos

---

## ¿Qué es la IA?

La **Inteligencia Artificial**, o simplemente **IA**, es el conjunto de técnicas informáticas que permiten a una máquina realizar tareas que, hechas por una persona, diríamos que requieren inteligencia: reconocer una cara, traducir un texto, mantener una conversación, conducir un coche.

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Definición clásica de John McCarthy (1956): <strong>«la ciencia e ingeniería de hacer máquinas inteligentes»</strong>.</div>
</div>

La IA **no** es lo mismo que la automatización (un script de Bash automatiza, pero no aprende), ni es magia: es **estadística aplicada a gran escala** sobre hardware muy potente.

---

## La IA que ya usas cada día

Aunque no lo parezca, llevas años conviviendo con IA:

<div class="cols-2" style="margin-top:0.7rem">

<div class="card card-blue">

- 📷 **Face ID** de tu móvil reconoce tu cara
- 🗣️ **Siri, Alexa o Google Assistant** entienden lo que dices
- 🎬 **Netflix o Spotify** te recomiendan qué ver o escuchar

</div>

<div class="card card-green">

- ✉️ El **filtro de spam** de tu correo decide qué es publicidad
- 🌐 **Google Translate** traduce un texto al instante
- 💬 **ChatGPT o Claude** responden a tus preguntas

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>💡</span><div>Todo esto son ejemplos de <strong>IA débil o estrecha</strong>: cada sistema resuelve muy bien <strong>una</strong> tarea concreta, y solo esa.</div>
</div>

---

## La idea clave: reglas frente a ejemplos

<div class="cols-2" style="margin-top:0.7rem">

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

## ¿Es realmente "inteligente" la IA actual?

Un chatbot escribe con fluidez, resuelve ejercicios, incluso programa. Es fácil pensar que "entiende" lo que dice. Pero conviene matizarlo:

- **No tiene conciencia** ni sabe que existe
- **No razona** como una persona: no "piensa" la respuesta, la calcula estadísticamente
- **No entiende el significado** como tú lo entiendes: predice qué palabra es más probable a continuación, a partir de patrones aprendidos en billones de textos

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>🎭</span><div>Es como un actor que interpreta a un médico de forma muy convincente en una serie: suena creíble, pero no ha estudiado medicina.</div>
</div>

Lo llamamos "inteligencia" porque el resultado se parece al de una persona inteligente, no porque la máquina piense como una.

---

## IA actual (estrecha) frente a IA general (AGI)

| | IA actual (estrecha) | IA general (AGI) |
|:--|:--|:--|
| Qué es | Resuelve **una** tarea concreta y ya está | Razonaría y aprendería en **cualquier** ámbito, como una persona |
| Ejemplos | Ajedrez, traducir, ChatGPT, Claude, Face ID | No existe todavía |
| Estado | Es **todo** lo que existe hoy en producción | Objetivo de investigación, no un producto |

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div><strong>AGI</strong> son las siglas de <em>Artificial General Intelligence</em> (Inteligencia Artificial General). Aunque algunas empresas dicen estar cerca, a día de hoy sigue siendo un objetivo, no una realidad.</div>
</div>

---

## Dos enfoques históricos

<div class="cols-2" style="margin-top:0.7rem">

<div class="card card-yellow">

### IA simbólica (años 50–80)

Codifica el conocimiento como **reglas explícitas** (*"si tiene fiebre y dolor de garganta, sospechar amigdalitis"*): los **sistemas expertos**. Funcionan en dominios cerrados, pero no escalan ante la incertidumbre.

*Ej.: **Akinator**, que adivina un personaje recalculando probabilidades sobre una base de datos — sin ninguna red neuronal.*

</div>

<div class="card card-blue">

### IA conexionista (desde 2010)

Abandona las reglas explícitas y **aprende de ejemplos** con redes inspiradas vagamente en el cerebro (neuronas conectadas).

Es donde está hoy **prácticamente toda la IA** que usamos: el **machine learning**.

</div>

</div>

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>👉</span><div>A partir de aquí, todo lo que veremos — machine learning, redes neuronales, LLM, ChatGPT, Claude — es <strong>IA conexionista</strong>: el paradigma que domina la IA actual.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">02</p>

# Cómo aprende una máquina

## Machine learning, redes neuronales y deep learning

---

## Aprendizaje automático (Machine Learning, ML)

El sistema mejora su rendimiento a partir de datos, sin ser programado explícitamente para la tarea. Tres modalidades:

<div class="cols-3" style="margin-top:0.7rem">

<div class="card card-blue">

### Supervisado

Ejemplos ya **etiquetados** ("este correo es spam"). Aprende a clasificar casos nuevos.

*Ej.: diagnosticar una radiografía a partir de miles ya etiquetadas por médicos.*

</div>

<div class="card card-green">

### No supervisado

Datos **sin etiquetar**. El sistema descubre patrones por sí mismo.

*Ej.: agrupar a los clientes de una tienda online según lo que compran.*

</div>

<div class="card card-purple">

### Por refuerzo

Aprende por **ensayo y error**, con recompensas al acertar.

*Ej.: AlphaGo aprendiendo a jugar al Go contra sí mismo.*

</div>

</div>

---

## ¿Qué es una red neuronal artificial?

Es la estructura matemática que hace posible que una máquina "aprenda". Está formada por **neuronas artificiales** conectadas entre sí y organizadas en capas, inspiradas (muy vagamente) en el cerebro.

Cada neurona:

1. Recibe varios números de entrada
2. Los combina aplicando un **peso** a cada uno (la importancia que le da a esa entrada)
3. Produce un número de salida que pasa a la siguiente capa

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Los <strong>pesos</strong> son justamente los <strong>parámetros</strong> del modelo de los que hablaremos más adelante: los números que se ajustan durante el entrenamiento.</div>
</div>

---

## De la entrada a la salida: capas y entrenamiento

```
píxeles  →  [capa: bordes]  →  [capa: formas]  →  [capa: partes]  →  "8"
```

- La **primera capa** recibe los datos en bruto (por ejemplo, los píxeles de una imagen)
- Las **capas intermedias** detectan patrones cada vez más complejos: bordes → formas → partes → concepto completo
- La **última capa** da el resultado: una predicción, una clasificación, la siguiente palabra…

<div class="alerta alerta-info" style="margin-top:0.5rem">
<span>ℹ️</span><div>Esto es lo que la red hace <strong>una vez ya entrenada</strong>. Pero, ¿cómo llega a saber qué pesos usar? Eso lo vemos en la siguiente diapositiva.</div>
</div>

---

## ¿Cómo aprende exactamente? El ciclo de entrenamiento

Para entrenar hacen falta dos cosas: **muchos ejemplos de entrada** y, para cada uno, **la respuesta correcta ya conocida** (por eso se llama aprendizaje *supervisado*).

1. Le mostramos un ejemplo: la imagen de un "8" escrito a mano
2. Con los pesos aún sin ajustar, la red hace una predicción — al principio, casi al azar (quizás dice "3")
3. Comparamos su predicción con la respuesta correcta ("8") y calculamos cuánto ha fallado: el **error**
4. Un **algoritmo** (no una persona) ajusta ligeramente cada peso, calculando matemáticamente en qué dirección habría reducido ese error
5. Se repite con **millones de ejemplos**, una y otra vez, hasta que el error es muy pequeño

<div class="alerta alerta-ok" style="margin-top:0.5rem">
<span>🎯</span><div>Es como encestar con los ojos vendados, pero quien corrige la puntería no eres tú: es un algoritmo automático (<em>descenso de gradiente</em>) que recalcula, cada intento, cómo mover cada peso. Nadie ajusta a mano miles de millones de números.</div>
</div>

---

## Aprendizaje profundo (Deep Learning)

Es como llamamos a una red neuronal cuando tiene **muchas capas** ("profundo" = muchas capas apiladas). Es el tipo de red que hay detrás de casi todos los avances recientes: reconocimiento facial, traducción automática, asistentes de voz y los chatbots como ChatGPT o Claude.

- Un modelo moderno puede tener desde millones hasta **cientos de miles de millones** de parámetros (pesos)
- Cuantas más capas y más parámetros, más patrones complejos puede aprender — pero también hace falta más dato y más potencia de cálculo

<div class="alerta alerta-info" style="margin-top:0.5rem">
<span>ℹ️</span><div>El salto en eficacia se explica por tres factores que coincidieron hacia 2010: muchísimos datos (internet), potencia de cálculo barata (<strong>GPU</strong>, <em>Graphics Processing Unit</em>, el chip gráfico usado normalmente en videojuegos) y mejores algoritmos.</div>
</div>

---

## ¿Por qué hacen falta tantas capas? El ejemplo de un chatbot

```
texto → [capa: caracteres] → [capa: palabras y gramática] → [capa: significado] → [capa: intención] → respuesta
```

- Una sola capa solo puede hacer una transformación simple: no puede "saltar" directamente de las letras al significado
- Cada capa construye sobre la abstracción de la anterior — igual que para reconocer un dígito hacían falta capas de bordes → formas → partes

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>💬</span><div>Ejemplo: en <em>"El banco estaba cerrado"</em>, una sola capa que mire palabra por palabra no sabe si es una entidad financiera o el asiento de un parque. Hacen falta capas que integren todo el contexto de la frase para desambiguarlo — por eso los LLM apilan tantas: <strong>Llama 3.1 405B tiene 126 capas</strong> (GPT-3, de 2020, tenía 96).</div>
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

<div class="cols-3" style="margin-top:0.7rem">

<div class="card card-purple">

### Modelos de difusión

Imagen y vídeo (Stable Diffusion, DALL·E, Midjourney). Parten de **ruido aleatorio** y lo van limpiando paso a paso hasta que emerge la imagen pedida.

</div>

<div class="card card-blue">

### LLM

*Large Language Model* — modelo de lenguaje de gran tamaño. Texto y código (ChatGPT, Claude, Gemini, Llama, Mistral).

</div>

<div class="card card-green">

### Multimodales

Aceptan o generan varios tipos de contenido a la vez: le subes una foto y la explica, le hablas y responde con voz.

</div>

</div>

---

## Sigamos un ejemplo: "¿Cuál es la capital de Francia?"

Para entender cómo un LLM llega de tu pregunta a la respuesta, vamos a seguirla paso a paso — cada concepto aparece justo en el momento en que entra en juego.

**Paso 1 — Tokenización**: el modelo no procesa letras ni palabras enteras, sino "trozos" llamados **tokens** (≈ media palabra en español):

```
"¿Cuál es la capital de Francia?"  →  ["¿Cuál", " es", " la", " capital", " de", " Francia", "?"]
```

<div class="alerta alerta-info" style="margin-top:0.5rem">
<span>ℹ️</span><div>Importa porque las APIs cobran por tokens, no por palabras: "ordenador" puede ser 1 token, o partirse en "orden" + "ador", según el modelo.</div>
</div>

---

## Paso 2 — Cada token se convierte en números

**Embedding**: cada token se transforma en un vector de números que captura su significado — el modelo no procesa palabras, solo números.

```
"capital"  →  [0.31, -0.55, 0.12, ..., 0.44]   (cientos de números)
```

<div class="alerta alerta-info" style="margin-top:0.5rem">
<span>ℹ️</span><div>Esos números se ajustaron durante el entrenamiento para que sean parecidos cuando el concepto también lo es: "perro" y "gato" acaban con embeddings cercanos; "termodinámica" queda lejos.</div>
</div>

---

## Paso 3 — El Transformer procesa toda la frase a la vez

El **Transformer** (la arquitectura de red neuronal detrás de los LLM actuales) recibe estos embeddings. Su pieza clave es la **atención**: no analiza palabra por palabra en orden, sino que examina toda la frase de golpe para entender el contexto — así resuelve ambigüedades como "el banco" (¿financiero o del parque?) que vimos antes.

<div class="alerta alerta-info" style="margin-top:0.5rem">
<span>ℹ️</span><div>Con la pregunta completa a la vista, el Transformer ya puede calcular qué palabra es más probable para empezar la respuesta.</div>
</div>

---

## Paso 4 — Predecir y repetir (generación "autoregresiva")

El modelo elige la palabra más probable, la añade a la frase, y **repite el proceso completo** mirando otra vez todo el texto (incluido lo que él mismo acaba de escribir):

```
"La"  →  "capital"  →  "de"  →  "Francia"  →  "es"  →  "París"  →  "."
```

<div class="alerta alerta-warning" style="margin-top:0.5rem">
<span>⚠️</span><div>Por eso a veces un chatbot empieza una respuesta y luego "se contradice": decide palabra a palabra, no piensa la frase completa de antemano. Y por eso existe la <strong>ventana de contexto</strong>: el límite de cuánto texto puede tener "en mente" a la vez — si se supera, empieza a "olvidar" lo más antiguo.</div>
</div>

---

## Resultado: "La capital de Francia es París."

Todo este proceso —tokenizar, convertir en embeddings, procesar con atención, predecir palabra a palabra— lo ejecutan los **parámetros** del modelo: los pesos ajustados durante el entrenamiento. Cifras como "Llama 70B" = 70 mil millones de parámetros; **GPT-3** (2020) tenía **175.000 millones**.

| | Entrenamiento | Inferencia |
|:--|:--|:--|
| Qué es | Ajustar los parámetros a partir de enormes corpus de texto | Usar el modelo ya entrenado para responder (lo que acabamos de ver) |
| Coste | Lento y **carísimo** | Rápido |
| Frecuencia | Se hace **una vez** | Cada vez que un usuario interactúa |

---

## ¿Recuerda la IA entre sesiones?

En una misma conversación, cada mensaje que escribes hace que se reenvíe **toda la conversación completa** como contexto — no es que el modelo recuerde, es que ese texto sigue estando físicamente ahí.

Al cerrar la conversación, ese contexto se pierde. Como la inferencia no modifica los pesos del modelo, **por defecto no hay memoria entre sesiones**: la próxima vez empieza completamente en blanco.

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>💡</span><div>¿Cómo "recuerdan" entonces herramientas como ChatGPT o Claude Code? Guardan tus notas o preferencias en un fichero aparte y, al empezar una sesión nueva, se las <strong>reinyectan como contexto</strong> antes de que escribas nada. No es memoria del modelo: es una chuleta que alguien le vuelve a leer.</div>
</div>

---

## ¿Por qué entiende aunque tengas faltas de ortografía?

Por tres cosas que ya hemos visto, trabajando juntas:

```
"ordenador"  →  "orden" + "ador"
"ordenaor"   →  "orden" + "aor"      (la mayoría del token se mantiene)
```

- **Tokenización**: un error rara vez rompe la palabra entera — el token parcial sigue dando señal
- **Embeddings**: el modelo se entrenó con billones de textos reales llenos de erratas — una palabra mal escrita que aparece en los mismos contextos que la correcta acaba con un embedding parecido
- **Atención**: examina toda la frase a la vez, y el resto de palabras bien escritas acotan el significado

<div class="alerta alerta-ok" style="margin-top:0.6rem">
<span>✅</span><div>No es que el modelo "corrija" el texto por dentro: tokens parecidos + embeddings parecidos + atención al contexto completo hacen que la falta de ortografía apenas mueva la aguja.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">04</p>

# Vocabulario que escuchamos cada día

## Términos que te vas a encontrar constantemente trabajando con IA

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

## Chatbots, asistentes y agentes de IA

- **Chatbot** — solo responde dentro de una conversación de texto: una pregunta, una respuesta
- **Asistente** — ejecuta **una acción concreta ya programada de antemano** por petición (poner una alarma, añadir un evento). Tiene un catálogo fijo de cosas que sabe hacer, ni una más
- **Agente** — usa un LLM como "cerebro" para **encadenar varias acciones, decidiendo sobre la marcha** según lo que va descubriendo — no sigue un guion prefabricado

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>🤖</span><div>La diferencia clave: pídele a Alexa "pon una alarma a las 7" y ejecuta esa única acción ya programada. Pídele a un agente como <strong>Claude Code</strong> "arregla este error" y él solo lee el código, ejecuta pruebas, interpreta el fallo y edita el fichero que corresponda — decidiendo cada paso según el resultado del anterior.</div>
</div>

---

## Herramientas y Skills

<div class="cols-2" style="margin-top:0.8rem">

<div class="card card-blue">

### Herramientas (*tool use*)

Capacidad de un agente de **usar programas externos**: una calculadora, un buscador web, un intérprete de código, una **API** (*Application Programming Interface*, la forma en que dos programas se comunican entre sí).

El modelo decide **cuándo** y **con qué datos** llamar a cada herramienta.

</div>

<div class="card card-green">

### Skills

Paquetes de **instrucciones y conocimiento especializado** que se le dan a un agente para una tarea concreta (por ejemplo, cómo corregir un examen con un formato determinado).

Son como "manuales" que el agente consulta cuando la tarea lo requiere.

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Es lo que hace que un agente pase de "saber hablar" a <strong>saber hacer</strong>: buscar en internet, ejecutar un script, consultar una base de datos.</div>
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

### RAG

*Retrieval-Augmented Generation* — generación aumentada por recuperación. En lugar de reentrenar, se le da al modelo acceso a una **fuente externa** (base documental, wiki interna) en el momento de la consulta.

Es el patrón **dominante hoy** en entornos profesionales.

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>🔎</span><div>Cuando un modelo <strong>busca en internet</strong> (por ejemplo, porque le preguntas algo posterior a su corte de conocimiento) está haciendo <strong>RAG en tiempo real</strong>: dispara una búsqueda real (código normal, sin IA), inserta los resultados en su contexto y genera la respuesta leyendo ese texto nuevo, citando la fuente.</div>
</div>

---

## Modelos abiertos frente a cerrados

<div class="cols-2" style="margin-top:0.7rem">

<div class="card card-blue">

### Abiertos

Llama, Mistral, DeepSeek, Qwen, Gemma. Se descargan y ejecutan en **infraestructura propia** — la frontera interesante para un sysadmin.

**Ollama** es la herramienta para esto: como `docker pull` + `docker run`, pero con modelos. `ollama pull llama3.2` y ya lo tienes corriendo en tu propio servidor.

</div>

<div class="card card-green">

### Cerrados

GPT, Claude, Gemini. Solo se usan vía **API** del fabricante: rendimiento de vanguardia y sin mantenimiento propio, pero sin control sobre dónde viven tus datos.

Ni siquiera el número de parámetros es público: a diferencia de un modelo abierto, aquí es secreto industrial.

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.5rem">
<span>ℹ️</span><div>El número en el nombre de un modelo (<code>llama3.2:3b</code>, <code>qwen3:30b</code>) son sus <strong>parámetros en miles de millones</strong> (<em>billion</em>): más B, más capacidad — pero también más memoria (RAM/VRAM) y GPU necesarias para ejecutarlo en local.</div>
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

Puede generar información falsa con total aplomo, como inventarse una opción de un comando que no existe. No es un fallo puntual: genera lo **plausible**, no lo verdadero.

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

## El coste ambiental de la IA

No hay un "coste fijo" por consulta: depende del modelo, el hardware y la refrigeración. Aun así, dan una idea del orden de magnitud:

<div class="cols-2" style="margin-top:0.5rem">

<div class="card card-blue">

- Una consulta puede consumir del orden de **10 veces más electricidad** que una búsqueda tradicional
- Generar **una imagen** puede gastar tanta energía como **cargar un móvil** entero
- Una conversación de 10-50 preguntas puede equivaler a **medio litro de agua potable**

</div>

<div class="card card-green">

- El **uso** (inferencia) puede suponer hasta el **90% de la huella ecológica total**, no solo el entrenamiento
- Las GPU de datacenter tienen una vida útil de solo **3-5 años** → mucho residuo electrónico

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.5rem">
<span>🌍</span><div>Cifras orientativas — Fuente: INTEF, <em>Guía sobre el uso de la IA en el ámbito educativo</em> (2026).</div>
</div>

---

## Marco ético y legal

- **Reglamento Europeo de IA**, conocido como **AI Act** (*Artificial Intelligence Act*) — aprobado en 2024, aplicación escalonada hasta 2027. Clasifica los sistemas por **nivel de riesgo**: inaceptable, alto, de transparencia y mínimo. Los sistemas de IA en selección de personal o evaluación educativa se consideran de **alto riesgo**
- **RGPD** (Reglamento General de Protección de Datos) — cualquier sistema de IA que trate datos personales sigue plenamente sujeto a él
- **LOPDGDD** (Ley Orgánica de Protección de Datos Personales y Garantía de los Derechos Digitales) — en España, el consentimiento propio para que tus datos sean tratados (también por una IA) solo es válido a partir de los **14 años**; por debajo, hace falta el de la familia
- **Propiedad intelectual** — zona gris en formación: ¿se puede entrenar con contenido protegido?, ¿quién es autor de lo generado? Hay litigios en curso

---

## Todo esto tiene un nombre: alfabetización en IA

Saber qué es la IA, interactuar con ella con criterio, crear y gestionar con su ayuda, y entender sus límites: eso es lo que el INTEF, la OCDE y la Comisión Europea llaman **alfabetización en IA**.

<div class="alerta alerta-ok" style="margin-top:0.6rem">
<span>🎓</span><div>No es un contenido más: es una <strong>competencia</strong> tan necesaria hoy como saber buscar información o usar una hoja de cálculo. Todo lo que hemos visto hasta aquí es, en el fondo, eso.</div>
</div>

---

<!-- _class: capitulo -->
<!-- _paginate: false -->

<p class="numero">06</p>

# Usar la IA para aprender

## Un cambio de mentalidad, no solo una herramienta nueva

---

## El cambio de mentalidad previo

El objetivo **ya no es** aprender a ejecutar tareas de memoria, sino **aprender a tomar decisiones técnicas fundamentadas**. Ese cambio de enfoque lo condiciona todo lo demás.

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

Otro ejemplo: en vez de *"configura DNS"*, pregunta *"configura un servidor DNS con BIND9 en Debian 13 que resuelva el dominio interno empresa.local y reenvíe el resto de consultas a 8.8.8.8"*. Esto se entrena y se evalúa, no se da por supuesto.

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

## El verdadero peligro: no darte cuenta cuando falla

No es solo que la IA se equivoque — es que, si sabes menos que ella, **no tienes forma de notarlo**.

**Ejemplo:** la IA te da una solución técnicamente impecable, pero incompatible con tu versión del sistema.

- **El alumno que sabe** lo detecta, lo corrige y sigue
- **El alumno que no sabe** la copia, la pega, y cuando falla piensa *"la IA se ha equivocado"* — sin darse cuenta de que **él no la ha verificado**

<div class="alerta alerta-warning" style="margin-top:0.6rem">
<span>⚠️</span><div>Por eso la Etapa 1 y esta etapa no son un trámite: son lo que te permite detectar el fallo. Sin base técnica, la IA no solo se puede equivocar — <strong>tú no lo sabrás</strong>.</div>
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
<span>✅</span><div>Consolida el aprendizaje real y genera un <strong>portfolio técnico</strong> que podrás llevar a una entrevista de trabajo. El propio INTEF propone en su guía un modelo oficial de <strong>"declaración de uso de IA"</strong> con esta misma idea.</div>
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

## Riesgos de depender demasiado de la IA

<div class="cols-3" style="margin-top:0.7rem">

<div class="card card-red">

### Externalización cognitiva

Delegar en la IA funciones clave como el juicio crítico, la revisión o la verificación, en lugar de ejercerlas tú.

</div>

<div class="card card-yellow">

### Ilusión de competencia

Creer que dominas un tema porque la IA te lo explicó bien, sin haberlo trabajado tú mismo.

</div>

<div class="card card-purple">

### Sedentarismo cognitivo

Perder práctica en memoria, comprensión o razonamiento por delegar en exceso, incluso en tareas que ya dominabas.

</div>

</div>

<div class="alerta alerta-info" style="margin-top:0.6rem">
<span>ℹ️</span><div>Términos del INTEF en su guía sobre IA en educación (2026): no son casos aislados, son los riesgos que el propio Ministerio pide vigilar en el aula.</div>
</div>

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

<!-- _class: destacado -->

# Si la IA puede hacer tu trabajo...

## ¿qué tienes que saber **tú**?

1. Saber **qué pedir**
2. Saber **cuándo está equivocada**
3. Saber **por qué** funciona la solución

<div class="alerta alerta-ok" style="margin-top:1rem">
<span>🎯</span><div><strong>Eso</strong> es lo que significa saber trabajar con IA.</div>
</div>

---

<!-- _class: cierre -->
<!-- _paginate: false -->

# ¡Gracias!

## IA y educación — Qué debe saber un alumno sobre IA

<div style="margin-top:2rem; display:flex; gap:2rem; justify-content:center; font-size:0.85rem; color:#64748b">
  <span>📧 José Domingo Muñoz</span>
  <span>🏫 IES Gonzalo Nazareno · Dos Hermanas</span>
</div>
