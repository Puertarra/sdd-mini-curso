# Mini curso de Spec Driven Development

**Andrés Leiva-Araos, Ph.D.** · 
<br>Facultad de Ingeniería, Universidad del Desarrollo

Material para un mini curso de **Spec Driven Development (SDD)** dirigido a estudiantes de pregrado de Ingeniería. El curso usa **GitHub Spec Kit** como capa de método y **Claude Code** o **Codex** como capa de agente, y tiene dos partes:

- **Introducción a SDD (módulos 00 a 08):** el ciclo completo de SDD, desde las reglas del proyecto hasta el código.
- **SDD — Tópicos avanzados (módulos 09 a 12):** lo que pasa cuando el producto ya existe y empieza a cambiar.

## El caso

Todo el curso trabaja sobre un único caso, que avanza módulo a módulo.

**ProductChat** es un chatbot interno para **Distribuidora Andes**, una empresa que vende equipamiento informático. Su catálogo de productos y servicios vive en una planilla Excel mantenida a mano, con SKU duplicados que se contradicen, categorías escritas de trece formas distintas, atributos sin unidad estable, precios que a veces son texto y datos escondidos en columnas de observaciones. Nadie va a limpiarla antes de construir el chatbot.

Ese desorden no es casual. Es lo que obliga a que la especificación decida cosas que una especificación sobre datos limpios nunca tendría que decidir, y es el hilo que atraviesa los trece módulos. En la segunda parte, el mismo catálogo sigue produciendo los casos difíciles: un precio en el límite exacto de un filtro, un parche de stock que oculta productos disponibles, dos capacidades que interpretan distinto el mismo SKU duplicado.

## Índice

### Primera parte: Introducción a SDD

| Módulo | Contenido |
|---|---|
| [`00-Entorno_de_trabajo.md`](00-Entorno_de_trabajo.md) | Las dos capas, instalación de uv y Spec Kit 1.0.8, creación del proyecto |
| [`01-SDD.md`](01-SDD.md) | Qué es SDD, SDD frente a Vibe Coding, el caso del curso, vocabulario y contrato de prompt |
| [`02-Constitución.md`](02-Constitución.md) | Etapa 1: las reglas permanentes del proyecto |
| [`03-Especificación.md`](03-Especificación.md) | Etapa 2: qué debe hacer el producto. Incluye la puerta Clarify |
| [`04-Plan.md`](04-Plan.md) | Etapa 3: cómo se va a construir |
| [`05-Tareas.md`](05-Tareas.md) | Etapa 4: unidades de trabajo pequeñas y verificables |
| [`06-Análisis_de_consistencia.md`](06-Análisis_de_consistencia.md) | Puerta Analyze: coherencia entre todos los artefactos |
| [`07-Implementación.md`](07-Implementación.md) | Ejecución: de las tareas al código, y validación manual |
| [`08-Modificaciones_post_implementación.md`](08-Modificaciones_post_implementación.md) | Segunda iteración sin volver al Vibe Coding |

### Segunda parte: SDD — Tópicos avanzados

| Módulo | Contenido |
|---|---|
| [`09-Git_y_trazabilidad.md`](09-Git_y_trazabilidad.md) | Control de versiones al servicio de la trazabilidad: una rama por Spec, commits por etapa y contexto para el agente |
| [`10-Spec_Drift.md`](10-Spec_Drift.md) | Cuando el código y la Spec dejan de decir lo mismo: provocarlo, auditarlo en dos direcciones y resolverlo. Cómo evolucionan las Specs y una vista de los requisitos vigentes |
| [`11-Trabajo_en_paralelo.md`](11-Trabajo_en_paralelo.md) | Dos agentes, dos capacidades y los conflictos de significado que Git no ve |
| [`12-Reverse_Specs.md`](12-Reverse_Specs.md) | Reconstruir la Spec de un sistema existente. Actividad final del curso |

### Anexo

| Archivo | Contenido |
|---|---|
| [`A-Mini_manual_de_Git.md`](A-Mini_manual_de_Git.md) | Los comandos de Git que usa el curso, con ejemplos de ProductChat, convenciones y cómo deshacer errores |

Los módulos están pensados para leerse en orden. El `01` contiene el vocabulario y el contrato de prompt que usan todos los demás, así que conviene no saltárselo. La segunda parte parte de un ProductChat ya implementado al terminar la primera.

## Archivos del caso

| Archivo | Qué es |
|---|---|
| [`spec_servilleta_productchat.md`](spec_servilleta_productchat.md) | La especificación escrita a mano, antes de usar ninguna herramienta. Es la entrada de `/speckit-specify`, no su salida |
| `catalogo_productos.xlsx` | El catálogo de Distribuidora Andes, con sus defectos. Hoja `Catálogo`, cabecera en la fila 3 |
| [`prompts/`](prompts/) | Los 34 prompts del curso, listos para copiar |

