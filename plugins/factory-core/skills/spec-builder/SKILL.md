---
name: spec-builder
description: "Use cuando alguien pide documentar un proyecto o feature partiendo de un requerimiento (una idea, un pedido de cliente, un mensaje de CS/soporte, un feature request) y hace falta que la documentación quede fundada en el sistema real: escanea el código de los repos relacionados que declara el repo de trabajo (front, back, servicios — resueltos por su `owner/repo` de GitHub, no por ruta fija) además de la documentación funcional ya existente, y en vez de tirar todo en una sola pasada, itera con el usuario ronda por ronda hasta que la documentación esté sólida — sin preguntas pendientes que dependen de una suposición. Activala para: armar el requerimiento y los criterios de aceptación de un feature nuevo, documentar un cambio grande o una integración, o escribir el brief técnico de traspaso a desarrollo. Si el pedido es chico y no necesita mirar código (redactar un requisito suelto, una especificación liviana dentro de un repo de código, el paquete requerimiento + criterios + diseño + plan sin scout), usá spec-package en su lugar."
argument-hint: "requerimiento, idea, pedido de cliente o feature a documentar"
user-invocable: true
---

# Spec Builder — Documentación de proyecto con scout de código

Convertí un requerimiento —una idea, un mensaje de cliente, un feature request, un flujo o una decisión pendiente— en documentación funcional sólida para el repo donde corrés la skill, fundada en lo que el código del producto **realmente hace hoy**, no en supuestos.

Lo que distingue a esta skill de `spec-package` (la liviana, que arma el paquete de cuatro capas sin mirar código):

1. **Hace scout de código real**, no solo de documentación en Markdown. Antes de escribir una línea, lee los repos relacionados (front, back o los que declare el repo de trabajo) para confirmar cómo se comporta hoy lo que el requerimiento toca.
2. **Itera de verdad.** No genera un borrador y una lista de preguntas al final para que las conteste quien lo pidió sin más intercambio. Trabaja en rondas: comparte lo que encontró, pregunta lo mínimo indispensable, espera la respuesta, refina, y solo cierra cuando no queda nada `Pendiente` que debería estar resuelto.
3. **Escribe en el estilo real de la documentación existente** (por ejemplo, archivos numerados que varían según el feature), no en una plantilla fija importada.

## Configuración del repo de trabajo

La skill no trae datos del producto precargados: los lee del archivo `.claude/factory/convenciones.md` del repo donde se corre. Usa tres secciones (nombres exactos):

- `## Repos relacionados` — lista de repos `owner/repo` con su rol (ej. front, back). Es lo que resuelve el Paso 0.
- `## Dominios sensibles` — temas que obligan a levantar el **flag de dominio sensible** y pedir validación explícita (ej. pagos, permisos, datos personales, integraciones con terceros; la lista real la define cada repo).
- `## Documentación` — dónde viven las docs funcionales, cómo están organizadas y con qué estilo se escriben.

Si el archivo no existe, o falta alguna de esas secciones, **preguntale al usuario** lo que falta en vez de asumirlo, y sugerile agregarlo a `.claude/factory/convenciones.md` para la próxima vez. Nunca completes una sección faltante con valores inventados.

## Principios que no se negocian

- **Español rioplatense**, tono funcional, directo, orientado a desarrollo — igual que el resto de la documentación.
- **El requerimiento dice QUÉ y POR QUÉ, no CÓMO.** El "cómo" es la especificación técnica, y ahí sí entra el código relevado en el scout.
- **No inventés.** Si el código no resuelve algo y tampoco lo resuelve el usuario en la iteración, marcalo `Pendiente` / `Supuesto` / `Necesita definición`. Nunca cierres una decisión inventando la respuesta.
- **Todo criterio de aceptación tiene que ser verificable** (se responde sí/no al probarlo) y **ninguno puede depender de algo `Pendiente`**. Ver `guia-redaccion.md`.
- **Cuando el scout encuentra algo, citalo** con `repo:ruta/archivo` (y línea si aplica), igual que citarías un documento. "El back considera vencida una tarea cuando `due_at < now()` en `api:src/tasks/status.ts:42`" pesa más que "el back debería calcular...".
- **Si el código contradice la documentación existente o lo que asume el pedido, decilo explícitamente** — no falles en silencio hacia ninguno de los dos lados. Un origen muy común de bugs es que un repo asuma el comportamiento de otro (por ejemplo, el front asumiendo lo que devuelve el back) sin verificarlo contra una respuesta real; el scout de esta skill existe justamente para no repetir ese error. Si algún repo relacionado documenta ese historial (en su `CLAUDE.md`, `AGENTS.md` o similar), leelo y no repitas lo que ya está verificado.
- **Flag de dominio sensible obligatorio.** Si el pedido toca alguno de los temas listados en `## Dominios sensibles` de `.claude/factory/convenciones.md`, marcalo y pedí validación explícita antes de asumir comportamiento. Si la sección no existe, preguntale al usuario qué temas considera sensibles antes de cerrar el borrador.
- **Trazabilidad simple.** Cada documento abre con un bloque de metadatos (por ejemplo una "Ficha documental") y una lista de "Relacionados" a los otros documentos del mismo paquete — si la documentación existente ya lo hace así, seguí ese esquema; no hace falta inventar uno de IDs nuevo.

