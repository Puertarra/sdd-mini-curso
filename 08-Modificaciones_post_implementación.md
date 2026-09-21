# Las modificaciones posteriores a la implementación en SDD

## Iterar después de la primera versión

La primera implementación no termina el proceso de **Spec Driven Development**. Al probar el producto pueden aparecer nuevas necesidades, mejoras o funcionalidades que no estaban contempladas originalmente.

La idea fundamental es:

> **Una nueva necesidad no se implementa directamente sobre el código. Primero se incorpora a la especificación y después se ejecuta nuevamente el ciclo de SDD.**

Supongamos que ProductChat ya está funcionando, pero después de utilizarlo queremos incorporar dos mejoras:

1. Una página inicial que permita acceder a las principales funciones del chatbot.
2. Una interfaz más profesional y consistente.

En lugar de pedir directamente a la IA que modifique el código, comenzamos una nueva iteración.

> **Nota sobre los identificadores.** Spec Kit crea un directorio por cada especificación, numerado en el orden en que se generan. En este material llamamos `SPEC001` a la especificación inicial de ProductChat y `SPEC002` a la de esta segunda iteración. El nombre exacto del directorio depende de la versión de Spec Kit; comprueba cómo los nombra la tuya y sustituye la referencia donde corresponda.

## 1. Crear la nueva especificación

Utilizamos nuevamente:

```text
/speckit-specify
```

```text
/speckit-specify

ROL
Actúa como analista de requisitos sobre un producto ya implementado.

CONTEXTO
[NOMBRE DEL PRODUCTO] está funcionando. Quiero incorporar cambios sin romper lo
que ya hay.

ENTRADA
La especificación existente, [RUTA DE LA SPEC ANTERIOR].

CAMBIOS SOLICITADOS
1. [CAMBIO 1]
2. [CAMBIO 2]
3. [CAMBIO N]

SALIDA ESPERADA
Una especificación nueva y separada, que describa únicamente estos cambios y
declare explícitamente qué comportamiento existente se mantiene intacto.

RESTRICCIONES
Mantén sin cambios el comportamiento existente que no esté afectado
explícitamente por estos requisitos. No reescribas la especificación anterior.

CRITERIO DE ACEPTACIÓN
La nueva especificación puede leerse junto a la anterior sin contradecirla, y
sus criterios de aceptación son verificables sin leer código.
```

> Este prompt está en `prompts/09-modificacion-specify.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

### Ejemplo ProductChat

```text
/speckit-specify

ROL
Actúa como analista de requisitos sobre un producto ya implementado.

CONTEXTO
ProductChat está funcionando sobre el catálogo Excel de Distribuidora Andes.
Quiero mejorar cómo se usa, sin tocar cómo lee los datos.

ENTRADA
La especificación existente, SPEC001/spec.md.

CAMBIOS SOLICITADOS
1. Una página de inicio desde la que el usuario pueda acceder al chatbot y
   consultar información básica del catálogo.
2. Una presentación visual consistente, clara y profesional.

SALIDA ESPERADA
Una especificación nueva y separada, SPEC002, que describa solo estos cambios.

RESTRICCIONES
Mantén sin cambios la búsqueda de productos, la normalización del catálogo y el
tratamiento de los SKU en conflicto. No reescribas SPEC001.

CRITERIO DE ACEPTACIÓN
Los criterios de aceptación de SPEC001 siguen cumpliéndose sin modificación.
```

Spec Kit puede crear una nueva especificación, por ejemplo `SPEC002`, manteniendo separada la nueva feature de la implementación original.

## 2. Clarificar los nuevos requisitos

Antes de planificar, ejecutamos Clarify con el prompt de `prompts/03-clarify.txt`, ajustando su bloque de contexto a los nuevos requisitos:

```text
/speckit-clarify
```

Clarify permite detectar expresiones ambiguas como *"interfaz profesional"* o *"navegación clara"* y transformarlas en requisitos más concretos.

Cuando la modificación afecta a un dominio que requiere criterios específicos, también puede utilizarse `/speckit-checklist`, una puerta de calidad adicional que genera una lista de comprobación temática —de usabilidad, seguridad, accesibilidad u otra— para revisar los nuevos requisitos antes de planificar:

```text
/speckit-checklist
```

```text
/speckit-checklist

ROL
Actúa como revisor especializado en [UX / seguridad / accesibilidad / otro].

