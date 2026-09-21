# Especificación de servilleta: ProductChat v0

> Esto es una **spec de servilleta**: la escribe una persona, a mano, antes de
> tocar ninguna herramienta. No es el artefacto formal que produce Spec Kit; es
> la materia prima que le vamos a entregar. Sirve para obligarnos a decidir qué
> queremos, y para dejar por escrito lo que todavía no hemos decidido.

## 1. Objetivo y contexto de negocio

ProductChat permite que cualquier persona de **Distribuidora Andes** consulte en
lenguaje natural el catálogo de productos y servicios de la empresa, sin abrir
la planilla, sin saber cómo está organizada y sin preguntarle a la persona que
"se la sabe".

La escena real es esta. Un vendedor está al teléfono con un cliente que pregunta
si hay notebooks con 32 GB de RAM. El vendedor abre `catalogo_productos.xlsx`,
filtra por la columna `categoria` y obtiene menos resultados de los que debería,
porque la mitad del catálogo está cargada como `Portatiles` y no como
`Notebooks`. Busca "32 GB" en la columna `Caracteristicas` y encuentra dos
equipos, pero hay un tercero cargado como `32768 MB` y un cuarto donde alguien
escribió solamente `32`. Encuentra además un Dell que parece cumplir, hasta que
lee la columna `Obs` y dice "ampliable a 32GB previa solicitud". Mientras tanto,
el cliente espera.

El catálogo no está limpio y **no se va a limpiar antes de construir esto**. Ese
es el punto de partida, no un problema que otro resolverá:

- El archivo es `catalogo_productos.xlsx`, mantenido a mano por el área de ventas.
- La cabecera no está en la primera fila: hay dos filas de texto suelto encima.
- Hay **SKU duplicados con datos que no coinciden** entre sí.
- Hay **un mismo producto cargado bajo dos SKU distintos**.
- La categoría está escrita de **13 formas distintas** para 5 familias reales
  (`Notebooks`, `notebooks`, `NOTEBOOKS`, `Portatiles`, `Portátiles`, …).
- Los atributos no tienen unidad estable: `32 GB`, `32768 MB`, `RAM 32 GB`, `32`.
- El precio a veces es un número, a veces texto (`949.990`, `$1.299.990`), a
  veces está vacío y a veces dice `consultar`.
- El stock mezcla números con `Si`, `si`, `agotado` y celdas vacías.
- Hay una columna `Obs` de texto libre donde a veces vive el dato que importa.
- Hay filas en blanco, espacios sobrantes y tildes usadas al azar.
- Hay una **segunda hoja**, `catalogo_antiguo`, con otras columnas, otra
  posición de cabecera y precios distintos para los mismos SKU.
- Productos y servicios conviven en la misma planilla, aunque a los servicios no
  les aplica el stock.

Éxito de negocio: responder correctamente una consulta de catálogo en menos de
diez segundos, **sin que el usuario tenga que saber nada de lo anterior**.

## 2. Usuarios

- **El vendedor (usa la aplicación).** Trabaja en Distribuidora Andes, atiende
  clientes por teléfono y por correo. No es técnico. Hoy consulta la planilla a
  mano. Necesita respuestas rápidas y, sobre todo, **confiables**: prefiere un
  "no estoy seguro" a un dato inventado que lo deje mal frente al cliente.
- **El encargado de catálogo (no usa la aplicación, pero le importa).** Es quien
  mantiene el Excel. No va a rehacerlo. Le sirve que el chatbot exponga las
  inconsistencias en vez de taparlas, porque eso le dice qué corregir primero.
- **El cliente final (no usa la aplicación).** Recibe la respuesta por boca del
  vendedor. Nunca habla con el chatbot en esta versión.

## 3. Escenarios de usuario

- **HU1 — Buscar productos.** Como vendedor, quiero encontrar productos por
  nombre, categoría o características escritas en lenguaje natural, para no
  depender de cómo esté escrito el dato en la planilla.
- **HU2 — Consultar los atributos de un producto.** Como vendedor, quiero pedir
  el precio, el stock o una característica concreta de un producto, para
  responderle al cliente en el momento.
- **HU3 — Gestionar información inexistente o insuficiente.** Como vendedor,
  quiero que el chatbot me diga claramente cuándo no sabe, para no repetirle al
  cliente un dato que no existe.
- **HU4 — Resolver un catálogo inconsistente.** Como vendedor, quiero que el
  chatbot entienda que `32768 MB` y `32 GB` son lo mismo y que `Portatiles` y
  `Notebooks` son la misma familia, y que **me avise cuando el catálogo se
  contradice a sí mismo**, para no elegir yo a ciegas entre dos precios.

## 4. Requisitos funcionales

