# El entorno de trabajo en SDD

## Las dos capas del entorno

Para trabajar con **Spec Driven Development (SDD)** conviene distinguir dos capas que cumplen funciones diferentes:

```text
┌──────────────────────────────────────┐
│           CAPA DE MÉTODO             │
│                                      │
│ GitHub Spec Kit                      │
│ OpenSpec                             │
│ Kiro                                 │
│                                      │
│ Organiza el proceso SDD              │
│ Especificación → Plan → Tareas → ... │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│           CAPA DE AGENTE             │
│                                      │
│ Claude Code                          │
│ Codex                                │
│ Otros agentes / modelos locales      │
│                                      │
│ Razona, modifica archivos,           │
│ programa, ejecuta y verifica         │
└──────────────────────────────────────┘
```

La **capa de método** define cómo organizamos el desarrollo. Gestiona especificaciones, planes, tareas, controles de calidad y el flujo general de SDD.

La **capa de agente** es el motor que ejecuta ese método. El agente lee los artefactos, razona sobre ellos, escribe código, ejecuta comandos y verifica los resultados.

La separación es importante porque **método y agente son intercambiables**. Podemos utilizar GitHub Spec Kit como método y trabajar con Claude Code, Codex u otro agente sin cambiar la lógica fundamental del proceso.

En este curso utilizaremos:

```text
Capa de método  → GitHub Spec Kit v1.0.8
Capa de agente  → Claude Code o Codex
```

> **Sobre la versión.** Spec Kit evoluciona rápido y los nombres de sus comandos han cambiado entre versiones. Todo el material de este curso está comprobado contra la **versión 1.0.8**. Si instalas otra, revisa la sección "Comprobar la integración" antes de seguir.

## 1. Preparar la capa de agente

Antes de instalar Spec Kit necesitamos disponer de un agente de código: Claude Code, Codex u otro agente compatible.

La instalación y autenticación dependen del agente seleccionado. Una vez instalado, debemos comprobar que podemos iniciarlo desde la terminal dentro del directorio de un proyecto. Por ejemplo, con Claude Code:

```bash
claude
```

o con Codex:

```bash
codex
```

El agente siempre debe iniciarse desde el directorio del proyecto sobre el que queremos trabajar.

## 2. Instalar uv

Spec Kit se distribuye como una herramienta de Python y se instala con **uv**, un gestor de paquetes y entornos de Python. En macOS:

```bash
brew install uv
```

En otros sistemas, o si no usas Homebrew, sigue el instalador oficial de uv. Comprueba que quedó disponible:

```bash
uv --version
```

## 3. Instalar GitHub Spec Kit

> ⚠️ **Antes de ejecutar este comando, comprueba la versión.** Este material está fijado a la
> v1.0.8 y comprobado contra ella. Spec Kit se desarrolla rápido y **ya cambió una vez la sintaxis
> de sus operaciones**: las versiones anteriores a la 1.0 usaban `/speckit.specify`, con punto,
> donde la 1.0.8 usa `/speckit-specify`, con guion. Antes de instalar, revisa las releases del
> repositorio (`github.com/github/spec-kit`) y decide conscientemente: o instalas la v1.0.8 y el
> material funciona tal cual, o instalas una más reciente y asumes que tendrás que comprobar los
> nombres de las operaciones y adaptar los prompts. Lo que no conviene es instalar `@main` sin
> mirar: es una rama de desarrollo y puede cambiar de un día para otro.

Instalamos la versión que usa el curso, fijándola explícitamente:

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@v1.0.8
```

Esto deja disponible el comando `specify`. Si la terminal responde que no lo encuentra, es porque el directorio de herramientas de uv no está en el `PATH`:

```bash
uv tool update-shell
```

Después abre una terminal nueva y comprueba la instalación:

```bash
specify --version
```

Debería responder:

```text
specify 1.0.8
```

Spec Kit no sustituye al agente. Su función es proporcionar la estructura, los templates, los scripts y las instrucciones que permiten al agente trabajar siguiendo SDD.

## 4. Crear un proyecto

Creamos primero un directorio de trabajo:

```bash
mkdir sdd-curso
cd sdd-curso
```

A continuación inicializamos un proyecto indicando el agente que utilizaremos.

### Con Claude Code

```bash
specify init proyecto1 --integration claude
```

### Con Codex

```bash
specify init proyecto1 --integration codex
```

Spec Kit prepara automáticamente el proyecto para trabajar con el agente seleccionado. Si quieres ver qué integraciones admite tu instalación:

```bash
specify check
```

## 5. Llevar los archivos del caso al proyecto

Accedemos al proyecto:

```bash
cd proyecto1
```

y copiamos dentro los dos archivos del caso:

```text
proyecto1/
├── catalogo_productos.xlsx
└── spec_servilleta_productchat.md
```

Este paso importa más de lo que parece. Todos los prompts del curso declaran sus archivos de entrada por nombre; si no están en el proyecto, el agente no puede leerlos y acabamos pegando contenido a mano, que es justo lo que queremos evitar.

Ya podemos arrancar el agente dentro del proyecto:

```bash
claude
```

A partir de este momento tenemos las dos capas trabajando conjuntamente:

```text
GitHub Spec Kit
      │
      │ proporciona el método,
      │ templates, scripts e instrucciones
      ▼
