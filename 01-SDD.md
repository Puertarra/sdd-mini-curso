# Spec Driven Development (SDD)

## ¿Qué es Spec Driven Development?

**Spec Driven Development (SDD)** es un enfoque para el desarrollo de software en el que la **especificación se convierte en la principal fuente de verdad del proyecto**.

En lugar de pedir directamente a una IA que genere una aplicación a partir de un prompt, primero se define con precisión qué debe construirse, qué restricciones debe respetar y qué condiciones debe cumplir. A partir de esa información, los agentes de IA pueden generar el plan técnico, dividirlo en tareas e implementar el software.

La idea fundamental puede resumirse como:

**Especificar → Planificar → Dividir en tareas → Implementar**

El desarrollador o responsable del proyecto se concentra principalmente en definir y revisar el **qué**, mientras que el agente de IA puede encargarse de gran parte del **cómo**.

---

## ¿Dónde nace SDD?

El concepto de desarrollar software a partir de especificaciones no es nuevo. La ingeniería de software utiliza desde hace décadas requisitos, especificaciones, diseño y planificación antes de implementar un sistema.

El **Spec Driven Development moderno** surge con la aparición de agentes de IA capaces de trabajar directamente sobre repositorios de software. La especificación deja de ser solamente documentación para humanos y pasa a convertirse en una entrada estructurada que guía al agente durante la planificación y la implementación.

Herramientas como **GitHub Spec Kit**, **OpenSpec** y **Kiro** formalizan diferentes variantes de este enfoque y permiten utilizar agentes como **Claude Code** o **Codex** para ejecutar el desarrollo.

La diferencia importante respecto del desarrollo tradicional es la velocidad del ciclo. La IA puede convertir una especificación en un plan, tareas y código en minutos, haciendo posible iterar rápidamente sobre la especificación.

---

## SDD frente a Vibe Coding

El **Vibe Coding** consiste, de forma simplificada, en describir mediante prompts lo que queremos construir y permitir que la IA genere directamente el código.

Por ejemplo:

> "Crea un chatbot para consultar el catálogo de productos de mi empresa."

La IA debe completar por sí misma muchos detalles que no fueron especificados. Puede producir rápidamente una aplicación funcional, pero también tomar decisiones incorrectas o inconsistentes.

Este enfoque resulta especialmente útil para **prototipos, pruebas de concepto y experimentación rápida**, donde el objetivo principal es obtener algo funcional en poco tiempo.

El problema aparece cuando el sistema aumenta en complejidad o debe convertirse en software estable y mantenible. Los requisitos implícitos, las decisiones tomadas por la IA y los cambios realizados mediante sucesivos prompts pueden producir inconsistencias y dificultar la trazabilidad.

SDD intenta resolver este problema cambiando el flujo:

**Vibe Coding:**

`Idea → Prompt → Código`

**Spec Driven Development:**

`Idea → Constitución → Especificación → Clarify → Plan → Tareas → Analyze → Código`

La diferencia principal es que en SDD **la IA no debería tener que adivinar requisitos importantes**. Estos se establecen antes de comenzar la implementación.

---

---

# El caso del curso

Todo el curso trabaja sobre **un único caso**, que iremos desarrollando etapa por
etapa hasta llegar al código.

**ProductChat** es un chatbot interno para **Distribuidora Andes**, una empresa
que vende equipamiento informático. Su catálogo de productos y servicios vive en
una planilla Excel mantenida a mano por el área de ventas: `catalogo_productos.xlsx`.

Esa planilla es el problema. Tiene SKU duplicados con precios que no coinciden,
la misma categoría escrita de trece formas distintas, atributos sin unidad
estable (`32 GB`, `32768 MB`, `32`), precios que a veces son números y a veces
texto, y una columna de observaciones donde a veces vive el dato que importa.
Nadie la va a limpiar antes de construir el chatbot.

Ese desorden no es un detalle de contexto que podamos olvidar después: es lo que
hace interesante el caso. Una especificación que asuma un catálogo limpio
producirá un chatbot que responde con seguridad datos equivocados.

El punto de partida está en dos archivos de esta carpeta:

```text
catalogo_productos.xlsx          ← los datos, tal como están
spec_servilleta_productchat.md   ← la spec escrita a mano, antes de usar Spec Kit
```

Conviene leer ambos antes de continuar.

# Las cuatro etapas de SDD

Antes de entrar en cada una conviene fijar el vocabulario, porque el ciclo tiene tres tipos de paso distintos y es fácil confundirlos.

Hay **cuatro etapas**, y cada una produce un artefacto: la Constitución, la Especificación, el Plan y las Tareas. Son las que vamos a estudiar en detalle.

