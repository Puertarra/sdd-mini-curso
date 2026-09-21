# La Constitución en SDD

## ¿Qué es la Constitución?

La **Constitución** es la primera etapa del ciclo de **Spec Driven Development (SDD)**. Define los principios y reglas fundamentales que deben respetarse durante todo el desarrollo del proyecto. Su objetivo es establecer un conjunto pequeño de reglas **globales y no negociables** que guíen posteriormente la especificación, el plan, las tareas y la implementación.

La Constitución responde principalmente a la pregunta: **¿qué reglas debe respetar siempre nuestro proyecto?**

No describe funcionalidades concretas ni determina cómo se implementará técnicamente el sistema. Por ejemplo, no debería indicar qué lenguaje de programación, framework, modelo de IA o base de datos utilizar. Esas decisiones corresponden principalmente al **Plan**.

## ¿Qué debe contener?

Los principios de una Constitución deben ser de **alto nivel, claros y accionables**. Normalmente representan decisiones relacionadas con el negocio, alcance y calidad del producto. Por ejemplo: mantener la solución simple, no implementar funcionalidades que no estén especificadas, establecer el idioma del producto, exigir resultados verificables y definir principios generales para el tratamiento de los datos.

La Constitución debe mantenerse **corta**, ya que sus principios se incorporarán repetidamente al contexto utilizado por los agentes de IA. Debe contener únicamente reglas que realmente deban gobernar todo el proyecto.

## Constitución, Especificación y Plan

Es importante separar correctamente estas etapas. La **Constitución** establece reglas permanentes, por ejemplo: *"El chatbot nunca debe inventar información sobre los productos"*. La **Especificación** define qué debe hacer el producto, por ejemplo: *"El usuario debe poder consultar el precio y disponibilidad de un producto"*. El **Plan** determina cómo implementar esa funcionalidad.

En forma resumida:

`Constitución = reglas permanentes`

`Especificación = qué debe hacer el producto`

`Plan = cómo se implementará`

## Ejemplo: la Constitución de ProductChat

Retomamos el caso del curso. **ProductChat** es un chatbot interno para Distribuidora Andes que permite consultar en lenguaje natural el catálogo de productos y servicios de la empresa. Ese catálogo vive en una planilla Excel mantenida a mano, con SKU duplicados, categorías escritas de varias formas, atributos sin unidad estable y columnas de texto libre. La planilla no se va a limpiar antes de construir el chatbot.

Ese último punto obliga a una decisión constitucional que no aparecería en un proyecto con datos limpios: qué debe hacer el producto cuando la fuente se contradice a sí misma. No es una decisión técnica ni una funcionalidad, es una regla permanente sobre cómo se comporta el sistema frente a un dato en el que no se puede confiar del todo.

La Constitución no debe especificar todavía cómo se leerá el Excel, qué modelo de lenguaje se utilizará ni cómo estará construido el backend. Esas decisiones corresponden al Plan.

### Principios

**1. Simplicidad ante todo.** Ante dos soluciones que satisfagan los mismos requisitos, se utilizará la más simple. Se trata de una primera versión del producto y no se añadirá complejidad para resolver necesidades futuras que todavía no hayan sido especificadas.

**2. El catálogo es la fuente de verdad, con todos sus defectos.** Toda respuesta sobre productos o servicios debe poder rastrearse hasta una fila concreta del archivo. El chatbot no inventa productos, precios, atributos ni disponibilidad.

**3. La inconsistencia se muestra, no se esconde.** Cuando el catálogo se contradice a sí mismo, el chatbot informa el conflicto en lugar de elegir un valor en silencio. Un dato entregado con falsa seguridad es peor que un dato ausente.

**4. El chatbot lee, no corrige.** El archivo de origen nunca se modifica. La normalización de categorías, unidades y formatos ocurre al interpretar el dato, no al escribirlo.

**5. Cero alcance fantasma.** No se implementará ninguna funcionalidad que no esté definida en la especificación. Si durante el desarrollo aparece una nueva funcionalidad potencialmente útil, deberá proponerse e incorporarse a la especificación antes de implementarla.

**6. Respuestas verificables.** La información proporcionada debe poder contrastarse abriendo la planilla. Los criterios de éxito deben poder comprobarse utilizando la aplicación, sin necesidad de revisar el código.

