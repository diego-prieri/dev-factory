# dev-factory

Fábrica de software para [Claude Code](https://claude.com/claude-code). Entran una doc funcional o un punteo de bugs y salen un PR o un diff listo para commitear. En el medio hay subagentes especializados, review independiente en cada paso y dos puntos donde decide una persona.

El motor es genérico. Lo propio de cada repo (stack, comandos de test, formato de commits, reglas de negocio) vive en un archivo de convenciones dentro de ese repo.

## Plugins

| Plugin | Qué trae |
|---|---|
| `factory` | `/factory:init`, `/factory:feature-spec` → `/factory:feature-build`, `/factory:bug-triage` → `/factory:bug-fix`, 12 subagentes, el método de bugfixing y la plantilla de convenciones |
| `factory-core` | Skills para redactar specs: `spec-builder` (documentación fundada en el código real, iterativa) y `spec-package` (requerimiento, criterios, diseño y plan) |

## Instalación

```bash
claude plugin marketplace add diego-prieri/dev-factory
claude plugin install factory@dev-factory
claude plugin install factory-core@dev-factory
```

Después, en la raíz de cada repo donde lo quieras usar:

```
/factory:init
```

Crea `.claude/factory/convenciones.md` a partir de [la plantilla](plugins/factory/plantillas/convenciones.md): completa lo que se puede leer del código y te pregunta el resto. Commitealo con el repo para que lo use todo el equipo.

## Flujo de features

| Input | Comando | Output |
|---|---|---|
| Doc funcional (+ prototipo HTML opcional) | `/factory:feature-spec <doc> [prototipo]` | `00-indice`, `01-flujo-usuario`, `02-definicion-tecnica`, `03-pasos-implementacion`, `04-documentacion` |
| `03-pasos-implementacion.md` | `/factory:feature-build <03> [--dry-run]` | Branch con un commit por paso, verificación y PR (draft si algo quedó en rojo) |

- **`feature-spec`** delega en `repo-scout`, `feature-flow-designer`, `feature-tech-architect`, `feature-impl-planner` y `feature-docs-writer`. Cada uno lee lo que dejó el anterior en disco. El plan sale con un prompt autocontenido por paso, pensado para un ejecutor con contexto limpio.
- **`feature-build`** ejecuta cada paso con `feature-step-runner`. Cada paso pasa por un gate doble: tests verdes **y** `VEREDICTO: GO` de `change-reviewer`, que juzga el cambio sin mirar los tests. Hay hasta 2 reintentos y cada veredicto queda en `reviews.log`. Con `--dry-run` no pushea: te muestra el log, el diff y los comandos para publicar.

## Flujo de bugs

| Input | Comando | Output |
|---|---|---|
| Punteo informal de QA/producto/soporte | `/factory:bug-triage <archivo o texto>` | `<branch>-plan.md` revisable |
| El plan, revisado a mano | `/factory:bug-fix <plan> [--solo G1,G3] [--sin-doc]` | Fixes grupo por grupo, doc y mensaje de commit sugerido |

Lo que un triage ingenuo hace mal, y este no:

- **Agrupa por causa raíz, no por ítem.** El mismo síntoma en dos pantallas suele ser un solo fix en una pieza compartida.
- **Verifica antes de fixear.** Cada grupo sale con veredicto `RESUELTO` / `PARCIAL` / `PENDIENTE` / `NO-VERIFICABLE` y su evidencia.
- **Separa lo que no es un bug.** Las mejoras, lo que depende de otro repo y las decisiones de producto pendientes se preguntan, no se asumen.

Y `bug-fix` garantiza:

- **Test rojo antes del fix**, confirmando que el rojo es la aserción esperada y no un `0 matches` o un timeout.
- **Baseline en vez de abortar.** El árbol sucio es normal: se registra qué estaba sucio y qué ya estaba en rojo, y se compara contra eso.
- **Review independiente** con el mismo gate doble que en features.
- **Reversión no destructiva**: `git stash push`, nunca `git checkout --`. Lo que ya estaba sucio no se toca.
- **Ledger en el plan** (`[ ] [~] [x] [!] [≁]`) para retomar si se corta la sesión.
- **Commits solo si las convenciones del repo lo permiten.** Si no, muestra el diff y el mensaje listos para copiar.

El método completo está en [`metodo/bugfix.md`](plugins/factory/metodo/bugfix.md).

## Cómo está armado

```
plugins/
├── factory/
│   ├── commands/      # orquestadores: init, feature-spec, feature-build, bug-triage, bug-fix
│   ├── agents/        # 12 subagentes, uno por responsabilidad
│   ├── metodo/        # reglas del motor, iguales para cualquier repo
│   └── plantillas/    # plantilla de .claude/factory/convenciones.md
└── factory-core/
    └── skills/        # spec-builder, spec-package
```

La regla que sostiene todo: **el método vive en el plugin y las convenciones viven en el repo.** Si cambia algo de tu stack, editás `.claude/factory/convenciones.md` de ese repo y listo. Si las dos cosas chocan, mandan las convenciones del repo.

## Licencia

[MIT](LICENSE)
