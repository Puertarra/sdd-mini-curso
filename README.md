# Mini curso de Spec Driven Development

Material para un mini curso de **Spec Driven Development (SDD)** dirigido a estudiantes de pregrado de Ingeniería. El curso recorre el ciclo completo, desde las reglas del proyecto hasta el código, usando **GitHub Spec Kit** como capa de método y **Claude Code** o **Codex** como capa de agente.

## El caso

Todo el curso trabaja sobre un único caso, que avanza módulo a módulo.

**ProductChat** es un chatbot interno para **Distribuidora Andes**, una empresa que vende equipamiento informático. Su catálogo de productos y servicios vive en una planilla Excel mantenida a mano, con SKU duplicados que se contradicen, categorías escritas de trece formas distintas, atributos sin unidad estable, precios que a veces son texto y datos escondidos en columnas de observaciones. Nadie va a limpiarla antes de construir el chatbot.

Ese desorden no es decorado. Es lo que obliga a que la especificación decida cosas que una especificación sobre datos limpios nunca tendría que decidir, y es el hilo que atraviesa los nueve módulos.

## Índice

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

Los módulos están pensados para leerse en orden. El `01` contiene el vocabulario y el contrato de prompt que usan todos los demás, así que conviene no saltárselo.

## Archivos del caso

| Archivo | Qué es |
|---|---|
| [`spec_servilleta_productchat.md`](spec_servilleta_productchat.md) | La especificación escrita a mano, antes de usar ninguna herramienta. Es la entrada de `/speckit-specify`, no su salida |
| `catalogo_productos.xlsx` | El catálogo de Distribuidora Andes, con sus defectos. Hoja `Catálogo`, cabecera en la fila 3 |
| [`prompts/`](prompts/) | Los diez prompts del curso, numerados por orden de uso, listos para copiar |

Los dos primeros se copian dentro del proyecto de Spec Kit antes de empezar, porque los prompts los nombran por su ruta.

## El ciclo

```text
Constitución → Especificación → Clarify → Plan → Tareas → Analyze → Implementación
```

Cuatro **etapas** que producen artefactos, dos **puertas de calidad** que revisan los artefactos existentes sin producir ninguno nuevo, y una **ejecución** que convierte todo lo anterior en código. El módulo `01` desarrolla esta distinción.

## Requisitos

- Un agente de código: Claude Code o Codex.
- [uv](https://docs.astral.sh/uv/), para instalar Spec Kit.
- GitHub Spec Kit **v1.0.8**:

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@v1.0.8
```

> ⚠️ **Verifica la versión antes de instalar.** Todo el material está fijado a la v1.0.8 y
> comprobado contra ella. Spec Kit se desarrolla rápido y ya cambió una vez la sintaxis de sus
> operaciones: antes de la 1.0 eran `/speckit.specify`, con punto, y en la 1.0.8 son
> `/speckit-specify`, con guion. Revisa las releases del repositorio antes de instalar. Si eliges
> una versión distinta de la v1.0.8, los prompts de `prompts/` pueden necesitar ajustes; el módulo
> `00` explica cómo comprobar los nombres reales de tu instalación.

Si vas a dictar el curso, conviene repetir esta comprobación antes de cada edición y actualizar la
versión fijada si hace falta, en lugar de asumir que lo que funcionó el semestre pasado sigue
funcionando.

## Sobre los prompts

Todos los prompts del curso declaran cinco cosas: **rol**, **contexto** del caso, **entrada** nombrada por su ruta, **salida esperada** y **criterio de aceptación** verificable sin leer código. Y encadenan: la salida de cada uno es la entrada declarada del siguiente. Esa propiedad es lo que convierte a SDD en un método y no en una sucesión de peticiones sueltas.

Los módulos reproducen cada prompt, pero la versión de referencia es siempre el archivo en `prompts/`.

## Nota

`_original/` conserva la versión del material anterior a la revisión de consistencia. No forma parte del curso.