CONTEXTO
[Qué parte de ProductChat se está revisando y por qué.]

ENTRADA
spec.md, en particular los requisitos incorporados en esta iteración.

SALIDA ESPERADA
Una lista de comprobación de [dominio], donde cada punto se responde con sí o no
usando la aplicación.

CRITERIO DE ACEPTACIÓN
Ningún punto de la lista requiere leer código para responderse, y ninguno es
subjetivo: "la interfaz se ve profesional" no es un punto válido.
```

> Este prompt está en `prompts/08-checklist.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

### Ejemplo ProductChat

```text
/speckit-checklist

ROL
Actúa como revisor especializado en UX.

CONTEXTO
Se está revisando la nueva página de inicio de ProductChat y el rediseño de la
presentación de resultados. El chatbot muestra a menudo varios productos y, a
veces, avisos de inconsistencia del catálogo.

ENTRADA
SPEC002/spec.md.

SALIDA ESPERADA
Una lista de comprobación de UX donde cada punto se responde con sí o no usando
la aplicación.

CRITERIO DE ACEPTACIÓN
Ningún punto requiere leer código ni emitir un juicio estético.
```

## 3. Crear el Plan respetando la implementación existente

El nuevo Plan debe reutilizar las decisiones técnicas existentes salvo que el cambio requiera explícitamente modificarlas.

```text
/speckit-plan

ROL
Actúa como arquitecto sobre una base de código existente.

CONTEXTO
Esta feature modifica [ÁREA AFECTADA] de una aplicación que ya funciona.

ENTRADA
La nueva especificación, más [RUTA DEL PLAN ANTERIOR] y [RUTA DEL MODELO DE
DATOS ANTERIOR], de donde debes tomar el stack tecnológico, las convenciones, el
modelo de datos y la estructura del proyecto.

SALIDA ESPERADA
Un plan que reutilice las decisiones técnicas existentes y describa únicamente
lo que cambia.

RESTRICCIONES
No sustituyas ni rediseñes el stack, las convenciones ni el modelo de datos salvo
que los nuevos requisitos lo hagan imprescindible. Limita los cambios a lo que
pide la nueva especificación.

CRITERIO DE ACEPTACIÓN
El plan enumera explícitamente qué partes del sistema no toca. Ningún componente
que ya funcionaba aparece rediseñado sin una justificación escrita.
```

> Este prompt está en `prompts/10-modificacion-plan.txt`, listo para copiar. Si lo modificas, hazlo allí: ese archivo es la versión de referencia.

### Ejemplo ProductChat

```text
/speckit-plan

ROL
Actúa como arquitecto sobre una base de código existente.

CONTEXTO
Esta feature modifica la capa de presentación de ProductChat.

ENTRADA
SPEC002/spec.md, más SPEC001/plan.md y SPEC001/data-model.md, de donde debes
tomar el stack, las convenciones, el modelo de datos y la estructura.

SALIDA ESPERADA
Un plan que reutilice esas decisiones y describa solo lo que cambia.

RESTRICCIONES
No sustituyas ni rediseñes el acceso al catálogo ni el modelo interno
normalizado. Limita los cambios a la página de inicio y a la interfaz.

CRITERIO DE ACEPTACIÓN
El plan enumera qué partes del sistema no toca.
```

Esta instrucción es importante porque evita que una mejora localizada provoque un rediseño innecesario de partes del sistema que ya funcionan.

## 4. Generar, analizar e implementar

A partir de aquí repetimos el ciclo normal:

```text
/speckit-tasks
        ↓
/speckit-analyze
        ↓
/speckit-implement
        ↓
Validación
```

`Tasks` descompone la modificación en trabajo ejecutable. `Analyze` comprueba que la nueva especificación, el Plan y las tareas sean consistentes con los artefactos existentes. Finalmente, `Implement` realiza los cambios y verifica las tareas.

Si durante la validación aparecen nuevas necesidades, iniciamos otra iteración:

```text
Producto implementado
        ↓
Nueva necesidad
        ↓
Especificación
        ↓
Clarify / Checklist
        ↓
Plan
        ↓
Tareas
        ↓
Analyze
        ↓
Implementación
        ↓
Validación
        ↺
```

La ventaja es que ProductChat puede evolucionar sin perder la trazabilidad de **qué se pidió, por qué se modificó, cómo se decidió implementarlo y qué tareas produjeron finalmente el cambio en el código**.