## Paso 0 — Resolver los repos relacionados (siempre, primero)

Los repos NO se asumen por ruta fija ni por nombre: se leen de `## Repos relacionados` en `.claude/factory/convenciones.md`, se identifican por su `owner/repo` de GitHub y se resuelven en el momento, porque quien corra esta skill puede tenerlos clonados en cualquier lado (o no tenerlos clonados todavía).

```bash
CONV=".claude/factory/convenciones.md"

# Imprime el cuerpo de una sección "## Título" de convenciones.md.
leer_seccion() {
  [ -f "$CONV" ] || return 1
  awk -v h="## $1" 'index($0, h) == 1 { f = 1; next } /^## / { f = 0 } f' "$CONV"
}

# Cada ítem de lista -> "owner/repo|rol". Tolera backticks, "—", "–", "-", ":" y "(rol)".
listar_repos() {
  leer_seccion "Repos relacionados" \
    | grep -E '^[[:space:]]*[-*+][[:space:]]' \
    | sed -E 's/^[[:space:]]*[-*+][[:space:]]*//; s/`//g' \
    | sed -nE 's#^([A-Za-z0-9_.-]+/[A-Za-z0-9_.-]+)(.*)$#\1|\2#p' \
    | sed -E 's/\|[[:space:]]*(\(|:|—|–|-)*[[:space:]]*/|/; s/\)[[:space:]]*$//; s/\|$/|sin-rol/'
}

resolve_repo() {
  local org_repo="$1"          # ej: acme/tasks-web
  local name="${org_repo##*/}"

  # 1) ¿Ya hay un clon local en algún lugar razonable? Buscá por remote, no por nombre de carpeta.
  for root in ".." "../.." "$HOME/Projects" "$HOME/Documents" "$HOME"; do
    [ -d "$root" ] || continue
    while IFS= read -r d; do
      if git -C "$d" remote get-url origin 2>/dev/null | grep -qi "$org_repo"; then
        echo "$d"
        return 0
      fi
    done < <(find "$root" -maxdepth 2 -type d -iname "$name" 2>/dev/null)
  done

  # 2) No se encontró local: clonalo a un cache dentro de este repo (gitignored).
  local cache_dir=".claude/.repo-cache"
  mkdir -p "$cache_dir"
  if [ ! -d "$cache_dir/$name/.git" ]; then
    gh repo clone "$org_repo" "$cache_dir/$name" 2>/dev/null \
      || git clone --quiet "https://github.com/$org_repo.git" "$cache_dir/$name" \
      || { echo "ERROR: no se pudo clonar $org_repo" >&2; return 1; }
  fi
  echo "$cache_dir/$name"
}

REPOS=$(listar_repos)
if [ -z "$REPOS" ]; then
  echo "SIN_REPOS: no hay $CONV o no tiene '## Repos relacionados' (o no se pudo parsear)."
else
  while IFS='|' read -r org_repo rol; do
    dir=$(resolve_repo "$org_repo") || continue
    # Mejor esfuerzo para no trabajar sobre algo muy viejo; si falla (sin red, etc.) seguí igual.
    git -C "$dir" fetch --quiet 2>/dev/null
    echo "REPO rol=\"$rol\" repo=$org_repo path=$dir"
  done <<< "$REPOS"
fi

# El cache no se commitea: verificá que esté ignorado.
if [ -d ".claude/.repo-cache" ] && ! git check-ignore -q ".claude/.repo-cache" 2>/dev/null; then
  echo "AVISO: .claude/.repo-cache no está en .gitignore"