**7. Incertidumbre explícita.** Cuando los datos disponibles no permitan responder una consulta, el chatbot debe indicarlo claramente en lugar de completar la respuesta mediante suposiciones.

**8. Protección de los datos.** Se solicitará únicamente la información necesaria para prestar el servicio. Rutas absolutas, credenciales y secretos nunca deberán incorporarse directamente al código ni exponerse al usuario.

## Flujo de trabajo y gobernanza

Toda nueva funcionalidad debe aparecer primero en la especificación. Si un requisito es ambiguo y su interpretación puede modificar significativamente el comportamiento del producto, se debe solicitar una aclaración antes de implementarlo. La especificación, el plan y las tareas deben ser coherentes con los principios establecidos en la Constitución.

La Constitución prevalece sobre decisiones individuales de implementación que entren en conflicto con sus principios. Cualquier modificación debe documentarse y versionarse, comprobando posteriormente que las especificaciones, planes y tareas existentes continúan siendo coherentes con ella.

## Ejemplo con GitHub Spec Kit

Utilizando **GitHub Spec Kit**, solicitamos la creación de la Constitución con este prompt. Fíjate en su estructura: rol, contexto, entrada, salida esperada y criterio de aceptación. Es el contrato que siguen todos los prompts del curso, descrito en el módulo anterior.

```text
/speckit-constitution

ROL
Actúa como responsable de definir las reglas permanentes de un proyecto de
software. No eres el arquitecto ni el programador: no decides tecnologías.

CONTEXTO
ProductChat es un chatbot interno para Distribuidora Andes que permite consultar
en lenguaje natural el catálogo de productos y servicios de la empresa. El
catálogo vive en catalogo_productos.xlsx, una planilla mantenida a mano, con SKU
duplicados, categorías escritas de varias formas, atributos sin unidad estable,
precios en texto y columnas de texto libre. La planilla no se va a limpiar antes
de construir el chatbot.

ENTRADA
Los ocho principios que se enumeran más abajo. No hay ningún artefacto previo:
esta es la primera etapa del ciclo.

SALIDA ESPERADA
El archivo de Constitución del proyecto, normalmente
.specify/memory/constitution.md, corto, accionable y en lenguaje claro.
Sin decisiones sobre lenguajes, frameworks, modelos de IA, arquitectura ni
tecnologías concretas.

PRINCIPIOS
1. Simplicidad ante todo: ante dos soluciones, siempre la más simple. Es una
   versión 1; nada de complejidad anticipada.
2. El catálogo es la fuente de verdad, con todos sus defectos: las respuestas
   deben poder rastrearse hasta una fila concreta del archivo. No inventar
   productos, precios, atributos ni disponibilidad.
3. La inconsistencia se muestra, no se esconde: cuando el catálogo se
   contradice a sí mismo, el chatbot lo informa en lugar de elegir un valor
   en silencio.
4. El chatbot lee, no corrige: nunca modifica el archivo de origen. La
   normalización ocurre al interpretar el dato, no al escribirlo.
5. Cero alcance fantasma: no implementar ninguna funcionalidad que no esté
   escrita en la Especificación. Si surge una idea nueva, se propone, no se
   construye.
6. Verificable por una persona no técnica: cada criterio de éxito debe poder
   comprobarse usando la aplicación y abriendo la planilla, sin leer código.
7. Incertidumbre explícita: cuando los datos no permitan responder, decirlo
   claramente en lugar de suponer.
8. Datos con respeto: pedir solo lo imprescindible. No introducir rutas
   absolutas, credenciales ni secretos en el código.

CRITERIO DE ACEPTACIÓN
La Constitución resultante cabe en una página, contiene los ocho principios sin
añadir otros, y ninguno de ellos menciona una tecnología concreta.
```

> Este prompt está en `prompts/01-constitution.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

Spec Kit utilizará estas instrucciones para generar el archivo de Constitución del proyecto, normalmente:

```text
.specify/memory/constitution.md
```

Este documento establece las reglas que gobernarán las siguientes etapas del desarrollo:

`Constitución → Especificación → Clarify → Plan → Tareas → Analyze → Implementación`

La Constitución se define inicialmente una sola vez y solo debería modificarse cuando cambien los principios fundamentales que gobiernan el proyecto.