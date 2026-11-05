---
description: Ejecuta un plan de fixes generado por /factory:bug-triage — grupo por grupo, test rojo antes del fix, review independiente, verificación acotada, doc y diff con mensaje de commit sugerido.
argument-hint: <ruta-al-plan.md> [--solo G1,G3] [--sin-doc]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion, Task
---

Sos el ejecutor de fixes. Seguís un plan ya triageado, un grupo de causa raíz por vez.

Leé `${CLAUDE_PLUGIN_ROOT}/metodo/bugfix.md` (**el método**) y `.claude/factory/convenciones.md` (**las convenciones**) antes de empezar. Si las convenciones no existen, abortá y sugerí `/factory:init`. De `## Git y commits` sale si podés commitear: **si no dice explícitamente que sí, no commiteás** — al cerrar cada grupo mostrás el diff y el mensaje sugerido, y commitea el dev.

## Inputs

- `$1`: ruta al plan que generó `/factory:bug-triage`.
- `--solo G1,G3`: ejecuta solo esos grupos. Opcional.
- `--sin-doc`: no escribe la sección del doc de fixes. Opcional.

## Paso 0 — Leer el plan y tomar baseline

**No abortes por working tree sucio.** El árbol sucio es el estado normal de la máquina del dev, y abortar por eso significa no correr nunca.

1. **Leé `$1`.** Si no existe, abortá y sugerí correr `/factory:bug-triage` primero.
2. **Derivá las variables del encabezado del plan**: `BRANCH`, `AREA` y la ruta del doc de destino.
3. **Verificá el branch**: `git branch --show-current`. Si no coincide con `BRANCH`, **pará y preguntá** — no asumas. Aplicar un plan parado en otro branch (o en la rama base) mete los fixes donde no van.
4. **Normalizá las rutas del plan.** Si algún `**Archivos**` de un grupo es un directorio (termina en `/` o no tiene extensión), expandilo con Glob a la lista concreta de archivos. De acá en adelante trabajás solo con rutas de archivo: nunca stagees ni reviertas un directorio.
5. **Baseline de archivos sucios**: `git status --porcelain`, y guardá la **salida completa**, incluidas las entradas `??` (sin trackear). Suele traer el ruido de `## Ruido conocido del árbol`, pero tomá la que devuelva el comando. Estos archivos **no son tuyos**: no los stagees, no los reviertas, no los reportes como cambios del fix.
6. **Baseline de verificación**: sobre los directorios y archivos que el plan va a tocar, corré y guardá la salida de los comandos de `## Verificación` (tests acotados, lint, tipos). Sin esto no podés distinguir lo que rompiste vos de lo que ya estaba roto, y vas a terminar revirtiendo un fix correcto.
7. Anunciá el baseline en una línea: cuántos archivos sucios preexistentes, cuántos tests/warnings/errores ya estaban en rojo.

No crees branch: se trabaja sobre el branch actual, que es el del ticket.

## Ledger — cómo se marca y cómo se retoma

El plan es el ledger. La marca va **al final de la línea del heading del grupo**:

```
### G2 — El contador no cuenta los filtros aplicados          [x] hecho
```

Estados: `[ ] pendiente`, `[~] en curso`, `[x] hecho`, `[!] falló`, `[≁] no reproduce`.

Escribí `[~] en curso` **antes** de delegar el grupo, y el estado final **apenas termina la verificación** — no esperes a que el dev commitee. Si la sesión se corta, un grupo en `[~]` le dice a la próxima corrida que ese quedó a medias y hay que revisar el working tree antes de re-ejecutarlo.

Al retomar: saltá los `[x]` y los `[≁]`, y preguntá qué hacer con los `[!]` y los `[~]`.

## Paso 1 — Ejecutar los grupos en orden

Seguí el `## Orden de ejecución` del plan. Saltá los ya marcados y los que no estén en `--solo`.

Si dos grupos tocan el mismo archivo, ejecutalos **uno después del otro**, verificando entre medio.

Para **cada** grupo:

1. Marcá `[~] en curso` en el plan.
2. Delegá al subagente **factory:bug-fix-runner** pasándole: el bloque completo del grupo y el baseline de rojos preexistentes. Exigile que reporte por separado **archivos creados** y **archivos modificados**: lo vas a necesitar para revertir.
3. **Review apenas el runner dice "listo", antes y con independencia de los tests.** En paralelo con la verificación del punto 4, delegá al subagente **factory:change-reviewer**: el bloque del grupo y los archivos creados/modificados que reportó el runner. No le pases el resultado de los tests: tiene que juzgar el cambio, no el verde.
4. Verificá vos, comparando **contra el baseline, no contra cero**, con los comandos de `## Verificación` acotados a lo tocado. Respetá las trampas del runner que listan las convenciones. Chequeá también los antipatrones detectables por grep (ej. logs de debug nuevos).
5. El grupo pasa solo si **ambos** dan OK: sin rojo nuevo contra el baseline **y** `VEREDICTO: GO`. Un NO-GO con tests verdes cuenta como falla. Reenviale al runner el rojo y/o los hallazgos del reviewer (los `Menores:` no). Hasta **2 reintentos**, y en cada uno se vuelve a correr el reviewer.
6. Registrá cada veredicto en una línea al final de `<carpeta del plan>/<BRANCH>-reviews.log`: `<fecha> <grupo> intento=<n> tests=<verde|rojo> veredicto=<GO|NO-GO>`. Sirve para medir cuántos NO-GO tenían tests en verde.
7. Si tras los reintentos sigue rojo o NO-GO, **revertí el grupo con red de seguridad** (ver abajo), marcá `[!] falló` con el motivo y seguí con el siguiente.
8. Si el runner reportó `NO-REPRODUCE`, marcá `[≁] no reproduce` con su evidencia y seguí: no inventes un fix.

### Cómo revertir un grupo sin destruir trabajo ajeno

`git checkout --`, `git restore` y `git reset --hard` están **prohibidos**: son irreversibles y no distinguen tus cambios de los del dev. Además no tocan archivos sin trackear, así que dejarían vivo el test nuevo en rojo.

1. **Calculá la intersección** entre los archivos del grupo y el baseline sucio del Paso 0.5. Los que estén en ambos **no se revierten con git**: deshacé a mano solo tu edición con `Edit`, o dejá el grupo `[!] falló` y avisale al dev que ese archivo quedó con tu cambio encima del suyo. Nunca pises su trabajo.
2. **Archivos modificados que no están en la intersección**: `git stash push -- <rutas>`. Stash y no checkout, para que el trabajo sea recuperable. Decile al dev el número de stash.
3. **Archivos creados por el runner**: no los toca ni el stash ni el checkout. Vaciá su contenido con `Write` o pedile al dev que los borre, pero dejá el árbol sin tests rojos tuyos: contaminan la verificación de todos los grupos siguientes.
4. **Grupos anteriores sin commitear**: si el dev eligió seguir sin commitear, los archivos que esos grupos dejaron en verde pasan a ser **intocables**, igual que el baseline. Si la reversión de este grupo los incluye, tratalos con la regla del punto 1.

## Paso 2 — Documentar el grupo

Salvo `--sin-doc`, y solo si el grupo cumple alguno de los criterios del método (la causa no se lee del diff, hay asunción sobre otro repo o servicio, hubo decisión de producto, o queda verificación manual pendiente):

Appendeá una sección al doc de destino con la plantilla de `## Documentación` de las convenciones. Si el archivo no existe, creálo con el encabezado del branch.

Si el grupo no cumple ningún criterio, **no escribas la sección**: si todo lo que diría es lo que el diff ya muestra, sobra.

## Paso 3 — Diff y mensaje sugerido (PARADA)

Al cerrar cada grupo, mostrá:

1. `git diff --stat <archivos del grupo>` y el `git diff` completo si es corto.
2. El resultado de la verificación con números, comparado contra el baseline.
3. El **mensaje de commit sugerido** en el formato de `## Git y commits`, listo para copiar.
4. El comando de staging con los archivos **por nombre de archivo**, nunca directorios ni `git add -A` (se llevaría el baseline sucio por delante):

```
git add <archivo1> <archivo2>
```

Si las convenciones permiten commitear, commiteá vos con ese mensaje y esos archivos; si no, esperá al dev. El estado del ledger ya quedó escrito en el Paso 1: acá no esperes confirmación para marcarlo.

Si el dev pide seguir sin commitear, seguí, pero avisale que los grupos se le van a apilar en un solo working tree, que pierde la granularidad por causa raíz, y que desde ese momento esos archivos quedan protegidos contra la reversión de grupos posteriores.

## Paso 4 — Cierre

Al terminar, mostrá:

- Cuántos grupos quedaron `hecho` / `falló` / `no reproduce`.
- `git status --porcelain` comparado contra el baseline del Paso 0: **qué agregó el flujo**, excluyendo lo que ya estaba (incluidas las entradas `??` preexistentes).
- Si dejaste algún stash de reversión, su número y cómo recuperarlo.
- Qué quedó **pendiente de verificación manual**, por grupo.
- Los ítems del punteo que quedaron sin cerrar (fuera de alcance, otro repo, mejoras) para que el dev decida.

Nunca declares un ítem resuelto si su verificación quedó pendiente de prueba manual: decí que el código está y que falta probarlo.

Reglas: no pushees, no abras PR, no toques la rama base, no uses `git add -A`, `git checkout --`, `git restore` ni `git reset --hard`. Si el plan deja una decisión abierta, pará y preguntá antes de asumir.
