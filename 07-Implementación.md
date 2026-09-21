# La Implementación en SDD

## ¿Qué es la Implementación?

La **Implementación (Implement)** es el paso en el que los artefactos desarrollados durante Spec Driven Development se transforman finalmente en código ejecutable. No es una quinta etapa: las cuatro etapas producen artefactos y ya terminaron; esto es la ejecución de lo que esos artefactos definen.

A diferencia del desarrollo tradicional, no comenzamos a programar directamente desde una idea o un prompt. Antes de llegar aquí hemos construido progresivamente el contexto necesario:

```text
Constitución
     ↓
Especificación
     ↓
Clarify
     ↓
Plan
     ↓
Tareas
     ↓
Analyze
     ↓
Implementación
```

En esta etapa, la implementación puede delegarse en gran medida al **agente de IA**, porque las principales decisiones ya fueron tomadas y documentadas.

## Implementación con GitHub Spec Kit

En **GitHub Spec Kit**, iniciamos este paso con:

```text
/speckit-implement

ROL
Actúa como desarrollador. Implementa lo que está especificado, ni más ni menos.

CONTEXTO
ProductChat, con los artefactos ya revisados por Analyze y corregidos.

ENTRADA
tasks.md como guía principal, más spec.md, plan.md, data-model.md y
.specify/memory/constitution.md. El archivo de datos es catalogo_productos.xlsx.

SALIDA ESPERADA
El código de la aplicación, con las tareas de tasks.md marcadas a medida que se
completan. Para cada tarea: implementar, ejecutar su verificación y corregir
hasta que pase.

RESTRICCIONES
No modifiques catalogo_productos.xlsx bajo ninguna circunstancia. No implementes
nada que no tenga una tarea en tasks.md; si detectas que falta algo, detente y
dímelo en lugar de añadirlo. Si una tarea resulta ambigua, pregunta antes de
decidir.

CRITERIO DE ACEPTACIÓN
La aplicación arranca. Cada criterio de aceptación de spec.md puede comprobarse
manualmente. El archivo catalogo_productos.xlsx tiene la misma fecha de
modificación que antes de empezar.
```

> Este prompt está en `prompts/07-implement.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.


El agente utiliza los artefactos generados previamente, especialmente:

```text
spec.md
plan.md
tasks.md
data-model.md
contracts/
research.md
quickstart.md
```

No todos los proyectos tendrán necesariamente todos estos archivos.

El elemento central es `tasks.md`. El agente recorre las tareas definidas, respeta sus dependencias y comienza progresivamente la construcción de la aplicación.

## Ejecución de las tareas

Para ProductChat, podríamos tener tareas provenientes del refinamiento de nuestras historias de usuario:

```text
[HU1.1] Buscar productos por nombre.
[HU1.2] Buscar productos por categoría.
[HU1.3] Buscar productos por características.
[HU1.4] Obtener múltiples resultados.

[HU2.1] Consultar los atributos disponibles de un producto.
[HU2.3] Informar cuando un atributo solicitado no está disponible.

[HU3.1] Detectar consultas sobre productos inexistentes.
[HU3.2] Informar al usuario cuando no se encuentran productos.
[HU3.3] Evitar respuestas con información que no existe en la fuente.

[HU4.1] Aplicar las equivalencias de categoría definidas en RN2.
[HU4.2] Convertir las unidades de memoria según RN3.
[HU4.3] Descartar los atributos condicionales según RN4.
[HU4.4] Informar los SKU duplicados con valores en conflicto según RN5.
[HU4.5] Indicar el SKU de origen en cada respuesta.
```

Esta lista ya no es la que generó `/speckit-tasks`: incorpora las correcciones que Analyze recomendó en el módulo anterior. `HU2.2` desapareció, consolidada con `HU2.1`, y las validaciones que faltaban para `HU3.2` y `HU4.4` se añadieron a la fase correspondiente. Ese es el efecto de la puerta de calidad: se implementa la versión corregida, no la primera que salió.

Durante la implementación, el agente va completando estas unidades de trabajo y marcándolas en `tasks.md`.

La trazabilidad que construimos durante las etapas anteriores llega entonces hasta el código:

```text
Requisito
   ↓
Historia de Usuario
   ↓
Historia refinada
   ↓
Plan
   ↓
Task
   ↓
Código
   ↓
Verificación
```

## El bucle agéntico

Una característica importante de esta etapa es que el agente no debería limitarse a generar código una sola vez. Para cada tarea puede ejecutar un pequeño **bucle agéntico**:

```text
  Analizar la tarea
         ↓
     Planificar
         ↓
    Implementar
         ↓
   ┌→  Verificar
   │       ↓
   │   ¿Cumple?
   │    ↙      ↘
   │   No        Sí  →  Tarea completada
   │    ↓
   └─ Corregir
