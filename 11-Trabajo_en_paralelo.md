# El trabajo en paralelo en SDD

Hasta aquí, un agente hizo todo el trabajo, una Spec detrás de otra. Con dos capacidades independientes, la tentación natural es abrir dos sesiones y dejar que cada agente avance por su lado. Puede funcionar, pero conviene saber de antemano dónde se rompe.

El riesgo principal no son los conflictos que detecta Git. Son los que Git no puede ver: dos agentes que entienden de forma distinta la misma regla del catálogo y producen resultados que se contradicen, aunque cada uno por separado pase sus pruebas. Este módulo trata de cómo evitarlo.

Una advertencia antes de empezar: agregar agentes no garantiza terminar antes. Paralelizar ahorra tiempo solo si las tareas son realmente independientes y si coordinar e integrar cuesta menos que hacerlas en serie. En un proyecto del tamaño de ProductChat, la lección vale más que el ahorro.

## Qué se puede paralelizar y qué no

La independencia no se juzga por el nombre de las tareas, sino por lo que comparten. Dos tareas con nombres distintos pueden depender de la misma regla, el mismo componente o el mismo archivo:

| Trabajo | ¿En paralelo? |
|---|---|
| Decidir cómo se trata un SKU con dos precios | No. Es una regla común: se decide una vez, antes de repartir |
| Redactar dos Specs que consumen el mismo resultado | Sí, con una revisión cruzada antes de implementar |
| Implementar dos capacidades en archivos separados | Sí, si cada agente trabaja en su propio directorio |
| Modificar el normalizador del catálogo | No. Un solo responsable, porque todo depende de él |

## Las dos capacidades

ProductChat va a sumar dos formas de sacar el resultado de una consulta fuera del chat:

| Spec | Capacidad | Responsable |
|---|---|---|
| SPEC005 | Exportar el resultado a CSV, para trabajarlo en una planilla | Agente A |
| SPEC006 | Resumir el resultado en texto, para pegarlo en un correo al cliente | Agente B |
| | Reglas comunes, revisión cruzada e integración | Tú, como coordinador |

Las dos capacidades consumen el mismo resultado, y ese resultado ya aplica reglas que pertenecen a otras Specs: los conflictos de precio y los duplicados son de SPEC001, los filtros de SPEC003 y el stock de SPEC004. Ninguna de las dos capacidades nuevas debería redefinirlas. Ese es el punto donde suele romperse el trabajo en paralelo, y por eso lo primero es dejarlo por escrito:

```text
ROL
Actúa como coordinador de un equipo de dos agentes. Repartes trabajo y
asignas responsables; no modificas archivos.

CONTEXTO
ProductChat va a sumar dos capacidades que consumen el mismo resultado de
consulta: exportarlo a CSV (SPEC005) y resumirlo en texto para un correo al
cliente (SPEC006). Ese resultado ya aplica reglas definidas en otras Specs:
conflictos de precio y duplicados (SPEC001), filtros (SPEC003) y stock
(SPEC004). Las dos capacidades las implementarán agentes distintos.

ENTRADA
Las Specs de specs/, el código, las pruebas y CLAUDE.md.

SALIDA ESPERADA
Dos tablas. La primera, de reglas comunes: cada regla que ambas capacidades
consumen y la única Spec dueña de esa regla. La segunda, de trabajo: Spec,
agente responsable, rama, archivos propios, archivos compartidos con su
responsable único, dependencias y pruebas necesarias.

RESTRICCIONES
Distingue entre leer una Spec y tener disponible su código. Si un componente
común necesita cambios, asígnalo a un solo agente.

CRITERIO DE ACEPTACIÓN
Cada regla común tiene exactamente una Spec dueña, y ningún archivo tiene dos
responsables.
```

> Este prompt está en `prompts/11.1-distribucion.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

## Por qué las Specs las crea el coordinador

Parece natural que cada agente cree su propia Spec. Con Spec Kit 1.0.8 eso trae dos problemas, y los dos son concretos.

El primero es la **Spec activa**. Spec Kit guarda en `.specify/feature.json` cuál es la Spec en curso, y hay un solo archivo por directorio de trabajo. Si dos agentes ejecutan `/speckit-specify` en el mismo directorio, el segundo pisa la Spec activa del primero, y los comandos siguientes del primer agente terminan trabajando sobre la Spec equivocada.

El segundo es la **numeración**. Spec Kit asigna a cada Spec el siguiente número libre que encuentra en la carpeta `specs/` de la rama donde se ejecuta. Una rama creada desde `main` no contiene las Specs que viven en otras ramas, así que dos Specs creadas en ramas distintas reciben el mismo número, 005. El choque aparece recién al integrar.

La salida más simple es que el coordinador cree las dos Specs, una después de la otra, en el directorio principal, y que fije en el prompt el directorio de cada una en lugar de dejar que Spec Kit elija el número. La versión 1.0.8 respeta el directorio si se lo indicas como `SPECIFY_FEATURE_DIRECTORY`, y los dos prompts lo hacen. Cada Spec, en su rama:

```bash
git switch -c 005-exportar-csv
```

```text
/speckit-specify