- **RF1.** El sistema debe leer el catálogo desde `catalogo_productos.xlsx`,
  localizando la fila de cabecera aunque no sea la primera del archivo.
- **RF2.** El sistema debe ignorar filas vacías y recortar espacios sobrantes en
  los valores de texto.
- **RF3.** El sistema debe normalizar la categoría, de modo que las variantes de
  mayúsculas, tildes y sinónimos declarados se traten como una misma familia.
- **RF4.** El sistema debe normalizar los atributos numéricos con unidad, de modo
  que `32 GB`, `32768 MB`, `RAM 32 GB` y `32` se interpreten como 32 GB.
- **RF5.** El sistema debe normalizar el precio a un valor numérico en pesos
  chilenos, y marcar como **precio no disponible** los casos vacíos o textuales
  (`consultar`, `según cotización`).
- **RF6.** El sistema debe normalizar el stock a tres estados: **disponible**,
  **sin stock** y **stock no informado**.
- **RF7.** El sistema debe detectar SKU duplicados y, cuando los registros
  duplicados no coincidan en un atributo consultado, **informar el conflicto en
  lugar de elegir un valor en silencio**.
- **RF8.** El sistema debe permitir consultas en lenguaje natural por nombre,
  categoría y características.
- **RF9.** El sistema debe poder devolver varios resultados cuando la consulta
  los admita.
- **RF10.** El sistema debe distinguir un atributo **declarado** de un atributo
  **condicional o potencial** mencionado en texto libre, y no contar el segundo
  como si fuera el primero.
- **RF11.** Cuando no exista información suficiente para responder, el sistema
  debe decirlo explícitamente.
- **RF12.** El sistema debe indicar, junto a cada respuesta, el SKU del que
  proviene el dato, para que el vendedor pueda verificarlo en la planilla.

## 5. Reglas de negocio

- **RN1.** La hoja `Catálogo` es la fuente de verdad. La hoja `catalogo_antiguo`
  es histórica y **no se usa** en esta versión.
- **RN2.** Familias equivalentes: `Notebooks` ≡ `Portatiles` ≡ `Portátiles`
  (con cualquier combinación de mayúsculas y tildes).
- **RN3.** Conversión de unidades de memoria: 1024 MB = 1 GB. Un número sin
  unidad en un campo de memoria se interpreta en GB.
- **RN4.** Un atributo mencionado en `Obs` como condición futura ("ampliable a",
  "previa solicitud", "bajo pedido") **no cuenta como atributo del producto**.
- **RN5.** Ante un SKU duplicado con valores en conflicto, el chatbot **no elige**:
  reporta ambos valores e indica que el catálogo está inconsistente en ese punto.
- **RN6.** El chatbot no inventa productos, precios ni atributos. Todo dato
  entregado debe poder rastrearse hasta una fila del archivo.

### Ejemplo de referencia (debe cuadrar exactamente)

Consulta: **"¿Qué notebooks tienen 32 GB de RAM?"**

| Forma de resolverlo | Resultado | Veredicto |
|---|---|---|
| Buscar el literal `32 GB` en `Caracteristicas` | NB-1005, NB-1012 | **Incorrecto**: 2 falsos negativos |
| Buscar el texto `32` en todos los campos | NB-1003, NB-1005, NB-1007, NB-1009, NB-1012 | **Incorrecto**: 1 falso positivo |
| Normalizar la memoria a GB y aplicar RN3 y RN4 | NB-1005, NB-1007, NB-1009, NB-1012 | **Correcto: 4 modelos** |

Detalle de por qué cada uno entra o queda fuera:

| SKU | Dato en la planilla | Interpretación | ¿Cumple? |
|---|---|---|---|
| NB-1005 | `32 GB RAM, SSD 1TB` | 32 GB declarados | Sí |
| NB-1007 | `RAM 32768 MB, SSD 512 GB` | 32768 MB = 32 GB (RN3) | Sí |
| NB-1009 | `32` | sin unidad, se lee GB (RN3) | Sí |
| NB-1012 | `RAM 32 GB, SSD 1 TB, i7` | 32 GB declarados | Sí |
| NB-1003 | `16GB RAM DDR4` + Obs `ampliable a 32GB` | condicional (RN4) | No |

Consulta: **"¿Cuál es el precio del ThinkBook 14?"**

El SKU `NB-1001` aparece **dos veces en la hoja `Catálogo`**, con precios
899.990 y 949.990, y una tercera vez en `catalogo_antiguo` con 879.990. Por RN1
la hoja histórica se descarta; por RN5 el chatbot debe responder que existen dos
precios registrados para ese SKU y que el catálogo está inconsistente. **No debe
elegir uno.** Cuál de los dos gana es una decisión de negocio que todavía no
está tomada: ver PA1.

## 6. Criterios de aceptación (verificables sin leer código)

