# Los Reverse-Specs en SDD

Todo el curso partió de los requisitos: primero la Constitución y la Especificación, después el código. El primer trabajo de un ingeniero casi nunca es así. Lo normal es llegar a un sistema que ya funciona, que alguien usa todos los días y que nadie documentó.

**Reverse-Specs** es el proceso inverso: reconstruir la especificación a partir del código, de su ejecución y de las personas que lo usan. Su objetivo no es criticar el sistema ni mejorarlo, sino entender y proteger lo que hace hoy antes de cambiarlo.

Conviene aclarar algo desde el principio. Para adoptar SDD en un proyecto existente no hace falta especificarlo entero: la guía de Spec Kit para proyectos existentes recomienda capturar las reglas que importan y aplicar el ciclo al siguiente cambio acotado. Aquí lo hacemos completo porque es el ejercicio, y porque es la mejor forma de cerrar el curso: obliga a usar todo lo anterior en la dirección contraria.

## Invertir la dirección

```text
Código existente → Observación y evidencia → Reglas reconstruidas
                                                    ↓
                                     Confirmación con el negocio
                                                    ↓
                               Pruebas de caracterización y Spec
```

Durante todo el proceso hay tres preguntas que no se deben mezclar: **qué hace el sistema**, **por qué se usa así** y **qué debería cambiar**. La ejecución responde la primera. Las personas responden la segunda. La tercera queda para después, como propuesta separada.

Un comportamiento observado no es automáticamente un requisito. Y un requisito deseado no describe necesariamente lo que el sistema hace.

## El sistema a estudiar

El sistema es el ProductChat de un compañero. Cada estudiante recibe la versión de otro, terminada al cierre del módulo 11, sin `specs/` y sin la servilleta. Así nadie tiene a mano los requisitos del sistema que estudia.

La copia se entrega **sin historial**: si llegara con su historial de Git, las Specs eliminadas seguirían visibles en commits anteriores. Quien la prepara, el docente o el propio compañero, la exporta así desde el repositorio original:

```bash
git archive --format=tar --prefix=productchat-recibido/ HEAD | tar -x -C ..
cd ../productchat-recibido
rm -rf specs spec_servilleta_productchat.md
```

Y quien la recibe registra esa versión como punto de partida, anotando el commit de origen:

```bash
git init -b main
git add .
git commit -m "Received snapshot of ProductChat (source commit abc1234)"
```

Hay una trampa en este ejercicio, y conviene nombrarla. Tú conoces las Specs que tu compañero tenía que cumplir: son las mismas que implementaste tú. La tentación será escribir lo que el sistema *debería* hacer. Pero un reverse-spec describe lo que el código **hace**, y es precisamente en las diferencias donde aparece el valor del ejercicio.

Fuera del aula, el sistema rara vez viene de un compañero que siguió el mismo curso, y la pregunta central pasa a ser qué hace de referencia. Hay dos situaciones típicas.

La primera es un proyecto propio hecho con *vibe coding*: software construido pidiéndole cambios a un agente, sin escribir una especificación (el término lo acuñó Andrej Karpathy en 2025). Ahí tienes acceso a parte de la historia: tu memoria, la conversación con el agente y, si los hubo, los commits. Pero ninguna de esas fuentes es confiable por sí sola. La memoria mezcla lo que querías con lo que quedó. Los prompts registran lo que pediste, no lo que el agente hizo. Y la intención probablemente cambió a mitad de camino sin quedar anotada. El riesgo es escribir la Spec del sistema que imaginaste. La regla es la de este módulo: lo que el sistema hace lo dicen las pruebas de caracterización; tus recuerdos y tus prompts son evidencia de intención, y se registran aparte, como una entrevista.

La segunda es un proyecto ajeno. Ahí la intención solo llega a través de personas o de rastros como el historial y los tickets. Si no hay a quién preguntar, el comportamiento observado es la única referencia, y cada regla queda como observada y no confirmada. El riesgo es el inverso: tomar por requisito lo que en realidad es un error, como el `NB-1009` mostrado a $0, que encontrarás más adelante en este módulo.

En ninguno de los dos casos tendrás, al final, unas Specs originales contra las cuales comparar tu reconstrucción. Esa comparación es un lujo del ejercicio de clase. En la práctica, la Spec inversa se valida de otra forma: la revisa quien usa o encargó el sistema, y las pruebas de caracterización pasan sobre la versión recibida.

