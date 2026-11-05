---
name: spec-package
description: "Use when: redactar requerimientos, especificaciones funcionales, criterios de aceptación, documentos de diseño / propuestas técnicas (RFC, ADR) o planes de implementación (tasks paso a paso) para proyectos de desarrollo de software. Activá esta skill siempre que alguien de producto, CS, soporte o desarrollo diga: documentar un proyecto o feature, armar el requerimiento, escribir la especificación, definir criterios de aceptación, hacer el design doc o la propuesta técnica, decidir la arquitectura o el enfoque, o armar el plan paso a paso para pasarle a desarrollo — aunque no use esas palabras exactas y aunque el pedido venga informal o pegado de un cliente. Esta skill es para el paquete de documentación completo de un proyecto (requerimiento, criterios, diseño y plan). Si además hace falta recorrer el código real de los repos relacionados e iterar con el usuario por rondas, usá spec-builder."
argument-hint: "idea, feature request, documento o contexto a documentar como proyecto"
user-invocable: true
---

# Spec Package — Documentación de proyectos

Convertí cualquier entrada —una idea, un mensaje de un cliente, una feature request, un flujo, una decisión pendiente o documentación existente— en un **paquete de documentación de proyecto** en Markdown, coherente y verificable, que va de **producto a desarrollo**.

El paquete tiene cuatro capas que responden, en orden, cuatro preguntas distintas:

1. **Requerimiento** — *¿qué problema resolvemos y para quién?* (el qué y el porqué, sin la solución).
2. **Criterios de aceptación** — *¿cómo sabemos que está bien?* (verificable, binario).
3. **Diseño / propuesta técnica** — *¿cómo lo vamos a resolver y por qué así?* (alternativas y decisiones).
4. **Plan de implementación (Task)** — *¿en qué pasos lo construimos?* (rebanadas verticales, secuenciadas).

Más una capa transversal de **trazabilidad y seguimiento** que liga necesidad → requisito → criterio → tarea → prueba.

Esta skill está pensada para que **la use cualquiera del equipo**, no solo perfiles técnicos. Quien la usa **no tiene que saber** qué es EARS, Gherkin o un ADR: la skill escribe bien por defecto y enseña en el camino.

## Relación con otras skills

- Esta skill es el **nivel proyecto**: produce el requerimiento + criterios + diseño + plan, a partir de la documentación existente y de lo que aporte quien la usa. No mira código.
- `spec-builder` es la hermana pesada: hace scout del código real de los repos relacionados e itera con el usuario por rondas. Si el pedido necesita confirmar cómo se comporta hoy el sistema antes de escribir, derivá ahí.
- Las dos leen la misma configuración del repo de trabajo (ver abajo): los disparadores del flag de dominio sensible y la ubicación y el estilo de la documentación son los mismos.

## Configuración del repo de trabajo

La skill no trae datos del producto precargados: los lee del archivo `.claude/factory/convenciones.md` del repo donde se corre.

- `## Dominios sensibles` — temas que obligan a levantar el **flag de dominio sensible** y pedir validación explícita.
- `## Documentación` — dónde viven las docs funcionales, cómo están organizadas (módulos, carpetas, índices), con qué estilo se escriben y, si existe, qué checklist de calidad propio usa el repo.
- `## Repos relacionados` — no lo usa esta skill directamente, pero sirve para listar dependencias en el brief de traspaso.

Si el archivo no existe, o falta la sección que necesitás, **preguntale al usuario** en vez de asumir, y sugerile agregarla a `.claude/factory/convenciones.md`. Nunca completes una sección faltante con valores inventados.

## Principios que no se negocian

