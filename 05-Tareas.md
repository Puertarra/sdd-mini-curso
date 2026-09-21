# Las Tareas en SDD

## ¿Qué son las Tareas?

Las **Tareas (Tasks)** constituyen la cuarta etapa del ciclo de **Spec Driven Development (SDD)**. Una vez que el Plan ha definido cómo se construirá el sistema, esta etapa lo descompone en **unidades de trabajo pequeñas, concretas y verificables** que posteriormente podrá ejecutar el agente de IA.

Las Tareas responden principalmente a la pregunta: **¿qué acciones concretas debemos ejecutar para implementar el Plan?**

En esta etapa la intervención humana es reducida. La IA dispone de la Constitución, la Especificación y el Plan, por lo que puede generar automáticamente la secuencia de trabajo necesaria.

## ¿Cómo debe ser una buena tarea?

Cada tarea debe representar una unidad de trabajo suficientemente pequeña para poder implementarse y comprobarse de manera independiente. Debe indicar claramente qué debe hacerse y permitir determinar cuándo está terminada.

Por ejemplo, una tarea demasiado general sería:

```text
Implementar el chatbot.
```

Una descomposición adecuada sería:

```text
Crear el componente de acceso al catálogo de productos.

Implementar la lectura de productos desde la fuente de datos.

Implementar la búsqueda de productos por nombre.

Implementar la búsqueda por categoría.

Implementar el tratamiento de consultas sin resultados.

Crear las pruebas para verificar las consultas de productos.
```

El objetivo es eliminar tareas demasiado grandes o ambiguas que obliguen al agente a tomar múltiples decisiones durante su ejecución.

## Organización de las Tareas

Las tareas normalmente se organizan en fases según sus dependencias y el orden lógico de implementación. Una estructura posible es:

```text
Fase 1: Configuración inicial
Fase 2: Componentes fundamentales
Fase 3: Historia de usuario 1
Fase 4: Historia de usuario 2
Fase 5: Historia de usuario 3
Fase 6: Integración y ajustes finales
```

Esta organización permite construir primero los elementos necesarios para el funcionamiento general y posteriormente implementar las distintas historias de usuario definidas en la Especificación.

## Dependencias entre tareas

No todas las tareas pueden ejecutarse en cualquier orden. Algunas necesitan que otras hayan terminado previamente.

Por ejemplo:

```text
Crear el mecanismo de acceso a los datos
        ↓
Implementar búsqueda de productos
        ↓
Integrar búsqueda con el chatbot
        ↓
Validar las respuestas
```

La etapa de Tareas (Tasks) debe identificar estas dependencias para establecer un orden de ejecución coherente.

## Tareas paralelizables

Cuando dos tareas no dependen entre sí, pueden potencialmente ejecutarse en paralelo. GitHub Spec Kit puede identificar estas tareas, normalmente mediante una marca como **[P]**.

Por ejemplo:

```text
[P] Implementar búsqueda por categoría
[P] Implementar búsqueda por nombre
[P] Crear formato de presentación de productos
```

La identificación de tareas paralelizables es especialmente útil cuando se trabaja con **varios agentes o subagentes**, ya que permite distribuir trabajo independiente y reducir el tiempo de implementación.

No todas las tareas deben paralelizarse. Las dependencias definidas en el Plan deben respetarse.

## Verificación

Una característica fundamental es que las tareas deben tener un resultado verificable. No basta con indicar qué debe construirse, debe ser posible determinar objetivamente si la tarea se completó correctamente.

Por ejemplo:

```text
Tarea:
Implementar la búsqueda de productos por nombre.

Verificación:
Al consultar un producto existente, el sistema devuelve
la información correspondiente al producto almacenado
en el catálogo.
```

Esta verificabilidad permite relacionar la implementación con los requisitos y criterios de aceptación definidos previamente en la Especificación.

## Generación con GitHub Spec Kit

En **GitHub Spec Kit**, las tareas se generan con:

```text
/speckit-tasks

ROL
Actúa como responsable de planificación de trabajo.

CONTEXTO
ProductChat, chatbot sobre un catálogo Excel inconsistente.

ENTRADA
plan.md, data-model.md, spec.md y .specify/memory/constitution.md.

SALIDA ESPERADA
tasks.md con las tareas organizadas por fases, cada tarea suficientemente
pequeña para implementarse y comprobarse de forma independiente. Marca con [P]
las que pueden ejecutarse en paralelo. Etiqueta cada tarea de funcionalidad con
la historia refinada que implementa, en el formato [HU1.1].

REQUISITOS DE LA SALIDA
Toda historia refinada debe tener al menos una tarea de implementación y una
tarea de verificación. Las tareas de verificación deben poder comprobarse contra
catalogo_productos.xlsx, nombrando el SKU concreto cuando aplique. No introduzcas
ninguna tarea que implemente algo que no esté en spec.md.

CRITERIO DE ACEPTACIÓN
Cada requisito funcional y cada criterio de aceptación de spec.md tiene al menos
una tarea asociada, y ninguna tarea carece de requisito que la origine.
```

> Este prompt está en `prompts/05-tasks.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.


Normalmente no es necesario proporcionar un nuevo prompt detallado. El agente utiliza los artefactos creados durante las etapas anteriores:

```text
Constitución
      ↓
Especificación
      ↓
Plan
      ↓
Tareas
```

A partir de ellos genera normalmente un archivo:

```text
tasks.md
```

Este archivo contiene las tareas organizadas por fases, sus dependencias, posibles oportunidades de paralelización y los elementos necesarios para verificar su ejecución.

## Ejemplo para ProductChat

Para ProductChat usamos el prompt anterior sin cambios: toda la información específica del caso ya está en los artefactos que declara como entrada.

El resultado podría contener una estructura similar a:

```text
Fase 1: Setup
- Crear la estructura inicial del proyecto.
- Inicializar las dependencias necesarias.
- Crear la configuración del entorno.

