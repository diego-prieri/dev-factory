# Plantillas — Documentación de proyecto

Usá estas estructuras al redactar. No elimines secciones: si una no aplica, dejala con `No aplica` o `Pendiente` para mantener consistencia con el resto de la documentación.

**Estructura de carpetas canónica** (ver detalle completo en SKILL.md → "Salida"; la raíz sale de `## Documentación` en `.claude/factory/convenciones.md`):

```
[raíz-docs]/[modulo]/[feature]/
├── 00-INDICE.md
└── REQ-[NN]-[area]/
    ├── requerimiento.md
    ├── diseno.md
    ├── plan.md
    └── trazabilidad.md
```

Por defecto, entregá **documentos separados y enlazados** dentro de `REQ-NN-[area]/`: `requerimiento.md`, `diseno.md`, `plan.md` y `trazabilidad.md`, todos con el mismo `REQ-NN` y una línea de enlaces a los otros. Para una mejora chica, alcanza con `requerimiento.md`. Siempre generá o actualizá el `00-INDICE.md` del feature.

## Índice de plantillas
1. Requerimiento (capa 1)
2. Criterios de aceptación (capa 2)
3. Diseño / brief técnico de traspaso — producto + desarrollo (capa 3)
4. Plan de implementación / Task (capa 4)
5. Trazabilidad y seguimiento (transversal)
6. Índice de feature (00-INDICE.md)

---

## 1. Requerimiento (requerimiento.md)

# Proyecto: [Título claro y accionable]
> Relacionados: [diseño](./diseno.md) · [plan](./plan.md) · [trazabilidad](./trazabilidad.md)

**Tipo:** Feature / Mejora / Bug / Spike / Deuda técnica / QA / UX
**Módulo:** [módulo funcional afectado] · **ID:** REQ-[NN]
**Prioridad sugerida:** Crítica · Alta / estratégica · Media · Baja
> Prioridad según impacto, urgencia, frecuencia de pedidos similares y alineación con el roadmap del producto.

### Fuente y trazabilidad
- **Origen:** (idea interna / cliente / CS / ventas / soporte / herramienta de seguimiento / documentación / decisión pendiente)
- **Cliente / Empresa:** · **Segmento / industria:** · **Responsable:**
- **Medio de contacto:** · **Link al ticket o CRM:**
- **Frecuencia / pedidos similares:**
- **Relación con el roadmap:**
> Si no viene de un cliente, completá `Origen` y dejá el resto en `No aplica`. No borres campos.

### Historia de usuario
Como [rol], quiero [objetivo], para [beneficio].

### Problema
El dolor operativo, la limitación o la necesidad actual. Concreto y SIN solución todavía.

### Objetivo
Qué resultado se espera lograr. Medible o verificable.

### Alcance incluido
Lista concreta de comportamientos, pantallas, reglas o casos que SÍ entran.

### Fuera de alcance
Qué NO se resuelve acá (y en qué otro proyecto/ticket va, si aplica).

### Requisitos funcionales
Numerados y en sintaxis EARS (ver guia-redaccion-requisitos.md). Citá el documento de origen cuando la regla salga de la documentación.
- **RF-01 —** Cuando [disparador], el sistema debe [respuesta].
- **RF-02 —** Si [condición no deseada], entonces el sistema debe [respuesta].
- **RF-03 —** Mientras [estado], el sistema debe [respuesta].

### Restricciones
Reglas duras impuestas (legales, regulatorias, de integración, de plataforma). No son diseño: son límites.

### Flujo esperado
Pasos principales del usuario o del sistema, en orden.

---

## 2. Criterios de aceptación

Verificables, numerados, binarios. Ligados al RF que cubren. Incluí camino feliz + error/límite.

- **CA-01 (RF-01) —** [condición simple verdadero/falso].
- **CA-02 (RF-02) —** Dado [contexto], Cuando [acción], Entonces [resultado esperado].
- **CA-03 (error/límite) —** [qué pasa ante el caso de error o el borde].

### Casos de prueba sugeridos (QA)
- **CP1 —** [precondición] -> [acción] -> [resultado esperado]   (caso feliz)
- **CP2 —** [precondición] -> [acción] -> [resultado esperado]   (error/límite)

---

## 3. Diseño / brief técnico de traspaso (diseno.md)

> Este documento tiene dos mitades claramente separadas. **Producto completa la parte A** (el contexto y las restricciones que conoce, y las preguntas que dev tiene que resolver). **Desarrollo completa la parte B** (la solución y las decisiones técnicas). Producto NO inventa la parte B: la deja marcada `A completar por desarrollo`.

# Diseño: [Título] · ID: DIS-[NN] · vincula a REQ-[NN]
> Relacionados: [requerimiento](./requerimiento.md) · [plan](./plan.md) · [trazabilidad](./trazabilidad.md)

## Parte A — Lo completa producto

### Contexto
Por qué existe esto, qué hay hoy, qué documentación funcional aplica (linkeá los .md del repo).

### Restricciones
Reglas duras impuestas que el diseño tiene que respetar: legales, regulatorias, de plataforma, de integración. No son la solución: son los límites dentro de los que dev tiene que diseñar.

### Impacto en dominios sensibles
> Obligatorio si toca alguno de los temas listados en `## Dominios sensibles` de `.claude/factory/convenciones.md` (si la sección no existe, preguntá cuáles son).
Qué dominios sensibles toca y por qué es delicado. Producto marca el riesgo; la confirmación de comportamiento contra los sistemas externos involucrados queda como punto a validar por dev.

