# El Spec Drift en SDD

El módulo 07 terminó con una advertencia. Si aparece en la planilla una inconsistencia que ninguna regla cubre, la tentación es parchear el código directamente, y lo correcto es llevar la regla a la Especificación. Este módulo trata de lo que pasa cuando se cede a esa tentación, una y otra vez.

**Spec Drift** es la divergencia entre lo que dicen los requisitos vigentes y lo que hace el código. No aparece de golpe: se acumula con cada corrección urgente que no pasó por la Spec. Git, que vimos en el módulo anterior, registra cada uno de esos cambios, pero no avisa de que el código y la Spec ya no dicen lo mismo. Para eso hace falta auditar.

## Qué papel cumple la Spec después de implementar

No todos los proyectos le piden lo mismo a una especificación una vez que el código existe. La documentación de Spec Kit distingue tres enfoques:

| Enfoque | Qué pasa con la Spec | En ProductChat |
|---|---|---|
| **Spec-first** | Guía el desarrollo y después se puede descartar | SPEC001 quedaría como registro histórico de cómo empezó el proyecto |
| **Spec-anchored** | Se conserva y se mantiene como referencia del comportamiento | Cada cambio pasa por una Spec, y las Specs vigentes describen lo que ProductChat hace hoy |
| **Spec-as-source** | Es lo único que editan las personas; el código se genera desde ella | No lo usamos en el curso |

Desde el módulo 08 el curso trabaja **spec-anchored**: un cambio entregado se registra en una Spec nueva que referencia a las anteriores, y el conjunto de Specs vigentes describe el comportamiento actual. Ese enfoque tiene un riesgo propio, y es justamente el drift: una Spec que se conserva como referencia solo sirve si sigue diciendo la verdad.

## Qué es el drift, y qué no

En ProductChat, un drift se ve así:

```text
Requisito vigente (SPEC001, CA1):  NB-1007 aparece entre los notebooks de 32 GB
Código después de un parche:       NB-1007 no aparece, porque su stock es "Si" y no un número
                                               ↓
                                   El código ya no hace lo que la Spec dice
```

No toda diferencia entre documento y código es drift. Que el código tenga funciones, variables o estructuras que la Spec no menciona es normal: la Spec describe comportamiento, no implementación. Y una Spec histórica que describe una versión anterior no está equivocada, solo está superada. Hay drift cuando un **requisito vigente** y el **comportamiento observable** se contradicen.

Las causas suelen ser estas:

| Situación | Qué produce |
|---|---|
| Un cambio urgente va directo al código | Comportamiento sin requisito ni prueba |
| Se le pide un ajuste al agente después de implementar | El resultado se aleja del alcance aceptado |
| Cambia un requisito, pero no el plan ni las tareas | El código sigue una decisión obsoleta |
| Dos Specs vigentes dicen cosas distintas | Nadie sabe qué comportamiento es el correcto |

Una advertencia: que las pruebas pasen no descarta el drift. Las pruebas comprueban lo que alguien decidió probar. Si un criterio de aceptación nunca se convirtió en prueba, el código puede dejar de cumplirlo sin que nada falle.

## El caso: ocultar lo que no hay

Los vendedores se quejan: ProductChat les sugiere notebooks que no hay en bodega, y terminan ofreciendo lo que no pueden vender. Llega un pedido urgente, y alguien lo resuelve directo en el código.

Para estudiarlo, vamos a provocar ese drift a propósito, en una rama de práctica:

```bash
git switch -c 004-ocultar-sin-stock
```

```text
EJERCICIO CONTROLADO. Este prompt no sigue el contrato del curso a propósito:
es un pedido urgente como los que producen Spec Drift. Úsalo solo en la rama
de práctica.

Urgente: los vendedores están ofreciendo productos que no tenemos. Modifica
directamente el código de ProductChat para que no muestre ningún producto cuyo
stock no sea un número mayor que cero. No toques las Specs, el plan ni las
tareas; las pruebas las agregamos después.
```

