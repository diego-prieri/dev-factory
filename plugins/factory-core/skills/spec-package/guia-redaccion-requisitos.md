# Guía de redacción de requisitos y criterios

Esta guía es el estándar de calidad de la skill. Tiene tres partes: **(1) qué debe y qué no debe tener un requisito**, **(2) cómo escribir requisitos sin ambigüedad (EARS)** y **(3) cómo escribir criterios de aceptación verificables (Gherkin + checklist)**. Está basada en marcos consolidados de la industria: ISO/IEC/IEEE 29148 (ingeniería de requisitos), EARS (Easy Approach to Requirements Syntax), INVEST para historias de usuario y BDD / Specification by Example para criterios.

## Índice
1. Las 6 propiedades de un buen requisito
2. Lo que un requisito NUNCA debe tener (anti-patrones)
3. Vocabulario prohibido
4. Sintaxis EARS (plantillas)
5. Historias de usuario e INVEST
6. Criterios de aceptación: regla vs escenario (Gherkin)
7. Checklist rápido antes de dar un requisito por bueno

---

## 1. Las 6 propiedades de un buen requisito

Cada requisito individual debe ser:

- **Necesario** — si lo eliminás, queda un hueco real. Si nadie lo extraña, sobra.
- **No ambiguo** — admite una sola interpretación. Si dos personas del equipo pueden entenderlo distinto, está mal escrito.
- **Singular** — describe **una** cosa. Nada de encadenar reglas con "y"/"o". Un requisito, una obligación.
- **Verificable** — existe una forma objetiva de probar que se cumple (test, inspección, demostración). Si no se puede responder sí/no, no es un requisito, es un deseo.
- **Factible** — se puede construir dentro de las restricciones técnicas, de tiempo y de negocio conocidas.
- **Trazable** — tiene un ID (`RF-01`) y se sabe de qué necesidad o documento proviene.

Y el **conjunto** de requisitos debe ser **completo** (cubre la necesidad), **consistente** (no se contradice entre sí) y **comprensible** sin contexto oculto.

---

## 2. Lo que un requisito NUNCA debe tener (anti-patrones)

- **Solución disfrazada.** El requisito dice *qué* tiene que pasar, no *cómo* implementarlo. ❌ "Guardar el email del usuario en una tabla indexada por hash." ✅ "El sistema debe recuperar un usuario por su email." La excepción: una restricción real impuesta (ej. "debe autenticar contra el proveedor de identidad (SSO) del cliente"), que se marca como **restricción**, no como diseño.
- **Requisito compuesto.** Si tiene "y", "o", "además", probablemente sean varios requisitos. Partilos: cada uno se prueba por separado.
- **Voz pasiva sin actor.** ❌ "Las invitaciones serán validadas." ¿Por quién? ✅ "El sistema debe validar que el email del invitado no pertenezca ya al espacio de trabajo antes de enviar la invitación."
- **Adjetivos no medibles.** Ver vocabulario prohibido abajo.
- **Negaciones difíciles de probar.** Preferí comportamiento observable. ❌ "El sistema no debe ser lento." ✅ "El sistema debe responder la búsqueda de tareas en menos de 2 segundos para espacios de trabajo de hasta 10.000 tareas."
- **Dependencia de algo sin definir.** Un requisito que depende de un `Pendiente` no puede ser parte de "Listo para desarrollo".
- **Scope creep colado.** "…y ya que estamos, estaría bueno que también…" → eso va a *Fuera de alcance* o a otro ticket.

---

## 3. Vocabulario prohibido

Estas palabras son banderas rojas: casi siempre esconden ambigüedad. Si aparecen, reescribí con un criterio medible.

| Palabra vaga | Reemplazá por algo medible |
|---|---|
| rápido / performante | un umbral: "< 2 s", "hasta N registros" |
| intuitivo / amigable / fácil | un comportamiento observable o un criterio de UX testeable |
| robusto / confiable | qué pasa ante qué error, concretamente |
| flexible / configurable | qué exactamente se puede configurar y por quién |
| óptimo / eficiente | el número o la condición concreta |
| varios / algunos / muchos | una cantidad o un rango |
| etc. / y/o | enumerá los casos; elegí "y" **o** "o" |
| si es necesario / cuando corresponda | la condición exacta que lo dispara |

---

## 4. Sintaxis EARS (plantillas de llenar huecos)

