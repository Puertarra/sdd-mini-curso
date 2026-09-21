# La Especificación en SDD

## ¿Qué es la Especificación?

La **Especificación (Spec)** es la segunda etapa del ciclo de **Spec Driven Development (SDD)** y probablemente la más importante. Define con precisión **qué debe hacer el producto, para quién y bajo qué condiciones se considera correcto**.

Mientras la Constitución establece las reglas permanentes del proyecto, la Especificación describe el comportamiento esperado del producto. Debe concentrarse en el **qué** y evitar definir el **cómo**, ya que las decisiones técnicas corresponden posteriormente al Plan.

La Especificación responde principalmente a la pregunta: **¿qué queremos construir y qué debe cumplir?**

## ¿Qué hace buena a una especificación?

Una buena especificación debe contener al menos cuatro elementos fundamentales.

**1. Intención clara.** Debe explicar qué problema se quiere resolver y quién es el usuario. El objetivo debe poder entenderse sin conocimientos técnicos.

**2. Contexto de negocio.** Debe explicar por qué el problema es importante y cómo se utilizará realmente el producto. Esto permite que los requisitos tengan relación con situaciones reales y no sean solamente una lista abstracta de funcionalidades.

**3. Restricciones y reglas de negocio.** Debe establecer las condiciones que el producto necesariamente debe respetar. Estas reglas deben ser concretas y evitar interpretaciones.

**4. Criterios de aceptación verificables.** Debe establecer condiciones que permitan determinar objetivamente si una funcionalidad está correctamente implementada. Idealmente, cada criterio debería poder responderse con un **sí o un no** utilizando la aplicación.

## ¿Qué hace mala a una especificación?

Una mala especificación suele presentar cuatro problemas. El primero es la **ambigüedad**, mediante expresiones como *"el chatbot debe responder bien"* o *"la interfaz debe verse profesional"*, que obligan a la IA a interpretar qué significa exactamente "bien" o "profesional".

El segundo es la **sobre especificación técnica**, incluyendo prematuramente lenguajes, frameworks, modelos, bases de datos o decisiones de arquitectura. El tercero, estrechamente relacionado, es **mezclar el qué con el cómo**.

El cuarto es más silencioso y es el que más nos va a ocupar en este curso: **suponer que los datos están limpios**. Es habitual describir el desorden de la fuente en la introducción y después escribir requisitos y criterios de aceptación que solo funcionan si ese desorden no existiera. Un criterio como *"el precio mostrado coincide con el registrado en la fuente"* parece verificable, pero deja de serlo en cuanto la fuente registra dos precios distintos para el mismo producto. Si la fuente es imperfecta, la especificación tiene que decir qué hacer con cada imperfección.

Por ejemplo:

> "El usuario debe poder consultar productos mediante lenguaje natural."

es una especificación del **qué**.

En cambio:

> "Implementar las consultas utilizando Python, FastAPI y un modelo determinado."

describe el **cómo** y corresponde al Plan.

## Lista de comprobación

Antes de aprobar una especificación conviene comprobar:

1. ¿Se entiende el objetivo sin conocimientos técnicos?
2. ¿Está claro quién es el usuario?
3. ¿Las reglas de negocio son concretas?
4. ¿Existen criterios de aceptación verificables con un sí o un no?
5. ¿Está separado el qué del cómo?
6. ¿Existe una sección que indique explícitamente qué está fuera de alcance?
7. ¿Se han considerado casos límite, como datos vacíos, inexistentes o incorrectos?
8. ¿Existe alguna decisión importante que la IA tendría que adivinar?
9. ¿Existen expresiones ambiguas que puedan interpretarse de distintas maneras?
10. ¿Otra persona podría leer la especificación y construir esencialmente el mismo producto?
11. ¿Se dice qué debe ocurrir cuando la fuente de datos se contradice a sí misma?
12. ¿Los criterios de aceptación siguen siendo verificables con los datos reales, tal como están hoy?

Si alguna decisión relevante queda abierta, debería resolverse en la especificación en lugar de permitir que el agente de IA la determine arbitrariamente.

## Estructura de una especificación

Una estructura sencilla y reutilizable puede contener:

```text
1. Objetivo y contexto de negocio
2. Usuarios
3. Historias de usuario
4. Requisitos funcionales
5. Reglas de negocio
6. Criterios de aceptación
7. Casos límite
8. Fuera de alcance
```

# Ejemplo: especificación de ProductChat

Retomamos **ProductChat**, el chatbot interno de Distribuidora Andes. Su catálogo de productos y servicios vive en `catalogo_productos.xlsx`, una planilla mantenida a mano, con SKU duplicados, categorías inconsistentes, atributos sin unidad estable y columnas de texto libre.