> Este prompt está en `prompts/10.1-drift-contraejemplo.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

El resultado parece correcto: `NB-1009`, con stock 0, ya no aparece, y `NB-1010`, que dice `agotado`, tampoco. Registra el parche como lo haría alguien con prisa:

```bash
git add src/
git commit -m "hotfix: hide out-of-stock products"
```

Ahora contrástalo con la planilla. La regla "solo stock numérico mayor que cero" choca con un catálogo que nunca fue numérico:

| SKU | Stock en la planilla | Qué dice SPEC001 | Qué hace el parche |
|---|---|---|---|
| `NB-1009` | `0` | Sin stock | Lo oculta |
| `NB-1010` | `agotado` | Sin stock | Lo oculta |
| `NB-1007` | `Si` | Disponible (CA5) | **Lo oculta** |
| `MON-3002` | vacío | Stock no informado (RF6) | **Lo oculta** |
| `SRV-5001`, `SRV-5002`, `SRV-5003` | vacío | Servicio: el stock no aplica (CL6) | **Los oculta** |

Aplicado a la consulta canónica del curso, "¿qué notebooks tienen 32 GB de RAM?", el parche devuelve `NB-1005` y `NB-1012`. SPEC001 dice que son cuatro.

Otra forma frecuente del mismo parche es ocultar solo cuando el stock vale `0`. Esa falla al revés: deja visible `NB-1010`, porque `agotado` no es `0`. Las dos variantes tienen la misma causa: la regla se escribió sin mirar la Spec, que ya definía tres estados de stock.

## Auditar en dos direcciones

Una auditoría de drift se hace siempre en dos direcciones, porque cada una encuentra un tipo de problema distinto:

```text
Spec → código:  ¿el código sigue cumpliendo los requisitos vigentes?
Código → Spec:  ¿el código hace cosas que ninguna Spec pide?
```

La primera dirección detecta incumplimientos. La segunda detecta reglas nuevas que entraron sin pasar por la Especificación. Un parche urgente suele producir los dos tipos a la vez.

La evidencia sale de la Constitución, las Specs vigentes con sus planes y tareas, el código, las pruebas y el historial. Este comando muestra todo lo que la rama cambió desde que se separó de `main`:

```bash
git diff main...HEAD
```

```text
ROL
Actúa como auditor de consistencia de ProductChat. Detectas y explicas
divergencias; no modificas archivos ni decides qué comportamiento es correcto.

CONTEXTO
ProductChat tiene varias Specs vigentes en specs/. Puede haber código que
cambió sin que cambiaran sus requisitos, o requisitos que cambiaron sin que
cambiara el código. El catálogo, catalogo_productos.xlsx, es la referencia
para reproducir cada comportamiento con SKU concretos.

ENTRADA
.specify/memory/constitution.md, las Specs de specs/ con sus planes y tareas,
el código, las pruebas, CLAUDE.md, el historial de la rama actual (git log y
git diff main...HEAD) y, si existe, el informe de una auditoría anterior.

SALIDA ESPERADA
Un informe con hallazgos numerados D-01, D-02… Cada uno indica dirección
(Spec → código o código → Spec), severidad, requisito afectado, comportamiento
observado, SKU que lo reproduce, evidencia con rutas y líneas, e impacto. Si
hay un informe anterior, agrega el estado de cada hallazgo previo: resuelto,
pendiente o no verificable.

QUÉ BUSCAR
Spec → código: requisitos y criterios de aceptación que el código ya no
cumple. Código → Spec: reglas de producto implementadas que ninguna Spec
respalda. Revisa en especial stock, precios, categorías y conflictos de SKU.

RESTRICCIONES
Distingue lo que ejecutaste de lo que solo leíste en el código. Si no puedes
determinar qué requisito está vigente, regístralo como ambigüedad.

CRITERIO DE ACEPTACIÓN
Cada hallazgo se reproduce con el SKU que cita, y ninguno es genérico del
tipo "conviene revisar la cobertura".
```

> Este prompt está en `prompts/10.2-auditoria-drift.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