ROL
Actúa como analista de requisitos. Especificas una capacidad que consume
reglas ya definidas; no las redefines.

CONTEXTO
El vendedor quiere descargar en CSV el resultado de una consulta de
ProductChat para trabajarlo en una planilla. El resultado ya aplica las
reglas de SPEC001 (conflictos de precio y duplicados), SPEC003 (filtros) y
SPEC004 (stock).

ENTRADA
Las spec.md de SPEC001, SPEC003 y SPEC004, y catalogo_productos.xlsx.

SALIDA ESPERADA
La spec.md de SPEC005, en el directorio specs/005-exportar-csv (úsalo como
SPECIFY_FEATURE_DIRECTORY): un CSV con una fila por producto del resultado y las
columnas SKU, nombre, categoría, precio, estado de stock y aviso.

RESTRICCIONES
Exporta exactamente el resultado mostrado: no vuelvas a consultar el catálogo
ni a seleccionar productos. Si una regla de las Specs anteriores no alcanza
para decidir cómo exportar un caso, regístralo como pregunta abierta; no
inventes una regla nueva. Un resultado vacío no genera archivo y se avisa.
Fuera de alcance: otros formatos, enviar el archivo y modificar el catálogo.

CRITERIO DE ACEPTACIÓN
La Spec define cómo se exportan NB-1001, con dos precios en conflicto, y
SRV-5001, cuyo stock no aplica, citando la regla de origen de cada caso.
```

> Este prompt está en `prompts/11.2-exportacion-specify.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

Clarifica la Spec con `prompts/03.2-clarify.txt`, regístrala y vuelve a `main` para la segunda:

```bash
git add specs/
git commit -m "SPEC005: specify CSV export"
git switch main
git switch -c 006-resumen-correo
```

```text
/speckit-specify

ROL
Actúa como analista de requisitos. Especificas una capacidad que consume
reglas ya definidas; no las redefines.

CONTEXTO
El vendedor responde por correo a sus clientes y hoy copia a mano los datos
que le da ProductChat. Quiere un resumen en texto del resultado de una
consulta, listo para pegar. El resultado ya aplica las reglas de SPEC001
(conflictos de precio y duplicados), SPEC003 (filtros) y SPEC004 (stock).

ENTRADA
Las spec.md de SPEC001, SPEC003 y SPEC004, y catalogo_productos.xlsx.

SALIDA ESPERADA
La spec.md de SPEC006, en el directorio specs/006-resumen-correo (úsalo como
SPECIFY_FEATURE_DIRECTORY): un texto plano con una línea por producto del
resultado, con nombre, SKU, precio y disponibilidad, más los avisos que
correspondan.

RESTRICCIONES
Resume exactamente el resultado mostrado: no vuelvas a consultar el catálogo
ni a seleccionar productos. Si una regla de las Specs anteriores no alcanza
para decidir cómo presentar un caso, regístralo como pregunta abierta; no
inventes una regla nueva. Fuera de alcance: enviar el correo, redactar el
mensaje completo y modificar el catálogo.

CRITERIO DE ACEPTACIÓN
La Spec define cómo aparecen NB-1001, con dos precios en conflicto, y
MON-3001 y MON-3004, que son el mismo monitor bajo dos SKU, citando la regla
de origen de cada caso.
```

> Este prompt está en `prompts/11.3-resumen-specify.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

Clarifica y registra igual que la anterior. Con las dos Specs en sus ramas, ya se pueden revisar en paralelo.

## Conflictos de texto y conflictos de significado

Un **conflicto de texto** ocurre cuando Git no puede combinar los cambios de un archivo: dos ramas modificaron las mismas líneas. Git lo detecta y obliga a resolverlo.

Un **conflicto de significado** ocurre cuando dos Specs, o dos implementaciones, interpretan distinto la misma regla. Git no lo detecta, porque los archivos no se tocan. El merge sale limpio y el resultado está mal.

En ProductChat aparecen con facilidad, porque el catálogo está lleno de casos que obligan a interpretar:

| SKU | Situación | Si el CSV… | Y el resumen… |
|---|---|---|---|
| `NB-1001` | Dos precios en conflicto, 899.990 y 949.990 | exporta solo el primero | dice "precio por confirmar" |
| `MON-3001`, `MON-3004` | El mismo monitor bajo dos SKU | exporta las dos filas | muestra una sola línea "para no repetir" |
| `SRV-5001` | Servicio: el stock no aplica | escribe stock 0 | no menciona el stock |

En los tres casos, el cliente recibe un correo que no coincide con la planilla del vendedor. Y en los tres, una de las dos capacidades redefinió una regla que ya tenía dueño: RN5 y CL5 de SPEC001, y la regla de servicios de SPEC004.

Para revisar las dos Specs no hace falta cambiar de rama. Git permite leer un archivo de otra rama directamente:

```bash
git show 005-exportar-csv:specs/005-exportar-csv/spec.md
```

Eso sí, `git show` solo ve lo que está registrado en un commit: un cambio sin commit en otro directorio no aparece.

```text
ROL
Actúa como revisor de consistencia entre Specs. Detectas conflictos; no
editas documentos ni decides cómo resolverlos.

