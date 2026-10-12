# Mini manual de Git

Este anexo reúne los comandos de Git que usa el curso, sobre todo a partir del módulo 09, con ejemplos tomados de ProductChat. No reemplaza a la documentación de Git: es la hoja que conviene tener abierta mientras trabajas. Si un comando no aparece aquí, probablemente no lo necesitas para este curso.

Todos los comandos requieren **Git 2.28 o posterior** (`git init -b` y `git switch` no existen en versiones antiguas). Compruébalo con `git --version`.

## Git y GitHub

**Git** es un sistema de control de versiones: registra cada cambio de un proyecto en tu computador y permite volver a cualquier punto de su historia. Funciona sin conexión. **GitHub** es un servicio web que aloja repositorios Git y agrega herramientas para colaborar: revisión de cambios, *issues*, *pull requests*, *releases*. Git es la herramienta; GitHub, uno de los lugares donde se publica lo que Git registra.

| | Git | GitHub |
|---|---|---|
| Qué es | Un programa | Un servicio web |
| Dónde vive el repositorio | En tu computador | En un servidor remoto |
| Funciona sin conexión | Sí | No |
| Para qué lo usamos | Registrar la historia de ProductChat y separar el trabajo por Spec | Publicar el proyecto y compartirlo |

```text
tu computador (repositorio local)  ── git push ──▶  GitHub (repositorio remoto)
                                   ◀── git pull ──
```

## Configuración inicial

Se hace una sola vez por computador. Git firma cada commit con estos datos.

| Comando | Para qué |
|---|---|
| `git --version` | Comprobar que Git está instalado y su versión |
| `git config --global user.name "Ana Pérez"` | Nombre que aparece en tus commits |
| `git config --global user.email "ana.perez@ejemplo.cl"` | Correo asociado a tus commits |
| `git config --global init.defaultBranch main` | Que los repositorios nuevos partan en la rama `main` |

## Las zonas de trabajo y los estados de un archivo

Git trabaja con tres zonas locales y una remota. Cada comando mueve cambios de una zona a otra:

```text
 carpeta de trabajo ── git add ──▶ área de preparación ── git commit ──▶ repositorio local ── git push ──▶ GitHub
       ◀── git restore ──                ◀── git restore --staged ──                         ◀── git pull ──
```

La **carpeta de trabajo** son tus archivos tal como los ves. El **área de preparación** (*staging*) es la selección de cambios que irán en el próximo commit. El **repositorio local** guarda la historia de commits. Separar la preparación del commit permite revisar qué se va a registrar antes de hacerlo.

| Estado | Qué significa | Cómo pasa al siguiente |
|---|---|---|
| Sin seguimiento (*untracked*) | Archivo nuevo que Git todavía no registra | `git add <archivo>` |
| Modificado | Archivo registrado que cambió desde el último commit | `git add <archivo>` |
| Preparado (*staged*) | Cambio seleccionado para el próximo commit | `git commit` |
| Confirmado (*committed*) | Cambio guardado en la historia | Ya está registrado |

`git status` muestra en qué estado está cada archivo. Por ejemplo, a mitad de SPEC003:

```text
$ git status
On branch 003-filtros-catalogo
Changes not staged for commit:
        modified:   src/productchat/catalog.py
Untracked files:
        specs/003-filtros-catalogo/
```

## Crear y guardar

| Comando | Para qué | Ejemplo |
|---|---|---|
| `git init -b main` | Convertir la carpeta actual en repositorio, con rama `main` | `git init -b main` |
| `git clone <url>` | Copiar un repositorio remoto a tu computador | `git clone https://github.com/usuario/productchat.git` |
| `git status` | Ver qué cambió y qué está preparado | `git status` |
| `git add <ruta>` | Preparar un archivo o carpeta | `git add specs/003-filtros-catalogo/` |
| `git add .` | Preparar todo lo que cambió | `git add .` |
| `git diff` | Ver los cambios aún no preparados | `git diff src/` |
| `git diff --cached` | Ver lo que ya está preparado, antes del commit | `git diff --cached` |
| `git commit -m "<mensaje>"` | Registrar lo preparado en la historia | `git commit -m "SPEC003: plan and tasks"` |

Revisa `git diff --cached` antes de cada commit. Es el último momento en que ves exactamente qué vas a registrar.

