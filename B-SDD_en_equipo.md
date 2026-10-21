# SDD en equipo

Este anexo reúne las reglas y prácticas para aplicar SDD cuando el proyecto lo desarrollan varias personas, cada una con su agente. Conviene leerlo al formar el grupo, antes de escribir la primera Spec, y tenerlo a mano durante el proyecto. Asume que ya conoces el ciclo de la primera parte y las ramas por Spec del módulo 09.

## La idea central

Cuando trabajas solo, la intención del producto vive en tu cabeza. En un equipo, eso deja de servir: cada persona y cada agente necesitan leer lo mismo para construir lo mismo. La Spec es ese acuerdo escrito, y todo lo que sigue sirve para protegerlo. Se resume en tres reglas:

1. **Cada regla compartida tiene un solo dueño.** El dueño puede ser una persona o una Spec, pero nunca dos a la vez.
2. **La Spec se revisa antes que el código.** Un error en la Spec cuesta minutos; el mismo error en el código cuesta días.
3. **Nada entra en `main` sin respaldo en una Spec.** Es la puerta que protege al resto del equipo.

## El acuerdo de equipo

Antes de empezar, el grupo toma algunas decisiones una sola vez. Si no se toman explícitamente, cada persona las toma a su manera y el repositorio se llena de estilos distintos.

| Decisión | Recomendación | Dónde queda |
|---|---|---|
| Modo de trabajo con las Specs | *Flow-forward*: cada cambio en una Spec nueva que declara qué reemplaza (módulo 10) | Constitución |
| Ramas | Ramas de Git, publicadas en GitHub: una por Spec, con el nombre de su carpeta, por ejemplo `003-filtros-catalogo` | Constitución |
| Mensajes de commit | En inglés, con el prefijo de la Spec: `SPEC003: plan and tasks` | Constitución |
| Quién aprueba una Spec | El responsable de producto y una persona distinta del autor (de preferencia, el dueño de una regla compartida que la Spec use, o quien la implementará) | Constitución |
| Quién integra en `main` | El integrador de turno, nunca el autor del cambio | Constitución |
| Agentes | Cada uno usa el que prefiera, Claude Code o Codex, con los mismos prompts (ver "Equipos con agentes distintos") | `.specify/` y `AGENTS.md` |
| Dónde se registran las Specs | En el repositorio compartido en GitHub; el índice `specs/README.md` de `main` reserva el número de cada una (ver "Dónde viven las Specs") | Repositorio |

Las decisiones que el equipo no negocia van en la Constitución, que ya cumple ese papel para el agente. Basta con agregar una sección como esta a `.specify/memory/constitution.md`:

```text
## Reglas de trabajo del equipo

1. Todo cambio de comportamiento empieza en una Spec. Ningún cambio de
   comportamiento se integra en main sin una Spec que lo respalde.
2. Las Specs se revisan y aprueban antes de planificar. Aprueban el
   responsable de producto y una persona distinta del autor.
3. Cada regla compartida (precios, stock, categorías, duplicados) tiene una
   Spec dueña. Las demás Specs la citan; no la redefinen.
4. Una rama por Spec, con el nombre de su carpeta en specs/. Un commit por
   etapa, con el prefijo SPECnnn.
5. Integra el integrador de turno, nunca el autor. Antes de integrar: pruebas
   en verde y auditoría de drift sin hallazgos pendientes.
```

Como el agente lee la Constitución antes de cada etapa, estas reglas también lo obligan a él: si le pides implementar algo sin Spec, debería señalarlo.

## Dónde viven las Specs

Cuando trabajas solo, las Specs están en tu disco y nadie más las toca. En un equipo, cada persona trabaja en su propio computador, con su propia copia del repositorio. La versión que vale para todos es la de GitHub: las Specs viajan de un computador a otro a través de ramas y *pull requests*, igual que el código. Lo que está solo en tu disco, para el resto del equipo, no existe.

Ese reparto produce tres problemas concretos:

| Problema | Por qué ocurre | Cómo se evita |
|---|---|---|
| Dos personas crean la Spec 007 | `/speckit-specify` asigna el siguiente número libre que encuentra en tu copia de `specs/`, y no sabe lo que otro creó en su computador (módulo 11) | Reservar el número en `main` antes de especificar |
| Alguien especifica sobre requisitos que ya cambiaron | Su copia de `main` está desactualizada | Actualizar `main` antes de empezar cada Spec |
| Una Spec nueva pasa días sin que nadie la vea | Está solo en el computador de su autor | Publicar la rama apenas exista `spec.md` |