Los dos primeros se copian dentro del proyecto de Spec Kit antes de empezar, porque los prompts los nombran por su ruta. Los criterios de aceptación de todo el curso citan SKU de la planilla: si se modifica, dejan de cuadrar.

## El ciclo

```text
Constitución → Especificación → Clarify → Plan → Tareas → Analyze → Implementación
```

Cuatro **etapas** que producen artefactos, dos **puertas de calidad** que revisan los artefactos existentes sin producir ninguno nuevo, y una **ejecución** que convierte todo lo anterior en código. El módulo `01` desarrolla esta distinción.

## Los prompts

Todos los prompts del curso declaran cinco cosas: **rol**, **contexto** del caso, **entrada** nombrada por su ruta, **salida esperada** y **criterio de aceptación** verificable sin leer código. Y encadenan: la salida de cada uno es la entrada declarada del siguiente. Esa propiedad es lo que convierte a SDD en un método y no en una sucesión de peticiones sueltas.

Hay una sola excepción, y es deliberada: `10.1` representa el pedido urgente sin método que produce un drift, y lo dice en su primera línea.

El número de cada prompt indica el módulo donde se usa y su orden dentro de él: `03.2-clarify.txt` es el segundo prompt del módulo 03. Los módulos reproducen cada prompt completo, pero la versión de referencia es siempre el archivo en `prompts/`.

| Prompt | Módulo | Para qué |
|---|---|---|
| [`02.1-constitution.txt`](prompts/02.1-constitution.txt) | 02 | Crear la Constitución de ProductChat |
| [`03.1-specify.txt`](prompts/03.1-specify.txt) | 03 | Formalizar la spec de servilleta como SPEC001 |
| [`03.2-clarify.txt`](prompts/03.2-clarify.txt) | 03 | Detectar decisiones pendientes en una Spec. Se reutiliza en la segunda parte |
| [`04.1-plan.txt`](prompts/04.1-plan.txt) | 04 | Planificar, incluido el tratamiento del dato sucio |
| [`05.1-tasks.txt`](prompts/05.1-tasks.txt) | 05 | Descomponer el plan en tareas verificables. Se reutiliza en la segunda parte |
| [`06.1-analyze.txt`](prompts/06.1-analyze.txt) | 06 | Auditar la coherencia entre todos los artefactos. Se reutiliza en la segunda parte |
| [`07.1-implement.txt`](prompts/07.1-implement.txt) | 07 | Implementar las tareas sin modificar el catálogo. Se reutiliza en la segunda parte |
| [`08.1-modificacion-specify.txt`](prompts/08.1-modificacion-specify.txt) | 08 | Plantilla: especificar un cambio sobre un producto ya implementado |
| [`08.2-checklist.txt`](prompts/08.2-checklist.txt) | 08 | Plantilla: lista de comprobación temática |
| [`08.3-modificacion-plan.txt`](prompts/08.3-modificacion-plan.txt) | 08 | Plantilla: planificar un cambio sin rediseñar lo existente |
| [`09.1-git-init.txt`](prompts/09.1-git-init.txt) | 09 | Crear el repositorio y su primer commit |
| [`09.2-contexto-agente.txt`](prompts/09.2-contexto-agente.txt) | 09 | Crear `CLAUDE.md` o `AGENTS.md` |
| [`09.3-filtros-specify.txt`](prompts/09.3-filtros-specify.txt) | 09 | Especificar los filtros por categoría y precio (SPEC003) |
| [`09.4-filtros-plan.txt`](prompts/09.4-filtros-plan.txt) | 09 | Planificar SPEC003 sobre el código existente |
| [`09.5-trazabilidad.txt`](prompts/09.5-trazabilidad.txt) | 09 | Demostrar la cadena requisito → plan → tarea → prueba → commit |
| [`09.6-precisar-spec-existente.txt`](prompts/09.6-precisar-spec-existente.txt) | 09 | Precisar una Spec abierta sin crear otra |
| [`10.1-drift-contraejemplo.txt`](prompts/10.1-drift-contraejemplo.txt) | 10 | Provocar un drift a propósito. No sigue el contrato, y lo declara |
| [`10.2-auditoria-drift.txt`](prompts/10.2-auditoria-drift.txt) | 10 | Auditar en dos direcciones. También verifica la resolución |
| [`10.3-sin-stock-specify.txt`](prompts/10.3-sin-stock-specify.txt) | 10 | Formalizar la regla de stock (SPEC004) |
| [`10.4-sin-stock-plan.txt`](prompts/10.4-sin-stock-plan.txt) | 10 | Corregir el parche desde el plan |
| [`10.5-vista-vigente.txt`](prompts/10.5-vista-vigente.txt) | 10 | Generar la vista consolidada de los requisitos vigentes, para quien no lee las Specs |
| [`11.1-distribucion.txt`](prompts/11.1-distribucion.txt) | 11 | Repartir el trabajo y asignar dueño a cada regla común |
| [`11.2-exportacion-specify.txt`](prompts/11.2-exportacion-specify.txt) | 11 | Especificar la exportación a CSV (SPEC005) |
| [`11.3-resumen-specify.txt`](prompts/11.3-resumen-specify.txt) | 11 | Especificar el resumen para correo (SPEC006) |
| [`11.4-revision-cruzada.txt`](prompts/11.4-revision-cruzada.txt) | 11 | Detectar conflictos de significado entre Specs |
| [`11.5-plan-agente.txt`](prompts/11.5-plan-agente.txt) | 11 | Planificar una sola capacidad dentro del equipo |
| [`11.6-integracion.txt`](prompts/11.6-integracion.txt) | 11 | Integrar rama por rama, con pruebas después de cada merge |
| [`12.1-explorar.txt`](prompts/12.1-explorar.txt) | 12 | Mapa del sistema y flujo de una consulta |
| [`12.2-reglas.txt`](prompts/12.2-reglas.txt) | 12 | Reconstruir las reglas con evidencia |
| [`12.3-anomalias.txt`](prompts/12.3-anomalias.txt) | 12 | Registrar y clasificar comportamientos raros |
| [`12.4-entrevista.txt`](prompts/12.4-entrevista.txt) | 12 | Preparar y registrar la entrevista al negocio |
| [`12.5-caracterizacion.txt`](prompts/12.5-caracterizacion.txt) | 12 | Fijar el comportamiento actual con pruebas |
| [`12.6-spec-inversa.txt`](prompts/12.6-spec-inversa.txt) | 12 | Escribir la Spec del sistema existente |
| [`12.7-spec-pack.txt`](prompts/12.7-spec-pack.txt) | 12 | Ensamblar el entregable final |

