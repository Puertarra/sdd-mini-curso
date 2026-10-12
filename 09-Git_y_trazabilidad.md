# El control de versiones en SDD

La primera parte terminó con ProductChat funcionando. A partir de aquí el producto empieza a cambiar, y eso plantea una pregunta que hasta ahora no hacía falta responder: **¿cómo demuestras que una línea de código existe por un requisito?**

En la primera parte afirmamos muchas veces que SDD es trazable, que cada parte del código se puede seguir hasta la especificación que la pidió. Sin control de versiones, esa afirmación es retórica: puedes decirla, pero no comprobarla. Git la vuelve verificable, porque guarda juntos, en el mismo commit, el requisito y el código que lo implementa.

Este módulo usa Git solo en lo que SDD necesita. No es un curso de Git: bastan una rama por Spec, commits por etapa y un archivo de contexto para el agente. Si Git es nuevo para ti, o necesitas recordar un comando, el anexo [`A-Mini_manual_de_Git.md`](A-Mini_manual_de_Git.md) reúne todos los que usa el curso, con ejemplos de ProductChat.

## Qué se versiona

En un proyecto SDD se versionan juntos el código y los documentos que explican por qué el código es así. Para ProductChat:

```text
ProductChat/
├── .specify/memory/constitution.md
├── specs/
│   ├── 001-consulta-catalogo/      ← SPEC001, primera parte
│   └── 002-pagina-inicio/          ← SPEC002, módulo 08
├── src/
├── tests/
├── catalogo_productos.xlsx
├── spec_servilleta_productchat.md
├── CLAUDE.md
└── .gitignore
```

Los nombres de las carpetas de `specs/` los genera Spec Kit a partir de la descripción de cada Spec; los tuyos pueden variar.

Hay un archivo que conviene mirar dos veces: el catálogo. En un proyecto real, los datos de un cliente no se suben a un repositorio. Aquí sí, por dos razones. El catálogo es ficticio. Y los criterios de aceptación de la Especificación citan sus SKU: si el archivo cambia sin quedar registrado, `CA1` deja de ser verificable y nadie sabe por qué. Versionarlo es lo que mantiene honestos a los criterios.

## Preparar el repositorio

Spec Kit 1.0.8 **no crea un repositorio Git** al ejecutar `specify init`. Si no lo creaste tú al terminar la primera parte, este es el momento:

```text
ROL
Actúa como responsable del repositorio de ProductChat. Preparas el control de
versiones; no modificas código ni artefactos.

CONTEXTO
ProductChat ya está implementado: la primera parte del curso terminó con su
código, sus artefactos SDD, catalogo_productos.xlsx y
spec_servilleta_productchat.md dentro del proyecto. Spec Kit 1.0.8 no crea un
repositorio Git al inicializar un proyecto.

ENTRADA
La raíz del proyecto ProductChat.

SALIDA ESPERADA
Un repositorio Git con la rama main, un .gitignore revisado y un primer commit
que incluya el código, las pruebas, .specify/memory/constitution.md, la carpeta
specs/, catalogo_productos.xlsx y spec_servilleta_productchat.md.

RESTRICCIONES
Si Git ya está inicializado, no lo reinicialices: informa su estado. Excluye
entornos virtuales, cachés y archivos generados. No excluyas el catálogo: sus
SKU son la referencia de los criterios de aceptación.

CRITERIO DE ACEPTACIÓN
git status no muestra cambios pendientes, y git show --stat HEAD lista el
catálogo y los archivos de specs/.
```