- **Escribí siempre en español rioplatense.** Tono funcional, claro, directo, orientado a desarrollo.
- **El requerimiento dice QUÉ y POR QUÉ, no CÓMO.** El "cómo" vive en la capa de diseño. No metas solución dentro del requerimiento salvo que sea una restricción real e impuesta (y entonces marcala como restricción, no como diseño).
- **Todo requisito y todo criterio tiene que ser verificable.** Si no se puede responder con sí/no al probarlo, está mal escrito: reescribilo. Ver `guia-redaccion-requisitos.md`.
- **No inventes.** Si falta un dato, marcalo como `Pendiente`, `Supuesto` (cuando asumís algo razonable y lo declarás) o `Necesita definición`. Nunca cierres una decisión pendiente inventando la respuesta.
- **El diseño técnico lo escribe desarrollo, no producto.** Producto NO propone la solución, ni elige arquitectura, ni inventa decisiones técnicas (eso sería violar "no inventes" en la capa más cara de equivocarse). Lo que produce esta skill en la capa de diseño es un **brief técnico de traspaso**: contexto, restricciones conocidas, impacto en dominios sensibles, dependencias y las preguntas/decisiones que desarrollo tiene que resolver. Las secciones de propuesta, alternativas y ADR quedan como esqueleto marcado `A completar por desarrollo`. Cuando dev las complete, sí valen las reglas del oficio: una propuesta sin "no-objetivos" ni "alternativas consideradas" es una lista de deseos.
- **Plan en rebanadas verticales.** El plan entrega valor de punta a punta lo antes posible (rebanadas finas que funcionan), no capas horizontales (todo el backend, después todo el front). Atacá primero la suposición más riesgosa.
- **IDs para todo lo que se referencia.** Requisitos `RF-01`, criterios `CA-01`, decisiones `ADR-01`, tareas `T-01`. Permite trazar y citar en commits y PRs.
- **Flag de dominio sensible obligatorio.** Si el pedido toca alguno de los temas listados en `## Dominios sensibles` de `.claude/factory/convenciones.md`, marcalo en *Riesgos* y pedí validación antes de asumir comportamiento. No declares "Listo para desarrollo" con un supuesto sobre un dominio sensible sin confirmar. Si la sección no existe, preguntale al usuario qué temas considera sensibles.
- **Trazabilidad.** Si viene de un cliente o de CS, conservá: cliente, empresa, segmento o industria, responsable, medio de contacto, link al ticket o al CRM, módulo y frecuencia.

## Qué tiene que tener (y qué NO) un buen requisito

Esto es el corazón de la skill. Antes de escribir cualquier requisito, leé `guia-redaccion-requisitos.md`. En resumen, cada requisito debe ser:

- **Necesario** — si lo sacás, falta algo real.
- **No ambiguo** — una sola interpretación posible.
- **Singular** — una cosa por requisito (nada de "y" / "o" encadenando varias reglas).
- **Verificable** — se puede probar, inspeccionar o demostrar que se cumple.
- **Factible** — se puede construir dentro de las restricciones conocidas.
- **Trazable** — tiene ID y se sabe de qué necesidad viene.

Y **nunca** debe tener: solución/implementación disfrazada de requisito; adjetivos vagos (*rápido, intuitivo, amigable, robusto, eficiente, flexible, óptimo*); voz pasiva que esconde quién hace qué; "etc." o "y/o"; ni depender de algo marcado `Pendiente`.

Para escribir requisitos funcionales con una sola interpretación, usá **sintaxis EARS** (plantilla de llenar huecos, sirve para gente no técnica):

- **Cuando** ‹disparador›, el sistema **debe** ‹respuesta›. *(comportamiento ante un evento)*
- **Mientras** ‹estado›, el sistema **debe** ‹respuesta›. *(comportamiento durante un estado)*
- **Si** ‹condición no deseada›, **entonces** el sistema **debe** ‹respuesta›. *(manejo de errores)*
- **Donde** ‹característica incluida›, el sistema **debe** ‹respuesta›. *(comportamiento opcional según config)*
- El sistema **debe** ‹respuesta›. *(regla siempre vigente)*

## Procedimiento

Seguí estos pasos en orden. No los narres al usuario; ejecutalos y entregá el resultado.