Evita `git commit -am`: prepara y registra de una vez los archivos ya conocidos, pero **omite los archivos nuevos**. En el módulo 10 eso dejaría fuera de la historia un archivo del parche.

### Qué no se versiona

El archivo `.gitignore` lista lo que Git debe ignorar. En ProductChat:

```text
.venv/
__pycache__/
.pytest_cache/
~$*.xlsx
.specify/feature.json
```

`~$*.xlsx` son los archivos temporales que Excel crea mientras la planilla está abierta. `.specify/feature.json` guarda cuál es la Spec activa en tu directorio, y cambia con cada `/speckit-specify`: no es historia del proyecto (ver módulo 11). El catálogo **sí** se versiona: los criterios de aceptación citan sus SKU.

## Mirar la historia

| Comando | Para qué | Ejemplo |
|---|---|---|
| `git log --oneline` | Historia compacta, un commit por línea | `git log --oneline` |
| `git log --oneline --graph --all` | Historia de todas las ramas, como gráfico | `git log --oneline --graph --all` |
| `git log -p <archivo>` | Cada cambio que tuvo un archivo | `git log -p specs/001-consulta-catalogo/spec.md` |
| `git show <commit>` | Qué cambió en un commit | `git show a1b2c3d` |
| `git show --stat <commit>` | Solo la lista de archivos de un commit | `git show --stat HEAD` |
| `git show <rama>:<ruta>` | Leer un archivo de otra rama sin cambiarte a ella | `git show 005-exportar-csv:specs/005-exportar-csv/spec.md` |

```text
$ git log --oneline
e4f5a6b (HEAD -> 003-filtros-catalogo) SPEC003: implement and test filters
c3d4e5f SPEC003: plan and tasks
a1b2c3d SPEC003: specify and clarify filters
9f8e7d6 (main) ProductChat: base implementation with SDD artifacts
```

`HEAD` es el commit en el que estás parado. Los siete caracteres iniciales son el identificador abreviado del commit (*hash*), el que usas en `git show`.

## Ramas

Una rama es una línea de trabajo separada. En el curso, cada Spec se desarrolla en su propia rama y se integra en `main` solo cuando está verificada.

| Comando | Para qué | Ejemplo |
|---|---|---|
| `git branch` | Listar las ramas y ver en cuál estás | `git branch` |
| `git switch -c <rama>` | Crear una rama y pasarte a ella | `git switch -c 003-filtros-catalogo` |
| `git switch <rama>` | Pasarte a una rama existente | `git switch main` |
| `git merge <rama>` | Integrar otra rama en la actual | `git merge 003-filtros-catalogo` |
| `git branch -d <rama>` | Borrar una rama ya integrada | `git branch -d 003-filtros-catalogo` |

```text
main     ──●──────────────────────●──   ← git merge 003-filtros-catalogo
            \                    /
003-…        ●──────●──────●────
           specify  plan  implement
```

Para integrar, primero te cambias a la rama que recibe (`git switch main`) y después integras la otra (`git merge …`). En material antiguo verás `git checkout` en lugar de `git switch`; hacen lo mismo, pero `switch` es más claro.

### Conflictos

Si dos ramas cambiaron las mismas líneas, Git no elige: detiene el merge y marca el archivo así:

```text
<<<<<<< HEAD
DEFAULT_SORT = "price"
=======
DEFAULT_SORT = "name"
>>>>>>> 005-exportar-csv
```

Lo de arriba es tu rama; lo de abajo, la que estás integrando. Para resolverlo, edita el archivo hasta dejar la versión correcta, sin las marcas, y después ejecuta `git add <archivo>` y `git commit`. En SDD, la versión correcta no la decides mirando el código: la decide la Spec. El módulo 11 trabaja este caso.

## Trabajo en paralelo y entregas

| Comando | Para qué | Ejemplo |
|---|---|---|
| `git worktree add <carpeta> <rama>` | Abrir una rama en otra carpeta, para trabajar en dos a la vez | `git worktree add ../ProductChat-csv 005-exportar-csv` |
| `git worktree list` | Ver las carpetas abiertas | `git worktree list` |
| `git worktree remove <carpeta>` | Cerrar una carpeta sin cambios pendientes | `git worktree remove ../ProductChat-csv` |
| `git archive` | Exportar una copia de los archivos, sin historia | Ver módulo 12 |
| `git tag -a <versión> -m "<mensaje>"` | Marcar un commit como versión | `git tag -a v1.0 -m "First release"` |

