# El análisis de consistencia en SDD
## ¿Qué es Analyze?

Una vez generadas las tareas y antes de comenzar la implementación, es recomendable incorporar una **puerta de calidad adicional** mediante **Analyze**.

En **GitHub Spec Kit** se ejecuta con:

```text
/speckit-analyze

ROL
Actúa como auditor de consistencia. Detecta y explica problemas; no corrijas los
artefactos por tu cuenta.

CONTEXTO
ProductChat, antes de comenzar la implementación.

ENTRADA
.specify/memory/constitution.md, spec.md, plan.md, data-model.md y tasks.md.

SALIDA ESPERADA
Un informe de hallazgos, cada uno con severidad (CRITICAL, HIGH, MEDIUM, LOW),
el artefacto y la sección donde está el problema, y una recomendación concreta.

QUÉ BUSCAR
Requisitos o criterios de aceptación sin tarea asociada. Tareas que implementan
algo que no aparece en la Especificación. Tareas duplicadas. Decisiones del Plan
que contradicen la Constitución. Historias refinadas con tarea de implementación
pero sin tarea de verificación. Y, en particular, cualquier punto donde un
artefacto haya vuelto a suponer que el catálogo está limpio.

CRITERIO DE ACEPTACIÓN
Cada hallazgo señala un archivo y una sección concretos, y puedo comprobarlo
abriendo ese archivo. No hay hallazgos genéricos del tipo "conviene revisar la
cobertura".
```

> Este prompt está en `prompts/06-analyze.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.


Analyze no es una etapa: es una **puerta de calidad**, igual que Clarify. No produce un artefacto nuevo, revisa los que ya existen. Y cumple una función distinta de la de Clarify. Mientras **Clarify revisa la Especificación individualmente**, Analyze comprueba la **consistencia y cobertura entre los diferentes artefactos del proyecto**.

Su objetivo es responder a la pregunta: **¿todo lo que hemos definido hasta ahora es consistente, está cubierto y respeta la Constitución?**

## ¿Qué analiza?

Analyze revisa conjuntamente los principales artefactos generados durante el proceso:

```text
Constitución
     ↓
Especificación
     ↓
Plan
     ↓
Tareas
```

Dependiendo del proyecto, también puede considerar otros documentos generados durante la planificación, como el modelo de datos o los contratos de las APIs.

El análisis busca principalmente problemas de **consistencia, cobertura y trazabilidad**. Por ejemplo:

- Un requisito de la Especificación que no tiene ninguna tarea asociada.
- Una tarea que implementa una funcionalidad inexistente en la Especificación.
- Dos tareas que implementan innecesariamente lo mismo.
- Una decisión del Plan que contradice la Constitución.
- Una historia de usuario que quedó parcialmente cubierta por las tareas.
- Un criterio de aceptación para el cual no existe una forma clara de verificación.

## Diferencia entre Clarify y Analyze

Aunque ambos funcionan como controles de calidad, actúan en momentos y niveles diferentes.

`Clarify = revisa la calidad de la Especificación`

`Analyze = revisa la coherencia entre todos los artefactos`

Por ejemplo, **Clarify** podría detectar que la Especificación de ProductChat no dice si, ante dos precios en conflicto para el mismo SKU, debe reportarlos ambos o aplicar una regla de desempate. Esa ambigüedad se resuelve directamente en la Especificación.

En cambio, **Analyze** trabaja sobre lo que ya está decidido. Una vez que RN5 y CA3 establecen que el conflicto debe reportarse, Analyze puede detectar que la tarea `HU4.4` implementa ese comportamiento pero que ninguna tarea de la fase de validación lo verifica. La Especificación está bien; lo que falla es su propagación.

## Trazabilidad

Uno de los aspectos más importantes de Analyze es comprobar la trazabilidad entre requisitos, historias de usuario, decisiones técnicas y tareas.

Para ProductChat podríamos tener:

```text
RN5 / CA3
   ↓
HU4: Resolver un catálogo inconsistente
   ↓
HU4.1: Equivalencias de categoría
HU4.2: Conversión de unidades
HU4.3: Atributos condicionales
HU4.4: Conflictos entre SKU duplicados
   ↓
Plan técnico
   ↓
Tasks de implementación
   ↓