### 1. Identificá la fuente y la escala
- **Fuente:** idea interna / cliente / CS / ventas / soporte / herramienta de seguimiento (tablero, planilla, tracker) / documentación / decisión pendiente. Define qué trazabilidad cargar.
- **Escala:** decidí qué capas generar (ver "Selección de profundidad" abajo). No todo proyecto necesita design doc; casi ninguno se salva de tener criterios de aceptación.

### 2. Detectá el módulo afectado
Identificá qué módulo del producto toca. Tomá los nombres canónicos de lo que declare `## Documentación` (una taxonomía de módulos, si la hay) o, si no, de las carpetas de módulo que ya existen bajo la raíz de documentación, y confirmalos contra sus índices (por ejemplo `00-INDICE.md`). Si no hay ninguna de las dos cosas, preguntale al usuario. Si no encaja limpio: `Módulo: Necesita definición`.

### 3. Leé la documentación relacionada del repo (en runtime)
La documentación vive en el repo y la leés acá, no la tenés precargada. Empezá por la raíz que declare `## Documentación` y descubrila por patrón, no por ruta fija:

```bash
find . -iname "00-INDICE.md"                       # índices de módulo/feature
find . \( -iname "*Requerimiento*.md" -o -iname "*Especificacion*.md" -o -iname "*Flujos*.md" \)
find . -iname "*valida*" \( -iname "*.md" -o -iname "*.json" \)   # validaciones = reglas duras
find . -ipath "*Feature Request*"
find . -iname "*ADR*" -o -iname "*decision*"        # decisiones técnicas previas
```

Ajustá los patrones a la convención de nombres que describa `## Documentación`. Empezá por el índice para ubicarte. Las validaciones `.json`/`.md` son **reglas duras**: respetalas tal cual. Si hay ADRs previos relacionados, no contradigas una decisión vigente sin marcarlo.

### 4. Extraé reglas existentes y detectá vacíos
Sacá reglas de negocio, validaciones, permisos, estados, cálculos e integraciones ya definidos, y citá el documento de origen. Todo lo que la documentación NO resuelve va marcado `Pendiente` / `Supuesto` / `Necesita definición`.

### 5. Escribí el requerimiento
Usá la plantilla de `plantillas.md`. Historia de usuario (formato Connextra, chequeada contra INVEST), problema sin solución, objetivo medible, alcance y fuera de alcance **ambos completos**, y requisitos funcionales numerados (`RF-01`…) en sintaxis EARS.

### 6. Escribí los criterios de aceptación
Numerados (`CA-01`…), binarios, ligados a los `RF`. Mezclá dos estilos según convenga:
- **Estilo regla (checklist):** cuando es una condición simple verdadero/falso.
- **Estilo escenario (Gherkin):** **Dado** ‹contexto›, **Cuando** ‹acción›, **Entonces** ‹resultado›. Úsalo para flujos con pasos.

Cubrí siempre: camino feliz + al menos un caso de error/límite + casos de dominio sensible si aplica. Ningún criterio puede depender de algo `Pendiente`.

### 7. Armá el brief técnico de traspaso (NO el diseño)
Producto no diseña la solución. Completá solo lo que producto legítimamente conoce y deja escrito para que desarrollo decida: **contexto**, **restricciones** impuestas (legales, regulatorias, de plataforma), **impacto en dominios sensibles** (qué temas de `## Dominios sensibles` toca y qué sistemas externos involucra), **dependencias** y las **decisiones/preguntas abiertas** que dev tiene que resolver. Las secciones de **propuesta, alternativas y ADR** se dejan como esqueleto con la marca `A completar por desarrollo` — no las inventes. Usá la plantilla de diseño de `plantillas.md`, que ya viene partida en "lo completa producto" y "lo completa desarrollo".

