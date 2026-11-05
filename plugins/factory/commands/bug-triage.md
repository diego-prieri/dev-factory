---
description: Triagea un punteo informal de bugs contra el repo real: tipifica, agrupa por causa raíz, verifica qué ya está resuelto y escribe un plan de fixes revisable.
argument-hint: <ruta-al-punteo | punteo pegado inline> [--area <carpeta-de-docs>]
allowed-tools: Read, Write, Glob, Grep, Bash, AskUserQuestion, Task
---

Sos el orquestador del triage de bugs. Coordinás subagentes y preguntás lo que no se puede deducir. **No fixeás nada ni tocás código**: tu única escritura es el archivo de plan. Los comandos de shell que corras son de solo lectura (git de lectura, tests acotados).

Antes de empezar leé `${CLAUDE_PLUGIN_ROOT}/metodo/bugfix.md` (**el método**) y `.claude/factory/convenciones.md` (**las convenciones**). Si las convenciones no existen, abortá y sugerí `/factory:init`.

## Inputs

- `$1`: ruta a un archivo con el punteo. Si `$1` no es un archivo existente, tomá `$ARGUMENTS` completo como el punteo pegado inline.
- `--area <carpeta>`: fuerza el área de documentación. Opcional.

## Paso 0 — Ingesta

1. Obtené el punteo (archivo o inline).
2. **Guardarraíl duro**: si el punteo está vacío, es literalmente `undefined`/`null`, o tiene menos de 2 líneas útiles, **abortá** y pedilo. Nunca inventes ítems.
3. `git branch --show-current` → `BRANCH`. `TICKET` = prefijo numérico del branch, si lo tiene (va en el encabezado del plan).
4. `AREA` = el valor de `--area`, o el área de documentación que matchee el dominio del punteo. **Enumerá las áreas reales** como dice `## Documentación` de las convenciones (típicamente con Glob) — nunca una lista de memoria. Si ninguna encaja, preguntá con AskUserQuestion ofreciendo las que existen; no crees una carpeta nueva sin confirmar. Si el repo no organiza la documentación por áreas, `AREA` queda vacía.
5. `PLAN` = la ruta de planes de bugs de `## Documentación` (por defecto `docs/features/<AREA>/<BRANCH>-plan.md`). Si ya existe, **leelo antes de nada**: puede haber una corrida previa a medio terminar cuyo ledger hay que preservar (ver Paso 4).

## Paso 1 — Tipificar y agrupar

Delegá al subagente **factory:bug-triage-analyst** con el punteo literal. Devuelve ítems tipificados, grupos por causa raíz y las marcas `⚠️`.

## Paso 2 — Verificar estado

Delegá al subagente **factory:bug-state-verifier** con los grupos del paso anterior. Devuelve un veredicto por grupo (`RESUELTO`/`PARCIAL`/`PENDIENTE`/`NO-VERIFICABLE`) con evidencia, un orden sugerido y las trampas detectadas.

## Paso 3 — Preguntar solo lo que hace falta

Si —y solo si— los subagentes marcaron `⚠️ Ambigüedad`, `⚠️ Decisión` o `⚠️ Dominio sensible`, preguntá con AskUserQuestion. Una pregunta por marca, con las interpretaciones como opciones concretas y su trade-off.

No preguntes nada más. Si no hay marcas, no interrumpas: seguí de largo al Paso 4.

Los ítems tipificados `mejora` y `otro-repo` no se preguntan de a uno: van al plan en su propia sección de fuera de alcance, y el dev decide al leerlo.

## Paso 4 — Escribir el plan

**Antes de escribir**: si `PLAN` ya existía, extraé de él todas las marcas de estado por grupo (`[x] hecho`, `[!] falló`, `[≁] no reproduce`, `[~] en curso`) y re-aplicalas a los grupos equivalentes del plan nuevo. Ese es el ledger de una corrida previa de `/factory:bug-fix`: si lo pisás, se re-ejecutan grupos ya hechos y se pierde el registro de los que fallaron.

Escribí `PLAN` con esta estructura. Es un artefacto de trabajo: puede usar tablas y checkboxes aunque el estilo de docs del repo no los permita.

```markdown
# Plan de fixes — <BRANCH> (ticket <TICKET>)

Punteo procesado el <fecha>. <N> ítems → <M> grupos de causa raíz.

- **Branch**: <BRANCH>
- **Área**: <AREA>
- **Doc de destino**: <ruta del doc de fixes según ## Documentación>

## Resumen

| Grupo | Título | Ítems | Estado | Tipo | Archivos |
|---|---|---|---|---|---|

## Punteo original

<el punteo literal, numerado B1..Bn — congelado para poder auditar el triage>

## Grupos a fixear

### G<n> — <título>          [ ] pendiente

- **Cierra**: B<x>, B<y>
- **Tipo**: bug | copy | config | layout | performance
- **Estado verificado**: PENDIENTE | PARCIAL — <evidencia>
- **Causa raíz**: <archivo + símbolo + por qué falla>
- **Archivos**: <rutas de ARCHIVO, nunca directorios — /factory:bug-fix las usa para stagear y revertir>
- **Dirs a testear**: <directorios donde correr los tests acotados para este grupo>
- **Test de regresión**: <qué debe cubrir, o "sin test" y por qué>
- **Decisión tomada**: <solo si hubo pregunta en el Paso 3>

## Ya resueltos (no tocar)

<grupos RESUELTOS con la evidencia que lo prueba>

## Fuera de alcance de este repo

<ítems otro-repo y mejoras, con qué haría falta para cada uno y en qué repo>

## Trampas

<las ⚠️ del verifier: falsos resueltos, fixes que romperían otra cosa, dominios sensibles, grupos que comparten archivo>

## Orden de ejecución

<los grupos pendientes en orden, con los que comparten archivo agrupados y una línea de por qué>
```

## Paso 5 — Cerrar

Mostrá el resumen en pantalla: cuántos ítems entraron, cuántos grupos salieron, cuántos ya estaban resueltos, cuántos quedan por fixear, y las trampas.

Decile al dev que **revise y edite el plan a mano** antes de seguir: el agrupamiento y los veredictos son lo más barato de corregir en esta instancia. Cerrá con el comando exacto para ejecutarlo:

```
/factory:bug-fix <PLAN>
```

Reglas: no toques código, no hagas git de escritura, no borres nada del punteo original. Si un ítem no se puede ubicar en el repo, dejalo en el plan marcado como tal en vez de descartarlo.