Tasks de verificación
```

Analyze permite detectar interrupciones en esta cadena. Si `HU4.4` está definida y tiene tarea de implementación, pero ninguna tarea la verifica, el criterio de aceptación `CA3` no está cubierto aunque todo *parezca* estar hecho. Ese es el tipo de hueco que no se ve leyendo un artefacto por separado.

También puede detectar el problema contrario: una tarea destinada a implementar recomendaciones personalizadas cuando esa funcionalidad nunca fue definida en la Especificación. En ese caso existe **alcance no especificado**.

## Ejecución con GitHub Spec Kit

Una vez revisado `tasks.md`, ejecutamos el prompt anterior. No hace falta añadir información del caso: el prompt nombra los artefactos como entrada y Spec Kit los lee.

Los hallazgos pueden clasificarse por severidad, por ejemplo:

```text
CRITICAL
HIGH
MEDIUM
LOW
```

Un problema crítico o alto debería revisarse antes de comenzar la implementación. Los problemas medios o bajos pueden representar inconsistencias menores, problemas de trazabilidad o aspectos que conviene revisar.

## Ejemplo de hallazgos

Ejecutemos Analyze sobre los artefactos que hemos ido construyendo. El `tasks.md` del módulo anterior tiene tres problemas reales, y son los que Analyze debería encontrar.

El primero es el más grave, porque afecta a un criterio de aceptación:

```text
[HIGH] La historia refinada HU4.4 (informar SKU duplicados con valores en
conflicto) tiene tarea de implementación en la Fase 3, pero no existe ninguna
tarea de verificación en la Fase 4. El criterio de aceptación CA3 depende
directamente de este comportamiento y quedaría sin comprobar.

Recomendación:
Añadir a la Fase 4 una tarea de validación asociada a HU4.4, verificable
contra el SKU NB-1001 del catálogo.
```

El segundo es una duplicación:

```text
[MEDIUM] Las tareas HU2.1 ("consultar los atributos disponibles de un
producto") y HU2.2 ("obtener las características disponibles de un producto")
describen el mismo comportamiento sobre la misma entidad. La Especificación
no distingue atributo de característica.

Recomendación:
Consolidar ambas en una sola historia refinada, o bien precisar en la
Especificación en qué se diferencian.
```

El tercero es una brecha de cobertura menor:

```text
[MEDIUM] HU3.2 (informar al usuario cuando no se encuentran productos) no
tiene tarea de verificación asociada.

Recomendación:
Añadir la validación correspondiente, o justificar por qué queda cubierta
por la verificación de HU3.1.
```

Analyze también puede detectar el problema inverso, una tarea que implementa algo que nadie pidió:

```text
[HIGH] Existe una tarea para corregir y reescribir el archivo Excel con los
datos normalizados. Esta funcionalidad no aparece en la Especificación y
contradice el principio constitucional "el chatbot lee, no corrige".

Recomendación:
Eliminar la tarea. Si la empresa quiere una versión limpia del catálogo,
debe especificarse como producto aparte.
```

Este último caso es instructivo: no es solo alcance no especificado, es una violación de la Constitución. Un agente que encuentra datos sucios tiende naturalmente a querer limpiarlos, y la Constitución existe precisamente para impedirlo sin una decisión humana de por medio.

## Corrección de inconsistencias

Analyze debe utilizarse principalmente para **detectar y explicar problemas**, no para modificar automáticamente los artefactos sin revisión.

Cuando encuentra una inconsistencia, puede proponerse una corrección concreta. Una vez revisada y aceptada, se actualiza el artefacto correspondiente.

La corrección debe realizarse en el nivel donde se encuentra realmente el problema:

`Problema en el qué → corregir la Especificación`

`Problema en el cómo → corregir el Plan`

`Problema en la ejecución → corregir Tasks`

`Problema con una regla global → revisar respecto de la Constitución`

Después de realizar cambios importantes puede ejecutarse nuevamente `/speckit-analyze` para comprobar que los artefactos han quedado alineados.

## Analyze como puerta previa a la implementación

Analyze funciona como la última revisión global antes de comenzar a generar código. Hasta este momento hemos trabajado principalmente sobre documentos y decisiones, por lo que corregir una inconsistencia todavía resulta relativamente sencillo.

El flujo completo antes de implementar queda:

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

De esta forma, **Clarify** reduce las ambigüedades de la Especificación y **Analyze** comprueba posteriormente que esa Especificación se haya propagado correctamente hacia el Plan y las Tareas. Solo después de superar estas revisiones comienza la implementación.