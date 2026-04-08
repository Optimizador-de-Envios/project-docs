# TEST PLAN — Optimizador de Envíos
## MVP base + MVP v2: HU-01 a HU-09

---

## 1. Identificación del Plan

| Campo | Detalle |
|---|---|
| **Proyecto** | Optimizador de Envíos |
| **Sistema bajo prueba** | Backend REST (`shipment-service` + `user-service`) + Frontend Web (Zustand + React Router) + PostgreSQL + integración OpenRouteService + Leaflet |
| **Versión** | MVP 2.0 |
| **Fecha** | 07/04/2026 |
| **Equipo** | Nahuel Lemes y Santiago Angarita |

---

## 2. Contexto

El Optimizador de Envíos automatiza la selección del proveedor de transporte más conveniente (FedEx, DHL o proveedor local — datos simulados) para envíos dentro de Colombia, evaluando costo y tiempo según la prioridad del usuario.

Este plan cubre el **MVP base** (HU-01 a HU-05) y el **MVP v2** (HU-06 a HU-09): cotización, recomendación, alternativas, confirmación, visualización de ruta en mapa, registro, login/logout y consulta de pedidos propios.

**Flujo principal:** registro → login obligatorio → origen/destino (autocompletado ORS, solo Colombia) → peso y prioridad → recomendación → alternativas y ruta → confirmación → persistencia → historial → logout. Sin sesión activa, toda funcionalidad operativa queda bloqueada.

---

## 3. Alcance de las Pruebas

### 3.1 Historias de Usuario validadas

| Microsprint | ID | Historia de Usuario | SP |
|---|---|---|---:|
| **MS1** (2 días) | HU-01 | Registrar pedido de envío | 5 |
| | HU-02 | Definir prioridad del envío | 3 |
| **MS2** (2 días) | HU-03 | Obtener recomendación principal | 8 |
| | HU-04 | Obtener opciones alternativas | 3 |
| | HU-05 | Seleccionar y confirmar proveedor | 5 |
| **MS3** (2 días) | HU-06 | Visualizar ruta en mapa | 5 |
| **MS4** (2 días) | HU-07 | Registrar usuario | 5 |
| | HU-08 | Iniciar sesión | 3 |
| | HU-09 | Consultar pedidos del usuario | 5 |
| **Total** | | | **42** |

### 3.2 Fuera de alcance

Gestión avanzada de perfiles · APIs reales de proveedores logísticos · Módulo de pagos · Tracking post-confirmación · Pedidos internacionales.

---

## 4. Estrategia de Pruebas

Trazabilidad: PRD → reglas de negocio → HU → criterios de aceptación → casos de prueba.

| Reglas PRD | HU | Foco |
|---|---|---|
| 1–4 | HU-01, HU-02 | Datos obligatorios, cobertura Colombia, peso, prioridad |
| 5–8 | HU-03 | Recomendación y desempate |
| 9 | HU-04 | Alternativas sin duplicar recomendación |
| 10–11, 24 | HU-05, HU-09 | Confirmación, persistencia, asociación a usuario |
| 12–16 | HU-06 | Ruta, ORS, coordenadas, marcadores, Leaflet |
| 17–23, 28 | HU-07, HU-08 | Registro, login obligatorio, logout |
| 25–27 | HU-09 | Historial propio, aislamiento entre usuarios |

### 4.1 Niveles de prueba

| Nivel | Herramienta | Alcance |
|---|---|---|
| Manuales (aceptación + exploratorias) | Navegador + DevTools | Flujos E2E, UX, mapa, rutas protegidas, casos donde la inspección humana agrega valor |
| Funcional E2E | Serenity Screenplay + Cucumber | Escenarios Gherkin con actores/tareas/preguntas modelando los criterios de aceptación |
| API / Integración | Karate | Contratos REST, códigos HTTP, estructura de respuesta, seguridad, datos persistidos |
| Rendimiento | k6 | Estabilidad y tiempos de respuesta bajo carga concurrente |

