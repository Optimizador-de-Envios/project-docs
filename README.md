# Optimizador de Envíos — Documentación del Proyecto

Herramienta logística que automatiza la selección del proveedor de transporte más conveniente (FedEx, DHL o proveedor local) para envíos dentro de Colombia, evaluando costo y tiempo de entrega según la prioridad del usuario.

---

## Acceso al proyecto

Para acceder al proyecto, utiliza el siguiente enlace: [Optimizador de Envíos - Infra](#aquí-va-el-enlace-del-repositorio).

## Documentación

| Documento | Descripción | Incluye |
|---|---|---|
| [PRD.md](PRD.md) | **Product Requirements Document** — Define el producto, su visión, objetivos y alcance del MVP. | Problema y propuesta de valor, objetivos del producto, reglas de negocio generales (R1–R11), alcance del MVP (in/out of scope), riesgos técnicos y de negocio con mitigaciones. |
| [USER_STORIES.md](USER_STORIES.md) | **Historias de Usuario** — Especificación funcional del sistema desde la perspectiva del usuario. | 5 historias de usuario (HU-01 a HU-05) con descripción, valor de negocio, reglas relacionadas, Definition of Ready, criterios de aceptación en Gherkin, Definition of Done y tabla de estimación en Story Points. |
| [SUBTASKS.md](SUBTASKS.md) | **Subtareas técnicas** — Desglose de trabajo por historia de usuario para DEV y QA. | Subtareas de backend, frontend y QA para cada HU, con identificadores únicos (ej. HU01-T01), estimaciones en puntos y justificación del esfuerzo por rol. |
| [TEST_PLAN.md](TEST_PLAN.md) | **Plan de Pruebas** — Estrategia y planificación de QA para el MVP. | Identificación del plan, contexto, alcance por microsprint, estrategia de pruebas (SerenityBDD + Cucumber, Karate, k6), criterios de entrada/salida, entorno, herramientas, roles y responsabilidades, cronograma, entregables y riesgos de producto/proyecto. |
| [TEST_CASES.md](TEST_CASES.md) | **Casos de Prueba** — Especificación detallada de los escenarios de prueba. | 21 casos de prueba (10 positivos, 7 negativos, 2 de borde, 2 de desempate) organizados por microsprint y HU, con prioridad, escenario BDD, precondiciones, datos, pasos de ejecución y resultado esperado. |

---

## Autores

| Rol | Nombre |
|---|---|
| **QA** | Nahuel Lemes |
| **DEV** | Santiago Angarita |

---