> Este prompt está en `prompts/09.1-git-init.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

Si prefieres hacerlo a mano, son tres comandos:

```bash
git init -b main
git add .
git commit -m "ProductChat: base implementation with SDD artifacts"
```

Antes del `add`, revisa el `.gitignore`: el entorno virtual y las cachés quedan fuera; el catálogo, dentro.

## Una rama por Spec

Cada Spec nueva se desarrolla en su propia rama y se integra en `main` solo cuando está verificada. La convención del curso es que la rama se llame igual que la carpeta de la Spec:

```bash
git switch -c 003-filtros-catalogo
```

Es una convención nuestra, no de Spec Kit. En la versión 1.0.8, `/speckit-specify` **no crea ramas**: eso solo ocurre si instalas la extensión `git`, que dejamos fuera del curso. Tampoco las necesita para funcionar: Spec Kit recuerda cuál es la Spec activa en `.specify/feature.json`, no por el nombre de la rama. Por eso la rama la creas tú, antes de ejecutar `/speckit-specify`.

Dentro de la rama, un commit por cada avance con sentido propio, que es lo que después permite reconstruir el razonamiento:

```text
SPEC003: specify and clarify filters      ← spec.md
SPEC003: plan and tasks                   ← plan.md, tasks.md
SPEC003: implement and test filters       ← código, pruebas y artefactos
```

## El contexto técnico del agente

El agente empieza cada sesión sin memoria. Sabe leer la Constitución y las Specs, pero no sabe dónde vive el normalizador del catálogo ni con qué comando se ejecutan las pruebas. Esa información va en un archivo de contexto en la raíz del proyecto: `CLAUDE.md` en Claude Code, `AGENTS.md` en Codex.

No es otro lugar para escribir reglas. Cada cosa tiene su sitio:

| Documento | Responde a | Ejemplo en ProductChat |
|---|---|---|
| Constitución | ¿Qué reglas se respetan siempre? | La inconsistencia del catálogo se muestra, no se esconde |
| Especificación | ¿Qué hace esta capacidad? | Con precio máximo 34.990, `PER-4001` aparece |
| Plan | ¿Cómo se construye esta capacidad? | Los filtros se aplican sobre el catálogo ya normalizado |
| `CLAUDE.md` | ¿Cómo se trabaja en este repositorio? | El normalizador está en `src/catalog/`; las pruebas se ejecutan con `pytest` |

Si una línea de `CLAUDE.md` podría estar en la Constitución, está en el lugar equivocado.

```text
ROL
Actúa como mantenedor técnico de ProductChat. Documentas cómo se trabaja en el
repositorio; no cambias el comportamiento del sistema.

CONTEXTO
ProductChat tiene Constitución, especificaciones y planes, pero el agente
empieza cada sesión sin saber cómo está organizado el código ni cómo se prueba.
Esa información vive en un archivo de contexto: CLAUDE.md en Claude Code,
AGENTS.md en Codex.

ENTRADA
.specify/memory/constitution.md, los plan.md de specs/ y el código del proyecto.

SALIDA ESPERADA
CLAUDE.md (o AGENTS.md) en la raíz, de una página como máximo, con: propósito
del sistema en dos líneas, stack vigente, organización del código, dónde se
normaliza el catálogo, comandos comprobados para ejecutar y probar, y la
convención de ramas y commits por Spec.

RESTRICCIONES
Referencia la Constitución y la carpeta specs/; no copies su contenido. Si no
pudiste comprobar un comando, márcalo como no verificado. No inventes datos.

CRITERIO DE ACEPTACIÓN
Cada comando del archivo se ejecuta sin error en el proyecto, y el archivo no
repite ninguna regla que ya esté en la Constitución.
```

> Este prompt está en `prompts/09.2-contexto-agente.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

## El ciclo con Git: los filtros de SPEC003

Con el repositorio listo, recorremos el ciclo completo para una capacidad nueva. El vendedor quiere filtrar por categoría y precio máximo. Es SPEC003, porque SPEC002 fue la página de inicio del módulo 08.

```text
main ──┬────────────────────────────────────────────┬── main
       │                                       merge │
       └── 003-filtros-catalogo                      │
             Especificación → Clarify  → commit      │
             Plan → Tareas → Analyze   → commit      │
             Implementación → pruebas  → commit ─────┘
```

Antes de empezar, `git status` debe estar limpio: un cambio pendiente de otra cosa terminaría mezclado en los commits de SPEC003.