Lo que sigue es la especificación formal derivada de `spec_servilleta_productchat.md`. Conviene leer ambas y comparar: la servilleta es lo que escribe una persona; esto es lo que esa persona quiere obtener de vuelta.

## 1. Objetivo y contexto de negocio

ProductChat permite consultar en lenguaje natural el catálogo de Distribuidora Andes, sin abrir la planilla y sin conocer su estructura.

El problema no es solo que la planilla sea incómoda de consultar. Es que **consultarla a mano da respuestas equivocadas**. Un vendedor que busca notebooks con 32 GB de RAM filtrando por categoría pierde los equipos cargados como `Portatiles`; buscando el texto `32 GB` pierde los cargados como `32768 MB` y como `32`; buscando `32` a secas incorpora un Dell que solo es ampliable a 32 GB bajo pedido. El catálogo no se va a limpiar antes de construir el chatbot, de modo que **interpretar correctamente un dato sucio es el requisito central del producto**, no un detalle de implementación.

## 2. Usuarios

El usuario principal es el **vendedor**: atiende clientes, no es técnico, y necesita respuestas rápidas y confiables. Prefiere un "no estoy seguro" a un dato inventado.

El **encargado de catálogo** no usa el chatbot, pero le interesa que las inconsistencias se hagan visibles en lugar de quedar disimuladas, porque eso le indica qué corregir primero en la planilla.

## 3. Historias de usuario

### HU1 — Buscar productos

El vendedor pregunta *"¿qué notebooks tienen 32 GB de RAM?"*. El chatbot devuelve los cuatro equipos que realmente cumplen, incluyendo los que declaran la memoria en otra unidad, y excluyendo los que solo la admiten como ampliación.

### HU2 — Consultar los atributos de un producto

El vendedor pregunta *"¿cuánta RAM tiene el Acer Swift Go?"*. El chatbot responde 32 GB, indicando el SKU de origen, aunque la planilla lo registre como `32768 MB`.

### HU3 — Gestionar información inexistente o insuficiente

El vendedor pregunta por un producto que no está en el catálogo, o por el precio del Monitor LG UltraWide, cuya celda dice `consultar`. El chatbot informa la ausencia en lugar de completarla.

### HU4 — Resolver un catálogo inconsistente

El vendedor pregunta *"¿cuál es el precio del ThinkBook 14?"*. El SKU `NB-1001` aparece dos veces con precios distintos. El chatbot informa que existen dos precios registrados y que el catálogo está inconsistente en ese punto, **sin elegir uno**.

## 4. Requisitos funcionales

**RF1.** El sistema debe leer el catálogo desde `catalogo_productos.xlsx`, localizando la fila de cabecera aunque no sea la primera del archivo.

**RF2.** El sistema debe descartar filas vacías y recortar los espacios sobrantes de los valores de texto.

**RF3.** El sistema debe normalizar la categoría, de modo que las variantes de mayúsculas, tildes y sinónimos declarados se traten como una misma familia.

**RF4.** El sistema debe normalizar los atributos numéricos con unidad, de modo que `32 GB`, `32768 MB`, `RAM 32 GB` y `32` se interpreten como el mismo valor.

**RF5.** El sistema debe normalizar el precio a un valor numérico, y marcar como **precio no disponible** los casos vacíos o textuales.

**RF6.** El sistema debe normalizar el stock a tres estados: disponible, sin stock y no informado.

**RF7.** El sistema debe detectar SKU duplicados y, cuando los registros duplicados no coincidan en el atributo consultado, informar el conflicto.

**RF8.** El usuario debe poder consultar productos por nombre, categoría o características mediante lenguaje natural.

**RF9.** Cuando varios productos satisfagan una consulta, el sistema debe poder presentar múltiples resultados.

**RF10.** El sistema debe distinguir un atributo declarado de un atributo condicional mencionado en texto libre, y no tratar el segundo como si fuera el primero.

**RF11.** Cuando no exista información suficiente para responder, el sistema debe indicarlo explícitamente.

**RF12.** Cada dato entregado debe ir acompañado del SKU del que proviene.

## 5. Reglas de negocio

**RN1.** La hoja `Catálogo` es la fuente de verdad. La hoja `catalogo_antiguo` es histórica y no se utiliza en esta versión.

**RN2.** Familias equivalentes: `Notebooks` ≡ `Portatiles` ≡ `Portátiles`, con cualquier combinación de mayúsculas y tildes.

**RN3.** Conversión de unidades de memoria: 1024 MB = 1 GB. Un número sin unidad en un campo de memoria se interpreta en GB.

**RN4.** Un atributo mencionado como condición futura ("ampliable a", "previa solicitud", "bajo pedido") no cuenta como atributo del producto.