### Dependencias
Documentos del repo, módulos, APIs, permisos, integraciones o datos necesarios.

### Decisiones / preguntas para desarrollo
Las preguntas técnicas que producto detecta pero no resuelve. No las respondas inventando.
- ¿…?
- ¿…?

## Parte B — Lo completa desarrollo
> Producto deja estas secciones con la marca `A completar por desarrollo`.

### Objetivos y no-objetivos
`A completar por desarrollo` — qué tiene que lograr la solución y qué explícitamente queda afuera.

### Propuesta
`A completar por desarrollo` — la solución elegida, con el detalle suficiente para construirla. Diagramar si ayuda (contexto + componentes).

### Alternativas consideradas
`A completar por desarrollo`
- **Opción A (elegida):** … — por qué gana.
- **Opción B:** … — por qué se descarta (trade-off).

### Decisiones (ADR)
`A completar por desarrollo` — una entrada por decisión, formato corto e inmutable:
- **ADR-01 — [título de la decisión]**
  - **Estado:** propuesta / aceptada / reemplazada por ADR-XX
  - **Contexto:** qué obliga a decidir.
  - **Decisión:** qué se decide.
  - **Consecuencias:** qué gana y qué cuesta.

### Migración / rollout
`A completar por desarrollo` — cómo se libera, qué pasa con los datos existentes, cómo se revierte, feature flag si aplica.

### Riesgos técnicos
`A completar por desarrollo` — performance, dependencia de terceros, migración.


---

## 4. Plan de implementación (plan.md)

> El plan se construye en rebanadas verticales: cada una entrega algo demostrable de punta a punta. Primero la que valida el riesgo más alto.

# Plan de implementación: [Título] · vincula a REQ-[NN] / DIS-[NN]
> Relacionados: [requerimiento](./requerimiento.md) · [diseño](./diseno.md) · [trazabilidad](./trazabilidad.md)

### Estrategia
Una línea: cuál es la rebanada riesgosa que atacamos primero y por qué.

### Rebanadas / tareas
Cada tarea entrega valor demostrable, no una capa horizontal.
- **T-01 — [nombre]**
  - **Qué entrega:** [resultado observable]
  - **Disciplina:** Producto / UX / Frontend / Backend / QA / Documentación
  - **Depende de:** — (o T-0X)
  - **Definición de hecho (DoD):** [cómo sabemos que esta tarea terminó]
  - **Estimación:** [gruesa, si se puede]
  - **¿Se vuelve ticket?** Sí / No
- **T-02 — …**
- **T-03 — …**

### Definition of Ready (antes de empezar)
- [ ] Requisitos con criterios verificables.
- [ ] Sin decisiones de producto pendientes que bloqueen.
- [ ] Sin flag de dominio sensible abierto sin validar.
- [ ] Dependencias y datos disponibles.

### Definition of Done (para cerrar el proyecto)
- [ ] Todos los CA pasan.
- [ ] Casos de prueba de QA ejecutados.
- [ ] Documentación del repo actualizada.
- [ ] Validación de dominios sensibles confirmada (si aplicaba).

---

## 5. Trazabilidad y seguimiento (trazabilidad.md)

> Relacionados: [requerimiento](./requerimiento.md) · [diseño](./diseno.md) · [plan](./plan.md)

Liga cada requisito con sus criterios y las tareas que lo construyen. Es lo que permite seguir el avance.

| Requisito | Criterios | Tareas | Estado |
|-----------|-----------|--------|--------|
| RF-01 | CA-01 | T-01 | Pendiente / En curso / Hecho |
| RF-02 | CA-02, CA-03 | T-02, T-03 | Pendiente |

### Estado del proyecto
> Marcá uno:
- [ ] Listo para desarrollo
- [ ] Necesita refinamiento
- [ ] Necesita definición de producto
- [ ] Necesita validación técnica

### Decisiones pendientes
Preguntas que deben resolverse antes de avanzar. No las respondas inventando.

### Dependencias
Documentos del repo, módulos, APIs, permisos, integraciones o datos necesarios. Linkealos.

---

## 6. Índice de feature (00-INDICE.md)

Ubicación: `[raíz-docs]/[modulo]/[feature]/00-INDICE.md`

Generalo o actualizalo cada vez que se agrega un `REQ-NN-[area]` al feature. No copies contenido de los paquetes: solo el índice con estado.

---

# Índice — [Feature]

## Módulo
[Modulo] → [Feature]

## Objetivo del feature
Una línea: qué resuelve este conjunto de requisitos y para quién.

## Paquetes de requisitos

| REQ | Área | Descripción | Estado |
|---|---|---|---|
| [REQ-01-area](./REQ-01-area/requerimiento.md) | [area] | [una línea de qué cubre] | Pendiente / En revisión / Listo para desarrollo / En desarrollo / Cerrado |
| [REQ-02-area](./REQ-02-area/requerimiento.md) | [area] | [una línea de qué cubre] | Pendiente |

## Orden de desarrollo sugerido
1. REQ-01 — [razón: riesgo más alto / dependencia base]
2. REQ-02 — depende de REQ-01
3. REQ-03 — puede desarrollarse en paralelo con REQ-02

## Decisiones pendientes globales del feature
Decisiones que afectan a más de un REQ y deben resolverse antes de iniciar cualquier paquete.
- DP-XXX: [descripción] — bloquea REQ-01, REQ-02

## Documentación de referencia
Links a documentos fuente del repo relacionados con este feature.