fi
```

Cómo seguir según lo que devuelva:

- **`SIN_REPOS`:** si el archivo existe pero el parseo no devolvió nada, leé la sección `## Repos relacionados` a mano (puede tener un formato que el parser no contempla) y resolvé cada repo con `resolve_repo`. Si el archivo o la sección no existen, **preguntale al usuario** qué repos tiene que mirar el scout (`owner/repo` y rol de cada uno). No adivines nombres de repo.
- **`ERROR: no se pudo clonar`:** avisale al usuario qué repo falló (permisos, red, nombre mal escrito) y preguntá si lo tiene clonado en otra ruta antes de seguir sin él. Si se sigue sin ese repo, todo lo que dependa de él queda `Necesita validación técnica`.
- **`AVISO: .claude/.repo-cache no está en .gitignore`:** agregalo al `.gitignore` del repo de trabajo o, si no corresponde tocarlo, avisale al usuario para que no se commiteen los clones.

Guardá el mapa rol → ruta (mentalmente, o en variables de shell dentro de la misma sesión de comandos) para el resto del procedimiento. Un repo puede tener cualquier rol; no asumas que siempre hay exactamente un front y un back.

## Procedimiento

Seguí estos pasos en orden. No los narres en detalle al usuario — ejecutalos y andá compartiendo hallazgos y preguntas a medida que iterás (ver "Cómo iterar" abajo).

### 1. Identificá la fuente y el módulo

Fuente: idea interna / cliente / CS / ventas / soporte / documentación existente. Módulo: tomá la raíz de la documentación de `## Documentación` en `.claude/factory/convenciones.md`, recorré sus carpetas (buscá el índice de cada área, por ejemplo `00-INDICE.md`) y, si la documentación declara una taxonomía de módulos (un `.yml`, un índice general), usala como referencia para ubicar en qué área entra el pedido. Si `## Documentación` no existe, preguntale al usuario dónde vive la documentación funcional antes de seguir. Si el pedido no encaja limpio en ningún área, es una señal para preguntar en la ronda 1, no para inventar un área nueva sin confirmar.

### 2. Leé la documentación funcional relacionada

```bash
DOCS="ruta/declarada/en/## Documentación"   # no la asumas: sale de convenciones.md o del usuario
find "$DOCS" -iname "00-INDICE.md" -o -iname "README.md" -o -iname "index.md"
find "$DOCS" \( -iname "*Requerimiento*" -o -iname "*Especificacion*" -o -iname "*Flujos*" -o -iname "*Validaciones*" -o -iname "*Decisiones*" -o -iname "*ADR*" \)
```

Ajustá los patrones a la convención de nombres que describa `## Documentación`. Empezá por el índice del área más cercana. Sacá reglas de negocio, validaciones, permisos y decisiones ya definidas, y citá el documento de origen.

### 3. Scout de código en los repos relacionados (lo que agrega esta skill)

Hacelo para **cada repo** que resolvió el Paso 0, con el rol que tiene declarado.

**Contexto primero.** Antes de buscar nada, leé los documentos de contexto que tenga cada repo: `CLAUDE.md`, `AGENTS.md`, `README.md`, `CONTRIBUTING.md`, docs de arquitectura o de roadmap si existen. De ahí sacá el stack, la organización de carpetas y las decisiones de negocio o bugs históricos ya verificados — no los repitas.

```bash
for f in CLAUDE.md AGENTS.md README.md CONTRIBUTING.md docs/architecture.md; do
  [ -f "$REPO/$f" ] && echo "== $REPO/$f"
done
ls "$REPO"   # para ver la estructura de primer nivel y deducir el stack
```

**Después, lo específico del requerimiento.** Buscá en las carpetas donde ese stack pone modelos, controladores/handlers, servicios, políticas de permisos, validaciones, rutas o endpoints, clientes de API, estado, páginas y componentes:

```bash
grep -rniE "PALABRA_CLAVE" "$REPO" \
  --exclude-dir=node_modules --exclude-dir=.git --exclude-dir=vendor \
  --exclude-dir=dist --exclude-dir=build --exclude-dir=.next \
  2>/dev/null | head -50
# y en el archivo de rutas/endpoints que use el stack (routes, router, urls, openapi...)
```

Acotá con `--include` a las extensiones del stack cuando el resultado sea muy ruidoso. Repetí la búsqueda con las palabras clave que hagan falta (nombre del módulo, entidades, verbos de negocio). El objetivo: saber qué existe hoy (modelos, endpoints, validaciones, estados, cálculos) antes de escribir el requerimiento, y detectar si el pedido choca con algo que el sistema ya hace de otra forma. Cuando un repo consume a otro (front que llama al back, servicio que llama a otro servicio), confirmá el contrato de los dos lados: lo que uno envía y lo que el otro realmente devuelve.

### 4. Elegí un espejo de estilo