CONTEXTO
SPEC005 (exportar a CSV) y SPEC006 (resumen para correo) se redactaron por
separado y consumen el mismo resultado de consulta. Las reglas que ese
resultado aplica pertenecen a SPEC001, SPEC003 y SPEC004.

ENTRADA
Las spec.md de SPEC005 y SPEC006, leídas desde sus ramas con
git show RAMA:RUTA, y las spec.md de SPEC001, SPEC003 y SPEC004 en main.
Registra el commit de cada rama que revisas.

SALIDA ESPERADA
Una tabla de hallazgos con: concepto, qué dice cada Spec consumidora, qué
dice la Spec dueña de la regla, SKU que lo reproduce y propuesta de
resolución.

QUÉ BUSCAR
Reglas redefinidas en una Spec consumidora. Casos que una Spec trata y la
otra no. En especial: precios en conflicto, stock vacío, servicios y
productos duplicados bajo dos SKU.

RESTRICCIONES
No cambies de rama ni edites archivos. Distingue un conflicto confirmado de
un riesgo que todavía no se puede confirmar.

CRITERIO DE ACEPTACIÓN
Cada hallazgo se reproduce con un SKU del catálogo, y la propuesta de
resolución apunta a la Spec que debe cambiar.
```

> Este prompt está en `prompts/11.4-revision-cruzada.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

## Cada regla tiene un dueño

La resolución sigue un principio simple: **una regla común vive en una sola Spec, y las demás la referencian**. La Spec que la redefinió se corrige; la Spec dueña no cambia.

Si el borrador del CSV exporta solo el primer precio de `NB-1001`, el responsable de SPEC005 corrige su Spec para aplicar RN5 de SPEC001: los dos precios, con el aviso de conflicto. Puede hacerlo con Clarify o editando la Spec, y después revisa los efectos sobre su plan y sus pruebas, si ya existen. No toca SPEC006 ni SPEC001: cada agente corrige solo su propio documento.

Si al revisar descubres que las dos capacidades necesitan algo que el resultado de consulta todavía no entrega, por ejemplo un campo que indique el estado de stock ya interpretado, eso no lo resuelve ninguno de los dos agentes. Es un cambio en el componente común, y se hace primero, con un solo responsable, antes de que los consumidores lo usen.

## Planificar e implementar

Las dependencias determinan qué puede avanzar a la vez:

```text
SPEC001 + SPEC003 + SPEC004, integradas y verificadas en main
                        ↓
            Resultado de consulta común
                ┌───────┴───────┐
                ↓               ↓
           SPEC005           SPEC006
        exportar CSV      resumen correo
                └───────┬───────┘
                        ↓
           Integración y pruebas conjuntas
```

Con las Specs revisadas, cada agente planifica la suya:

```text
/speckit-plan

ROL
Actúa como arquitecto de una sola capacidad dentro de un equipo. Planificas
tu Spec; no tocas el trabajo del otro agente.

CONTEXTO
Trabajas en la rama y la carpeta que te asignó el coordinador. Tu capacidad
consume el resultado de consulta que ya construye ProductChat.

ENTRADA
La spec.md de tu Spec, revisada después de la revisión cruzada, la tabla de
distribución del coordinador, el plan.md de SPEC001 y CLAUDE.md.

SALIDA ESPERADA
El plan.md de tu Spec, que reutilice el stack y el resultado de consulta
existentes, liste los archivos que vas a crear o modificar y defina las
pruebas con los SKU de tus criterios de aceptación.

RESTRICCIONES
Modifica solo los archivos que la tabla te asigna. Si necesitas cambiar un
componente compartido, informa al coordinador en lugar de hacerlo. No
reimplementes la selección de productos.

CRITERIO DE ACEPTACIÓN
Todos los archivos del plan están en tu columna de la tabla de distribución.
```