Hay además **dos puertas de calidad**, Clarify y Analyze, que no producen artefactos nuevos: revisan los existentes y proponen correcciones. La primera revisa la Especificación por dentro; la segunda revisa la coherencia entre todos los artefactos. Formalmente son opcionales, pero en este curso las usamos siempre, porque es donde se detectan los problemas mientras todavía son baratos de corregir.

Y hay una **ejecución**, la Implementación, que convierte todo lo anterior en código.

El flujo completo queda así:

`Constitución → Especificación → Clarify → Plan → Tareas → Analyze → Implementación`

## 1. Constitución

La **Constitución** establece los principios, restricciones y reglas no negociables que gobernarán todo el proyecto.

Son reglas globales que deben respetarse independientemente de la funcionalidad que se esté implementando.

Por ejemplo:

- El producto debe estar disponible en español.
- El desarrollo debe mantenerse lo más simple posible.
- No deben implementarse funcionalidades que no estén especificadas.
- Deben respetarse determinadas reglas de seguridad o arquitectura.

La Constitución normalmente se define una vez y gobierna las etapas posteriores.

**Pregunta que responde:**  
`¿Qué reglas debemos respetar siempre?`

---

## 2. Especificación

La **Especificación (Spec)** define **qué debe construirse y por qué**.

Describe los requisitos del sistema desde el punto de vista funcional, sin entrar innecesariamente en decisiones técnicas de implementación.

Puede incluir:

- Objetivo del producto.
- Usuarios.
- Funcionalidades.
- Requisitos.
- Historias de usuario.
- Restricciones funcionales.
- Criterios de aceptación.
- Condiciones que determinan cuándo una funcionalidad está terminada.

Una buena especificación debe permitir verificar objetivamente si el resultado cumple lo solicitado.

La especificación debe concentrarse principalmente en el **qué**, no en el **cómo**.

**Pregunta que responde:**  
`¿Qué queremos construir y qué debe cumplir?`

---

## 3. Plan

El **Plan** transforma la especificación en una solución técnica.

Aquí se determina **cómo se va a construir** el sistema.

Puede establecer:

- Arquitectura.
- Tecnologías.
- Base de datos.
- Backend.
- Frontend.
- APIs.
- Componentes.
- Estructura del proyecto.
- Dependencias.
- Decisiones técnicas.

En un flujo de desarrollo agéntico, gran parte de este plan puede ser generado por la IA a partir de la especificación y posteriormente revisado por una persona.

Esto permite separar claramente dos responsabilidades:

`Especificación = qué`

`Plan = cómo`

**Pregunta que responde:**  
`¿Cómo vamos a construirlo?`

---

## 4. Tareas

La etapa de **Tareas** convierte el plan técnico en unidades pequeñas de trabajo que puedan ejecutarse y verificarse individualmente.

Por ejemplo:

- Leer el catálogo desde la planilla y localizar la fila de cabecera.
- Normalizar las categorías equivalentes.
- Normalizar los atributos de memoria a una unidad única.
- Implementar la búsqueda de productos por características.
- Detectar los SKU duplicados con valores en conflicto.
- Crear las pruebas correspondientes.

Las tareas deben ser **concretas, pequeñas y verificables**, de manera que sea posible determinar claramente cuándo cada una está terminada.

El agente de IA puede ejecutar estas tareas progresivamente hasta completar la implementación.

**Pregunta que responde:**  
`¿Qué debemos hacer concretamente para implementar el plan?`

---

# Flujo completo

Las cuatro etapas pueden resumirse de la siguiente forma:

| Etapa | Pregunta principal | Resultado |
|---|---|---|
| **Constitución** | ¿Qué reglas debemos respetar siempre? | Principios y restricciones |
| **Especificación** | ¿Qué queremos construir? | Requisitos y criterios de aceptación |
| **Plan** | ¿Cómo vamos a construirlo? | Diseño y decisiones técnicas |
| **Tareas** | ¿Qué debemos hacer concretamente? | Lista de trabajo ejecutable |

A esas cuatro etapas se suman las dos puertas de calidad y la ejecución:

| Paso | Tipo | Función |
|---|---|---|
| **Clarify** | Puerta de calidad | Detecta ambigüedades dentro de la Especificación |
| **Analyze** | Puerta de calidad | Detecta incoherencias entre Constitución, Especificación, Plan y Tareas |
| **Implementación** | Ejecución | Convierte las Tareas en código verificado |

El flujo resultante es:

`Constitución → Especificación → Clarify → Plan → Tareas → Analyze → Implementación`

Una característica importante del SDD aplicado a agentes de IA es que este proceso no tiene por qué ser estrictamente lineal. Si durante la implementación se descubre que el resultado no corresponde con lo esperado, se puede volver a la especificación, corregirla y generar nuevamente las etapas posteriores.