**Endpoints cubiertos:** `POST /api/users/register` · `POST /api/users/login` · `POST /api/v1/pedido` · `POST /api/v1/pedido/confirmar` · `GET /api/v1/pedido/mis-pedidos` · integración ORS.

**Seguridad transversal:** autenticación obligatoria, rechazo sin token, aislamiento de pedidos por usuario, bloqueo post-logout — validado en pruebas manuales, Serenity y Karate.

### 4.2 Tipos de casos de prueba

- **Positivos:** registro, login, pedido válido, recomendación, alternativas, ruta, confirmación, historial, logout.
- **Negativos:** campos vacíos, correo duplicado, contraseña inválida, credenciales incorrectas, uso sin sesión, prioridad/proveedor no seleccionados, token ausente/inválido, origen/destino fuera de Colombia.
- **Borde:** peso 0,0009 / 0,001 / 70 / 70,001 Kg; coordenadas ausentes; ruta vacía; usuario sin pedidos; un solo proveedor.
- **Desempate:** empate costo → desempata por tiempo; empate tiempo → desempata por costo (Reglas 7–8).
- **Geoespaciales:** autocompletado ORS filtrado Colombia, conversión `[lng,lat]→[lat,lng]`, marcadores, ajuste viewport, error sin datos.
- **Seguridad:** aislamiento por usuario, rechazo no autenticado, logout, confirmación asociada a usuario autenticado.

**Matriz de borde mínima:**

| Dimensión | Datos | Resultado esperado |
|---|---|---|
| Peso < mínimo | 0,0009 Kg | Rechazado |
| Peso límite válido | 0,001 Kg / 70 Kg | Aceptado |
| Peso > máximo | 70,001 Kg | Rechazado |
| Cobertura válida | Col → Col | Aceptado |
| Origen inválido | Ext → Col | Rechazado |
| Destino inválido | Col → Ext | Rechazado |

### 4.3 Pruebas manuales

| Grupo | HU | Objetivo |
|---|---|---|
| Flujo base cotización | HU-01–05 | Flujo completo autenticado: origen, destino, peso, prioridad → recomendación → alternativas → confirmación |
| Validaciones de entrada | HU-01, 02, 07, 08 | Mensajes de error para campos vacíos, peso fuera de rango, cobertura, correo duplicado, credenciales |
| Ruta en mapa | HU-06 | Ruta visible, marcadores correctos, ajuste automático, caso sin datos |
| Autenticación | HU-07, 08, 09 | Registro, login, rutas protegidas, logout |
| Historial | HU-09 | Pedidos propios, aislamiento, estado sin pedidos |
| Exploratoria UX | HU-01–09 | Navegación, mensajes ambiguos, inconsistencias, bloqueos |

**Criterios mínimos:** ≥1 flujo feliz E2E por microsprint · casos críticos antes de cerrar HU · validar bloqueo sin sesión y post-logout · HU-06 en navegador real · HU-09 con dos usuarios · evidencia con pasos reproducibles.

### 4.4 Pruebas de rendimiento (k6)

| Escenario | Endpoint | Carga | Umbral |
|---|---|---|---|
| Smoke | `POST /api/v1/pedido` | 1 VU × 1 min | `failed < 1%`, `checks > 95%` |
| Carga recomendación | `POST /api/v1/pedido` | 5→25 VU × 5 min | `p95 < 1200 ms`, `failed < 1%` |
| Confirmación | `POST /api/v1/pedido/confirmar` | 10 VU × 3 min | `p95 < 1500 ms`, `failed < 1%` |
| Historial | `GET /api/v1/pedido/mis-pedidos` | 10→20 VU × 5 min | `p95 < 800 ms`, `failed < 1%` |
| Login | `POST /api/users/login` | 5→10 VU × 3 min | `p95 < 1000 ms`, `failed < 1%` |