**RN5.** Ante un SKU duplicado con valores en conflicto, el chatbot no elige: reporta ambos valores e indica la inconsistencia.

**RN6.** El chatbot no inventa productos, precios ni atributos. Todo dato entregado debe poder rastrearse hasta una fila del archivo.

## 6. Criterios de aceptación

**CA1.** Ante "¿qué notebooks tienen 32 GB de RAM?", el chatbot devuelve `NB-1005`, `NB-1007`, `NB-1009` y `NB-1012`, y no incluye `NB-1003`.

**CA2.** Ante "¿qué notebooks hay?", el resultado incluye los equipos cargados como `Portatiles` y `Portátiles`.

**CA3.** Ante "¿cuál es el precio del ThinkBook 14?", el chatbot informa que hay dos precios en conflicto para `NB-1001` y no entrega uno solo como si fuera el correcto.

**CA4.** Ante una consulta por el precio de `MON-3002`, el chatbot responde que el precio no está disponible.

**CA5.** Ante una consulta por el stock de `NB-1007`, cuyo valor es `Si`, el chatbot responde "disponible" y no un número.

**CA6.** Ante una consulta por un producto inexistente, el chatbot indica que no encontró información.

**CA7.** Cada dato entregado va acompañado de su SKU de origen.

**CA8.** Una persona no técnica puede ejecutar las consultas anteriores y verificar cada respuesta abriendo la planilla.

## 7. Casos límite

**CL1.** Fila completamente vacía en medio de los datos: se ignora.

**CL2.** Producto sin características (`MON-3003`): se encuentra por nombre y categoría; al pedir una característica se responde que no está registrada.

**CL3.** Producto sin precio (`NB-1009`): aparece en los resultados, con el precio marcado como no disponible.

**CL4.** Nombres muy parecidos (`NB-1001` y `NB-1015`): se devuelven ambos y se distinguen por SKU.

**CL5.** Mismo producto bajo dos SKU (`NB-1002` y `NB-2002`): se devuelven ambos; el chatbot no los fusiona por su cuenta.

**CL6.** Servicios (`SRV-*`): no tienen stock; preguntar por su stock debe responder "no aplica", no "sin stock".

**CL7.** El archivo no se puede leer: el chatbot lo informa y no responde con datos antiguos.

## 8. Fuera de alcance

Para esta primera versión quedan fuera de alcance:

- Modificar, crear o eliminar productos desde el chatbot.
- **Corregir el archivo Excel.** El chatbot lee; no escribe.
- Comprar, cotizar formalmente o gestionar pedidos.
- Recomendar productos con criterios externos al catálogo.
- Responder preguntas que no sean sobre el catálogo.
- Utilizar la hoja `catalogo_antiguo`.
- Conectarse a una base de datos SQL. En esta versión la fuente es el Excel.
- Cuentas de usuario, permisos y multiusuario.
- Incorporar funcionalidades no descritas explícitamente en esta especificación.

## Validación de la Especificación con Clarify

Una vez creada la especificación, es conveniente revisarla antes de pasar al Plan. En **GitHub Spec Kit**, esta revisión se realiza con:

```text
/speckit-clarify

ROL
Actúa como revisor de requisitos. Tu tarea es encontrar lo que falta decidir, no
decidirlo tú.

CONTEXTO
Estás revisando la especificación de ProductChat, un chatbot sobre un catálogo
Excel con datos inconsistentes.

ENTRADA
spec.md y .specify/memory/constitution.md.

SALIDA ESPERADA
Una lista de preguntas concretas, cada una con las alternativas posibles y su
consecuencia, para que yo elija. Después de que responda, incorpora las
decisiones a spec.md.

FOCO
Presta atención especial a los puntos donde la especificación supone un dato
limpio que el catálogo no garantiza: qué ocurre ante dos valores en conflicto
para el mismo SKU, qué unidad se asume cuando un atributo no la declara, si una
consulta por disponibilidad se refiere al catálogo o al stock, y qué se hace con
los atributos mencionados solo como condición futura en texto libre.

CRITERIO DE ACEPTACIÓN
Al terminar, spec.md no contiene ninguna decisión pendiente que pueda cambiar el
comportamiento observable del producto. Toda decisión incorporada la tomé yo, no
tú.
```