Fase 2: Componentes fundamentales
- Implementar la lectura del Excel y la localización de la fila de cabecera.
- Implementar el descarte de filas vacías y el recorte de espacios.
- Crear el modelo interno de producto.
- Implementar la normalización de categorías, unidades, precio y stock.
- Implementar la detección de SKU duplicados.
- Crear los componentes comunes para procesar consultas.

Fase 3: Funcionalidades
- [HU1.1] Buscar productos por nombre.
- [HU1.2] Buscar productos por categoría.
- [HU1.3] Buscar productos por características.
- [HU1.4] Obtener múltiples productos que satisfacen una consulta.

- [HU2.1] Consultar los atributos disponibles de un producto.
- [HU2.2] Obtener las características disponibles de un producto.
- [HU2.3] Informar cuando un atributo solicitado no está disponible.

- [HU3.1] Detectar consultas sobre productos inexistentes.
- [HU3.2] Informar al usuario cuando no se encuentran productos.
- [HU3.3] Evitar respuestas con información que no existe en la fuente.

- [HU4.1] Aplicar las equivalencias de categoría definidas en RN2.
- [HU4.2] Convertir las unidades de memoria según RN3.
- [HU4.3] Descartar los atributos condicionales según RN4.
- [HU4.4] Informar los SKU duplicados con valores en conflicto según RN5.
- [HU4.5] Indicar el SKU de origen en cada respuesta.

Fase 4: Validación
- [HU1.1] Verificar búsquedas por nombre.
- [HU1.2] Verificar búsquedas por categoría.
- [HU1.3] Verificar búsquedas por características.
- [HU1.4] Verificar consultas con múltiples resultados.
- [HU2.1] Verificar que los atributos coinciden con la fuente de datos.
- [HU2.3] Verificar consultas sobre atributos no disponibles.
- [HU3.1] Verificar consultas sobre productos inexistentes.
- [HU3.3] Verificar que el chatbot no inventa productos ni atributos.
- [HU4.1] Verificar que Portatiles y Notebooks se tratan como una familia.
- [HU4.2] Verificar que 32768 MB se interpreta como 32 GB.
- [HU4.3] Verificar que NB-1003 no aparece entre los equipos de 32 GB.
- [HU4.5] Verificar que cada respuesta indica su SKU de origen.

Fase 5: Integración final
- Verificar la integración completa de las historias refinadas.
- Comprobar los criterios de aceptación de la especificación.
- Verificar los casos límite.
- Comprobar el cumplimiento de la Constitución.
```

Conceptualmente, esto permite mostrar el refinamiento como un refinamiento de Historias de Usuario a partir de la aplicación de INVEST. **INVEST** es una regla mnemotécnica para evaluar si una historia de usuario está bien formulada: debe ser *Independent*, *Negotiable*, *Valuable*, *Estimable*, *Small* y *Testable*. Una historia que no cumple *Small* o *Testable* es exactamente la que conviene refinar antes de convertirla en tareas:

```text
HU1: Buscar productos
 ├── HU1.1: Buscar por nombre
 ├── HU1.2: Buscar por categoría
 ├── HU1.3: Buscar por características
 └── HU1.4: Obtener múltiples resultados

HU2: Consultar los atributos de un producto
 ├── HU2.1: Consultar atributos
 ├── HU2.2: Obtener características
 └── HU2.3: Gestionar atributos no disponibles

HU3: Gestionar información inexistente
 ├── HU3.1: Detectar productos inexistentes
 ├── HU3.2: Informar ausencia de resultados
 └── HU3.3: Evitar información inventada

HU4: Resolver un catálogo inconsistente
 ├── HU4.1: Equivalencias de categoría
 ├── HU4.2: Conversión de unidades
 ├── HU4.3: Atributos condicionales
 ├── HU4.4: Conflictos entre SKU duplicados
 └── HU4.5: Trazabilidad al SKU de origen
```

Este `tasks.md` de ejemplo **no está bien**: tiene una duplicación y deja tres historias refinadas sin verificar. Lo dejamos así a propósito. En el módulo siguiente, `/speckit-analyze` encontrará exactamente esos problemas, que es justamente para lo que sirve.

Obviamente, la lista real será más detallada y dependerá del Plan generado previamente.

## Revisión de las Tareas

Aunque la generación puede realizarse automáticamente, conviene revisar `tasks.md` antes de comenzar la implementación. La revisión debe comprobar principalmente que las tareas sean suficientemente pequeñas, tengan un resultado verificable, respeten las dependencias y cubran las historias de usuario y criterios de aceptación de la Especificación.

También debe comprobarse que no hayan aparecido funcionalidades nuevas. Si una tarea implementa algo que no existe en la Especificación, debe revisarse antes de continuar.

## Del Plan a la implementación

Las Tareas constituyen el puente entre el diseño técnico y la ejecución. El Plan puede describir componentes relativamente grandes, mientras que `tasks.md` los transforma en acciones que el agente puede ejecutar progresivamente y marcar como completadas.

El ciclo queda entonces:

`Constitución → Especificación → Clarify → Plan → Tareas → Analyze → Implementación`

Donde:

`Constitución = reglas que gobiernan el proyecto`

`Especificación = qué debe hacer el sistema`

`Clarify = eliminar ambigüedades`

`Plan = cómo se construirá`

`Tasks = acciones pequeñas y verificables`

Una vez generado y revisado `tasks.md`, el proyecto queda preparado para que el agente comience la **implementación progresiva de las tareas**.