Por tanto, la **especificación funciona como la referencia central del desarrollo**, mientras que el plan, las tareas y el código evolucionan a partir de ella.

---

# Vocabulario del curso

Un término por concepto, con el mismo significado en todos los módulos.

| Término | Significado |
|---|---|
| **Etapa** | Paso del ciclo que produce un artefacto. Hay cuatro: Constitución, Especificación, Plan y Tareas. |
| **Puerta de calidad** | Paso que revisa artefactos existentes sin producir uno nuevo. Hay dos: Clarify y Analyze. |
| **Constitución** | Reglas permanentes del proyecto. Archivo: `constitution.md`. Comando: `/speckit-constitution`. |
| **Especificación** | Qué debe hacer el producto. Archivo: `spec.md`. Comando: `/speckit-specify`. |
| **Clarify** | Puerta que detecta ambigüedades dentro de la Especificación. Comando: `/speckit-clarify`. |
| **Plan** | Cómo se construirá. Archivo: `plan.md`. Comando: `/speckit-plan`. |
| **Tareas** | Unidades de trabajo verificables. Archivo: `tasks.md`. Comando: `/speckit-tasks`. |
| **Analyze** | Puerta que detecta incoherencias entre todos los artefactos. Comando: `/speckit-analyze`. |
| **Implementación** | Ejecución de las Tareas hasta obtener código verificado. Comando: `/speckit-implement`. |
| **Spec de servilleta** | La especificación escrita a mano por una persona, antes de usar Spec Kit. Es la entrada de `/speckit-specify`, no su salida. |
| **Historia de usuario (HU)** | Necesidad del usuario expresada como escenario. Se identifican `HU1`, `HU2`… |
| **Historia refinada** | Subdivisión verificable de una historia de usuario. Se identifican `HU1.1`, `HU1.2`… |
| **El catálogo** | La fuente de datos del caso: `catalogo_productos.xlsx`, hoja `Catálogo`. Único nombre para referirse a ella. |
| **Capa de método** | Spec Kit: organiza el proceso. |
| **Capa de agente** | Claude Code, Codex u otro: ejecuta el proceso. |

Dos precisiones que conviene no perder de vista. La **Especificación** es el artefacto formal que produce Spec Kit; la **spec de servilleta** es lo que le entregamos para que lo produzca. Y la **Implementación** no es una quinta etapa: las cuatro etapas ya terminaron cuando empieza.

---

# Cómo escribimos los prompts en este curso

Un prompt de SDD no es una petición conversacional: es la entrada de una etapa y la fuente de un artefacto que otras etapas van a consumir. Si no es reproducible, la trazabilidad se rompe en el primer eslabón.

Por eso, todos los prompts de este curso declaran cinco cosas:

| Bloque | Qué responde |
|---|---|
| **Rol** | Quién es el agente en este paso, y qué decisiones **no** le tocan |
| **Contexto** | Qué proyecto es y qué particularidad del caso condiciona la respuesta |
| **Entrada** | Qué artefactos consume, nombrados por su ruta |
| **Salida esperada** | Qué produce, con qué estructura y en qué archivo |
| **Criterio de aceptación** | Cómo sabemos, sin leer código, que la salida sirve |

Los prompts están en la carpeta `prompts/`, numerados según el orden en que se usan, listos para copiar. Los módulos los reproducen, pero la versión de referencia es siempre el archivo.

## El encadenamiento

Cada prompt consume lo que produjo el anterior. Esa es la propiedad que hace que SDD sea un método y no una sucesión de peticiones sueltas:

| # | Prompt | Entrada | Salida |
|---|---|---|---|
| 1 | `prompts/01-constitution.txt` | decisiones humanas | `.specify/memory/constitution.md` |
| 2 | `prompts/02-specify.txt` | `spec_servilleta_productchat.md`, `constitution.md`, `catalogo_productos.xlsx` | `spec.md` |
| 3 | `prompts/03-clarify.txt` | `spec.md`, `constitution.md` | `spec.md` actualizado |
| 4 | `prompts/04-plan.txt` | `spec.md`, `constitution.md`, `catalogo_productos.xlsx` | `plan.md`, `data-model.md`, `research.md` |
| 5 | `prompts/05-tasks.txt` | `plan.md`, `data-model.md`, `spec.md` | `tasks.md` |
| 6 | `prompts/06-analyze.txt` | todos los anteriores | informe de hallazgos |
| 7 | `prompts/07-implement.txt` | `tasks.md` y los demás artefactos | código verificado |

Si al llegar a un paso no existe el archivo que su prompt declara como entrada, es que algo se saltó. Esa comprobación, hecha antes de pegar el prompt, ahorra la mayoría de los problemas.