**Preparación:** usuarios precreados · JWT pre-generados · datos dentro de Colombia · mocks de ORS para carga · limpieza post-ejecución.

**Métricas reportadas:** p95, p99, tasa de error, checks, throughput, recursos observados, conclusión (aprobado / con observaciones / rechazado).

### 4.5 Pruebas funcionales (Serenity Screenplay)

| Flujo | HU | Validaciones principales |
|---|---|---|
| Registro y autenticación | HU-07, 08 | Registro, correo duplicado, contraseña inválida, credenciales incorrectas, rutas protegidas, logout |
| Cotización | HU-01, 02 | Origen, destino, cobertura, peso, prioridad, validaciones de campos y límites |
| Recomendación y alternativas | HU-03, 04 | Recomendación por prioridad, desempate, alternativas distintas a recomendación |
| Ruta en mapa | HU-06 | Ruta, marcadores, ajuste viewport, mensaje sin datos |
| Confirmación e historial | HU-05, 09 | Persistencia, proveedor seleccionado, historial propio, aislamiento |

**Convenciones:** `Actor` (autenticado/no autenticado) · `Task` (acciones de negocio) · `Question` (validaciones) · `Interaction` (acciones UI) · trazabilidad Gherkin ↔ HU ↔ criterio de aceptación.

---

## 5. Criterios de Entrada y Salida

### 5.1 Entrada (para iniciar pruebas de una HU)

- Build QA estable con `user-service`, `shipment-service` y frontend desplegados.
- PostgreSQL accesible con schemas/migraciones aplicados.
- Mocks de proveedores y secreto JWT configurados.
- ORS operativo o mock/stub disponible; Leaflet renderizable.
- Serenity Screenplay y k6 configurados en ambiente QA.
- Datos de prueba definidos (usuarios, JWT semilla, coordenadas, pesos, prioridades).
- Casos de prueba revisados y aprobados.
- DEV entregó la HU sin defectos bloqueantes conocidos.

### 5.2 Salida (para cerrar una HU)

- 100% de casos diseñados ejecutados y aprobados (críticos, positivos, negativos, borde, seguridad).
- Sin defectos abiertos de severidad Alta o Crítica; los de severidad Media tienen plan de resolución.
- Escenarios Serenity y umbrales k6 aprobados.
- Evidencia manual registrada.
- Resultados reportados con visto bueno de QA.

---

## 6. Entorno de Pruebas

| Ítem | Detalle |
|---|---|
| **Ambiente** | QA (staging) aislado de producción |
| **Base de datos** | PostgreSQL exclusiva para pruebas |
| **Backend** | `shipment-service` + `user-service` en QA |
| **Frontend** | React + Zustand + React Router en QA |
| **Autenticación** | JWT con secreto compartido entre servicios |
| **Proveedores** | Mock FedEx, DHL, Local |
| **Geolocalización** | OpenRouteService (real controlado o mock/stub) |
| **Mapa** | Leaflet |
| **Navegador** | Chrome |

---

## 7. Herramientas

| Herramienta | Propósito |
|---|---|
| Navegador + DevTools | Pruebas manuales, validación visual, captura de evidencia |
| Serenity Screenplay + Cucumber | Automatización funcional E2E |
| Karate | Pruebas de API: contratos, seguridad, datos persistidos |
| k6 | Rendimiento: smoke y carga |
| GitHub Projects | Gestión de casos, defectos y trazabilidad |
| Git | Versionado de scripts de prueba |

---

## 8. Roles y Responsabilidades

**QA — Nahuel Lemes:** diseño de matrices y casos de prueba · escenarios Gherkin · ejecución manual · automatización Serenity, Karate y k6 · análisis de resultados · validación de seguridad funcional · reporte de defectos · métricas e informe final.

**DEV — Santiago Angarita:** implementación de endpoints y lógica · mocks de proveedores · PostgreSQL y migraciones · JWT · integración ORS · resolución de defectos · soporte entorno QA y datos semilla · revisión de DTOs y contratos.