EARS sirve para escribir requisitos funcionales con una sola interpretación, y es ideal para gente no técnica porque es completar una plantilla. Hay cinco patrones:

- **Ubicuo (regla siempre vigente):**
  *El sistema debe ‹respuesta›.*
  → "El sistema debe registrar la fecha y el usuario en cada cambio de estado de una tarea."

- **Por evento (Cuando):**
  *Cuando ‹disparador›, el sistema debe ‹respuesta›.*
  → "Cuando el administrador invita a un colaborador a un proyecto, el sistema debe enviarle un email con un enlace de acceso válido por 7 días."

- **Por estado (Mientras):**
  *Mientras ‹estado›, el sistema debe ‹respuesta›.*
  → "Mientras un proyecto está archivado, el sistema debe impedir crear tareas nuevas en él."

- **Comportamiento no deseado (Si… entonces):**
  *Si ‹condición no deseada›, entonces el sistema debe ‹respuesta›.*
  → "Si el servicio de email rechaza el envío de la invitación, entonces el sistema debe dejar la invitación como No enviada y mostrar el motivo devuelto."

- **Opcional según configuración (Donde):**
  *Donde ‹característica incluida›, el sistema debe ‹respuesta›.*
  → "Donde el espacio de trabajo tiene conectado un calendario externo, el sistema debe crear un evento al asignar una fecha de vencimiento a una tarea."

Se pueden combinar (ej. "Cuando… mientras…"), pero no encadenes varias respuestas en uno solo.

---

## 5. Historias de usuario e INVEST

Para el *porqué*, escribí una historia de usuario en formato Connextra:

> **Como** ‹rol›, **quiero** ‹objetivo›, **para** ‹beneficio›.

El beneficio ("para…") es lo más importante: si no sabés por qué lo quiere, probablemente el requisito esté mal entendido.

Chequeá la historia contra **INVEST**:
- **I**ndependiente — se puede construir y entregar por sí sola.
- **N**egociable — es un punto de partida para conversar, no un contrato cerrado.
- **V**aliosa — entrega valor a un usuario o al negocio.
- **E**stimable — el equipo puede dimensionar el esfuerzo.
- **S**mall (chica) — entra en una iteración; si no, partila.
- **T**esteable — tiene criterios de aceptación verificables.

Si la historia falla "Small", **rebánala** (por flujo, por tipo de dato, por regla de negocio), no la metas entera.

---

## 6. Criterios de aceptación: regla vs escenario

Un criterio de aceptación define **cómo sabemos que el requisito está cumplido**. Dos estilos, usá el que mejor describa cada caso:

**Estilo regla (checklist)** — para condiciones simples sí/no:
> CA-01 — El nombre de una etiqueta acepta entre 1 y 30 caracteres.
> CA-02 — Una tarea archivada no se puede editar.

**Estilo escenario (Gherkin / Dado-Cuando-Entonces)** — para flujos con pasos, viene de BDD:
> **CA-03**
> **Dado** que el usuario es administrador del espacio de trabajo y el plan admite más miembros,
> **Cuando** invita a un colaborador con un email válido,
> **Entonces** el sistema envía la invitación y muestra al invitado en la lista de miembros con estado "Invitación pendiente".

Reglas para criterios, sin excepción:
- Cada uno se responde con **sí/no** al probarlo. Si necesitás interpretar, está mal.
- Cubrí **camino feliz + al menos un caso de error/límite**. Lo que se rompe importa tanto como lo que funciona.
- Si toca un dominio sensible (los de `## Dominios sensibles` en `.claude/factory/convenciones.md`) que depende de un sistema externo, agregá un criterio explícito sobre el comportamiento ante rechazo o error de ese sistema.
- Numerá (`CA-01`…) y ligá cada uno al `RF` que verifica.
- Ningún criterio puede depender de algo `Pendiente`.

---

## 7. Checklist rápido antes de dar un requisito por bueno

- [ ] ¿Se entiende con una sola lectura y una sola interpretación?
- [ ] ¿Dice QUÉ, no CÓMO?
- [ ] ¿Es una sola cosa (sin "y"/"o" encadenando reglas)?
- [ ] ¿Puedo escribir un criterio sí/no para probarlo?
- [ ] ¿Está libre de vocabulario vago?
- [ ] ¿Tiene ID y sé de dónde viene?
- [ ] ¿No depende de nada marcado `Pendiente`?

Si alguna da que no, reescribí antes de seguir.