### 8. Armá el plan de implementación (Task)
Usá la plantilla de plan en `plantillas.md`. Reglas:
- **Rebanadas verticales** numeradas (`T-01`…), cada una entregando algo demostrable de punta a punta.
- Primero la **rebanada que valida el riesgo más alto** (un *spike* si hace falta).
- Cada tarea: objetivo, dependencias (`depende de T-0X`), disciplina (Producto/UX/Front/Back/QA/Doc), criterio de hecho (DoD) y estimación gruesa si se puede.
- Anotá qué tareas se vuelven tickets individuales para seguimiento.

### 9. Completá la trazabilidad y el seguimiento
Llená la tabla de trazabilidad (`plantillas.md`): cada `RF` ligado a sus `CA` y a las `T` que lo construyen. Esto es lo que después permite seguir el avance y saber qué falta.

### 10. Determiná la estructura de carpetas y generá el `00-INDICE.md`

Antes de escribir los archivos, decidí:
1. **`[Modulo]`** — el dominio funcional principal (ver "Salida → Reglas de nombrado").
2. **`[Feature]`** — el producto, canal o subsistema específico.
3. **Áreas** — si el proyecto es grande, descomponé en áreas lógicas. Cada área = una carpeta `REQ-NN-[area]`. Si es chico, una sola `REQ-01-[slug]`.

Luego generá o actualizá el `00-INDICE.md` en la carpeta `[Feature]/` usando la plantilla de `plantillas.md`. El índice lista cada `REQ-NN-[area]` con una línea de descripción y su estado actual.

### 11. Cerrá con un veredicto de estado
Indicá cuál de estos aplica al paquete:
- **Listo para desarrollo** — pasa la guía de calidad y no hay flags de dominio sensible abiertos ni decisiones pendientes.
- **Necesita refinamiento** — faltan criterios o casos.
- **Necesita definición de producto** — hay huecos de negocio o decisiones sin resolver.
- **Necesita validación técnica** — hay flag de dominio sensible o integración sin confirmar.

Si `## Documentación` declara un checklist de calidad propio del repo, verificá el paquete contra ese checklist antes de declarar "Listo para desarrollo".

## Selección de profundidad (qué capas generar)

No abrumes con documentación que el proyecto no necesita. La capa de diseño que genera producto es **siempre un brief de traspaso** (lo completa dev), nunca un diseño cerrado. Elegí qué capas generar:

- **Mejora chica / bug:** requerimiento liviano + criterios + plan corto. Sin brief de diseño separado.
- **Feature mediana:** requerimiento completo + criterios + plan + un brief técnico breve (contexto + restricciones + dependencias + preguntas para dev).
- **Proyecto / cambio estructural / toca un dominio sensible o integraciones:** las cuatro capas completas. El brief de traspaso es obligatorio y detallado en impacto sobre dominios sensibles, pero la propuesta y los ADR quedan para que los complete desarrollo.

Ante la duda, generá de más en criterios y de menos en diseño: los criterios casi nunca sobran; un brief técnico para algo trivial es ruido.

## Manejo de información incompleta

Si quien usa la skill da poca información, **no la frenes con un interrogatorio**:

1. Generá igual una **versión preliminar** del paquete con lo que haya, marcando huecos como `Pendiente` / `Necesita definición`.
2. Debajo, agregá una **lista corta de preguntas** (3 a 5 máximo) con lo mínimo indispensable para cerrar. Si falta configuración del repo (dominios sensibles, ubicación de la documentación), esas preguntas entran acá.

Así la persona ve algo concreto de entrada y solo responde lo que falta, sin conocer la estructura interna.

## Reglas sobre prototipos

Si hay un prototipo HTML o referencia visual, usalo **solo** para UX e interacción (cómo se ve, cómo se navega). El comportamiento funcional se valida contra los documentos funcionales, no contra el prototipo. Si el prototipo contradice la especificación, gana la especificación y lo marcás como punto de atención.

## Salida

Entregá el paquete como **documentos separados y enlazados entre sí** (no un solo archivo), siguiendo las plantillas de `plantillas.md` y la convención de estructura de carpetas definida abajo.

