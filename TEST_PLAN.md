# TEST PLAN — Optimizador de Envíos
## Microsprint: HU-01, HU-02, HU-03, HU-05

---

## 1. Identificación del Plan

| Campo | Detalle |
|---|---|
| **Nombre del proyecto** | Optimizador de Envíos |
| **Sistema bajo prueba** | Backend REST + Frontend Web (Zustand) + PostgreSQL — Módulo de Registro, Prioridad, Recomendación y Confirmación de Envíos |
| **Versión** | MVP 1.0 |
| **Fecha** | 25/03/2026 |
| **Equipo** | Nahuel Lemes y Santiago Angarita |

---

## 2. Contexto

El Optimizador de Envíos es una herramienta logística dirigida a empresas y personas que necesitan enviar productos dentro de Colombia y no saben qué proveedor de transporte elegir. El sistema automatiza la selección del proveedor más conveniente entre FedEx, DHL y un proveedor local, evaluando costo y tiempo de entrega según la prioridad del usuario (económico o rápido). Los proveedores se modelan con datos simulados (mock) en esta versión del MVP.

Este plan de pruebas cubre la versión 1.0 del MVP, que incluye las funcionalidades de registro de pedido, selección de prioridad, obtención de recomendación principal y confirmación del proveedor con persistencia en PostgreSQL. El flujo de datos va desde el estado global del frontend (Zustand) hasta PostgreSQL una vez el usuario confirma su selección.

---

## 3. Alcance de las Pruebas

### 3.1 Historias de Usuario que serán validadas

| ID | Historia de Usuario | Story Points |
|---|---|---|
| HU-01 | Registrar pedido de envío | 5 |
| HU-02 | Definir prioridad del envío | 3 |
| HU-03 | Obtener recomendación principal de proveedor de envío | 8 |
| HU-05 | Seleccionar y confirmar proveedor | 5 |

### 3.2 Fuera del ciclo

| ID | Historia de Usuario |
|---|---|
| HU-04 | Consultar opciones alternativas |

---

## 4. Estrategia de Pruebas

### 4.1 Niveles de prueba

**Pruebas funcionales (SerenityBDD + Cucumber)**
Se utilizará SerenityBDD con Cucumber para automatizar los escenarios BDD escritos en Gherkin, cubriendo los criterios de aceptación de cada HU. Los escenarios incluirán flujos exitosos, flujos alternativos y casos de borde.

**Pruebas de API / Integración (Karate)**
Se utilizará Karate para validar directamente los contratos de los endpoints REST expuestos por el backend, verificando códigos HTTP, estructura de respuesta, manejo de errores y consistencia de datos persistidos en PostgreSQL.

Endpoints cubiertos en este ciclo:
- `POST /api/v1/pedido` — recibe origen, destino, peso y prioridad; retorna la recomendación principal y las alternativas
- `POST /api/v1/pedido/confirmar`

**Pruebas de rendimiento (k6)**
Se utilizará k6 para validar que el motor de recomendación responde dentro de un umbral de tiempo aceptable bajo carga concurrente. Las pruebas se ejecutarán sobre el endpoint `POST /api/v1/pedido`, que es la operación más intensiva del sistema al consultar y comparar cotizaciones de los tres proveedores simultáneamente.

### 4.2 Tipos de casos de prueba

- **Casos positivos:** flujos válidos con datos correctos (registro exitoso, selección de prioridad, recomendación acertada, confirmación con persistencia)
- **Casos negativos:** datos inválidos y campos vacíos (origen/destino/peso faltantes, prioridad no seleccionada, proveedor inexistente)
- **Casos de borde:** valores límite de peso (0,001 Kg y 70 Kg), empates en costo o tiempo entre proveedores, empate total, un solo proveedor disponible
- **Casos de desempate:** validación de las reglas R7 y R8 — empate en costo desempatado por tiempo y empate en tiempo desempatado por costo

---

## 5. Criterios de Entrada y Salida

### 5.1 Criterios de entrada

Para iniciar la ejecución de pruebas de una Historia de Usuario se requiere:

- El build del entorno de QA está estable y desplegado
- Los endpoints de la HU están implementados y documentados
- La base de datos PostgreSQL está accesible y con el schema creado
- Los datos mock de proveedores (FedEx, DHL, Local) están configurados
- El servicio de OpenRouteService está operativo (autocompletado filtrado por Colombia)
- Los datos de prueba y la matriz de datos están definidos
- Los casos de prueba han sido revisados y aprobados
- DEV entregó la HU con las subtasks técnicas completadas

### 5.2 Criterios de salida

Una Historia de Usuario se considera probada y cerrada cuando:

- El 100% de los casos de prueba diseñados fueron ejecutados
- El 100% de los casos positivos pasan exitosamente
- No existen defectos abiertos de severidad **Alta** o **Crítica**
- Los defectos de severidad **Media** tienen plan de resolución acordado
- Los resultados fueron reportados y tienen el visto bueno del responsable de QA

---

## 6. Entorno de Pruebas

| Ítem | Detalle |
|---|---|
| **Ambiente** | Entorno de QA (staging) — aislado de producción |
| **Base de datos** | PostgreSQL — instancia exclusiva para pruebas, con datos controlados |
| **Backend** | API REST desplegada en ambiente QA |
| **Frontend** | Aplicación web con Zustand como estado global, desplegada en ambiente QA |
| **Proveedores** | Datos mock de FedEx, DHL y proveedor local — valores predefinidos por DEV |
| **Geolocalización** | OpenRouteService — autocompletado de ubicaciones filtrado por Colombia |
| **Datos de prueba** | Matrices de datos definidas por QA por cada HU (orígenes, destinos, pesos y prioridades) |
| **Navegadores (frontend)** | Chrome |

---

## 7. Herramientas

| Herramienta | Propósito |
|---|---|
| **SerenityBDD + Cucumber** | Automatización de pruebas funcionales end-to-end basadas en escenarios Gherkin (criterios de aceptación de las HU) |
| **Karate** | Pruebas de API: validación de contratos de endpoints, códigos HTTP, estructura de respuesta y datos persistidos en PostgreSQL |
| **k6** | Pruebas de rendimiento: validación del tiempo de respuesta del motor de recomendación bajo carga concurrente (`POST /api/v1/pedido`) |
| **GitHub Projects** | Gestión de casos de prueba, reporte de defectos y trazabilidad HU → casos de prueba |
| **Git** | Control de versiones de los scripts de prueba |

---

## 8. Roles y Responsabilidades

### QA: Nahuel Lemes

- Diseño de la matriz de datos de prueba
- Diseño y documentación de casos de prueba (positivos, negativos, de borde y de desempate)
- Escritura de escenarios Gherkin (.feature)
- Implementación de automatización con SerenityBDD + Cucumber
- Implementación de colecciones de prueba en Karate
- Implementación de scripts de rendimiento en k6
- Ejecución de pruebas y registro de resultados
- Reporte y seguimiento de defectos
- Generación de métricas e informe final

### DEV: Santiago Angarita

- Implementación de endpoints y lógica de negocio
- Configuración de datos mock de proveedores (FedEx, DHL, Local)
- Configuración de base de datos PostgreSQL y migraciones
- Resolución de defectos reportados por QA
- Soporte en la configuración del entorno de QA
- Revisión de DTOs y validaciones de entrada

### Responsabilidad compartida

- Refinamiento de criterios de aceptación
- Definición de datos de prueba complejos (escenarios de empate, valores límite)
- Acuerdos sobre severidad y prioridad de defectos

---

## 9. Cronograma y Estimación

La estimación de esfuerzo de QA se calcula tomando como referencia los Story Points del microsprint.

| HU | Story Points | Tareas QA | Esfuerzo estimado QA |
|---|---|---|---|
| HU-01 | 5 | 7 subtasks (T13–T19) | Alto |
| HU-02 | 3 | 7 subtasks (T13–T19) | Medio |
| HU-03 | 8 | 7 subtasks (T13–T19) | Alto |
| HU-05 | 5 | 7 subtasks (T12–T18) | Alto |
| **Total** | **21** | **28 subtasks** | **Alto** |

**Distribución sugerida:**

