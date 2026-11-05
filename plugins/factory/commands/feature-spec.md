---
description: Genera flujo de usuario, definición técnica, pasos+prompts y documentación de una feature a partir de su doc funcional (y prototipo HTML opcional).
argument-hint: <ruta-doc-funcional> [ruta-prototipo-html]
allowed-tools: Read, Write, Glob, Grep, Bash(mkdir:*), Bash(date:*), Task
---

Sos el orquestador del flujo de especificación de features. Tu trabajo es coordinar subagentes, NO escribir vos los deliverables (salvo el índice). Mantené la coherencia entre pasos pasándote los artefactos vía filesystem.

## Inputs
- Doc funcional: `$1` (obligatorio). Puede contener un prototipo HTML embebido en un bloque de código.
- Prototipo HTML: `$2` (opcional). Si se pasa, es un archivo `.html` aparte.

## Paso 0 — Preparación
1. Leé `.claude/factory/convenciones.md` (en adelante, **las convenciones**). Si no existe, **abortá** y sugerí correr `/factory:init` primero: sin convenciones los subagentes inventan el estilo del repo.
2. Leé el archivo de `$1`. Si `$2` existe, leelo también.
3. Derivá el nombre de la feature del título de la doc funcional y calculá `SLUG` = `<titulo-en-kebab-case>`.
4. Definí `OUTDIR` = `<carpeta de features de ## Documentación>/<SLUG>/` y creala con `mkdir -p`.

## Paso 1 — Contexto del repo
Delegá al subagente **factory:repo-scout** con la doc funcional. Pedile un brief con: piezas existentes parecidas que sirvan de modelo, acceso a datos reutilizable, tipos/interfaces relevantes, textos/claves de i18n ya existentes del dominio y la ubicación probable de la feature. Guardá su salida en `OUTDIR/_contexto-repo.md`.

## Paso 2 — Flujo de usuario
Delegá al subagente **factory:feature-flow-designer**. Pasale: ruta de la doc funcional, ruta del prototipo (`$2` o el bloque embebido) y `OUTDIR/_contexto-repo.md`. Debe escribir `OUTDIR/01-flujo-usuario.md`.

## Paso 3 — Definición técnica
Delegá al subagente **factory:feature-tech-architect**. Pasale: doc funcional, `OUTDIR/01-flujo-usuario.md` y `OUTDIR/_contexto-repo.md`. Debe escribir `OUTDIR/02-definicion-tecnica.md`.

## Paso 4 — Pasos de implementación + prompts
Delegá al subagente **factory:feature-impl-planner**. Pasale `OUTDIR/02-definicion-tecnica.md` y `OUTDIR/_contexto-repo.md`. Debe escribir `OUTDIR/03-pasos-implementacion.md` con un paso por unidad de trabajo, cada uno con su prompt autocontenido.

## Paso 5 — Documentación de la funcionalidad
Delegá al subagente **factory:feature-docs-writer**. Pasale los 3 archivos anteriores (`01`, `02`, `03`). Debe escribir `OUTDIR/04-documentacion.md`.

## Paso 6 — Índice
Escribí vos `OUTDIR/00-indice.md` con: nombre de la feature, fecha, links relativos a los 4 archivos y un resumen de 2-3 líneas. Borrá `OUTDIR/_contexto-repo.md` si querés mantener limpia la carpeta (opcional).

Al terminar, listá los archivos generados y avisá si algún subagente reportó ambigüedades o decisiones pendientes que conviene resolver antes de implementar. Cerrá con el comando para seguir:

```
/factory:feature-build <OUTDIR>/03-pasos-implementacion.md --dry-run
```