Sobre el parche, la auditoría debería encontrar al menos esto:

| Hallazgo | Dirección | Severidad | Qué encuentra |
|---|---|---|---|
| D-01 | Spec → código | Alta | CA1 de SPEC001 falla: los notebooks de 32 GB son dos en vez de cuatro. Faltan `NB-1007` y `NB-1009` |
| D-02 | Spec → código | Alta | Los tres servicios desaparecen, contra CL6 |
| D-03 | Spec → código | Media | `MON-3002` desaparece, aunque RF6 distingue "no informado" de "sin stock" |
| D-04 | Código → Spec | Alta | Hay una regla de ocultamiento que ninguna Spec respalda |

D-01 tiene una sutileza que vale la pena discutir en clase. Que falte `NB-1007` es un error: tiene stock. Que falte `NB-1009` quizás no: tiene stock 0, y si el negocio decide que no se muestra lo que no hay, su ausencia es correcta. La auditoría no lo puede decidir. Solo puede mostrar que el código y la Spec ya no coinciden.

## Resolver el drift

Resolver un drift empieza por decidir qué comportamiento se quiere. Ni el código ni la Spec tienen razón por defecto:

| Si se decide que… | Entonces |
|---|---|
| El requisito vigente es el correcto | Se corrige el código |
| El comportamiento nuevo es el correcto | Se formaliza en una Spec y se completan las pruebas |
| La necesidad es correcta pero la regla está mal | Se especifica la regla correcta y se ajusta el código |
| No hay acuerdo | Se resuelve antes de tocar nada más |

Aquí estamos en el tercer caso. La necesidad es legítima: no ofrecer lo que no hay. Y esa decisión cierra, de paso, una pregunta que la servilleta dejó abierta desde el principio. La PA2 preguntaba si "¿qué notebooks tienen 32 GB?" se refiere al catálogo o al stock. El negocio acaba de responder: al stock.

Es una Spec nueva, SPEC004, porque cambia un comportamiento que SPEC001 ya entregó. Y tiene que decirlo explícitamente: CA1 cambia de cuatro notebooks a tres.

```text
/speckit-specify

ROL
Actúa como analista de requisitos. Formalizas una decisión de negocio ya
tomada; no la diseñas ni la implementas.

CONTEXTO
Una corrección urgente ocultó del chatbot los productos sin stock sin pasar
por SDD, y la auditoría encontró que también ocultaba productos disponibles y
servicios. La necesidad es legítima; la regla estaba mal. Esta Spec resuelve
además la pregunta abierta PA2 de la servilleta: cuando el vendedor pregunta
qué productos "tienen", pregunta por lo que se puede vender hoy.

ENTRADA
La spec.md de SPEC001, el informe de auditoría, .specify/memory/constitution.md
y catalogo_productos.xlsx.

SALIDA ESPERADA
La spec.md de la nueva Spec, que referencie a SPEC001 y SPEC003 y declare de
forma explícita qué criterios de aceptación de SPEC001 cambian.

REGLAS YA DECIDIDAS
- Sin stock (0 o "agotado"): el producto no aparece en las respuestas.
- Disponible (número mayor que cero, "Si" o "si"): aparece.
- Stock no informado (celda vacía): aparece, con un aviso.
- Servicios: el stock no aplica y nunca los oculta.
- Las reglas valen para todas las consultas, con o sin filtros.

RESTRICCIONES
Inspecciona el código existente y distingue lo que ya cumple la regla de lo
que hay que corregir. No aceptes un comportamiento solo porque ya está
implementado. Fuera de alcance: modificar el catálogo y gestionar inventario.

CRITERIO DE ACEPTACIÓN
La Spec actualiza CA1 de SPEC001: "¿qué notebooks tienen 32 GB de RAM?" pasa
a devolver NB-1005, NB-1007 y NB-1012. NB-1007 aparece, NB-1009 no, y SRV-5001
sigue apareciendo.
```