> Este prompt está en `prompts/03-clarify.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.


**Clarify** analiza la especificación buscando **ambigüedades, decisiones no resueltas, requisitos incompletos o situaciones en las que la IA tendría que asumir información que no ha sido definida**. Cuando detecta estos puntos, formula preguntas para que sea el usuario quien tome las decisiones necesarias.

El objetivo es reducir al mínimo las suposiciones antes de comenzar la planificación. Las respuestas obtenidas durante este proceso se incorporan a la especificación, dejando una versión más precisa y preparada para generar el Plan.

En ProductChat, la sección 9 de la servilleta ya anticipa buena parte de lo que Clarify debería preguntar. Dos ejemplos concretos. Primero, RN5 dice que ante dos precios en conflicto el chatbot reporta ambos, pero eso es una decisión de negocio que nadie ha confirmado: quizá la empresa prefiera una regla de desempate. Segundo, cuando el vendedor pregunta qué notebooks *tienen* 32 GB, no está claro si pregunta por el catálogo o por lo que puede vender hoy: de los cuatro equipos que cumplen, solo dos tienen stock numérico mayor que cero. Ninguna de las dos decisiones debería tomarla el agente por su cuenta.

El flujo queda entonces:

`Constitución → Especificación → Clarify → Plan → Tareas → Analyze → Implementación`

# La especificación como responsabilidad humana

La IA puede ayudar a estructurar, revisar y formalizar una especificación, pero las decisiones fundamentales deberían proceder de quien conoce el problema y el negocio.

En SDD, la persona define principalmente la **intención, las reglas de negocio, el alcance y los criterios de aceptación**. El agente puede posteriormente transformar esa información en documentos más formales, generar el Plan y convertirlo en tareas ejecutables.

Por tanto, la especificación es la etapa donde debería existir una mayor participación humana:

`Persona: define y valida el qué`

`IA: ayuda a formalizar el qué y desarrolla el cómo`

# Uso con GitHub Spec Kit

Una vez definidos los criterios anteriores, pueden utilizarse como entrada para **GitHub Spec Kit** mediante `/speckit-specify`.

El objetivo no es pedir a la IA simplemente:

```text
Crea un chatbot para consultar productos.
```

sino proporcionarle la intención, usuarios, historias de usuario, requisitos, reglas de negocio, criterios de aceptación, casos límite y elementos fuera de alcance que acabamos de definir. Todo eso ya está escrito en `spec_servilleta_productchat.md`, así que el prompt no repite su contenido: lo nombra como archivo de entrada.

```text
/speckit-specify

ROL
Actúa como analista de requisitos. Tu tarea es formalizar una especificación
escrita a mano, no diseñar la solución ni elegir tecnologías.

CONTEXTO
ProductChat es un chatbot interno para Distribuidora Andes que permite consultar
en lenguaje natural el catálogo de productos y servicios de la empresa. El
catálogo vive en catalogo_productos.xlsx y está sucio a propósito: SKU
duplicados con precios distintos, categorías escritas de varias formas,
atributos sin unidad estable, precios en texto y datos escondidos en columnas de
texto libre. La planilla no se va a limpiar.

ENTRADA
El archivo spec_servilleta_productchat.md, en la raíz del proyecto. Léelo
completo antes de empezar. Puedes consultar también catalogo_productos.xlsx para
comprobar los ejemplos que la servilleta menciona, y .specify/memory/constitution.md
para no contradecir sus principios.

SALIDA ESPERADA
La especificación formal del proyecto, spec.md, con estas secciones: objetivo y
contexto de negocio, usuarios, historias de usuario identificadas HU1, HU2…,
requisitos funcionales numerados RF1, RF2…, reglas de negocio numeradas RN1,
RN2…, criterios de aceptación numerados CA1, CA2…, casos límite y fuera de
alcance.

RESTRICCIONES
No decidas lenguajes, frameworks, modelos de IA, arquitectura ni bibliotecas: eso
corresponde al Plan. No resuelvas por tu cuenta ninguna de las preguntas abiertas
de la sección 9 de la servilleta. Antes de redactar, hazme las preguntas que
necesites para resolver cualquier ambigüedad.

CRITERIO DE ACEPTACIÓN
Cada criterio de aceptación de spec.md se puede comprobar abriendo
catalogo_productos.xlsx y usando la aplicación, sin leer código. Ningún requisito
supone que el catálogo esté limpio. Las preguntas abiertas siguen abiertas.
```

> Este prompt está en `prompts/02-specify.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

Dos detalles merecen atención. El prompt prohíbe explícitamente resolver las preguntas abiertas de la servilleta, porque ese es trabajo de Clarify y de una persona. Y su criterio de aceptación exige que ningún requisito suponga un catálogo limpio, que es la forma de comprobar que la precariedad del dato sobrevivió a la formalización.

A partir de esta información, Spec Kit puede generar la especificación formal que servirá como fuente de verdad para las siguientes etapas:

`Constitución → Especificación → Clarify → Plan → Tareas → Analyze → Implementación`

Si durante las etapas posteriores aparece una ambigüedad sobre **qué debe hacer el producto**, la solución no debería ser improvisarla en el código. Se debe regresar a la especificación, resolver la decisión y continuar el ciclo desde una definición más precisa.