Agente de IA
      │
      │ interpreta y ejecuta
      ▼
Proyecto
```

## 6. ¿Qué añade Spec Kit al proyecto?

Al inicializarlo aparecen dos grupos de archivos, todos ocultos.

El primero depende del agente. Con Claude Code, Spec Kit instala sus instrucciones como **skills** en `.claude/skills/`. Con Codex, en `.agents/skills/`. Son las instrucciones que seguirá el agente en cada etapa de SDD.

El segundo son los recursos propios de Spec Kit, en `.specify/`:

```text
.specify/
├── memory/         ← aquí vivirá constitution.md
├── templates/      ← estructuras de los artefactos
├── scripts/        ← operaciones auxiliares
├── integrations/   ← configuración del agente elegido
└── workflows/
```

Los `templates` proporcionan la estructura de cada artefacto:

```text
constitution-template.md
spec-template.md
plan-template.md
tasks-template.md
checklist-template.md
```

No es necesario modificar manualmente estos archivos. El agente los utilizará cuando ejecutemos las distintas etapas.

## 7. Comprobar la integración

Una vez iniciado el agente dentro del proyecto, comprobamos que aparecen disponibles las operaciones de Spec Kit. Se invocan como **slash commands**, escribiendo una barra seguida del nombre:

```text
/speckit-constitution
/speckit-specify
/speckit-clarify
/speckit-plan
/speckit-tasks
/speckit-analyze
/speckit-implement
```

Conviene entender bien qué es cada cosa, porque es fácil confundirlas. Lo que Spec Kit instala en el proyecto es una **skill**: un archivo de instrucciones que el agente leerá. Lo que nosotros escribimos es un **slash command**: la forma de invocar esa skill. Usamos la barra precisamente para que la acción quede explícita, tanto para el agente como para quien lee después lo que se hizo. Escribir `/speckit-plan` deja constancia de que se ejecutó la etapa de planificación; pedirlo en lenguaje corriente, no.

Por tanto, cuando escribimos:

```text
/speckit-specify
```

no estamos cambiando de agente. Seguimos utilizando Claude Code, Codex o el que hayamos elegido, pero ahora ese agente recibe las instrucciones proporcionadas por la **capa de método de Spec Kit**.

Esta separación puede resumirse así:

```text
Spec Kit dice QUÉ PROCESO seguir.
El agente ejecuta ese proceso.
```

Spec Kit 1.0.8 instala además `/speckit-checklist`, que usaremos en el último módulo, y otras dos operaciones que quedan fuera de este curso: `/speckit-converge`, que evalúa el código existente y añade el trabajo pendiente como tareas, y `/speckit-taskstoissues`.

> **Si instalaste otra versión.** Los nombres de las operaciones han cambiado entre versiones de Spec Kit: las anteriores a la 1.0 usaban un punto, `/speckit.specify`, en lugar del guion. Puedes ver los nombres exactos de tu instalación listando el directorio correspondiente, `ls .claude/skills/` o `ls .agents/skills/`, y sustituirlos en los prompts del curso.

## Entorno preparado

Una vez configuradas ambas capas, nuestro entorno queda:

```text
proyecto1
│
├── GitHub Spec Kit           ← Capa de método
│   ├── Skills
│   ├── Templates
│   └── Scripts
│
├── Claude Code / Codex       ← Capa de agente
│   ├── Razona
│   ├── Modifica archivos
│   ├── Genera código
│   ├── Ejecuta
│   └── Verifica
│
└── Archivos del caso
    ├── catalogo_productos.xlsx
    └── spec_servilleta_productchat.md
```

Con este entorno preparado podemos comenzar el ciclo de Spec Driven Development:

`Constitución → Especificación → Clarify → Plan → Tareas → Analyze → Implementación`
