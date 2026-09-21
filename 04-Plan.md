# El Plan en SDD

## ¿Qué es el Plan?

El **Plan** es la tercera etapa del ciclo de **Spec Driven Development (SDD)**. Una vez definida la Constitución, creada la Especificación y resueltas sus ambigüedades mediante Clarify, el Plan transforma el **qué** en el **cómo**.

En esta etapa la IA adquiere un papel mucho más activo. A partir de las reglas de la Constitución y los requisitos de la Especificación, analiza las alternativas y propone las decisiones técnicas necesarias para implementar el sistema.

El Plan responde principalmente a la pregunta: **¿cómo vamos a construir lo que hemos especificado?**

## ¿Qué debe contener el Plan?

A diferencia de la Especificación, en el Plan sí corresponde tomar decisiones técnicas. Dependiendo del proyecto, puede definir aspectos como:

- Arquitectura de la solución.
- Lenguajes y frameworks.
- Componentes principales.
- Organización del frontend y backend.
- Modelo y almacenamiento de datos.
- APIs e interfaces entre componentes.
- Dependencias principales.
- Estrategia de pruebas.
- Estructura del proyecto.
- Estrategia de despliegue.

Estas decisiones deben derivarse de necesidades reales establecidas en la Especificación y respetar siempre los principios definidos en la Constitución.

## El papel de la IA

En SDD, el Plan puede ser elaborado principalmente por el **agente de IA**. La responsabilidad humana cambia respecto de la etapa anterior.

En la Especificación, la persona define activamente qué debe hacer el sistema. En el Plan, la IA propone cómo construirlo y la persona **revisa, cuestiona y valida las decisiones**.

En forma resumida:

`Persona + IA: Constitución`

`Persona + IA: Especificación`

`IA + revisión humana: Plan`

Esto permite aprovechar el conocimiento técnico del agente sin delegarle las decisiones sobre qué necesita realmente el negocio.

## Información adicional para generar el Plan

Aunque el Plan se genera a partir de la Constitución y la Especificación, es posible proporcionar instrucciones adicionales antes de generarlo. Estas instrucciones deben ser breves y utilizarse solamente cuando exista algún criterio relevante que convenga enfatizar.

Por ejemplo:

```text
Prioriza la simplicidad según lo establecido en la Constitución.

Es una versión 1 que debe poder desplegarse fácilmente.

No añadas infraestructura que la especificación no necesite.

Explica las decisiones técnicas importantes también en términos comprensibles
para una persona no técnica.
```

Estas instrucciones no deberían utilizarse para introducir nuevos requisitos funcionales. Si aparece un nuevo requisito sobre **qué debe hacer el producto**, debe incorporarse primero a la Especificación.

## Generación del Plan con GitHub Spec Kit

En **GitHub Spec Kit**, el Plan se genera mediante:

```text
/speckit-plan
```

El agente utiliza la Constitución y la Especificación existentes como contexto y comienza a diseñar la solución técnica.

Para **ProductChat** utilizamos:

```text
/speckit-plan

ROL
Actúa como arquitecto de software. Propón el cómo y justifícalo; la decisión
final es mía.

CONTEXTO
ProductChat es una primera versión que debe ser fácil de desplegar y mantener.
La fuente de datos es catalogo_productos.xlsx, una planilla con SKU duplicados,
categorías inconsistentes, atributos sin unidad estable y precios en texto. La
planilla no se corrige ni se sustituye.

ENTRADA
spec.md, .specify/memory/constitution.md y el propio archivo
catalogo_productos.xlsx, que puedes inspeccionar para conocer la forma real de
los datos.

SALIDA ESPERADA
plan.md con la arquitectura, el stack y la estructura del proyecto;
data-model.md distinguiendo explícitamente la forma cruda de la planilla del
modelo interno normalizado, y describiendo la transformación entre ambos;
research.md justificando las decisiones con alternativas.

DECISIONES QUE DEBES RESOLVER EXPLÍCITAMENTE
Dónde se aplica la normalización, al cargar o en cada consulta. Cuándo se lee el
Excel. Cómo se representa un dato ausente frente a un cero y frente a un texto
como "consultar". Cómo se representa un SKU con dos valores en conflicto sin
colapsarlos. Y qué parte del trabajo hace el modelo de lenguaje y qué parte es
código determinista.

RESTRICCIONES
Prioriza la simplicidad según la Constitución. No añadas infraestructura que la
especificación no necesite. No propongas generar una copia limpia de la planilla
ni deduplicar en silencio: ambas cosas contradicen la Constitución. Explica cada
decisión relevante también en términos comprensibles para una persona no técnica.

CRITERIO DE ACEPTACIÓN
El modelo de datos permite sostener dos valores distintos para el mismo SKU. La
interpretación de las unidades es determinista y no depende del modelo de
lenguaje. Cada componente que añade complejidad está justificado por un
requisito concreto de spec.md.
```