La reserva se hace en el índice `specs/README.md`. Quien quiere empezar una Spec actualiza `main`, agrega su línea al índice y la publica. Si al publicar GitHub rechaza el cambio porque otra persona reservó primero, actualiza de nuevo y toma el número siguiente:

```bash
git switch main
git pull
# agregar al índice la línea: | SPEC007 | Ordenar resultados por precio | Reservada | — |
git add specs/README.md
git commit -m "SPEC007: reserve number"
git push

git switch -c 007-ordenar-resultados
```

Después, al ejecutar `/speckit-specify`, se fija el directorio en el prompt, como en el módulo 11: "La spec.md de SPEC007, en el directorio `specs/007-ordenar-resultados` (úsalo como SPECIFY_FEATURE_DIRECTORY)". Así el número lo decide el índice compartido y no la copia local de cada uno.

Una última regla de convivencia: no se edita la Spec de otra persona en su rama. Las observaciones se hacen como comentarios en su *pull request*, y el autor decide cómo incorporarlas.

## Equipos con agentes distintos

Que unos miembros usen Claude Code y otros Codex no es un problema: las Specs, los planes y las tareas son archivos de texto, y los prompts del curso sirven para los dos agentes. Hay que preparar tres cosas:

1. **Instalar las dos integraciones en el mismo proyecto.** Spec Kit 1.0.8 lo permite sin tocar la Constitución ni las Specs:

   ```bash
   specify init . --integration claude     # al crear el proyecto
   specify integration install codex       # agrega Codex al mismo proyecto
   specify integration status              # debe listar claude y codex
   ```

   El repositorio queda con `.claude/skills/` y `.agents/skills/`, y cada miembro invoca las operaciones con la sintaxis de su agente: `/speckit-specify` en Claude Code, `$speckit-specify` en Codex. No uses `specify init --force` sobre un proyecto que ya tiene Constitución: puede sobrescribirla.
2. **Mantener un solo archivo de contexto.** Codex lee `AGENTS.md` y Claude Code lee `CLAUDE.md`. Para no mantener dos archivos que terminan diciendo cosas distintas, escribe el contexto en `AGENTS.md` y deja en `CLAUDE.md` una sola línea que lo importe: `@AGENTS.md`.
3. **Revisar igual a todos.** Dos modelos distintos redactan de forma distinta, y uno puede producir Specs más detalladas que otro. La revisión de la Spec y la puerta de integración son las mismas para todos, y son las que mantienen pareja la calidad.

## Roles

En un grupo de tres o cuatro personas, los roles no son cargos fijos. Conviene rotarlos en cada entrega, para que todos pasen por todos.

| Rol | Qué decide | Qué no decide |
|---|---|---|
| Responsable de producto | Qué entra en cada Spec, cómo se resuelven las preguntas abiertas, qué regla manda cuando dos chocan | Cómo se implementa |
| Responsable técnico | El Plan: stack, modelo de datos, estructura del código | Qué hace el producto |
| Integrador | Si una rama está lista para entrar en `main` | El contenido de la Spec o del código |
| Todos | Especifican, implementan y revisan el trabajo de otros | Una regla compartida que tiene otro dueño |

> **Si vienes de Scrum.** El responsable de producto es el equivalente del *Product Owner*. La diferencia es que aquí sus decisiones quedan escritas en la Spec, porque el agente que implementa no estuvo en la conversación.

## El ciclo de un cambio en equipo

Así recorre el repositorio un cambio, desde que alguien lo propone hasta que entra en `main`:

| Paso | Quién | Qué se hace | Resultado |
|---|---|---|---|
| 1. Proponer | Cualquiera | Describe la necesidad en una línea, con un ejemplo concreto del catálogo | Un *issue* o una entrada en el índice |
| 2. Especificar | El autor | Reserva el número en el índice, crea la rama y la Spec con `/speckit-specify` y `/speckit-clarify`, y publica la rama | `spec.md` en GitHub |
| 3. Revisar la Spec | Responsable de producto y una segunda persona | Revisan solo `specs/`, antes de que exista código | Spec aprobada |
| 4. Planificar | El autor, con el responsable técnico | Plan, tareas y Analyze | `plan.md` y `tasks.md` |
| 5. Implementar | El autor y su agente | Implementación y pruebas por criterio de aceptación | Código en la rama |
| 6. Pedir integración | El autor | Abre el *pull request* con la plantilla de este anexo | PR listo para revisar |
| 7. Integrar | El integrador | Pasa la puerta de integración y hace el merge | Cambio en `main` |
| 8. Actualizar lo vigente | El integrador | Regenera la vista de requisitos vigentes con `10.5` | `specs/requisitos-vigentes.md` |

