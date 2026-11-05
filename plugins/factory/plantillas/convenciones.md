# Convenciones del repo para dev-factory

> Copiá este archivo a `.claude/factory/convenciones.md` en la raíz de tu repo (o corré `/factory:init`) y completá cada sección.
> Es la **única fuente de verdad** que leen los comandos y subagentes de `factory`: el motor trae el método, este archivo trae lo que es propio de tu repo.
> Si una sección no aplica, escribí `No aplica.` — no la borres: los agentes distinguen "no aplica" de "nadie lo completó".
> Todo lo que está entre `<...>` es un ejemplo para reemplazar.

## Stack

<Lenguaje, framework y versión, librería de UI, manejo de estado, cliente HTTP, i18n, framework de tests.>

<Ej.: TypeScript 5 + React 19 + Vite. Estado global con Zustand. Tests con Vitest + Testing Library.>

## Estructura de directorios

<Una línea por carpeta relevante: qué vive ahí.>

- `<src/components/>` → <componentes de UI reutilizables>
- `<src/api/>` → <funciones de acceso a la API, por dominio>
- `<src/pages/>` → <rutas>

## Patrones y nombres

<Cómo se nombran y se escriben las piezas típicas. Citá un archivo real como modelo de cada una: es la forma más barata de que el código nuevo salga con el estilo de la casa.>

- <Componentes: `{Dominio}{Acción}Panel.tsx` — modelo: `src/components/TaskEditPanel.tsx`>
- <Respuesta de la API: `{ status, data }` → validar `status !== 200`>

## Antipatrones

<Lo que nunca hay que hacer en este repo. Los agentes lo usan para revisar y para escribir "qué NO hacer" en cada paso.>

- <Llamar a la API directo desde un componente.>
- <Strings de UI hardcodeados.>
- <`console.log` sin remover.>

## Capas para cortar pasos

<Orden de abajo hacia arriba en que se construye una feature en este repo. `feature-impl-planner` corta los pasos siguiendo este orden.>

1. <Tipos e interfaces>
2. <Acceso a datos>
3. <Componentes, de los internos a los que los componen>
4. <Página / ruta>
5. <Cableado final>

## Verificación

<Comandos exactos. Siempre ACOTADOS a lo que se tocó: los agentes reemplazan `<dirs>`/`<archivos>`/`<patrón>`.>

- Tests acotados: `<npx vitest run <dirs> --passWithNoTests>`
- Un test puntual: `<npx vitest run <patrón>>`
- Lint: `<npx eslint <archivos>>`
- Tipos: `<npx tsc --noEmit>` (solo si se tocaron tipos)
- Cobertura de lo nuevo: `<npx vitest run <dirs> --coverage --coverage.include='<archivos>'>` — exigencia: <100% sobre lo nuevo | no aplica>

Trampas del runner (qué significa realmente un rojo o un verde acá):

- <Ej.: el threshold global de cobertura falla siempre; nunca correr la suite con cobertura global.>
- <Ej.: sin `--passWithNoTests`, un directorio sin tests da error.>

## Ruido conocido del árbol

<Archivos que suelen aparecer modificados en la máquina del dev y NO son parte del trabajo. Los flujos los toman como baseline y nunca los stagean ni revierten.>

- <`.env.development` — config local apuntada a otro ambiente>

## Git y commits

- Rama base: `<main>`
- Branch de feature: `<feature/<slug>>`
- ¿El flujo puede commitear solo?: `<sí | no — solo muestra el diff y el mensaje sugerido>`
- ¿Puede pushear y abrir PR?: `<sí, como draft | no>`
- Formato del mensaje: <ej.: `Fix:` / `Feat:` + subject en infinitivo; cuerpo síntoma → causa → solución; trailer `Co-Authored-By`.>
- Granularidad: <ej.: un commit por paso de feature; un commit por grupo de causa raíz en fixes.>

## Documentación

- Carpeta de features: `<docs/features/>` (cada feature en `<docs/features/<slug>/>`)
- Planes de bugs: `<docs/features/<área>/<branch>-plan.md>`
- Doc de fixes: `<docs/features/<área>/<branch>.md>` — <uno por branch | uno por bug>
- Áreas: <cómo se enumeran (ej.: `Glob docs/features/*/`); nunca una lista fija>
- Estilo y plantilla de la sección de un fix:

```markdown
<## Fix: <síntoma>
### Síntoma
### Causa
### Solución
### Verificación>
```

## Reglas de negocio sensibles

<Reglas que un cambio puede romper sin que ningún test lo detecte. `change-reviewer` las verifica explícitamente si el cambio las toca.>

- <Ej.: el total de una orden se calcula siempre en el backend; el front nunca lo recalcula.>

## Dominios sensibles

<Temas que obligan a frenar y pedir validación explícita antes de asumir comportamiento (dinero, impuestos, permisos, datos de terceros, multi-tenant...).>

- <Ej.: cobros y reembolsos>
- <Ej.: datos cruzados entre cuentas (multi-tenant)>

## Repos relacionados

<Otros repos del mismo producto. Sirve para tipificar ítems que no son de este repo y para el scout de `spec-builder`.>

- `<owner/repo-front>` — <front>
- `<owner/repo-back>` — <back>

## E2E

<Opcional. Si no hay E2E, `No aplica.`>

- Herramienta: `<Playwright>` — comando: `<npx playwright test <spec> --reporter=list>`
- Carpeta de specs: `<e2e/tests/>`
- Helper de login: `<e2e/support/auth.ts → loginAsTestUser>`
- URL de la app: `<${APP_URL:-http://localhost:3000}>`
- Variables de credenciales de QA (nombres, nunca valores): `<QA_TEST_EMAIL>`, `<QA_TEST_PASSWORD>`