### Especificar

```text
/speckit-specify

ROL
Actúa como analista de requisitos. Especificas una capacidad nueva de
ProductChat; no la diseñas ni la implementas.

CONTEXTO
ProductChat consulta catalogo_productos.xlsx, un catálogo con categorías
escritas de varias formas, precios en texto o vacíos y SKU duplicados con
precios en conflicto. La consulta básica (SPEC001) y la página de inicio
(SPEC002) ya están implementadas. El vendedor necesita ahora filtrar por
categoría y precio máximo.

ENTRADA
.specify/memory/constitution.md, la spec.md de SPEC001 y catalogo_productos.xlsx.

SALIDA ESPERADA
La spec.md de la nueva Spec, con historias de usuario, requisitos funcionales,
reglas de negocio, criterios de aceptación y casos límite.

REGLAS YA DECIDIDAS
- Los filtros de categoría y precio máximo pueden combinarse.
- El precio máximo incluye el valor límite.
- La categoría usa las mismas equivalencias que la consulta básica.
- Un producto sin precio utilizable no cumple un filtro de precio: se excluye
  y la respuesta lo informa.
- Un SKU con precios en conflicto se informa como establece SPEC001: no se
  elige un precio en silencio.
- Un precio máximo negativo es una entrada inválida y se avisa.

RESTRICCIONES
No cambies el comportamiento de SPEC001. Fuera de alcance: ordenar resultados,
filtrar por otros atributos, exportar y modificar el catálogo.

CRITERIO DE ACEPTACIÓN
Los criterios citan SKU reales del catálogo. "Periféricos con precio máximo
34.990" devuelve PER-4001 y PER-4002, y "Notebooks con precio máximo
1.000.000" excluye NB-1009 informando que no tiene precio.
```

> Este prompt está en `prompts/09.3-filtros-specify.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

Los resultados esperados se comprueban contra el catálogo real:

| Consulta | Resultado esperado | Por qué |
|---|---|---|
| Periféricos, máximo 34.990 | `PER-4001`, `PER-4002` | `PER-4001` cuesta exactamente 34.990: prueba el límite inclusivo. Los dos están cargados como `Perifericos` y `Periféricos`, así que la consulta solo funciona si la categoría se normaliza |
| Notebooks, máximo 1.000.000 | `NB-1010`, `NB-1015`, y `NB-1001` con aviso de conflicto | Los dos precios de `NB-1001`, 899.990 y 949.990, cumplen el filtro. `NB-1009` queda fuera porque no tiene precio, y la respuesta lo dice |
| Periféricos, máximo -1 | Aviso de entrada inválida | Un precio negativo no es un filtro |

Ahora ejecuta Clarify con el prompt `prompts/03.2-clarify.txt` del módulo 03. Hay un caso que conviene que aparezca: con un máximo de 920.000, uno de los dos precios de `NB-1001` cumple el filtro y el otro no. La Spec dice que el conflicto se informa, pero no dice si el producto aparece. Si Clarify no lo detecta, lo encontrarás al implementar, y la sección siguiente explica qué hacer entonces.

Con la Spec clarificada, registra el avance:

```bash
git add specs/
git commit -m "SPEC003: specify and clarify filters"
```

### Planificar

```text
/speckit-plan

ROL
Actúa como arquitecto sobre una base de código que ya funciona. Propones el
cómo de SPEC003; la decisión final es mía.

CONTEXTO
ProductChat ya normaliza categorías, precios y conflictos de SKU al leer el
catálogo. Los filtros deben aplicarse sobre ese catálogo normalizado, no
rehacer la normalización.

ENTRADA
La spec.md de SPEC003, el plan.md y el data-model.md de SPEC001,
.specify/memory/constitution.md y CLAUDE.md.

SALIDA ESPERADA
El plan.md de SPEC003, que reutilice el stack y el modelo de datos de SPEC001
y describa solo lo que cambia: dónde se aplican los filtros, cómo se trata un
precio ausente y cómo se trata un SKU en conflicto frente a un precio máximo.