> Este prompt está en `prompts/04-plan.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

A partir de este punto, el agente puede formular preguntas cuando una decisión técnica importante no pueda determinarse razonablemente a partir de la información disponible.

## Decisiones técnicas durante el Plan

En esta etapa pueden aparecer preguntas sobre arquitectura, almacenamiento, integración, despliegue o tecnologías. En ProductChat, buena parte de esas preguntas no vienen de la arquitectura sino del estado del catálogo, y conviene resolverlas explícitamente en el Plan:

- **Dónde vive la normalización.** Las reglas RN2, RN3 y RN4 pueden aplicarse al cargar el archivo, construyendo un modelo interno limpio, o en cada consulta. La primera opción es más simple de razonar y de probar; la segunda refleja antes los cambios que el encargado haga en la planilla. No es una decisión indiferente: condiciona PA5 de la especificación.
- **Cuándo se lee el Excel.** Una sola vez al arrancar, en cada consulta, o con una recarga explícita.
- **Cómo se representa un dato ausente.** Un precio vacío, un `consultar` y un `0` no son lo mismo, y el modelo interno debe poder distinguirlos para que RF5 y CA4 sean verificables.
- **Cómo se representa un conflicto.** RN5 obliga a que el modelo interno pueda sostener dos valores distintos para el mismo SKU sin colapsarlos. Un modelo de datos que use el SKU como clave única hace imposible cumplir CA3, y es el error más fácil de cometer en esta etapa.
- **Qué hace el modelo de lenguaje y qué no.** Interpretar la pregunta del usuario y redactar la respuesta es una cosa; decidir si `32768 MB` son 32 GB es otra. Si la normalización queda en manos del modelo, el resultado deja de ser reproducible y CA1 deja de poder verificarse.

Si el usuario tiene conocimientos técnicos puede seleccionar directamente una alternativa. Si no los tiene, puede pedir al agente que recomiende una opción y explique la decisión en términos de negocio.

Por ejemplo:

```text
Estoy decidiendo cómo ProductChat debe representar un SKU con dos precios
distintos, según RN5 de spec.md. Recomiéndame la alternativa más simple
para esta primera versión y explícame ventajas y desventajas sin asumir
conocimientos técnicos. No la implementes todavía.
```

El objetivo no es evitar las decisiones técnicas, sino conseguir que sean **explícitas, justificadas y coherentes con la Especificación y la Constitución**.

## Artefactos generados

Dependiendo de la versión y configuración de Spec Kit, la etapa de planificación puede generar distintos documentos dentro del directorio de la especificación. Entre ellos pueden encontrarse:

```text
plan.md
research.md
data-model.md
quickstart.md
contracts/
```

`plan.md` contiene el plan técnico principal, incluyendo arquitectura, tecnologías, estructura y decisiones de implementación.

`research.md` documenta y justifica decisiones técnicas relevantes, especialmente cuando existen varias alternativas posibles.

`data-model.md` describe las entidades, campos, relaciones y estructuras de datos necesarias para implementar la especificación. En un caso como ProductChat debe distinguir con claridad **la forma cruda de la fuente** (las columnas de la planilla, tal como están) del **modelo interno normalizado** sobre el que operan las consultas, y explicitar qué transformación lleva de una a otro.

`contracts/` puede contener contratos de APIs o interfaces necesarias entre los componentes del sistema.

`quickstart.md` puede incluir escenarios que permiten comprobar manualmente que la solución resultante satisface los requisitos principales.

No todos los proyectos necesitan necesariamente todos estos artefactos. El contenido dependerá de la naturaleza del sistema y de las decisiones tomadas durante la planificación.

## Comprobación de la Constitución

Una parte importante de la planificación consiste en comprobar que las decisiones técnicas respetan la Constitución.

Por ejemplo, si la Constitución establece **simplicidad ante todo**, una arquitectura distribuida con numerosos servicios debería estar claramente justificada por los requisitos. Si establece que el catálogo es la fuente de verdad, el Plan debe preservar ese principio en la arquitectura propuesta.

En ProductChat hay dos principios que resultan especialmente fáciles de violar sin darse cuenta. El principio **la inconsistencia se muestra, no se esconde** queda incumplido por cualquier diseño que deduplique el catálogo en silencio al cargarlo. El principio **el chatbot lee, no corrige** queda incumplido por cualquier plan que proponga generar una versión limpia de la planilla y trabajar sobre ella. Ambas son decisiones técnicas razonables en abstracto y contrarias a la Constitución de este proyecto.

El Plan no puede utilizarse para ignorar o modificar silenciosamente principios establecidos previamente.

## Revisión e iteración del Plan

El Plan generado por la IA debe revisarse antes de convertirlo en tareas. No es necesario comprender cada detalle técnico, pero sí verificar que las decisiones sean coherentes con las necesidades del proyecto.

Puede utilizarse lenguaje natural para pedir explicaciones:

```text
Explícame la decisión de plan.md sobre cuándo se lee catalogo_productos.xlsx
como si yo fuera el responsable del negocio y no un desarrollador: qué
nota el vendedor si el encargado edita la planilla a mediodía.
```

También pueden solicitarse alternativas:

```text
Dame dos alternativas para la decisión sobre dónde se aplica la normalización
de los datos del catálogo. Para cada una, dime qué requisito de spec.md
favorece, cuál dificulta, y cuál elegirías si el criterio dominante
fuera la simplicidad.
```

O pedir modificaciones concretas:

```text
Simplifica esta arquitectura manteniendo todos los requisitos y criterios
de aceptación de spec.md. Indícame explícitamente qué componentes
eliminas y qué requisito garantizaba cada uno antes del cambio.
```

Si el problema detectado corresponde al **cómo**, se modifica el Plan. Si durante la revisión se descubre que el problema está realmente en el **qué**, se debe regresar a la Especificación y corregirla antes de continuar.

## Resultado de la etapa

Al finalizar esta etapa debemos disponer de un diseño técnico suficientemente preciso para que el agente pueda convertirlo en unidades concretas de trabajo.

La separación entre las etapas queda así:

`Constitución = qué reglas debemos respetar`

`Especificación = qué debe hacer el sistema`

`Plan = cómo vamos a construirlo`

`Tareas = qué acciones concretas debemos ejecutar`

Una vez revisado y aceptado el Plan, el siguiente paso es transformarlo en **tareas pequeñas, ordenadas y verificables** mediante `/speckit-tasks`.