## Trabajar con GitHub

| Comando | Para qué | Ejemplo |
|---|---|---|
| `git remote add origin <url>` | Conectar el repositorio local con uno en GitHub | `git remote add origin https://github.com/usuario/productchat.git` |
| `git remote -v` | Ver a qué repositorio remoto está conectado | `git remote -v` |
| `git push -u origin main` | Subir una rama por primera vez | `git push -u origin main` |
| `git push` | Subir los commits nuevos | `git push` |
| `git push origin --tags` | Subir las etiquetas de versión | `git push origin --tags` |
| `git pull` | Traer y aplicar los cambios del remoto | `git pull` |
| `git fetch` | Traer los cambios del remoto sin aplicarlos | `git fetch origin` |

En la web de GitHub, además, se trabaja con *forks* (copias de un repositorio en tu cuenta), *pull requests* (propuestas de cambio que otros revisan antes de integrarlas), *issues* (registro de problemas y tareas) y *releases* (versiones publicadas a partir de una etiqueta, descargables como zip).

## Deshacer sin miedo

Qué usar depende de hasta dónde llegó el cambio:

| Situación | Comando | Qué hace |
|---|---|---|
| Cambiaste un archivo y quieres volver al último commit | `git restore <archivo>` | Descarta tus cambios en ese archivo |
| Preparaste algo que no debía ir en el commit | `git restore --staged <archivo>` | Lo saca del área de preparación; tus cambios siguen ahí |
| Hiciste commit, todavía sin push, y falta algo o el mensaje está mal | `git commit --amend` | Rehace el último commit |
| El commit ya está en GitHub | `git revert <commit>` | Crea un commit nuevo que anula el anterior |

`git restore <archivo>` no tiene vuelta atrás: lo que descarta no queda en ninguna parte.

Dos comandos que verás recomendados en internet y que conviene evitar mientras aprendes: `git reset --hard`, que borra sin aviso todos los cambios no registrados, y `git push --force`, que reescribe la historia en GitHub y puede borrar el trabajo de otros. `git revert` resuelve casi siempre el mismo problema sin destruir nada.

## Convenciones del curso

- **Una rama por Spec**, con el mismo nombre que su carpeta en `specs/`: `003-filtros-catalogo`, `004-ocultar-sin-stock`.
- **Mensajes de commit en inglés**, con el identificador de la Spec como prefijo: `SPEC003: specify and clarify filters`. Un commit por etapa del ciclo, no uno al final.
- **Commits pequeños y con sentido.** Si el mensaje necesita la palabra *and* para unir dos cosas sin relación, probablemente eran dos commits.
- **Nada se integra en `main` sin verificar.** Primero las pruebas y los criterios de aceptación; después el merge.

## Referencia rápida

| Comando | Para qué | Módulo |
|---|---|---|
| `git init -b main` | Crear el repositorio | 09, 12 |
| `git status` | Ver el estado | 09 |
| `git add` | Preparar cambios | 09–12 |
| `git diff --cached` | Revisar lo preparado | 09 |
| `git commit -m` | Registrar | 09–12 |
| `git switch -c` | Crear rama | 09–11 |
| `git switch` | Cambiar de rama | 09–11 |
| `git merge` | Integrar una rama | 09–11 |
| `git log --oneline` | Ver la historia | 09 |
| `git log -p` | Historia de un archivo | 09 |
| `git show` | Ver un commit o un archivo de otra rama | 09, 11 |
| `git worktree` | Trabajar en dos ramas a la vez | 11 |
| `git archive` | Exportar sin historia | 12 |
| `git restore` | Deshacer cambios locales | Anexo |
| `git revert` | Anular un commit publicado | Anexo |
| `git push` / `git pull` | Sincronizar con GitHub | Anexo |

Para profundizar: Chacon, S. y Straub, B. (2014). [*Pro Git*](https://git-scm.com/book/es/v2) (2.ª ed.). Apress. Disponible en español, de libre acceso.

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/deed.es) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