Los documentos del reconocimiento van en una carpeta `reconocimiento/` en la raíz, no dentro de `specs/`. Spec Kit numera sus Specs según lo que encuentra en `specs/`, y no conviene mezclar en esa carpeta material que no es una Spec.

## Reconocer: del mapa al flujo

La exploración va de lo general a lo particular: primero qué módulos existen y qué hace cada uno, después el recorrido completo de una consulta concreta. Así se sabe dónde mirar antes de leer código línea a línea.

```text
ROL
Actúa como ingeniero que recibe un sistema sin documentación. Observas y
describes; no modificas código ni datos.

CONTEXTO
Recibiste una copia de ProductChat hecha por otra persona, sin specs/ y sin
la servilleta. Sabes que es un chatbot que consulta catalogo_productos.xlsx,
pero no sabes qué reglas implementa. El primer commit del repositorio es la
versión recibida.

ENTRADA
El código, las pruebas existentes, CLAUDE.md si existe y
catalogo_productos.xlsx.

SALIDA ESPERADA
Dos archivos en reconocimiento/: mapa.md, con los módulos, su
responsabilidad en lenguaje de negocio, el punto de entrada y los comandos
comprobados para ejecutar y probar; y flujo-consulta.md, con el recorrido de
la consulta "Periféricos con precio máximo 34.990" desde la pregunta hasta la
respuesta, indicando en qué función se lee, se normaliza, se filtra y se
responde.

RESTRICCIONES
Cada afirmación cita su fuente: ruta y líneas, o una ejecución con su entrada
y su salida. Distingue lo que ejecutaste de lo que solo leíste. No corrijas
nada que parezca un error: anótalo.

CRITERIO DE ACEPTACIÓN
Otra persona puede seguir flujo-consulta.md abriendo las rutas que cita y
llegar a la misma respuesta ejecutando la consulta.
```

> Este prompt está en `prompts/12.1-explorar.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

Cada afirmación del reconocimiento cita su fuente: una ruta con líneas, o una ejecución con su entrada y su salida. Los comentarios del código ayudan, pero pueden estar desactualizados: no son evidencia.

## Extraer reglas con evidencia

Una regla se escribe en términos del comportamiento, no del código. "Un producto con precio igual al máximo aparece" es una regla. "Se usa el operador `<=`" es una descripción de la implementación.

La forma más eficaz de extraer reglas es sondear el sistema con los casos difíciles del catálogo. En ProductChat ya sabemos cuáles son:

| Consulta | Qué observar |
|---|---|
| Periféricos, máximo 34.990 | ¿Aparece `PER-4001`, que cuesta exactamente 34.990? |
| Notebooks con 32 GB de RAM | ¿Cuáles devuelve? ¿Cuenta `NB-1003`, que solo es ampliable? ¿Y `NB-1009`, que no tiene stock? |
| Precio de `NB-1001` | ¿Uno o dos precios? ¿Avisa del conflicto? |
| Precio de `NB-1009` | ¿Vacío, 0 o "no disponible"? |
| Stock de `SRV-5001` | ¿"No aplica", "0" o "sin stock"? |
| RAM de `NB-1009` | ¿"32 GB", "32" o algo distinto? |
| Monitores, máximo 150.000 | ¿`MON-3001` y `MON-3004` por separado o fusionados? |

```text
ROL
Actúa como analista que reconstruye reglas de negocio desde el
comportamiento. Describes lo que el sistema hace, no lo que debería hacer.

CONTEXTO
Ya existe el mapa de ProductChat y el flujo de una consulta. Ahora hay que
extraer las reglas que el sistema aplica sobre un catálogo lleno de casos
ambiguos: precios vacíos o en texto, SKU duplicados, categorías con varias
grafías, unidades de memoria distintas y stock en texto.

ENTRADA
reconocimiento/mapa.md, reconocimiento/flujo-consulta.md, el código y
catalogo_productos.xlsx.

SALIDA ESPERADA
reconocimiento/reglas.md con reglas numeradas RN-01, RN-02… Cada una con
enunciado en lenguaje de negocio, SKU de ejemplo, salida observada, evidencia
y estado: observada (se ejecutó) o inferida (solo se leyó en el código).

QUÉ SONDEAR
Ejecuta al menos estas consultas y registra la respuesta exacta: Periféricos
con precio máximo 34.990; notebooks con 32 GB de RAM; precio de NB-1001;
precio de NB-1009; stock de SRV-5001; RAM de NB-1009; Monitores con precio
máximo 150.000.

RESTRICCIONES
No atribuyas intención a partir del código. Si un resultado parece un error,
regístralo igual como regla observada y anótalo para el registro de
anomalías.