RESTRICCIONES
No sustituyas el stack ni el modelo de datos. Si cambia algún comando o
convención, incluye una tarea para actualizar CLAUDE.md, registrando solo lo
que sirva a futuras Specs.

CRITERIO DE ACEPTACIÓN
El plan enumera los componentes que no toca, y cada componente nuevo se
justifica con un requisito de SPEC003.
```

> Este prompt está en `prompts/09.4-filtros-plan.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

Después ejecuta Tareas y Analyze con los prompts `prompts/05.1-tasks.txt` y `prompts/06.1-analyze.txt`. Funcionan sin cambios, porque nombran los artefactos de la Spec activa y Spec Kit sabe cuál es por `.specify/feature.json`. Corrige lo que Analyze encuentre en el artefacto donde está el problema, y registra:

```bash
git add specs/
git commit -m "SPEC003: plan and tasks"
```

### Implementar e integrar

Implementa con `prompts/07.1-implement.txt`. Cuando las tres consultas de la tabla den el resultado esperado, registra el código, las pruebas y los artefactos actualizados en un mismo commit:

```bash
git add src/ tests/ specs/ CLAUDE.md
git diff --cached
git commit -m "SPEC003: implement and test filters"
```

`git diff --cached` muestra exactamente qué va a entrar en el commit. Es el último punto donde un cambio accidental se ve antes de quedar registrado.

Para integrar, vuelve a `main` e incorpora la rama:

```bash
git switch main
git merge 003-filtros-catalogo
```

Después del merge, ejecuta otra vez las pruebas. Que Git no haya reportado conflictos solo significa que los textos se combinaron; no garantiza que el comportamiento siga siendo correcto.

## Comprobar la trazabilidad

Ahora la trazabilidad se puede demostrar, no solo afirmar. Para el requisito "el precio máximo incluye el valor límite", la cadena completa es:

| Eslabón | Evidencia |
|---|---|
| Requisito | `spec.md` de SPEC003: un producto con precio igual al máximo cumple el filtro |
| Decisión | `plan.md`: la comparación es `precio <= máximo` |
| Tarea | `tasks.md`: implementar el filtro y verificar el límite |
| Prueba | `PER-4001`, de 34.990, aparece con precio máximo 34.990 |
| Commit | `SPEC003: implement and test filters` |

Git permite recorrer esa cadena hacia atrás desde cualquier punto:

```bash
git log --oneline -- specs/003-filtros-catalogo/
git log -p -- specs/003-filtros-catalogo/spec.md
```

El primero lista los commits que tocaron la Spec. El segundo muestra cómo cambió cada requisito y cuándo.

```text
ROL
Actúa como auditor de trazabilidad. Revisas; no modificas archivos.

CONTEXTO
SPEC003 está implementada e integrada en main. Hay que demostrar que un
requisito concreto llega desde la especificación hasta una prueba y un commit.

ENTRADA
La carpeta de SPEC003 en specs/ (spec.md, plan.md, tasks.md), el código, las
pruebas y el historial de Git.

SALIDA ESPERADA
Una tabla para el requisito "el precio máximo incluye el valor límite" con
cinco filas: requisito, decisión del plan, tarea, prueba y commit, cada una
con su ruta, nombre o hash real.

RESTRICCIONES
Cita solo rutas, nombres de prueba y hashes que existan. Si falta un eslabón,
márcalo como faltante en lugar de suponerlo.

CRITERIO DE ACEPTACIÓN
Puedo abrir cada ruta y ejecutar git show con cada hash, y la prueba citada
verifica que PER-4001, de 34.990, aparece con un precio máximo de 34.990.
```