> Este prompt está en `prompts/10.3-sin-stock-specify.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

```text
/speckit-plan

ROL
Actúa como arquitecto sobre código que ya contiene una versión defectuosa de
esta regla. Propones el cómo; la decisión final es mía.

CONTEXTO
La rama ya tiene un parche que oculta productos según su stock. No hay que
rehacerlo, sino corregirlo: la normalización del stock en tres estados que
definió SPEC001 ya existe y es la que debe usarse.

ENTRADA
La spec.md de SPEC004, el plan.md y el data-model.md de SPEC001, el informe
de auditoría y CLAUDE.md.

SALIDA ESPERADA
El plan.md de SPEC004, que reutilice el stack y la normalización existentes,
describa solo los ajustes al parche y defina las pruebas con los SKU de los
criterios de aceptación.

RESTRICCIONES
No sustituyas el stack ni el modelo de datos. No dupliques la normalización
del stock en otro componente.

CRITERIO DE ACEPTACIÓN
El plan indica, para cada hallazgo de la auditoría, qué cambio lo resuelve.
```

> Este prompt está en `prompts/10.4-sin-stock-plan.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

Después vienen Tareas, Analyze e Implementación con los prompts `prompts/05.1-tasks.txt`, `prompts/06.1-analyze.txt` y `prompts/07.1-implement.txt`. Las tareas no deben reconstruir lo que ya funciona: el parche ya oculta correctamente `NB-1009` y `NB-1010`, y eso se conserva.

El resultado esperado, verificable contra la planilla:

| SKU | Stock | Con SPEC004 |
|---|---|---|
| `NB-1009` | `0` | No aparece |
| `NB-1010` | `agotado` | No aparece |
| `NB-1007` | `Si` | Aparece |
| `MON-3002` | vacío | Aparece, con aviso de stock no informado |
| `SRV-5001` | vacío | Aparece; su stock no se evalúa |

Y la consulta de los notebooks de 32 GB devuelve `NB-1005`, `NB-1007` y `NB-1012`.

Spec Kit 1.0.8 incluye además `/speckit-converge`, que revisa el código existente y agrega como tareas el trabajo que falta para cumplir la Spec. Es útil como revisión adicional después de implementar, pero no reemplaza la auditoría: converge mira de la Spec al código, y el drift también viaja en la dirección contraria.

Cuando la implementación pase las pruebas, vuelve a ejecutar la auditoría con `10.2`, ahora con el informe inicial como entrada. Cada hallazgo debería quedar marcado como resuelto. Recién entonces registra e integra:

```bash
git add src/ tests/ specs/
git commit -m "SPEC004: hide out-of-stock products per spec"
git switch main
git merge 004-ocultar-sin-stock
```

El historial de la rama cuenta la historia completa: primero el parche, después la auditoría, y al final la Spec que lo corrige. Eso también es trazabilidad.

## Cómo prevenirlo

La práctica que más drift evita es aburrida: cada criterio de aceptación tiene una prueba. Si `CA5` hubiera sido una prueba, el parche habría fallado en el primer intento y el drift nunca habría llegado al historial.

La segunda es auditar después de cada cambio urgente, no al final del semestre. El prompt `10.2` no depende de este caso: sirve igual para cualquier rama.

## Actividad práctica

Reproduce el caso completo en tu ProductChat: crea la rama, aplica el parche con `10.1`, comprueba la tabla del caso contra tu implementación, audita, formaliza SPEC004, corrige y vuelve a auditar.

| Qué se entrega | Cómo se comprueba |
|---|---|
| Diagnóstico | El informe inicial reproduce cada hallazgo con un SKU |
| Decisión | SPEC004 declara qué criterios de SPEC001 cambian y por qué |
| Corrección | La tabla de resultados esperados se cumple completa |
| Verificación | El segundo informe marca resuelto cada hallazgo inicial |
| Trazabilidad | El historial muestra parche, Spec y corrección en ese orden |
