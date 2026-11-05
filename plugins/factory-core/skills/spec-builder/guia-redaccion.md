# Guía de redacción de requisitos y criterios

Estándar de calidad de `spec-builder`, adaptado a documentación funcional escrita en prosa: acá los requisitos y criterios se escriben en prosa numerada llana (como suelen estar los requerimientos funcionales de un repo de documentación; respetá el estilo que describa `## Documentación` en `.claude/factory/convenciones.md`), no con IDs `RF-`/`CA-` ni sintaxis EARS visible. Las propiedades de fondo (verificable, singular, no ambiguo) siguen valiendo igual — cambia el envoltorio, no el estándar.

## 1. Las propiedades de un buen requisito

Cada requisito individual debe ser:

- **Necesario** — si lo eliminás, queda un hueco real. Si nadie lo extraña, sobra.
- **No ambiguo** — admite una sola interpretación.
- **Singular** — describe una cosa. Nada de encadenar reglas con "y"/"o".
- **Verificable** — existe una forma objetiva de probar que se cumple. Si no se puede responder sí/no, es un deseo, no un requisito.
- **Factible** — se puede construir dentro de las restricciones conocidas (y confirmadas por el scout de código, no supuestas).
- **Trazable** — se sabe de qué necesidad, mensaje de cliente o comportamiento de código existente proviene. No hace falta un ID nuevo: alcanza con que quede claro en el texto o en la sección "Relacionados" del documento.

El conjunto de requisitos debe ser **completo**, **consistente entre sí** y **consistente con lo que el scout encontró en el código** — si un requisito contradice el comportamiento actual de alguno de los repos relacionados, decilo explícitamente en vez de dejarlo pasar.

## 2. Lo que un requisito NUNCA debe tener

- **Solución disfrazada.** El requisito dice *qué* tiene que pasar, no *cómo* implementarlo. Excepción: una restricción real impuesta (ej. "debe autenticar contra el proveedor de identidad (SSO) del cliente"), que se marca como restricción, no como diseño.
- **Requisito compuesto.** Si tiene "y", "o", "además" encadenando reglas, son varios requisitos. Partilos.
- **Voz pasiva sin actor.** ❌ "Las invitaciones serán validadas." ✅ "El sistema debe validar que el email del invitado no pertenezca ya al espacio de trabajo antes de enviar la invitación."
- **Adjetivos no medibles.** Ver vocabulario prohibido abajo.
- **Negaciones difíciles de probar.** ❌ "El sistema no debe ser lento." ✅ "El sistema debe responder la búsqueda en menos de 2 segundos para espacios de trabajo de hasta 10.000 tareas."
- **Dependencia de algo sin definir.** Un requisito o criterio que depende de un `Pendiente` no puede formar parte de "Listo para desarrollo".
- **Un supuesto no verificado contra el código.** Si el scout puede confirmarlo (mirando el modelo, el endpoint, el componente), no lo dejes como supuesto — confirmalo o marcalo `Necesita validación técnica`.

## 3. Vocabulario prohibido

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

## 4. EARS como andamiaje interno (no como formato de salida)

Cuando un requisito te salga ambiguo, probalo primero con estas plantillas de EARS para encontrar la única interpretación correcta — pero después escribilo en prosa llana en el documento, como el resto de la documentación:

- **Regla siempre vigente:** *El sistema debe ‹respuesta›.*
- **Por evento:** *Cuando ‹disparador›, el sistema debe ‹respuesta›.* → "Cuando el administrador invita a un colaborador a un proyecto, el sistema debe enviarle un email con un enlace de acceso válido por 7 días."
- **Por estado:** *Mientras ‹estado›, el sistema debe ‹respuesta›.* → "Mientras un proyecto está archivado, el sistema debe impedir crear tareas nuevas en él."
- **Comportamiento no deseado:** *Si ‹condición no deseada›, entonces el sistema debe ‹respuesta›.* → "Si el servicio de email rechaza el envío de la invitación, entonces el sistema debe dejar la invitación como No enviada y mostrar el motivo devuelto."
- **Opcional según configuración:** *Donde ‹característica incluida›, el sistema debe ‹respuesta›.* → "Donde el espacio de trabajo tiene conectado un calendario externo, el sistema debe crear un evento al asignar una fecha de vencimiento a una tarea."

## 5. Criterios de aceptación: regla vs escenario

Dos estilos, elegí el que mejor describa cada caso — numerados en prosa, sin prefijo `CA-` obligatorio (aunque no molesta si el documento que estás espejando ya numera así):

**Estilo regla (checklist)** — condiciones simples sí/no:
> 1. El nombre de una etiqueta acepta entre 1 y 30 caracteres.
> 2. Una tarea archivada no se puede editar.

**Estilo escenario (Gherkin)** — para flujos con pasos:
> **Dado** que el usuario es administrador del espacio de trabajo y el plan admite más miembros, **cuando** invita a un colaborador con un email válido, **entonces** el sistema envía la invitación y muestra al invitado en la lista de miembros con estado "Invitación pendiente".

Reglas sin excepción:
- Cada uno se responde con **sí/no** al probarlo.
- Cubrí **camino feliz + al menos un caso de error/límite**.
- Si toca un dominio sensible (los de `## Dominios sensibles` en `.claude/factory/convenciones.md`) que depende de un sistema externo, agregá un criterio explícito sobre el comportamiento ante rechazo o error de ese sistema.
- Ningún criterio puede depender de algo `Pendiente`.

## 6. Checklist rápido antes de dar un requisito por bueno

- [ ] ¿Se entiende con una sola lectura y una sola interpretación?
- [ ] ¿Dice QUÉ, no CÓMO?
- [ ] ¿Es una sola cosa (sin "y"/"o" encadenando reglas)?
- [ ] ¿Puedo escribir un criterio sí/no para probarlo?
- [ ] ¿Está libre de vocabulario vago?
- [ ] ¿Lo verifiqué contra el código real (scout) en vez de asumirlo?
- [ ] ¿No depende de nada marcado `Pendiente`?

Si alguna da que no, reescribí antes de seguir.
