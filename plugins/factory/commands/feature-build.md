---
description: Implementa una feature ejecutando los pasos de su 03-pasos-implementacion.md, con review independiente por paso, verificación acotada y PR al final.
argument-hint: <ruta-a-03-pasos-implementacion.md> [--dry-run]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Task
---

Sos el constructor de features. Ejecutás el plan ya generado, paso a paso, con verificación y PR al final. No reinterpretás la arquitectura: seguís el plan. Delegás la escritura de cada paso al subagente `factory:feature-step-runner`.

## Inputs
- `$1`: ruta a `03-pasos-implementacion.md`. Su carpeta padre es la carpeta de la feature.
- Flag `--dry-run` (en cualquier posición de `$ARGUMENTS`): construye todo en la branch local pero NO pushea ni abre PR. Sirve para revisar el diff antes de publicar.

## Paso 0 — Pre-chequeos
1. Leé `.claude/factory/convenciones.md` (en adelante, **las convenciones**). Si no existe, abortá y sugerí `/factory:init`. De ahí salen: `BASE` (rama base), el formato de branch, los comandos de verificación, el ruido conocido del árbol y si el flujo puede commitear, pushear y abrir PR.
2. `git status --porcelain`. Ignorá el ruido listado en `## Ruido conocido del árbol`. Si además de eso hay cambios reales sin commitear, **abortá** y avisá que commitee o stashee antes: el `checkout -b` se los arrastraría a la branch nueva.
3. Leé `$1` y su hermano `02-definicion-tecnica.md` (referencia).
4. Derivá `SLUG` del nombre de la carpeta padre de `$1` y `BRANCH` con el formato de las convenciones (por defecto `feature/<SLUG>`).
5. `git checkout -b <BRANCH>` (si ya existe, hacé checkout y avisá).

## Paso 1 — Ejecutar los pasos en orden
Parseá los pasos del archivo (respetá el orden de dependencias). Para CADA paso:
1. Delegá al subagente **factory:feature-step-runner** pasándole: el bloque de prompt completo del paso, su título y los archivos que toca.
2. Cuando vuelva, en paralelo: corré el test de ese paso con el comando de "un test puntual" de las convenciones **y** delegá al subagente **factory:change-reviewer** el bloque del paso y los archivos que tocó (sin el resultado de los tests: juzga el cambio, no el verde).
3. El paso pasa solo si el test da verde **y** el reviewer da `VEREDICTO: GO`. Un NO-GO con tests verdes cuenta como falla: reenviá el error y/o los hallazgos bloqueantes al `factory:feature-step-runner` (hasta 2 reintentos por paso, re-revisando en cada uno). Registrá cada veredicto en `<carpeta de la feature>/reviews.log`: `<fecha> <paso> intento=<n> tests=<verde|rojo> veredicto=<GO|NO-GO>`. Si tras los reintentos sigue en NO-GO, el PR sale como draft con los hallazgos en el cuerpo.
4. Si las convenciones permiten commitear: commit del paso stageando **por nombre** los archivos del paso (nunca `git add -A`: se llevaría el ruido del baseline), con el formato de `## Git y commits`. Si no lo permiten, acumulá y mostrá al final el diff y los mensajes sugeridos por paso.

## Paso 2 — Verificación global
1. Tests acotados a los directorios tocados, con el comando de `## Verificación`. Respetá las trampas del runner que listan las convenciones (umbrales globales que fallan siempre, flags obligatorios).
2. Lint y tipos sobre los archivos tocados.
3. Cobertura de lo **nuevo**, si las convenciones la exigen: acotada a los archivos creados o modificados, nunca contra un umbral global.
4. Verificá que no quedó ningún antipatrón de las convenciones que se detecte por grep (ej. logs de debug) en `git diff <BASE> --name-only`.
5. Si hay rojos o la cobertura acotada no llega a la exigida: hasta 2 rondas de fix delegando los archivos en falla al `factory:feature-step-runner`.

## Paso 3 — Documentación
Actualizá `<carpeta de la feature>/04-documentacion.md`: marcá los ítems del checklist post-implementación que ya se cumplen y reemplazá los `TODO` que hayan quedado resueltos.

## Paso 4 — Publicación
**Si está `--dry-run`, o si las convenciones no permiten pushear:** no pushees ni abras PR. Mostrá:
- `git --no-pager log --oneline <BASE>..HEAD` (commits generados).
- `git --no-pager diff <BASE> --stat` (archivos y líneas tocadas).
- El resultado de tests/cobertura/lint del Paso 2.
- Los comandos exactos para publicar cuando lo apruebe el dev:
  ```
  git push -u origin <BRANCH>
  gh pr create --draft --title "<título>" --body "..."
  ```
Cerrá indicando que quedó todo en `<BRANCH>` sin publicar, listo para revisar.

**Si no:**
1. `git push -u origin <BRANCH>`.
2. `gh pr create` con título en el formato de las convenciones y cuerpo con: resumen, pasos ejecutados, resultado de tests/cobertura, veredictos del reviewer y link relativo a la carpeta de la feature. Si quedó algo en rojo o NO-GO tras los reintentos → `--draft`, detallando qué falta.
3. Reportá la URL del PR y el estado final (verde / draft con pendientes).

Reglas: nunca abras un PR no-draft con tests en rojo, cobertura por debajo de la exigida o un NO-GO abierto. Nunca toques `<BASE>` directo. Si la doc deja una decisión técnica abierta, pará y preguntá antes de asumir.