El paso 3 es el que más se salta y el que más ahorra. Se puede abrir un *pull request* que contenga solo la carpeta de la Spec: así la revisión ocurre en GitHub, queda registrada y nadie empieza a programar sobre una Spec que el resto no ha visto.

## Revisar una Spec: cuándo está lista

Antes de planificar, una Spec tiene que estar **lista**: otra persona, o su agente, debe poder implementarla sin preguntar nada. Revisarla no es corregir la redacción, sino comprobar eso. Esta lista sirve para cualquier proyecto:

1. **Propósito.** ¿Se entiende qué necesidad resuelve y para quién, sin leer el código?
2. **Criterios verificables.** ¿Cada criterio de aceptación tiene un dato de entrada concreto y un resultado esperado que se puede comprobar? En ProductChat, un SKU y lo que el chatbot debe responder.
3. **Reglas compartidas.** ¿Las reglas que pertenecen a otra Spec están citadas, y no redefinidas?
4. **Cambios declarados.** Si modifica algo que ya existía, ¿dice qué criterios de otras Specs reemplaza?
5. **Casos límite.** ¿Dice qué pasa con los datos faltantes, vacíos o contradictorios?
6. **Preguntas abiertas.** ¿Están todas resueltas? Si queda alguna, la Spec no está lista.
7. **Alcance.** ¿Queda claro qué está fuera de alcance?

Si alguna respuesta es no, la Spec vuelve a su autor. Devolver una Spec cuesta minutos; devolver un plan o un código construidos sobre ella cuesta mucho más.

> **Si vienes de Scrum.** Esta lista cumple el papel de una *Definition of Ready*: el acuerdo que algunos equipos usan para decidir cuándo un ítem del backlog puede entrar en un sprint. La puerta de integración de la sección siguiente se parece, a su vez, a la *Definition of Done*.

## La puerta de integración

Antes de cada merge en `main`, el integrador comprueba cuatro cosas:

1. **Las pruebas pasan,** incluidas las de las Specs anteriores. Que una Spec nueva rompa un criterio vigente es la forma más común de drift.
2. **Todo cambio en `src/` tiene respaldo en `specs/`.** El siguiente comando compara la rama con `main` y avisa si cambió `src/` sin ningún cambio en `specs/`:

   ```bash
   changed=$(git diff --name-only main...HEAD)
   if echo "$changed" | grep -q '^src/' && ! echo "$changed" | grep -q '^specs/'; then
     echo "Warning: src/ changed without any change in specs/"
   fi
   ```

   No toda modificación de código necesita una Spec: un cambio de nombre interno o una optimización no cambian el comportamiento. Pero el aviso obliga a preguntarlo.
3. **La auditoría de drift no tiene hallazgos pendientes.** Se ejecuta el prompt `prompts/10.2-auditoria-drift.txt` sobre la rama.
4. **La rama está al día con `main`.** Si no lo está, se actualiza primero y se vuelven a correr las pruebas.

Si el equipo quiere automatizar los pasos 1 y 2, puede llevarlos a un *hook* del agente o a la integración continua de GitHub. Conviene hacerlo solo cuando el equipo ya aplica la puerta a mano sin olvidarla: automatizar una práctica que todavía no existe no la crea.

## Plantilla de pull request

Si el repositorio incluye el archivo `.github/pull_request_template.md`, GitHub lo usa como texto inicial de cada *pull request*:

```markdown
## Spec

- Spec: SPEC00X, `specs/00X-nombre/`
- Reemplaza criterios de otras Specs: no / sí (cuáles)

## Qué cambia

Una o dos frases, desde el punto de vista del usuario.

## Cómo se verifica

| Criterio | SKU | Resultado esperado |
|---|---|---|
| CA1 | NB-1005 | Aparece entre los notebooks de 32 GB |

## Puerta de integración

- [ ] La Spec fue aprobada antes de implementar
- [ ] Todas las pruebas pasan, incluidas las de Specs anteriores
- [ ] Todo cambio en src/ tiene respaldo en specs/
- [ ] La auditoría de drift (10.2) no tiene hallazgos pendientes
```

## Cuando no todos usan SDD

Es habitual que alguien del grupo no quiera, o no alcance, a usar Spec Kit. Se puede trabajar igual, siempre que la regla se aplique al resultado y no a la herramienta:

- **Se exige la Spec, no Spec Kit.** Quien no use la herramienta puede escribir o editar `spec.md` a mano con la estructura del módulo 03. Lo que importa es que el cambio quede respaldado.
- **La puerta es la misma para todos.** Un cambio sin Spec no se integra, venga de quien venga.
- **El agente también puede ayudar a quien no usa SDD.** Pídele que redacte la Spec a partir del cambio ya hecho; es un *reverse spec* en pequeño, como en el módulo 12.