CRITERIO DE ACEPTACIÓN
Cada consulta de la lista tiene su salida registrada, y cada regla cita al
menos un SKU que la reproduce.
```

> Este prompt está en `prompts/12.2-reglas.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

Cada regla queda marcada como **observada**, si se ejecutó, o **inferida**, si solo se leyó en el código. La diferencia importa: una regla inferida puede estar equivocada sin que nadie lo note.

## Anomalías: lo raro no se corrige, se registra

El reconocimiento siempre encuentra comportamientos raros. La reacción natural es corregirlos. Es un error: antes hay que saber si alguien depende de ellos.

Supón que en la versión recibida el precio vacío de `NB-1009` aparece como \$0. Parece un defecto, y probablemente lo es. Pero si los vendedores aprendieron a ignorar los productos en \$0, "corregirlo" cambia lo que ven todos los días sin que nadie lo haya pedido. La anomalía se registra, se pregunta, y recién después se decide.

| Clasificación | Qué se hace en esta fase |
|---|---|
| Intencional | Se especifica como regla |
| Accidental | Se describe como comportamiento observado, no como requisito |
| Defecto | Se registra con su evidencia y sus dependencias, y se propone una corrección futura |
| Pendiente | Se deja abierta la pregunta |

Una fuente frecuente de anomalías son los **caminos paralelos**: la misma regla implementada dos veces. En ProductChat es fácil que ocurra con la RAM de `NB-1009`, cuyo valor en la planilla es solo `32`. La búsqueda por características puede interpretarlo como 32 GB, mientras la respuesta a "¿cuánta RAM tiene el MSI Modern 15?" muestra "32" sin unidad. Dos caminos, dos interpretaciones del mismo dato.

```text
ROL
Actúa como auditor de comportamiento. Clasificas lo extraño; no lo corriges.

CONTEXTO
Las reglas de ProductChat ya están registradas. Algunas parecen errores, y
algunas reglas se aplican de forma distinta según el camino: la misma
característica puede interpretarse de una manera al buscar y de otra al
responder por un producto.

ENTRADA
reconocimiento/reglas.md, reconocimiento/flujo-consulta.md y el código.

SALIDA ESPERADA
reconocimiento/anomalias.md con entradas A-01, A-02… Cada una con
comportamiento, SKU que la reproduce, evidencia, impacto, clasificación
provisional (intencional, accidental, defecto o pendiente) y la pregunta que
la resolvería. Incluye los casos donde dos caminos tratan distinto el mismo
dato, por ejemplo la RAM de NB-1009 en una búsqueda y en una consulta de
atributo.

RESTRICCIONES
No cierres una clasificación sin evidencia; ante la duda, déjala pendiente.
No decidas qué camino es el correcto.

CRITERIO DE ACEPTACIÓN
Cada anomalía se reproduce con su SKU, y cada clasificación que no sea
"pendiente" cita su evidencia.
```

> Este prompt está en `prompts/12.3-anomalias.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

## Preguntar al negocio

El código dice qué hace el sistema, pero no por qué. La intención, los procesos manuales y las dependencias solo los conoce quien lo usa o lo mantiene. Cada pregunta separa el presente del futuro:

> En esta versión, `NB-1009` aparece con precio $0 porque su celda está vacía. ¿Hoy alguien usa ese $0 para algo, o lo ignoran? ¿Qué quieren que pase en adelante?

En clase, la entrevista puede ser simulada: un compañero hace de vendedor o de encargado del catálogo. Eso es válido, siempre que cada respuesta quede marcada como simulación. Una respuesta generada por el agente no cuenta como respuesta del negocio.

```text
ROL
Actúa como entrevistador. Preparas preguntas y registras respuestas; no
respondes por el negocio.

CONTEXTO
El registro de anomalías de ProductChat tiene preguntas que el código no
puede responder: si un comportamiento es intencional, si alguien depende de
él y qué se quiere conservar. Las responde quien usa o mantiene el sistema.

ENTRADA
reconocimiento/anomalias.md y reconocimiento/reglas.md.

SALIDA ESPERADA
reconocimiento/entrevista.md con las preguntas ordenadas por impacto, cada
una dirigida a un rol (vendedor, encargado del catálogo o mantenedor), con
las respuestas recibidas, la fecha y el cambio de clasificación que provoca
en anomalias.md.

RESTRICCIONES
Haz una pregunta por vez y espera la respuesta. Cada pregunta separa qué pasa
hoy de qué se quiere cambiar. Si la entrevista es simulada en clase, marca
cada respuesta como SIMULACIÓN. No inventes respuestas, personas ni fechas.