```

Por ejemplo, para:

```text
[HU1.2] Implementar búsqueda de productos por categoría.
```

el agente puede identificar los componentes afectados, implementar la funcionalidad, ejecutar las pruebas correspondientes y comprobar que el comportamiento satisface los requisitos definidos. En ProductChat esa verificación es concreta y tiene una respuesta correcta conocida: la búsqueda por categoría debe devolver también los equipos cargados como `Portatiles` y `Portátiles`, no solo los cargados como `Notebooks`.

Si la verificación falla, vuelve sobre la implementación hasta resolver el problema.

Por eso la calidad de las etapas anteriores resulta fundamental. Cuanto más claros sean la Especificación, el Plan y las Tasks, menos decisiones tendrá que improvisar el agente durante estos ciclos.

## Ejecución en paralelo

Cuando `tasks.md` identifica tareas independientes, algunas pueden ejecutarse en paralelo.

Por ejemplo:

```text
[P] [HU1.1] Implementar búsqueda por nombre.
[P] [HU1.2] Implementar búsqueda por categoría.
[P] [HU1.3] Implementar búsqueda por características.
```

Si no existen dependencias entre ellas, distintos subagentes pueden trabajar simultáneamente sobre estas tareas.

Esto permite que `tasks.md` no sea simplemente una lista de trabajo, sino también una estructura que ayuda a coordinar la ejecución agéntica.

## Verificación automática y manual

Que todas las tareas aparezcan como completadas no significa que el desarrollo haya terminado. Una vez finalizada la implementación automática debemos **ejecutar la aplicación y validarla**.

En ProductChat deberíamos comprobar manualmente, entre otras cosas:

```text
¿Puedo consultar un producto por nombre?

Al preguntar por notebooks, ¿aparecen también los cargados como Portatiles?

Al preguntar por notebooks con 32 GB, ¿salen los cuatro correctos?

¿Queda fuera el NB-1003, que solo es ampliable a 32 GB?

Al preguntar el precio del ThinkBook 14, ¿me advierte del conflicto en NB-1001?

Al preguntar el precio del Monitor LG UltraWide, ¿dice que no está disponible?

Al preguntar el stock del NB-1007, cuyo valor es Si, ¿responde disponible?

¿Cada respuesta indica el SKU del que salió el dato?

¿Qué ocurre cuando el producto no existe?

¿El chatbot evita inventar productos o características?
```

Cada una de estas preguntas tiene una respuesta correcta que podemos comprobar abriendo `catalogo_productos.xlsx`. Esa es la diferencia entre una validación manual y una impresión general de que la aplicación funciona.

Esta validación permite detectar problemas que las verificaciones automáticas pueden no haber identificado, especialmente relacionados con interacción, presentación y experiencia de usuario.

## La implementación no termina el ciclo

**Implement es la última etapa de esta primera vuelta, pero no necesariamente el final del desarrollo.**

Al ejecutar y utilizar la aplicación pueden aparecer errores, comportamientos mejorables o nuevas necesidades.

Por ejemplo, durante la validación de ProductChat podríamos descubrir que:

```text
La presentación de múltiples productos no es suficientemente clara.

Queremos incorporar una página inicial.

Necesitamos mejorar la interfaz gráfica.

Una determinada consulta no se interpreta correctamente.

Aparece una nueva inconsistencia en la planilla que ninguna regla cubre.

Queremos añadir una nueva funcionalidad.
```

El quinto caso es el más probable en un proyecto como este, y también el que más fácilmente nos devuelve al Vibe Coding. Si la semana que viene alguien carga un producto con la RAM en gigabytes escritos con coma, la tentación es parchear el código de normalización. Lo correcto es añadir la regla a la Especificación y dejar que el cambio baje por el ciclo: es una regla de negocio nueva, no un error de implementación.

Aquí es importante distinguir entre **corregir una implementación incorrecta** y **cambiar lo que queremos construir**.

Si la aplicación no cumple algo que ya estaba especificado, corregimos la implementación correspondiente.

Si descubrimos una nueva necesidad, debemos incorporarla al ciclo de SDD:

```text
Nueva necesidad
      ↓
Actualizar Especificación
      ↓
Revisar Plan
      ↓
Actualizar Tasks
      ↓
Analyze
      ↓
Implementación
      ↓
Validación
```

De esta forma evitamos volver al Vibe Coding introduciendo cambios directamente mediante prompts sobre el código sin actualizar los artefactos que constituyen la fuente de verdad del proyecto.

## SDD como ciclo

El proceso completo puede entenderse finalmente como:

```text
Constitución
     ↓
Especificación
     ↓
Clarify
     ↓
Plan
     ↓
Tareas
     ↓
Analyze
     ↓
Implementación
     ↓
Validación
     ↓
Nuevos requisitos / correcciones
     ↺
```

La primera implementación no tiene por qué ser definitiva. **Spec Driven Development es iterativo**: cada ciclo permite evolucionar el producto manteniendo alineados los requisitos, las decisiones técnicas, las tareas y el código.

La diferencia fundamental es que la iteración no consiste simplemente en pedirle a la IA que modifique el código. Cuando cambia el producto, **actualizamos primero su especificación y propagamos el cambio de manera controlada hasta la implementación**.