La segunda parte reutiliza sin cambios `03.2`, `05.1`, `06.1` y `07.1` para Clarify, Tareas, Analyze e Implementación: nombran los artefactos de la Spec activa, y Spec Kit sabe cuál es.

## Requisitos

- Un agente de código: Claude Code o Codex.
- [uv](https://docs.astral.sh/uv/), para instalar Spec Kit.
- Git 2.28 o posterior, desde el módulo 09. El anexo `A` resume los comandos necesarios.
- GitHub Spec Kit **v1.0.8**:

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@v1.0.8
```

> ⚠️ **Verifica la versión antes de instalar.** Todo el material está fijado a la v1.0.8 y
> comprobado contra ella. Spec Kit se desarrolla rápido, y la forma de invocar sus operaciones no
> es estable: en la 1.0.8 depende del agente, y no coincide con la notación que usa su propia
> documentación. Revisa las releases del repositorio antes de instalar. Si eliges una versión
> distinta de la v1.0.8, los prompts de `prompts/` pueden necesitar ajustes; el módulo `00` explica
> cómo comprobar los nombres reales de tu instalación.

En la v1.0.8, las operaciones se invocan así:

| Agente | Forma |
|---|---|
| Claude Code | `/speckit-specify` |
| Codex | `$speckit-specify` |

El curso usa la forma de Claude Code. Con Codex basta cambiar la barra por el signo pesos. La notación con punto, `/speckit.specify`, es la que usa la documentación oficial como referencia; no es la que se escribe en el agente.

Si vas a dictar el curso, conviene repetir esta comprobación antes de cada edición y actualizar la versión fijada si hace falta, en lugar de asumir que lo que funcionó el semestre pasado sigue funcionando.

## Autoría

Material desarrollado por **Andrés Leiva-Araos**, Facultad de Ingeniería, Universidad del Desarrollo.

## Licencia

Este material está licenciado bajo [Creative Commons Attribution-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-sa/4.0/deed.es) (CC BY-SA 4.0). Puedes usarlo, adaptarlo y redistribuirlo, incluso con fines comerciales, siempre que cites la autoría y distribuyas las obras derivadas bajo la misma licencia. Ver [`LICENSE`](LICENSE).

Cita sugerida:

> Leiva-Araos, A. (2026). *Mini curso de Spec Driven Development*. Facultad de Ingeniería, Universidad del Desarrollo. CC BY-SA 4.0.

## Nota

`_original/` conserva versiones anteriores del material: la de antes de la revisión de consistencia de la primera parte y, en `_original/parte2_v0/`, el primer borrador de la segunda parte. `_fuera_del_curso/` guarda archivos de otros proyectos que llegaron a esta carpeta por error. Ninguna de las dos forma parte del curso, y ambas están excluidas del repositorio por `.gitignore`.