- **CA1.** Ante "¿qué notebooks tienen 32 GB de RAM?", el chatbot devuelve
  exactamente los 4 modelos de la tabla anterior; no incluye NB-1003.
- **CA2.** Ante "¿qué notebooks hay?", el resultado incluye los equipos cargados
  como `Portatiles` y `Portátiles`, no solo los cargados como `Notebooks`.
- **CA3.** Ante "¿cuál es el precio del ThinkBook 14?", el chatbot informa que
  hay dos precios en conflicto para NB-1001 y no entrega uno solo como si fuera
  el correcto.
- **CA4.** Ante una consulta por el precio de `MON-3002` (Monitor LG UltraWide),
  el chatbot responde que el precio no está disponible y que debe consultarse,
  en vez de omitir el producto o inventar un valor.
- **CA5.** Ante una consulta sobre el stock de `NB-1007`, cuyo valor es `Si`, el
  chatbot responde "disponible" y no un número.
- **CA6.** Ante una consulta por un producto que no existe en el catálogo, el
  chatbot dice que no encontró información.
- **CA7.** Cada dato entregado va acompañado de su SKU de origen.
- **CA8.** Una persona no técnica puede ejecutar las consultas anteriores y
  verificar cada respuesta abriendo `catalogo_productos.xlsx`.

## 7. Casos límite

- **CL1.** Fila completamente vacía en medio de los datos: se ignora.
- **CL2.** Producto sin características (`MON-3003`): se puede encontrar por
  nombre y categoría, y al pedir una característica se responde que no está
  registrada.
- **CL3.** Producto sin precio (`NB-1009`): aparece en los resultados, con el
  precio marcado como no disponible.
- **CL4.** Nombres muy parecidos (`NB-1001` "Notebook Lenovo ThinkBook 14" y
  `NB-1015` "notebook lenovo thinkbook 14 gen 4"): se devuelven ambos y se
  distinguen por SKU.
- **CL5.** Mismo producto bajo dos SKU (`NB-1002` y `NB-2002`; `MON-3001` y
  `MON-3004`): se devuelven ambos; el chatbot no los fusiona por su cuenta.
- **CL6.** Servicios (`SRV-*`): no tienen stock; preguntar por su stock no debe
  producir "sin stock" sino "no aplica".
- **CL7.** La planilla está abierta o el archivo no se puede leer: el chatbot lo
  informa y no responde con datos antiguos.

## 8. Fuera de alcance (v0)

- Modificar, crear o eliminar productos desde el chatbot.
- **Corregir el archivo Excel.** El chatbot lee; no escribe.
- Comprar, cotizar formalmente o gestionar pedidos.
- Recomendar productos con criterios externos al catálogo.
- Responder preguntas que no sean sobre el catálogo.
- Usar la hoja `catalogo_antiguo`.
- Conectarse a una base de datos SQL. En esta versión la fuente es el Excel, y
  solo el Excel.
- Cuentas de usuario, permisos y multiusuario.
- Cualquier funcionalidad no descrita explícitamente aquí.

## 9. Preguntas abiertas (para la sesión de Clarify)

Cosas que sé que no he decidido. No las escondo: las apunto para resolverlas en
el siguiente paso.

- **PA1. Duplicados en conflicto.** RN5 dice "reportar ambos". ¿Es esa la
  conducta definitiva, o el negocio prefiere una regla de desempate (el precio
  más alto, el más reciente según `Obs`, la última fila del archivo)?
- **PA2. ¿"Tienen" significa catálogo o stock?** De los 4 notebooks con 32 GB,
  solo NB-1005 y NB-1012 tienen stock numérico mayor que cero. ¿La consulta del
  vendedor pregunta por lo que existe en catálogo o por lo que se puede vender
  hoy?
- **PA3. Sinónimos de categoría.** RN2 declara la equivalencia notebook/portátil
  a mano. ¿Se mantiene una lista fija de sinónimos, o el sistema debe inferirlos?
  Si es una lista, ¿quién la mantiene?
- **PA4. Alcance del texto libre.** RN4 descarta lo condicional. ¿Debe el chatbot
  **mencionar** igualmente que el Dell es ampliable a 32 GB, como información
  adicional, o callarlo?
- **PA5. Frescura del dato.** ¿Se lee el Excel en cada consulta, o se carga una
  vez al arrancar? Si el encargado edita la planilla a mediodía, ¿qué ve el
  vendedor a las 12:01?
- **PA6. Umbral de confianza.** ¿Cuándo el chatbot debe decir "no estoy seguro"
  en vez de responder? ¿Existe un caso en que responda con reservas?
- **PA7. Productos y servicios.** ¿Se consultan juntos o el usuario debe poder
  pedir "solo servicios"?
