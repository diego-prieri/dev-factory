---
description: Crea .claude/factory/convenciones.md en el repo actual a partir de la plantilla, completándolo con lo que se puede leer del código y preguntando el resto.
argument-hint: "[--forzar]"
allowed-tools: Read, Write, Glob, Grep, Bash(mkdir:*), Bash(git log:*), Bash(git branch:*), Bash(git remote:*), AskUserQuestion
---

Preparás el repo actual para usar los flujos de `factory`. El resultado es un único archivo: `.claude/factory/convenciones.md`.

## Paso 0 — ¿Ya existe?

Si `.claude/factory/convenciones.md` existe y no vino `--forzar` en `$ARGUMENTS`, no lo pises: leelo, listá las secciones que siguen con placeholders `<...>` o vacías, y ofrecé completar solo esas.

## Paso 1 — Leer la plantilla

Leé `${CLAUDE_PLUGIN_ROOT}/plantillas/convenciones.md`. Respetá sus secciones y su orden exactos: los comandos y agentes las buscan por nombre.

## Paso 2 — Completar lo deducible del repo

Completá cada sección con evidencia real, citando archivos:

- **Stack**: manifiestos (`package.json`, `Gemfile`, `pyproject.toml`, `go.mod`, ...) y configs de framework.
- **Estructura de directorios**: `Glob` del primer y segundo nivel; describí solo las carpetas con código propio.
- **Patrones y nombres**: buscá 2-3 archivos representativos por tipo de pieza y describí el patrón que comparten, citando uno como modelo.
- **Verificación**: scripts de `package.json`/`Makefile`/`Rakefile`, config del runner de tests y su umbral de cobertura. Si hay un umbral global, chequeá si el repo lo cumple antes de recomendar usarlo; si no lo cumple, anotalo como trampa.
- **Git y commits**: rama por defecto (`git remote show origin` o `git branch`), formato real de los mensajes (`git log --format=%s -n 50`), convención de nombres de branch.
- **Documentación**: carpetas de docs existentes.
- **E2E**: config de la herramienta, carpeta de specs, helper de login. Variables de credenciales: solo los **nombres**, nunca valores.

Leé también `CLAUDE.md` del repo si existe: suele tener antipatrones y reglas de negocio ya escritas.

## Paso 3 — Preguntar lo que no se deduce

Con AskUserQuestion, en una sola ronda, preguntá lo que no salga del código. Típicamente:

- ¿El flujo puede commitear solo, o solo sugerir el mensaje?
- ¿Puede pushear y abrir PR (draft)?
- Dominios sensibles y reglas de negocio que un test no detecta.
- Repos relacionados del mismo producto.

## Paso 4 — Escribir

`mkdir -p .claude/factory` y escribí el archivo. Lo que no se pudo deducir ni fue respondido queda con su placeholder `<...>` — nunca lo inventes.

Cerrá mostrando qué secciones quedaron completas, cuáles con placeholder, y recordá que el archivo se commitea con el repo para que lo use todo el equipo.