> Este prompt está en `prompts/11.5-plan-agente.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

Después, cada uno sigue con Tareas, Analyze e Implementación usando `prompts/05.1-tasks.txt`, `prompts/06.1-analyze.txt` y `prompts/07.1-implement.txt`, sobre su propia Spec. Al terminar, entrega sus commits, los archivos que modificó, las pruebas que ejecutó y lo que quedó pendiente. Decir "terminé" sin esa evidencia no habilita la integración.

### Un directorio o dos

Una rama no es una copia de los archivos. Dos agentes que trabajan en el mismo directorio comparten los archivos y la Spec activa, y cualquier cambio de rama de uno afecta al otro.

Con un solo directorio, lo razonable es paralelizar la revisión, que es solo lectura, e implementar una capacidad después de la otra. Ya obtienes buena parte del beneficio: las Specs se revisaron juntas antes de escribir código.

Para implementar realmente en paralelo, cada agente necesita su propio directorio. Git lo permite con **worktrees**: carpetas distintas del mismo repositorio, cada una con su rama. Y como cada worktree tiene su propio `.specify/feature.json`, cada agente mantiene su propia Spec activa. Desde el directorio principal, en `main` y sin cambios pendientes:

```bash
git worktree add ../ProductChat-csv 005-exportar-csv
git worktree add ../ProductChat-resumen 006-resumen-correo
git worktree list
```

Cada agente abre su sesión en su carpeta. Es una ampliación opcional: la actividad se puede completar entera con un solo directorio.

## Integrar por orden de riesgo

Las dos ramas no dependen una de la otra, así que el orden lo decide el riesgo. El resumen solo produce texto; la exportación escribe archivos en disco, lo que agrega nombres, rutas y permisos que pueden fallar. Conviene integrar primero lo de menor riesgo, verificar, y recién entonces lo siguiente:

```bash
git switch main
git merge 006-resumen-correo
```

Ejecuta las pruebas. Si pasan:

```bash
git merge 005-exportar-csv
```

Y otra vez las pruebas, ahora con las dos capacidades juntas. Si el segundo merge produce conflictos, se resuelven según las Specs, conservando el trabajo válido de ambas ramas. Nunca eligiendo en bloque la versión de una, ni borrando pruebas para que pasen.

```text
ROL
Actúa como coordinador de la integración. Integras ramas ya verificadas, una
por vez; no amplías el alcance de ninguna.

CONTEXTO
SPEC005 y SPEC006 están implementadas en sus ramas, cada una con sus pruebas
aprobadas. Hay que llevarlas a main sin romper lo que ya funciona.

ENTRADA
Las ramas de SPEC005 y SPEC006, la tabla de distribución y los informes de
pruebas de cada agente.

SALIDA ESPERADA
Un orden de integración justificado por dependencias y por riesgo, la
integración rama por rama con las pruebas ejecutadas después de cada merge,
y un informe con commits, conflictos resueltos y pendientes.

RESTRICCIONES
Confirma que main está limpio antes de empezar. No integres la segunda rama
hasta que la primera pase las pruebas. Si aparece un conflicto, resuélvelo
según las Specs; no elijas en bloque la versión de una rama ni elimines
pruebas para que pasen.

CRITERIO DE ACEPTACIÓN
En main, la consulta "Notebooks con precio máximo 1.000.000" produce el
mismo resultado en el CSV y en el resumen: NB-1001 con sus dos precios y el
aviso de conflicto, y NB-1015.
```

> Este prompt está en `prompts/11.6-integracion.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

Si usaste worktrees, retíralos con `git worktree remove` solo cuando no tengan cambios pendientes.

## Actividad práctica

Reparte SPEC005 y SPEC006 entre dos sesiones de agente. Como coordinador, crea las Specs, ejecuta la revisión cruzada, resuelve los conflictos que encuentres y luego integra.

El resultado se verifica comparando las dos salidas para las mismas consultas:

| Consulta | CSV y resumen deben mostrar |
|---|---|
| Notebooks, máximo 1.000.000 | `NB-1001` con sus dos precios y el aviso de conflicto, y `NB-1015`. `NB-1010` no aparece: está agotado |
| Servicios, máximo 100.000 | `SRV-5001`, sin presentarlo como "sin stock" |
| Monitores, máximo 150.000 | `MON-3001` y `MON-3004`, por separado |

| Qué se entrega | Cómo se comprueba |
|---|---|
| Distribución | Cada regla común tiene una sola Spec dueña |
| Revisión cruzada | Cada conflicto encontrado se reproduce con un SKU |
| Resolución | Las Specs consumidoras referencian la regla en lugar de redefinirla |
| Integración | Un merge por vez, con pruebas después de cada uno |
| Consistencia | El CSV y el resumen coinciden en las tres consultas |