CRITERIO DE ACEPTACIÓN
Cada pregunta tiene una respuesta registrada o queda marcada como pendiente,
y anomalias.md refleja los cambios de clasificación.
```

> Este prompt está en `prompts/12.4-entrevista.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

## Pruebas de caracterización

Una **prueba de caracterización** registra lo que el sistema hace hoy ante una entrada conocida, para detectar cualquier cambio posterior. No pregunta si el resultado es correcto, sino si sigue siendo el mismo. Por eso puede, y a veces debe, proteger un comportamiento que parece un error: hasta que alguien decida cambiarlo, es el comportamiento del sistema.

En Python, con `pytest`, tienen esta forma:

```python
"""Characterization tests for the received ProductChat snapshot.

They pin down what this version does today, not what it should do.
Expected values were observed on the baseline commit; change them only
through a recorded decision.
"""
from productchat.catalog import load_catalog, search, get_product  # adapt to the snapshot

CATALOG = load_catalog("catalogo_productos.xlsx")


def test_rn01_max_price_is_inclusive():
    """RN-01: a product priced exactly at the maximum is returned."""
    result = search(CATALOG, category="periféricos", max_price=34_990)
    assert {item.sku for item in result} == {"PER-4001", "PER-4002"}


def test_a01_missing_price_shown_as_zero():
    """A-01: NB-1009 has no price and this version shows it as 0. Kept on purpose."""
    assert get_product(CATALOG, "NB-1009").price == 0
```

Los nombres de módulos y funciones son de ejemplo: cada versión recibida tiene los suyos. Lo que no cambia son tres reglas.

**Los valores esperados salen de la observación.** Se ejecutó la versión recibida, se anotó el resultado, y ese resultado va escrito tal cual en la prueba. Nunca se calcula dentro de la prueba, ni llamando a la misma función ni reproduciendo su algoritmo.

**Se prueba la capa determinista, no la conversación.** Las pruebas apuntan a la búsqueda y la normalización, no al texto que redacta el modelo de lenguaje, que puede cambiar de una ejecución a otra. Esa separación se decidió en el Plan del módulo 04, y aquí rinde.

**Una prueba que falla no se arregla cambiando el valor esperado.** Primero se averigua qué cambió: el entorno, los datos o el comportamiento.

Si una salida es grande y completamente determinista, existe una variante llamada *golden master*: se guarda la salida completa revisada en un archivo y las pruebas comparan contra él. Para ProductChat no hace falta.

```text
ROL
Actúa como desarrollador de pruebas. Fijas el comportamiento actual; no lo
corriges ni lo juzgas.

CONTEXTO
Antes de cambiar ProductChat hay que proteger lo que hace hoy, incluidos sus
comportamientos raros. Una prueba de caracterización registra la salida que
la versión recibida produce ante una entrada conocida.

ENTRADA
reconocimiento/reglas.md, reconocimiento/anomalias.md, el código y
catalogo_productos.xlsx.

SALIDA ESPERADA
tests/test_characterization.py con una prueba por cada regla RN y cada
anomalía A ejecutable, usando el punto de entrada real del sistema y el
catálogo real. Cada prueba indica en su docstring la regla o anomalía que
protege.

RESTRICCIONES
Los valores esperados salen de ejecutar la versión recibida: no los calcules
dentro de la prueba ni reproduzcas el algoritmo. Si la respuesta del chatbot
incluye texto generado, compara solo los datos estables (SKU, precios,
avisos), no el texto completo. Usa el runner existente y no cambies código de
producción.

CRITERIO DE ACEPTACIÓN
Todas las pruebas pasan sobre la versión recibida, y ninguna pasaría si el
comportamiento que protege cambiara.
```

> Este prompt está en `prompts/12.5-caracterizacion.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

## La Spec inversa

Con el reconocimiento, la entrevista y las pruebas, ya se puede escribir la Spec. Es distinta de las del resto del curso en un punto clave: separa cuatro cosas que en una Spec normal no conviven.

| Parte | Qué contiene |
|---|---|
| Requisitos vigentes | Lo que el negocio confirmó que el sistema debe seguir haciendo |
| Comportamientos observados | Lo que el sistema hace y nadie confirmó todavía |
| Anomalías conocidas | Lo que parece un error, con su referencia al registro |
| Cambios propuestos | Lo que se quiere cambiar en adelante, separado del estado actual |

Spec Kit sirve para redactarla, pero el ciclo se detiene ahí: no hay Plan, ni Tareas, ni Implementación, porque el sistema ya existe.

```text
/speckit-specify