**Compartido:** refinamiento de criterios de aceptación · datos de prueba complejos (empates, límites, coordenadas) · acuerdos de severidad/prioridad · trazabilidad PRD → HU → casos → resultados.

---

## 9. Cronograma y Estimación

| Microsprint | HU | SP | Tareas QA principales | Esfuerzo QA |
|---|---|---:|---|---|
| MS1 (2d) | HU-01 | 5 | Matriz de datos, validaciones de campos, cobertura, límites de peso | Alto |
| | HU-02 | 3 | Selección de prioridad, prioridad ausente | Medio |
| MS2 (2d) | HU-03 | 8 | Recomendación por prioridad, desempate, contratos | Alto |
| | HU-04 | 3 | Alternativas, ausencia, consistencia con recomendación | Medio |
| | HU-05 | 5 | Confirmación, persistencia, token obligatorio | Alto |
| MS3 (2d) | HU-06 | 5 | Ruta, marcadores, coordenadas, datos insuficientes | Alto |
| MS4 (2d) | HU-07 | 5 | Registro, correo duplicado, campos obligatorios | Alto |
| | HU-08 | 3 | Login, credenciales inválidas, rutas protegidas, logout | Medio |
| | HU-09 | 5 | Historial propio, aislamiento, sin pedidos | Alto |
| **Total** | | **42** | | **Alto** |

---

## 10. Entregables de Prueba

| Artefacto | Descripción |
|---|---|
| `TEST_PLAN.md` | Este documento |
| Matrices de datos | Combinaciones válidas e inválidas por HU |
| Casos de prueba | Precondiciones, pasos, resultado esperado y obtenido |
| Evidencias manuales | Capturas, notas exploratorias, defectos |
| Escenarios Gherkin (`.feature`) | Escenarios BDD por criterio de aceptación |
| Scripts Serenity Screenplay | Actores, tareas, preguntas e interacciones |
| Colecciones Karate | Pruebas de API por endpoint |
| Scripts k6 | Smoke, carga y rendimiento |
| Reportes Serenity / Karate / k6 | Resultados de ejecución con métricas |
| Informe de defectos | Severidad, prioridad, estado |
| Métricas de cobertura | % ejecutados, pasados, fallidos, defectos/HU |

---

## 11. Riesgos y Contingencias

### Riesgos de producto

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Lógica de recomendación incorrecta | Alto | Casos exhaustivos para todas las combinaciones de prioridad y desempate |
| Alternativas inconsistentes con recomendación | Medio | Validar que alternativas sean distintas y consistentes |
| Inconsistencia frontend ↔ API ↔ PostgreSQL | Alto | Validar coincidencia de datos en HU-05 y HU-09 |
| Conversión errónea de coordenadas ORS → Leaflet | Alto | Cubrir conversión `[lng,lat]→[lat,lng]` en HU-06 |
| ORS no disponible o con límite de cuota | Medio | Mocks/stubs como fallback; validar manejo de error |
| Exposición de pedidos entre usuarios | Crítico | Validar tokens, aislamiento y consulta de pedidos ajenos |
| Registro/login/logout sin validaciones | Alto | Cubrir correo único, contraseña, credenciales, rutas protegidas, post-logout |

### Riesgos de proyecto

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Entorno QA inestable | Alto | No iniciar ejecución sin build estable |
| Contratos de autenticación desalineados | Alto | Revisar contratos y JWT compartido antes de ejecución |
| ORS no disponible durante pruebas | Medio | Mock/stub como fallback |
| HU sin criterios de aceptación completos | Medio | QA participa en refinamiento previo |
| Tiempo insuficiente para automatización | Medio | Priorizar flujos críticos; completar con manuales documentadas |
| Cambio de scope en microsprint | Alto | Refinamiento formal antes de testear cambios |