### Convención de estructura de carpetas

La raíz de la documentación sale de `## Documentación` en `.claude/factory/convenciones.md`; si esa sección no existe, preguntale al usuario dónde guardar el paquete y proponé `docs/features/` como sugerencia. Si `## Documentación` define otra jerarquía, respetala. Si no define ninguna, usá esta jerarquía de tres niveles bajo la raíz:

```
[raíz-docs]/                           ← p. ej. docs/features/
└── [modulo]/                          ← kebab-case, sin número
    └── [feature]/                     ← kebab-case, sin número
        ├── 00-INDICE.md               ← índice del feature; generalo o actualizalo con cada REQ nuevo
        └── REQ-[NN]-[area]/           ← NN = 01, 02… sin saltar; area en kebab-case
            ├── requerimiento.md       ← capa 1 + 2: requisitos y criterios
            ├── diseno.md              ← capa 3: brief de traspaso
            ├── plan.md                ← capa 4: plan de implementación
            └── trazabilidad.md        ← tabla RF → CA → T + estado
```

**Reglas de nombrado:**

- **`[modulo]`** — dominio funcional principal. Los valores canónicos salen de `## Documentación` o de las carpetas de módulo que ya existen bajo la raíz (por ejemplo, en un gestor de tareas: `tareas`, `proyectos`, `colaboracion`, `notificaciones`, `general`). No crees un módulo nuevo sin confirmarlo. Sin número. Si no encaja limpio, usá `necesita-definicion`.
- **`[feature]`** — producto, canal, flujo o subsistema específico dentro del módulo. Ejemplos: `tableros-compartidos`, `recordatorios`, `importacion-csv`. Sin número, kebab-case.
- **`REQ-[NN]-[area]`** — carpeta de un paquete de requisitos. `NN` empieza en `01` y se incrementa por orden de creación dentro del feature. `area` en kebab-case, describe el tema central: `invitaciones`, `permisos-por-rol`, `recordatorios-por-email`, `vista-calendario`.

**Descomposición en áreas:**

Cuando el proyecto es grande, no metas todo en un solo REQ. Identificá las áreas lógicas del proyecto —un área es un conjunto cohesivo de requisitos que se puede construir y probar de forma relativamente independiente— y creá un `REQ-NN-[area]` por cada una. Ejemplo para tableros compartidos en un gestor de tareas:

```
REQ-01-invitaciones/              ← invitar por email, aceptar, expirar, revocar
REQ-02-permisos-por-rol/          ← roles del tablero, qué puede ver y editar cada uno
REQ-03-comentarios/               ← hilos de comentarios en tareas, edición y borrado
REQ-04-menciones-notificaciones/  ← @menciones, notificaciones in-app y por email
REQ-05-historial-actividad/       ← registro de cambios del tablero, filtros y exportación
```

**Proyecto chico (mejora / bug):**

Si el proyecto es pequeño, usá un único `REQ-01-[slug]/` con solo `requerimiento.md` o los archivos que justifiquen.

**`00-INDICE.md` del feature:**

Siempre generá o actualizá el `00-INDICE.md` en la carpeta `[Feature]/`. Debe listar cada `REQ-NN-[area]` con su estado (Pendiente / En revisión / Listo para desarrollo / En desarrollo / Cerrado). Usá la plantilla de `plantillas.md`.

**Enlazado entre documentos:**

Cada documento abre con una línea de enlaces a los otros del mismo paquete:
```
> Relacionados: [requerimiento](./requerimiento.md) · [diseño](./diseno.md) · [plan](./plan.md) · [trazabilidad](./trazabilidad.md)
```
Todos los documentos de un mismo paquete comparten el mismo `REQ-NN`. Si dividiste un pedido grande en varios paquetes, el `00-INDICE.md` del feature los liga y cierra la trazabilidad con dependencias y orden sugerido de desarrollo.
