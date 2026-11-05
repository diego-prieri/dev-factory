# Método de bugfixing

> Reglas del motor, iguales para cualquier repo. Lo propio de cada repo (comandos de verificación, ruido del árbol, formato de commits y de docs) está en `.claude/factory/convenciones.md` del repo de trabajo. Si este método y las convenciones del repo chocan, **mandan las convenciones del repo**.

## Regla de oro: agrupar por causa raíz, no por ítem

Un punteo de QA lista **síntomas**, no bugs. Dos líneas distintas del punteo suelen ser el mismo bug visto desde dos pantallas, y se arreglan con un solo cambio en un componente compartido.

Ejemplo: "no deja scrollear el panel de filtros" aparece reportado en la pantalla de Pedidos y en la de Clientes. Si las dos usan el mismo panel compartido, es **un** fix ahí. Fixear por ítem duplica el trabajo y genera dos cambios en conflicto sobre el mismo archivo.

Corolario: la unidad de trabajo del flujo es el **grupo de causa raíz**, no la línea del punteo. Un grupo puede cerrar varios ítems a la vez.

La regla vale en las dos direcciones: un síntoma igual en dos módulos puede ser **dos** causas distintas si cada módulo tiene su propia copia o su propia configuración encima de una pieza compartida. Se confirma leyendo el código, nunca por el parecido del síntoma.

## Regla de oro: verificar antes de fixear

En un punteo típico, buena parte de los ítems **ya están resueltos**: el punteo se junta durante días y se procesa después. Fixear a ciegas es trabajo perdido y riesgo de romper lo que ya anda.

Todo grupo se verifica contra el código real antes de tocarlo, y el resultado se registra: `RESUELTO` / `PARCIAL` / `PENDIENTE` / `NO-VERIFICABLE`.

## Regla de oro: test rojo antes del fix

Todo grupo tipificado como `bug` necesita un test que **falle antes del cambio y pase después**. Sin el rojo previo no hay prueba de que el test cubra el bug: puede estar pasando por otro motivo.

Un rojo solo cuenta si es **la aserción esperada fallando con el valor que describe el síntoma** (ver "El exit code no alcanza").

Excepción: los grupos `copy`, `config` o `layout` sin lógica testeable van sin test. Borrar un campo de un archivo de config o renombrar un label no lleva test propio.

## Tipos de ítem

| Tipo | Qué es | Tratamiento |
|---|---|---|
| `bug` | Algo anda mal | Fix + test de regresión |
| `mejora` | Feature request disfrazada ("agregar el filtro por estado") | Confirmar alcance antes; puede no entrar al branch |
| `copy` | Texto/label a cambiar | Fix sin test |
| `config` | Sacar/renombrar/reordenar algo de configuración | Fix sin test |
| `layout` | Espaciado, alineación, overflow o responsive | Sin test si es solo estilo; con test si hay lógica condicional |
| `performance` | Anda, pero lento o con consumo excesivo | Medir antes y después; test si hay lógica |
| `otro-repo` | Depende de otro repo del producto (ver `## Repos relacionados`) | No se fixea acá; se registra con qué haría falta del otro lado |
| `decision` | No está roto: falta que alguien decida qué debe pasar | **Preguntar antes de tocar nada** |

Un ítem `decision` es el que más caro sale confundir con un `bug`. Señal típica: el reporte describe un comportamiento sin decir cuál sería el correcto, o el "correcto" contradice una decisión anterior deliberada. Ante la duda, es `decision`.

Un `otro-repo` sin haber mirado el código de este repo es una asunción, no una conclusión: antes de tipificarlo así, buscá el endpoint, payload o componente relevante de este lado.

## El árbol sucio es normal

`git status --porcelain` en la máquina del dev suele tener ruido permanente (ver `## Ruido conocido del árbol` en las convenciones del repo).

**Nunca abortar por working tree sucio.** Se toma un baseline al empezar (qué archivos ya estaban modificados, qué tests ya estaban en rojo) y se compara contra eso al final. Lo que importa es que el flujo no haya *empeorado* el baseline, no que el árbol esté impecable.

## Verificación acotada

Se verifica **solo lo tocado**, con los comandos de `## Verificación` de las convenciones del repo. Nunca la suite entera con umbrales globales: un umbral global que el repo no cumple da un rojo que no dice nada sobre el fix.

### El exit code no alcanza

Un runner de tests sale con error por al menos cuatro motivos distintos, y solo uno significa lo que buscás:

1. Una aserción falló — el único caso útil.
2. No encontró tests con ese patrón (`0 matches`, `0 examples`).
3. La suite tiene un error de sintaxis o de import y no llegó a correr.
4. Un test se pasó del timeout.

Antes de dar por válido un rojo o un verde, **leé la salida** y confirmá cuál de los cuatro es.

## Reversión no destructiva

Nunca `git checkout --`, `git restore` ni `git reset --hard` para deshacer un grupo: son irreversibles y no distinguen tu cambio del trabajo del dev. Se usa `git stash push -- <archivos>` (recuperable), y los archivos que ya estaban sucios en el baseline no se tocan con git.

## Commits

Granularidad: **un commit por causa raíz**, no por ítem del punteo. Qué entra: código + su test + la sección del `.md`, todo junto. Se stagean archivos **por nombre**, jamás `git add -A`: se llevaría el ruido del baseline.

Si el flujo puede commitear solo o solo sugiere el mensaje, y con qué formato, lo dice `## Git y commits` de las convenciones del repo.

## Documentación de un fix

Dónde va y con qué plantilla lo dice `## Documentación` de las convenciones del repo. Cuándo documentar un grupo — solo si se cumple al menos una:

- La causa raíz no se lee del diff.
- Hay una asunción sobre el contrato con otro repo o servicio.
- Hubo una decisión de producto.
- Queda algo pendiente de verificación manual.

Si todo lo que la sección diría es lo que el diff ya muestra, no se escribe.

Cuando hay una asunción sin verificar, se documenta en cuatro partes: qué se asumió → de qué patrón se infirió → qué exactamente *no* se verificó → qué archivo tocar si la asunción falla.