ROL
Actúa como analista que documenta un sistema existente. Describes lo que
hace hoy; no especificas una funcionalidad nueva.

CONTEXTO
ProductChat está en producción sin especificación. El reconocimiento ya
reconstruyó sus reglas, sus anomalías y las decisiones de la entrevista, y
las pruebas de caracterización protegen su comportamiento actual.

ENTRADA
Los archivos de reconocimiento/, tests/test_characterization.py y
catalogo_productos.xlsx.

SALIDA ESPERADA
Una spec.md de la consulta del catálogo con cuatro partes separadas:
requisitos vigentes confirmados en la entrevista, comportamientos observados
que nadie ha confirmado, anomalías conocidas con su referencia A, y cambios
propuestos para el futuro. Los criterios de aceptación citan pruebas de
caracterización.

RESTRICCIONES
No conviertas una anomalía accidental en requisito. No declares confirmado
lo que solo respondió una simulación. No generes plan, tareas ni código.

CRITERIO DE ACEPTACIÓN
Cada requisito vigente remite a una prueba que pasa, y cada anomalía de
anomalias.md aparece en la Spec.
```

> Este prompt está en `prompts/12.6-spec-inversa.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

Después de generarla, revisa con `git diff` que solo cambiaron documentos. Una fase de reconocimiento que modifica el código dejó de ser reconocimiento.

## El Spec Pack

El **Spec Pack** es el entregable final: todo lo necesario para que otra persona entienda el sistema y pueda cambiarlo con seguridad, reunido y enlazado desde un solo documento.

```text
SPEC-PACK.md                    ← punto de entrada
reconocimiento/
├── mapa.md
├── flujo-consulta.md
├── reglas.md
├── anomalias.md
└── entrevista.md
specs/
└── 001-…/spec.md               ← la Spec inversa
tests/
└── test_characterization.py
```

```text
ROL
Actúa como responsable de la entrega. Ensamblas lo que existe; no agregas
evidencia nueva.

CONTEXTO
El reconocimiento de ProductChat terminó. Hay que entregar a otra persona
todo lo necesario para entender el sistema y cambiarlo con seguridad.

ENTRADA
Los archivos de reconocimiento/, la Spec inversa en specs/ y
tests/test_characterization.py.

SALIDA ESPERADA
SPEC-PACK.md en la raíz, con: versión estudiada y alcance; un resumen de una
página para un lector no técnico; un índice con enlaces a cada documento,
la Spec y las pruebas; una tabla de trazabilidad RN → A → prueba; los
comandos para ejecutar las pruebas; y lo que quedó sin verificar.

RESTRICCIONES
Enlaza solo archivos que existan. No declares el sistema completamente
documentado si el alcance no lo permite.

CRITERIO DE ACEPTACIÓN
Todos los enlaces abren, los comandos de prueba se ejecutan, y SPEC-PACK.md
declara explícitamente qué quedó fuera del alcance o sin verificar.
```

> Este prompt está en `prompts/12.7-spec-pack.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

Un buen Spec Pack es honesto sobre sus límites. Haber probado la búsqueda no equivale a haber probado todo el chatbot, y el documento debe decirlo.

## Actividad final del curso

Haz el reconocimiento completo del ProductChat de tu compañero: mapa y flujo, reglas, anomalías, entrevista, pruebas de caracterización, Spec inversa y Spec Pack. Durante toda la actividad, el código de producción no cambia.

Al terminar, y solo entonces, compara tu reconstrucción con las Specs originales de tu compañero. Las diferencias son de dos tipos, y las dos enseñan algo. Si tu reconstrucción no captó una regla que estaba en la Spec, el código no la deja ver con claridad. Si el código hace algo que la Spec no decía, encontraste un drift, como los del módulo 10, desde el otro lado.

| Qué se entrega | Cómo se comprueba |
|---|---|
| Preservación | El primer commit es la versión recibida, y el código de producción no cambió después |
| Reconstrucción | Cada regla cita un SKU y su evidencia, observada o inferida |
| Clasificación | Lo observado, lo confirmado y lo propuesto aparecen separados |
| Caracterización | Las pruebas pasan sobre la versión recibida y cada una referencia su regla o anomalía |
| Entrega | El Spec Pack enlaza todo y declara qué quedó sin verificar |
| Comparación | Un párrafo final con las diferencias entre tu reconstrucción y las Specs originales |

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/deed.es) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