La documentación no tiene por qué tener una plantilla fija: muchas veces cada feature tiene el set de archivos que necesita. Antes de escribir, leé lo que diga `## Documentación` sobre el estilo y elegí 1-2 features ya documentados en el área más parecida (o el área mejor documentada como referencia general si el área es nueva) y calcá su esqueleto: mismo bloque de metadatos, misma numeración de archivos (por ejemplo `00-INDICE.md`, `01-Resumen.md`, `02-Requerimiento-Funcional.md`...), mismas secciones cuando el tipo de feature sea comparable, mismo tono. No copies contenido, copiá estructura. Si no hay ningún feature documentado todavía, proponé la estructura de `spec-package` y confirmala con el usuario en la ronda 1.

### 5. Armá un borrador inicial (no el final)

Con lo que ya sabés (documentación + scout de código), armá un primer borrador de: descripción general, alcance incluido, fuera de alcance, y una lista preliminar de criterios de aceptación. Marcá explícitamente cada hueco como `Pendiente`, `Supuesto` o `Necesita definición` — no lo completes con una suposición razonable sin decirlo.

### 6. Iterá con el usuario hasta que quede sólida

Esta es la parte central de la skill — leé esto con cuidado:

- **Ronda 1:** compartí un resumen corto de lo que encontró el scout (qué existe hoy en cada repo relacionado, qué dice la documentación relacionada) y el borrador de alcance. Debajo, hacé **como máximo 5 preguntas**, solo las que **cambian** el alcance, el diseño o un criterio de aceptación — nada cosmético. Si falta configuración (repos, dominios sensibles, ubicación de la documentación), esas preguntas entran acá. Esperá la respuesta real antes de seguir escribiendo.
- **Rondas siguientes:** incorporá las respuestas, refiná el borrador, y volvé a mirar si las respuestas nuevas abrieron una pregunta que antes no existía (esto pasa seguido: una respuesta de alcance suele destapar un caso de borde). Si es así, abrí una nueva ronda corta. Si no, no preguntes por preguntar.
- **No hay un número fijo de rondas.** Puede ser una si el pedido viene claro y el scout confirma todo; pueden ser varias si el área es nueva o toca un dominio sensible. Lo que manda es el criterio de cierre, no la cantidad de rondas.
- **Criterio de cierre (recién ahí escribís los archivos finales):**
  - Ningún criterio de aceptación depende de algo marcado `Pendiente`.
  - Alcance y fuera de alcance están completos, no a medias.
  - Si el pedido toca un dominio sensible o integraciones, esa parte quedó validada explícitamente por el usuario o marcada, sin ambigüedad, como "Necesita validación técnica" — nunca asumida.
  - Todo lo que el scout de código contradice respecto de lo pedido o de la documentación previa quedó resuelto o marcado como punto de atención, no callado.

No frenes con un interrogatorio inicial de diez preguntas antes de mostrar nada: siempre arrancás mostrando el borrador con lo que ya sabés (paso 5), y las preguntas van después, acotadas y en rondas.

### 7. Escribí los documentos finales

Recién cuando se cumple el criterio de cierre, escribí (o actualizá) los archivos en la carpeta del área y del feature dentro de la raíz declarada en `## Documentación` (por ejemplo `[raíz-docs]/[area]/[feature]/`), siguiendo el espejo de estilo elegido en el paso 4. Cada documento abre con:

```
> Relacionados: [enlace a cada otro documento del mismo paquete]
```

y con el bloque de metadatos que ya usa el resto de la documentación (por ejemplo una "Ficha documental" con Versión, Estado, Owner, Última actualización, Fuente de verdad, Relacionados).

Si el feature es nuevo, generá o actualizá el índice del área (por ejemplo `00-INDICE.md`) listando el feature con una línea de descripción y su estado.

### 8. Cerrá con un veredicto de estado

Indicá cuál aplica:
- **Listo para desarrollo** — pasa `guia-redaccion.md` y no quedan flags de dominio sensible ni decisiones pendientes sin marcar.
- **Necesita refinamiento** — faltan criterios o casos de borde.
- **Necesita definición de producto** — hay huecos de negocio sin resolver ni con el scout ni con el usuario.
- **Necesita validación técnica** — hay flag de dominio sensible o de integración sin confirmar, o el scout encontró una contradicción sin resolver.

## Referencia

- `guia-redaccion.md` — qué tiene que tener (y qué no) un requisito y un criterio de aceptación, adaptado al estilo de documentación en prosa (sin exigir IDs `RF-`/`CA-` ni sintaxis EARS en el documento final; EARS es una herramienta interna opcional para redactar sin ambigüedad, no un formato de salida).
- `.claude/factory/convenciones.md` del repo de trabajo — `## Repos relacionados`, `## Dominios sensibles` y `## Documentación`.
- `spec-package` — la skill hermana para el paquete de cuatro capas (requerimiento, criterios, diseño y plan) cuando no hace falta scout de código.