Cuando llega un cambio hecho directamente en el código, hay tres salidas:

| Situación | Qué hacer |
|---|---|
| El cambio es correcto y se quiere conservar | Escribir la Spec que lo respalda antes de integrar |
| El cambio contradice una Spec vigente | No integrar. Decidir con el dueño de la regla cuál manda y, si cambia, escribir una Spec nueva |
| Hubo una urgencia y ya está en `main` | Aceptarlo como parche y escribir su Spec en la siguiente sesión de trabajo, como en el módulo 10 |

El objetivo no es sancionar a nadie. Es que la Spec siga diciendo la verdad, porque el resto del equipo, y sus agentes, trabajan sobre ella.

## Situaciones frecuentes

| Situación | Qué hacer | Dónde se ve |
|---|---|---|
| Dos personas crearon una Spec con el mismo número | La que se reservó después en el índice cambia de número y de carpeta antes de integrarse | Este anexo, módulo 11 |
| Dos personas necesitan cambiar la misma regla | La decide el dueño de la regla, en una sola Spec; la otra persona la cita | Módulo 11 |
| Un compañero cambió el código sin Spec | No se integra hasta que exista la Spec, o se revierte | Este anexo, módulo 10 |
| Al implementar aparece una ambigüedad en la Spec | Se vuelve a la Spec y se precisa antes de seguir | Módulo 09, prompt `09.6` |
| Git marca un conflicto | La versión correcta la decide la Spec, no quien llegó primero | Anexo A |
| Dos Specs vigentes se contradicen | Aparece en la sección de conflictos de la vista vigente; se resuelve con una Spec nueva | Módulo 10, prompt `10.5` |
| Se integra una persona nueva al equipo | Lee `specs/README.md`, `specs/requisitos-vigentes.md` y `CLAUDE.md`, en ese orden | Módulos 09 y 10 |
| Cada uno usa un agente distinto | Las dos integraciones en el mismo proyecto y un solo archivo de contexto | Este anexo, módulos 00 y 09 |
| Alguien trabaja con *vibe coding* y entrega algo que funciona | Se reconstruye su Spec antes de integrar | Módulo 12 |

## Señales de alerta

Si aparece alguna de estas señales, el equipo está dejando de trabajar con Specs aunque siga creyendo que lo hace:

- Hay commits en `src/` sin ningún commit en `specs/` durante varios días.
- Las Specs se aprueban después de implementar, o no las revisa nadie.
- La misma regla (por ejemplo, qué hacer con un SKU con dos precios) aparece redactada en más de una Spec.
- La vista de requisitos vigentes no se ha regenerado desde hace varias integraciones.
- Alguien pregunta "¿al final cómo quedó lo del stock?" y la respuesta no está en ninguna Spec.

## Ritmo de trabajo

Si el equipo trabaja por *sprints* o por entregas semanales, la práctica se organiza así:

| Momento | Qué se hace |
|---|---|
| Inicio del sprint | Revisar y aprobar las Specs que se trabajarán. Es el equivalente del "qué" de la *Sprint Planning* (nota del módulo 04) |
| Durante el sprint | Una rama por Spec, commits por etapa, Spec revisada antes de implementar |
| Antes de cada merge | Puerta de integración: pruebas, respaldo en `specs/` y auditoría `10.2` |
| Después de cada merge | Regenerar `specs/requisitos-vigentes.md` con `10.5` |
| Cierre del sprint | Revisar los hallazgos de drift del período: ¿qué los causó y qué regla del acuerdo hay que ajustar? |

## Actividad breve

Para grupos de tres o cuatro personas, antes de empezar el proyecto:

1. Escriban el acuerdo de equipo y agréguenlo a la Constitución.
2. Asignen los roles de la primera entrega.
3. Hagan pasar un cambio pequeño por los ocho pasos del ciclo, por ejemplo, ordenar los resultados de una búsqueda por precio.

| Qué se entrega | Cómo se comprueba |
|---|---|
| Acuerdo | La Constitución tiene la sección de reglas de trabajo del equipo |
| Revisión de la Spec | Hay un *pull request* con solo `specs/`, aprobado por dos personas antes de que exista código |
| Integración | El merge lo hizo una persona distinta del autor, y el PR tiene la puerta de integración completa |
| Vista vigente | `specs/requisitos-vigentes.md` incluye el cambio nuevo |

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/deed.es) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