> Este prompt está en `prompts/09.5-trazabilidad.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

## Modificar una Spec o crear otra

No todo cambio necesita una Spec nueva. El criterio del curso es simple: **si aclara algo de una Spec todavía abierta, se modifica esa Spec; si agrega una capacidad, o cambia algo ya entregado, se crea una nueva.**

| Situación en ProductChat | Qué hacer |
|---|---|
| Durante SPEC003 aparece el caso de `NB-1001` con máximo 920.000 | Precisar SPEC003 |
| `PER-4001` no aparece con máximo 34.990 | Corregir el código: el requisito ya es correcto |
| El vendedor quiere exportar los resultados filtrados | Nueva Spec que referencia SPEC003 |
| Se quiere cambiar cómo se muestra un conflicto de precio en todas las consultas | Nueva Spec: cambia un comportamiento ya entregado en SPEC001 |

El primer caso es el que dejamos pendiente al especificar. Así se precisa:

```text
ROL
Actúa como analista de requisitos sobre una Spec todavía en desarrollo.
Precisas un requisito; no amplías el alcance.

CONTEXTO
Durante la implementación de SPEC003 apareció un caso que la Spec no resuelve:
NB-1001 tiene dos precios en conflicto, 899.990 y 949.990, y con un precio
máximo de 920.000 uno cumple el filtro y el otro no. La decisión ya está
tomada: el producto aparece, con el aviso de conflicto e indicando cuál de sus
precios cumple el filtro.

ENTRADA
La spec.md, el plan.md y el tasks.md de SPEC003.

SALIDA ESPERADA
spec.md actualizado con esa regla y su criterio de aceptación, más una lista de
los cambios que exige en plan.md, tasks.md y las pruebas, separando lo ya hecho
de lo pendiente.

RESTRICCIONES
Conserva el identificador y la carpeta: no crees una Spec nueva. No borres
decisiones vigentes del plan ni tareas ya completadas.

CRITERIO DE ACEPTACIÓN
El diff solo toca la carpeta de SPEC003, y spec.md contiene un criterio
verificable con NB-1001 y un precio máximo de 920.000.
```

> Este prompt está en `prompts/09.6-precisar-spec-existente.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

Después de precisar la Spec, revisa los artefactos que dependen de ella. No regeneres el plan y las tareas desde cero: contienen decisiones y trabajo que siguen siendo válidos.

## Actividad práctica

Implementa SPEC003 sobre tu ProductChat de la primera parte, siguiendo el ciclo de este módulo.

La entrega es el repositorio, con:

| Qué | Cómo se comprueba |
|---|---|
| Coherencia | Spec, plan, tareas y código describen el mismo comportamiento |
| Historial | La rama tiene un commit por etapa, y cada uno referencia SPEC003 |
| Contexto | `CLAUDE.md` describe el proyecto actual y sus comandos funcionan |
| Verificación | Las tres consultas de la tabla dan el resultado esperado |
| Trazabilidad | La tabla de la sección anterior, completada con rutas y hashes reales |

Si tu proyecto acumula varias Specs, puedes agregar un `specs/README.md` con una línea por Spec: identificador, propósito y estado. Con tres Specs que no se pisan entre sí no hace falta; el módulo 10 muestra cuándo se vuelve necesario.

> **Para más adelante: la extensión `git` de Spec Kit.** Spec Kit 1.0.8 incluye una extensión opcional que automatiza parte de lo que en este módulo hiciste a mano. Se instala con `specify extension add git` y funciona mediante *hooks*, acciones que se ejecutan solas antes o después de cada operación: crea la rama `NNN-nombre` antes de cada `/speckit-specify` y, si la configuras, hace un commit antes o después de cada etapa. Es una funcionalidad avanzada, y conviene usarla solo cuando ya domines el flujo manual: crear la rama, decidir qué entra en cada commit y leer el historial resultante. Automatizar un proceso que todavía no entiendes produce un historial que tampoco sabrás leer. Ten en cuenta además que los mensajes automáticos, como `[Spec Kit] Add tasks`, no dicen de qué Spec son: habría que configurarlos para mantener la convención `SPEC003: …`. Y la extensión no resuelve el trabajo en paralelo del módulo 11: crea ramas, pero en una sola carpeta de trabajo.

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/deed.es) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
