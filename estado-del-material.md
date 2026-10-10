# Mini curso SDD — estado del material

Carpeta: `_RESEARCH/LangChain-Andres/Spec-Driven-Development` (local, Mac Studio).
Repositorio: https://github.com/Puertarra/sdd-mini-curso
Autor: Andrés Leiva-Araos. Licencia: CC BY-SA 4.0 (solo resumen + aviso en `LICENSE`).
Índice y puerta de entrada: `README.md` (único; los README de `prompts/` se eliminaron).
Respaldos: `_original/` (parte 1 previa a la revisión), `_original/parte2_v0/` (versión
OpenAI de la parte 2: 4 módulos, 37 prompts, 2 README). `_fuera_del_curso/` guarda
material ajeno. Ambos, más `Claude outputs/`, están en `.gitignore`.

## Estructura: trece módulos en dos partes

Parte 1 (fundamentos): `00` Entorno · `01`–`07` las etapas y puertas · `08` Modificación.
Parte 2 (tópicos avanzados):

| Módulo | Tema | Spec del caso |
|---|---|---|
| `09` El control de versiones en SDD | Git, rama por Spec, trazabilidad, `CLAUDE.md` | SPEC003 Filtros |
| `10` El Spec Drift en SDD | spec-first/anchored/as-source, auditoría D-01..D-04, hotfix | SPEC004 Ocultar sin stock |
| `11` El trabajo en paralelo en SDD | coordinador, `SPECIFY_FEATURE_DIRECTORY`, revisión cruzada, integración 006→005 | SPEC005 Export CSV, SPEC006 Resumen correo |
| `12` Los Reverse-Specs en SDD | snapshot sin historia, `reconocimiento/`, caracterización pytest, spec inversa, SPEC-PACK | Spec inversa (`specs/001-…` en el snapshot) |

Numeración de Specs en todo el curso: SPEC001 Consulta (parte 1), SPEC002 Página de
inicio (`08`), SPEC003–SPEC006 como en la tabla.

## Caso conductor

**ProductChat**, chatbot interno de **Distribuidora Andes** sobre un catálogo Excel
precario. Caso único en las dos partes: la servilleta y el catálogo son el mismo caso.
La parte 2 se reancló entera sobre el catálogo real (se eliminaron SQL, catálogos de
juguete e invoicelegacy de la versión OpenAI).

Dataset: `catalogo_productos.xlsx`, 24 filas útiles en la hoja `Catálogo` (cabecera en
la fila 3) más la hoja `catalogo_antiguo` con otro esquema. Defectos plantados y
verificados: SKU duplicados con precios en conflicto (`NB-1001`, `IMP-2002`), mismo
producto bajo dos SKU (`NB-1002`/`NB-2002`, `MON-3001`/`MON-3004`), 13 grafías de
categoría para 5 familias, RAM como `32 GB` / `32768 MB` / `32`, precios como número,
texto, símbolo, vacío y `consultar`, stock con `Si` / `si` / `agotado` / vacío, fila en
blanco, espacios y tildes inconsistentes, servicios mezclados con productos.

Ejemplo canónico: *"¿qué notebooks tienen 32 GB de RAM?"* → `NB-1005`, `NB-1007`,
`NB-1009`, `NB-1012`; falso positivo `NB-1003` ("ampliable a 32GB"). `NB-1001` tiene
tres precios (899.990, "949.990", 879.990 en la hoja antigua); desempate sin decidir (PA1).

Hechos usados en la parte 2: periféricos ≤34.990 → `PER-4001` (exacto), `PER-4002`.
Estados de stock RF6: sin stock `NB-1009` ("0") y `NB-1010` ("agotado"); disponible
`NB-1007` ("Si") e `IMP-2002` ("si"); no informado `MON-3002` (vacío); `SRV-*` no aplica.
El parche B del módulo `10` (stock numérico > 0) oculta de más; con SPEC004 el CA1 queda
`NB-1005`, `NB-1007`, `NB-1012`. Tras SPEC004: notebooks ≤1M → `NB-1001` (ambos precios)
y `NB-1015`; servicios ≤100.000 → `SRV-5001`; monitores ≤150.000 → `MON-3001`, `MON-3004`.

## Entorno fijado (verificado ejecutando Spec Kit, no de memoria)

- **GitHub Spec Kit v1.0.8.** Instalación:
  `uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@v1.0.8`
  (`uv tool update-shell` si `specify` no queda en el PATH). `specify --version` → `specify 1.0.8`.
- `specify init proyecto1 --integration claude` (o `--integration codex`). El flag es
  `--integration`, **no** `--ai`. `init` no crea repositorio Git.
- Las operaciones se instalan como **skills** y la invocación depende del agente:
  Claude Code → `/speckit-specify`, en `.claude/skills/` (slash command);
  Codex → `$speckit-specify`, en `.agents/skills/`.
  La documentación oficial usa la notación con punto (`/speckit.specify`) solo como
  referencia. El curso no afirma nada sobre versiones anteriores a la 1.0 (no verificado).
- Skills en 1.0.8: analyze, checklist, clarify, constitution, converge, implement, plan,
  specify, tasks, taskstoissues. `checklist` se usa en `08`, `converge` aparece en `10`
  (solo verifica spec→código), `taskstoissues` queda fuera del curso.
- `/speckit-specify` no crea ramas salvo con la extensión `git`. La Spec activa vive en
  `.specify/feature.json`, ignorado por Git y propio de cada checkout. La numeración
  secuencial mira `specs/` del checkout actual; `SPECIFY_FEATURE_DIRECTORY` la fija.
  Ambos hechos sostienen el diseño del módulo `11`.

## Decisiones cerradas

1. **Caso único** en los trece módulos.
2. **Precariedad propagada** a todos los módulos; la parte 2 trabaja sobre el mismo catálogo.
3. **Vocabulario fijo.** Cuatro etapas + dos puertas de calidad (Clarify, Analyze) + una
   ejecución. Cadena canónica:
   `Constitución → Especificación → Clarify → Plan → Tareas → Analyze → Implementación`.
   Títulos "El/La X en SDD". Registro de curso práctico, no de transcripción.
4. **Contrato de prompt.** ROL / CONTEXTO / ENTRADA / SALIDA ESPERADA / [bloques
   opcionales] / CRITERIO DE ACEPTACIÓN. Los prompts van incrustados en el módulo con la
   nota de que el `.txt` es la versión de referencia. Excepción deliberada: `10.1`
   (contraejemplo de drift, sin contrato).
5. **Numeración de prompts por módulo:** `MM.n-nombre.txt` (p. ej. `03.2-clarify.txt`).
   33 prompts: 10 en la parte 1, 23 en la parte 2 (se redujeron desde 37). Se reutilizan
   en la parte 2 `03.2`, `05.1`, `06.1`, `07.1`. Tabla completa en `README.md`.
6. **Sin implementación de referencia.** Decisión explícita del autor. No volver a proponerlo.
7. **Mensajes de commit en inglés** en los ejemplos (p. ej. `SPEC003: implement and test filters`).

## Mantenimiento

- **Verificar la versión fijada antes de cada dictado.** Si se sube de versión, comprobar
  nombres reales (`ls .claude/skills/`, `ls .agents/skills/`), el comportamiento de
  `feature.json` y `SPECIFY_FEATURE_DIRECTORY` (módulo `11`), y actualizar módulos,
  `prompts/`, la tabla por agente del `00` y las menciones a v1.0.8 en `README.md` y `00`.
- Al cambiar prompts: editar el `.txt`, copiar al bloque incrustado del módulo y revisar la
  tabla de `README.md`.