| Fase | Actividad |
|---|---|
| Fase 1 — Diseño | Diseño de matrices de datos y casos de prueba (paralelo al desarrollo) |
| Fase 2 — Automatización | Implementación de scripts SerenityBDD y Karate |
| Fase 3 — Ejecución | Ejecución de pruebas sobre build estable en ambiente QA |
| Fase 4 — Cierre | Reporte de defectos, métricas y entregables finales |

---

## 10. Entregables de Prueba

| Artefacto | Descripción |
|---|---|
| `TEST_PLAN.md` | Este documento — plan de pruebas del microsprint |
| Matrices de datos de prueba | Tablas con combinaciones de datos válidos e inválidos por HU |
| Casos de prueba documentados | Descripción, precondiciones, pasos, resultado esperado y resultado obtenido |
| Escenarios Gherkin (.feature) | Escenarios BDD alineados a los criterios de aceptación de cada HU |
| Scripts de automatización SerenityBDD | Implementación de los escenarios funcionales automatizados |
| Colecciones Karate | Scripts de prueba de API para cada endpoint cubierto |
| Scripts k6 | Script de prueba de rendimiento para `POST /api/v1/pedido` |
| Reporte de ejecución SerenityBDD | Reporte HTML generado por Serenity con resultados de pruebas funcionales |
| Reporte de ejecución Karate | Reporte de resultados de pruebas de API |
| Reporte k6 | Métricas de rendimiento: tiempo de respuesta, throughput y tasa de error bajo carga concurrente (`POST /api/v1/pedido`) |
| Informe de defectos | Listado de defectos encontrados con severidad, prioridad y estado |
| Métricas de cobertura | % de casos ejecutados, % pasados, % fallidos, defectos por HU |

---

## 11. Riesgos y Contingencias

### Riesgos de producto

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Fallos de lógica en el motor de recomendación que asigne la opción más cara en lugar de la más económica | Alto — el usuario recibe una recomendación incorrecta que genera mayores costos de envío | QA diseña casos de prueba exhaustivos para todas las combinaciones de prioridad y escenarios de desempate (Reglas R5–R8) |
| Aplicación incorrecta de las reglas de desempate (R7 y R8) | Alto — ante empates, el sistema podría recomendar una opción subóptima | QA diseña casos de prueba específicos para empate doble, triple y empate total en costo y tiempo |
| Inconsistencia entre datos del estado global (Zustand) y los persistidos en PostgreSQL | Alto — el usuario confirma datos que no coinciden con lo almacenado en base de datos | QA valida en HU-05 que los datos persistidos en PostgreSQL coinciden exactamente con los del estado global |
| Datos de entrada incorrectos que llegan al motor de recomendación | Medio — cotizaciones con cálculos inválidos por peso o ubicación incorrectos | QA cubre validaciones de entrada en HU-01 (campos obligatorios, rango de peso, cobertura Colombia) antes de que el flujo llegue a HU-03 |
| Peso fuera de rango aceptado por el sistema sin validación adecuada | Medio — el sistema podría cotizar pedidos con peso inválido (< 0,001 Kg o > 70 Kg) | QA diseña casos de borde específicos para los límites de peso: 0,001 Kg, 70 Kg, 0,0009 Kg, 70,001 Kg, 0 Kg y negativos |

### Riesgos de proyecto

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Entorno de QA inestable o no disponible | Alto — bloquea la ejecución de pruebas | Acordar criterios de entrada claros; QA no inicia ejecución sin build estable |
| Servicio de OpenRouteService no disponible durante pruebas | Medio — bloquea validación de autocompletado y geolocalización | Preparar datos mock de ubicaciones como fallback para pruebas de backend |
| HU entregada por DEV sin criterios de aceptación completos | Medio — casos de prueba mal definidos | QA participa del refinamiento previo al desarrollo |
| Tiempo insuficiente para automatización completa en el ciclo | Medio — cobertura automatizada parcial | Priorizar automatización de flujos críticos (motor de recomendación y reglas de desempate) y ejecutar el resto manualmente |
| Cambio de scope durante el microsprint | Alto — casos de prueba desactualizados | Cualquier cambio en HU debe pasar por refinamiento formal antes de ser testeado |
