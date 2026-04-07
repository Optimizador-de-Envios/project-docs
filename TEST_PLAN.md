# TEST PLAN — Optimizador de Envíos
## MVP base + MVP v2: HU-01 a HU-09

---

## 1. Identificación del Plan

| Campo | Detalle |
|---|---|
| **Nombre del proyecto** | Optimizador de Envíos |
| **Sistema bajo prueba** | Backend REST (`shipment-service` + `user-service`) + Frontend Web (Zustand + React Router) + PostgreSQL. OpenRouteService se valida como integración externa del sistema; Leaflet se valida como librería de visualización en frontend. |
| **Versión** | MVP 2.0 |
| **Fecha** | 07/04/2026 |
| **Equipo** | Nahuel Lemes y Santiago Angarita |

---

## 2. Contexto

El Optimizador de Envíos es una herramienta logística dirigida a empresas y personas que necesitan enviar productos dentro de Colombia y no saben qué proveedor de transporte elegir. El sistema automatiza la selección del proveedor más conveniente entre FedEx, DHL y un proveedor local, evaluando costo y tiempo de entrega según la prioridad del usuario: menor costo o menor tiempo de entrega. Los proveedores se modelan con datos simulados (mock) en esta versión del MVP.

Este plan de pruebas cubre el MVP base y las funcionalidades de crecimiento definidas para el **MVP v2** en el PRD. Además del flujo de cotización, recomendación, alternativas y confirmación del proveedor, se incorporan la visualización de la ruta en mapa (HU-06), el registro de usuario (HU-07), el inicio de sesión (HU-08) y la consulta de pedidos asociados al usuario autenticado (HU-09).

La aplicación exige iniciar sesión antes de usar cualquier funcionalidad operativa. Si no existe una sesión activa, el usuario no puede registrar pedidos, seleccionar prioridad, calcular recomendaciones, visualizar rutas, confirmar proveedores ni consultar historial; el sistema debe solicitar autenticación o redirigir al login. El registro permite crear una cuenta, pero el uso de la plataforma queda condicionado a una sesión autenticada. La funcionalidad de cierre de sesión forma parte del comportamiento actual y debe validarse.

El flujo principal esperado es: registro si el usuario aún no tiene cuenta, inicio de sesión obligatorio, ingreso de origen y destino dentro de Colombia con autocompletado de OpenRouteService, ingreso de peso y prioridad, cálculo de recomendación, visualización de alternativas y ruta, selección y confirmación del proveedor, persistencia del pedido asociado al usuario autenticado, posterior consulta del historial propio y cierre de sesión cuando corresponda.

---

## 3. Alcance de las Pruebas

### 3.1 Historias de Usuario que serán validadas

**Microsprint 1 (2 días) — MVP base**

| ID | Historia de Usuario | Story Points |
|---|---|---:|
| HU-01 | Registrar pedido de envío | 5 |
| HU-02 | Definir prioridad del envío | 3 |
| **Subtotal MS1** | | **8** |

**Microsprint 2 (2 días) — MVP base**

| ID | Historia de Usuario | Story Points |
|---|---|---:|
| HU-03 | Obtener recomendación principal de proveedor de envío | 8 |
| HU-04 | Obtener opciones alternativas de proveedores | 3 |
| HU-05 | Seleccionar y confirmar proveedor | 5 |
| **Subtotal MS2** | | **16** |

**Microsprint 3 (2 días) — MVP v2**

| ID | Historia de Usuario | Story Points |
|---|---|---:|
| HU-06 | Visualizar ruta del envío en el mapa | 5 |
| **Subtotal MS3** | | **5** |

**Microsprint 4 (2 días) — MVP v2**

| ID | Historia de Usuario | Story Points |
|---|---|---:|
| HU-07 | Registrar usuario | 5 |
| HU-08 | Iniciar sesión | 3 |
| HU-09 | Consultar pedidos del usuario | 5 |
| **Subtotal MS4** | | **13** |

### 3.2 Fuera de alcance del MVP base + MVP v2

| Ítem | Descripción |
|---|---|
| Gestión avanzada de perfiles | Edición de perfil, roles, recuperación de contraseña o administración de cuentas |
| Integración en tiempo real con proveedores logísticos | Consumo directo de APIs reales de FedEx, DHL o proveedor local |
| Módulo de pagos | Cobro, facturación o pasarelas de pago |
| Seguimiento del envío | Tracking posterior a la confirmación |
| Pedidos internacionales | Origen o destino fuera de Colombia |

---

## 4. Estrategia de Pruebas

La trazabilidad de este plan se mantiene desde el PRD hacia reglas de negocio, historias de usuario, criterios de aceptación y estrategia de pruebas. Cada escenario manual, Serenity Screenplay, Karate o k6 debe indicar la HU y regla funcional que valida, especialmente en reglas críticas de autenticación obligatoria, cobertura Colombia, recomendación, persistencia, historial y dependencia con OpenRouteService.

| Reglas del PRD | Historias relacionadas | Foco de prueba |
|---|---|---|
| Reglas 1-4 | HU-01, HU-02 | Datos obligatorios, cobertura Colombia, límites de peso y prioridad obligatoria |
| Reglas 5-8 | HU-03 | Recomendación por menor costo, menor tiempo y reglas de desempate |
| Regla 9 | HU-04 | Alternativas disponibles, ausencia de alternativas y no duplicación de recomendación |
| Reglas 10-11 y 24 | HU-05, HU-09 | Selección de proveedor, confirmación, persistencia y asociación al usuario autenticado |
| Reglas 12-16 | HU-06 | Ruta en mapa, OpenRouteService, conversión de coordenadas, marcadores y Leaflet |
| Reglas 17-23 y 28 | HU-07, HU-08 | Registro, login obligatorio, rechazo sin autenticación y logout |
| Reglas 25-27 | HU-09 | Historial propio, aislamiento entre usuarios y estado sin pedidos |

### 4.1 Niveles de prueba

Las pruebas requeridas para este ciclo son: **manuales**, **Serenity Screenplay**, **Karate** y **k6**.

**Pruebas manuales de aceptación y exploratorias**
Se ejecutarán pruebas manuales sobre los flujos críticos antes de automatizar o cerrar cada HU. Estas pruebas validarán la experiencia completa desde la interfaz, mensajes de error, comportamiento visual, navegación, rutas protegidas, visualización del mapa, consistencia de datos y casos donde la inspección humana agrega valor, como usabilidad, claridad de mensajes y validación visual de Leaflet.

**Pruebas funcionales E2E (Serenity Screenplay + Cucumber)**
Se utilizará Serenity Screenplay con Cucumber para automatizar escenarios end-to-end escritos en Gherkin, cubriendo los criterios de aceptación de cada HU desde la perspectiva del usuario. Los escenarios se modelarán con actores, tareas, preguntas e interacciones para representar acciones como registrarse, iniciar sesión, registrar pedido, seleccionar prioridad, consultar recomendación, confirmar proveedor, visualizar ruta y consultar historial.

**Pruebas de API / Integración (Karate)**
Se utilizará Karate para validar directamente los contratos de los endpoints REST expuestos por el backend, verificando códigos HTTP, estructura de respuesta, manejo de errores, seguridad y consistencia de datos persistidos en PostgreSQL.

Endpoints e integraciones cubiertos en este ciclo:
- `POST /api/users/register` — registra usuarios con nombre, correo electrónico y contraseña
- `POST /api/users/login` — autentica usuarios registrados y retorna el JWT según el contrato de MVP v2
- `POST /api/v1/pedido` — recibe origen, destino, peso y prioridad; retorna recomendación principal y alternativas disponibles
- `POST /api/v1/pedido/confirmar` — confirma el proveedor seleccionado y persiste el pedido asociado al usuario autenticado
- `GET /api/v1/pedido/mis-pedidos` — retorna únicamente los pedidos del usuario autenticado
- Integración externa OpenRouteService — autocompletado de ubicaciones y cálculo de ruta para visualización en mapa

OpenRouteService no se considera el sistema bajo prueba principal. Se valida la integración del Optimizador de Envíos con ese servicio externo: construcción de solicitudes, restricción de ubicaciones a Colombia, parseo de respuestas, conversión de coordenadas, uso de mocks/stubs, manejo de errores y comportamiento de la aplicación frente a indisponibilidad, respuestas vacías o fallas del proveedor externo.

**Validaciones transversales de seguridad y autorización**
Se validará dentro de las pruebas manuales, Serenity Screenplay y Karate que las funcionalidades operativas requieran autenticación, que los endpoints protegidos rechacen tokens ausentes o inválidos, que un usuario autenticado no pueda consultar pedidos de otra cuenta y que el logout bloquee nuevamente el acceso hasta un nuevo inicio de sesión.

**Pruebas de rendimiento (k6)**
Se utilizará k6 para validar que el motor de recomendación responde dentro de un umbral de tiempo aceptable bajo carga concurrente. Las pruebas se ejecutarán principalmente sobre `POST /api/v1/pedido`, y de forma secundaria sobre `POST /api/users/login`, `POST /api/v1/pedido/confirmar` y `GET /api/v1/pedido/mis-pedidos`, usando usuarios autenticados y JWT válidos para los flujos operativos protegidos. Para evitar consumo innecesario de cuotas externas, las pruebas de carga deben usar mocks o stubs de OpenRouteService cuando sea posible.

### 4.2 Tipos de casos de prueba

- **Casos positivos:** registro exitoso, login exitoso, registro de pedido válido con usuario autenticado, selección de prioridad, recomendación acertada, visualización de alternativas, visualización de ruta, confirmación con persistencia, consulta de historial y logout exitoso.
- **Casos negativos:** campos obligatorios vacíos, correo duplicado, contraseña inválida, credenciales inválidas, intento de uso de la plataforma sin sesión, prioridad no seleccionada, proveedor no seleccionado, token ausente o inválido, consulta protegida sin autenticación, origen fuera de Colombia, destino fuera de Colombia y origen/destino fuera de Colombia simultáneamente.
- **Casos de borde:** valores de peso 0 Kg, 0,0009 Kg, 0,001 Kg, 70 Kg y 70,001 Kg; coordenadas ausentes, respuesta vacía de ruta, usuario autenticado sin pedidos y un solo proveedor disponible.
- **Casos de desempate:** validación de las reglas 7 y 8: empate en costo desempatado por tiempo y empate en tiempo desempatado por costo.
- **Casos geoespaciales:** validación de autocompletado con OpenRouteService filtrado por Colombia, coordenadas válidas, conversión `[lng, lat]` a `[lat, lng]`, marcadores de origen y destino, ajuste automático del mapa y manejo de error cuando no hay datos suficientes.
- **Casos de seguridad:** aislamiento de pedidos por usuario, rechazo de acceso no autenticado, cierre de sesión y validación de que la confirmación del pedido queda asociada al usuario autenticado.

**Matriz mínima de borde y cobertura geográfica**

| Dimensión | Datos mínimos | Resultado esperado |
|---|---|---|
| Peso inválido inferior | 0 Kg y 0,0009 Kg | El sistema rechaza el cálculo por peso fuera del mínimo permitido. |
| Peso válido límite | 0,001 Kg y 70 Kg | El sistema permite continuar si el resto de datos es válido. |
| Peso inválido superior | 70,001 Kg | El sistema rechaza el cálculo por superar el máximo permitido. |
| Cobertura válida | Origen Colombia + destino Colombia | El sistema permite continuar si el resto de datos es válido. |
| Cobertura inválida por origen | Origen fuera de Colombia + destino Colombia | El sistema rechaza el flujo por fuera de cobertura. |
| Cobertura inválida por destino | Origen Colombia + destino fuera de Colombia | El sistema rechaza el flujo por fuera de cobertura. |
| Cobertura inválida total | Origen fuera de Colombia + destino fuera de Colombia | El sistema rechaza el flujo por fuera de cobertura. |

### 4.3 Pruebas manuales

Las pruebas manuales se utilizarán para validar el flujo end-to-end y complementar la automatización. Cada ejecución manual debe registrar evidencia, resultado obtenido, ambiente, datos utilizados y defectos asociados si aplica.

| Grupo | HU | Objetivo manual | Evidencia esperada |
|---|---|---|---|
| Flujo base de cotización | HU-01, HU-02, HU-03, HU-04, HU-05 | Validar que el usuario autenticado complete origen, destino, peso y prioridad, reciba recomendación, compare alternativas y confirme proveedor. | Capturas del formulario, resultados, proveedor seleccionado y confirmación final. |
| Validaciones de entrada | HU-01, HU-02, HU-07, HU-08 | Verificar mensajes visibles para campos obligatorios, peso fuera de rango, origen/destino fuera de Colombia, correo duplicado, contraseña inválida y credenciales incorrectas. | Capturas de mensajes de error y datos usados. |
| Ruta en mapa | HU-06 | Confirmar visualmente que la ruta aparece, los marcadores de origen/destino son correctos y el mapa se ajusta automáticamente. | Captura del mapa con ruta y marcadores, más caso sin datos suficientes. |
| Autenticación y rutas protegidas | HU-07, HU-08, HU-09 | Validar registro, login obligatorio, persistencia de sesión, redirección desde rutas protegidas y logout implementado. | Capturas de login/registro, acceso permitido, acceso bloqueado sin autenticación y bloqueo posterior al logout. |
| Historial de pedidos | HU-09 | Confirmar que cada usuario ve solo sus pedidos y que el estado sin pedidos se informa correctamente. | Capturas de historial para usuario con pedidos y usuario sin pedidos. |
| Exploratoria de UX | HU-01 a HU-09 | Detectar problemas de navegación, mensajes ambiguos, inconsistencias visuales o bloqueos no cubiertos por automatización. | Notas exploratorias con pasos, hallazgos y severidad sugerida. |

**Criterios mínimos de ejecución manual**
- Ejecutar al menos un flujo feliz end-to-end por microsprint.
- Ejecutar manualmente los casos críticos antes de marcar una HU como cerrada.
- Validar que sin sesión activa no se puede iniciar ni continuar el flujo operativo de la plataforma.
- Validar que después del logout las funcionalidades protegidas quedan bloqueadas hasta un nuevo login.
- Validar manualmente HU-06 en navegador real por depender de visualización de mapa.
- Validar HU-09 con dos usuarios distintos para comprobar aislamiento de datos.
- Adjuntar evidencia de defectos con pasos reproducibles, datos usados y resultado esperado.

### 4.4 Pruebas de rendimiento con k6

Las pruebas k6 se enfocarán en validar estabilidad, tiempos de respuesta y tasa de error de los endpoints críticos. No reemplazan las pruebas funcionales; se ejecutan después de que los flujos principales estén funcionalmente estables.

| Escenario k6 | Endpoint principal | Objetivo | Carga inicial propuesta | Umbral esperado |
|---|---|---|---|---|
| Smoke API autenticado | `POST /api/v1/pedido` | Verificar que el script, datos, JWT y ambiente funcionan antes de carga. | 1 VU durante 1 minuto | `http_req_failed < 1%` y `checks > 95%` |
| Carga de recomendación | `POST /api/v1/pedido` | Medir respuesta del motor de recomendación con usuario autenticado y proveedores mock. | Rampa 5 → 25 VU durante 5 minutos | `p95 < 1200 ms` y `http_req_failed < 1%` |
| Confirmación autenticada | `POST /api/v1/pedido/confirmar` | Medir confirmación y persistencia con token JWT válido. | 10 VU durante 3 minutos | `p95 < 1500 ms` y `http_req_failed < 1%` |
| Historial autenticado | `GET /api/v1/pedido/mis-pedidos` | Medir consulta de pedidos por usuario autenticado. | 10 → 20 VU durante 5 minutos | `p95 < 800 ms` y `http_req_failed < 1%` |
| Login controlado | `POST /api/users/login` | Validar autenticación bajo carga moderada sin crear usuarios masivos. | 5 → 10 VU durante 3 minutos | `p95 < 1000 ms` y `http_req_failed < 1%` |

**Datos y preparación para k6**
- Usar usuarios de prueba precreados para evitar que el registro masivo distorsione las métricas.
- Pre-generar o reutilizar JWT válidos para escenarios protegidos cuando el objetivo no sea medir login.
- Usar orígenes, destinos y coordenadas dentro de Colombia.
- Usar pesos representativos, incluyendo valores nominales y algunos límites válidos.
- Reservar los datos fuera de Colombia y pesos inválidos para pruebas funcionales negativas, no para carga sostenida.
- Usar mocks/stubs de OpenRouteService en pruebas de carga para evitar consumo de cuota externa y variabilidad de red.
- Limpiar o aislar datos generados por confirmaciones para no contaminar ejecuciones posteriores.

**Métricas reportadas**
- Tiempo de respuesta `p95` y `p99`.
- Tasa de error HTTP.
- Tasa de checks exitosos.
- Throughput por endpoint.
- Uso observado de recursos si el ambiente de QA lo expone.
- Comparación contra umbrales y conclusión: aprobado, aprobado con observaciones o rechazado.

### 4.5 Pruebas funcionales con Serenity Screenplay

Las pruebas Serenity Screenplay cubrirán flujos de usuario completos, reutilizando tareas e interacciones para mantener trazabilidad con los criterios de aceptación y reducir duplicación en la automatización.

| Flujo Screenplay | HU | Actor / intención | Validaciones principales |
|---|---|---|---|
| Registro y autenticación | HU-07, HU-08 | Usuario que crea cuenta, inicia sesión y cierra sesión | Registro exitoso, correo duplicado, contraseña inválida, credenciales inválidas, acceso a ruta protegida y logout. |
| Cotización de envío | HU-01, HU-02 | Usuario autenticado que registra un pedido | Origen, destino, cobertura Colombia, peso, prioridad, validaciones de campos y límites de peso. |
| Recomendación y alternativas | HU-03, HU-04 | Usuario que compara opciones de envío | Recomendación por menor costo/tiempo, reglas de desempate y alternativas distintas a la recomendación. |
| Ruta en mapa | HU-06 | Usuario que visualiza el recorrido estimado | Presencia de ruta, marcadores de origen/destino, ajuste del mapa y mensaje cuando no hay datos suficientes. |
| Confirmación e historial | HU-05, HU-09 | Usuario que confirma proveedor y consulta pedidos | Persistencia del pedido, proveedor seleccionado, historial propio y aislamiento frente a pedidos de otros usuarios. |

**Convenciones de implementación**
- Usar `Actor` para representar el usuario autenticado o no autenticado.
- Usar `Task` para acciones de negocio: registrarse, iniciar sesión, registrar pedido, seleccionar prioridad, confirmar proveedor.
- Usar `Question` para validar estado visible, mensajes, recomendación, alternativas, mapa e historial.
- Usar `Interaction` para acciones de UI: completar campos, seleccionar opciones, navegar y hacer clic.
- Modelar como usuario autenticado todos los escenarios operativos salvo registro, login y validaciones explícitas de acceso no autenticado.
- Mantener trazabilidad entre cada escenario Gherkin, HU y criterio de aceptación.

---

## 5. Criterios de Entrada y Salida

### 5.1 Criterios de entrada

Para iniciar la ejecución de pruebas de una Historia de Usuario se requiere:

- El build del entorno de QA está estable y desplegado.
- `user-service`, `shipment-service` y el frontend están disponibles en ambiente QA.
- Los endpoints de la HU están implementados y documentados.
- La base de datos PostgreSQL está accesible y con los schemas/migraciones correspondientes.
- Los datos mock de proveedores (FedEx, DHL, Local) están configurados.
- El secreto JWT compartido entre servicios está configurado para pruebas de autenticación.
- OpenRouteService está operativo o existe un mock/stub controlado para pruebas que no deban consumir la API externa.
- Leaflet puede renderizarse o simularse en el entorno de pruebas de frontend.
- El framework de Serenity Screenplay está configurado y puede ejecutar escenarios Cucumber en el ambiente de QA.
- k6 está instalado o disponible en el pipeline/ambiente donde se ejecutarán pruebas de rendimiento.
- Existen usuarios, JWT y datos semilla para escenarios autenticados de k6.
- Los datos de prueba están definidos: usuarios, credenciales, orígenes, destinos, coordenadas, pesos, prioridades y proveedores.
- Los casos de prueba han sido revisados y aprobados.
- DEV entregó la HU con criterios de aceptación implementados y sin defectos bloqueantes conocidos.

### 5.2 Criterios de salida

Una Historia de Usuario se considera probada y cerrada cuando:

- El 100% de los casos de prueba diseñados para la HU fueron ejecutados.
- El 100% de los casos críticos, positivos, negativos, de borde, geoespaciales y de seguridad aplicables pasan exitosamente.
- No existen defectos abiertos de severidad **Alta** o **Crítica**.
- Los defectos de severidad **Media** tienen plan de resolución acordado.
- Las validaciones de seguridad aplicables pasan sin defectos abiertos.
- Los casos manuales críticos tienen evidencia registrada.
- Los escenarios Serenity Screenplay requeridos para la HU pasan exitosamente.
- Los escenarios k6 definidos para el ciclo cumplen los umbrales acordados.
- Los resultados fueron reportados y tienen el visto bueno del responsable de QA.

---

## 6. Entorno de Pruebas

| Ítem | Detalle |
|---|---|
| **Ambiente** | Entorno de QA (staging) — aislado de producción |
| **Base de datos** | PostgreSQL — instancia exclusiva para pruebas, con datos controlados de usuarios y pedidos |
| **Backend de envíos** | `shipment-service` API REST desplegada en ambiente QA |
| **Backend de usuarios** | `user-service` API REST desplegada en ambiente QA |
| **Frontend** | Aplicación web con Zustand, React Router y rutas protegidas, desplegada en ambiente QA |
| **Autenticación** | JWT con secreto compartido entre `user-service` y `shipment-service` |
| **Proveedores** | Datos mock de FedEx, DHL y proveedor local — valores predefinidos por DEV |
| **Geolocalización y rutas** | Integración externa OpenRouteService — autocompletado de ubicaciones y cálculo de ruta filtrado por Colombia; validable con servicio real controlado o mock/stub |
| **Mapa** | Leaflet — librería frontend para visualización de ruta, marcadores y ajuste automático del viewport |
| **Datos de prueba** | Matrices de datos definidas por QA por cada HU |
| **Navegadores (frontend)** | Chrome |

---

## 7. Herramientas

| Herramienta | Propósito |
|---|---|
| **Navegador + DevTools** | Ejecución de pruebas manuales, validación visual, revisión de red y captura de evidencia |
| **Serenity Screenplay + Cucumber** | Automatización funcional end-to-end basada en actores, tareas, preguntas e interacciones |
| **Karate** | Pruebas de API: contratos de endpoints, códigos HTTP, estructura de respuesta, seguridad y datos persistidos |
| **k6** | Pruebas de rendimiento, smoke y carga sobre recomendación, confirmación, login e historial autenticado |
| **GitHub Projects** | Gestión de casos de prueba, reporte de defectos y trazabilidad HU → casos de prueba |
| **Git** | Control de versiones de los scripts de prueba |

---

## 8. Roles y Responsabilidades

### QA: Nahuel Lemes

- Diseño de la matriz de datos de prueba.
- Diseño y documentación de casos de prueba por HU.
- Escritura de escenarios Gherkin (`.feature`).
- Ejecución de pruebas manuales de aceptación y exploratorias.
- Implementación de automatización con Serenity Screenplay + Cucumber.
- Implementación de colecciones de prueba en Karate.
- Implementación de scripts de rendimiento en k6.
- Análisis de resultados k6 y comparación contra umbrales.
- Validación de seguridad funcional: autenticación, autorización y aislamiento de datos.
- Ejecución de pruebas y registro de resultados.
- Reporte y seguimiento de defectos.
- Generación de métricas e informe final.

### DEV: Santiago Angarita

- Implementación de endpoints y lógica de negocio.
- Configuración de datos mock de proveedores (FedEx, DHL, Local).
- Configuración de base de datos PostgreSQL y migraciones.
- Implementación y configuración de autenticación JWT.
- Integración con OpenRouteService para autocompletado y rutas, más manejo de errores frente a fallas del servicio externo.
- Resolución de defectos reportados por QA.
- Soporte en la configuración del entorno de QA.
- Soporte en preparación de datos semilla para pruebas manuales y k6.
- Revisión de DTOs, contratos de API y validaciones de entrada.

### Responsabilidad compartida

- Refinamiento de criterios de aceptación.
- Definición de datos de prueba complejos: escenarios de empate, valores límite, coordenadas válidas, usuarios con y sin pedidos.
- Acuerdos sobre severidad y prioridad de defectos.
- Validación de trazabilidad PRD → HU → casos de prueba → resultados.

---

## 9. Cronograma y Estimación

La estimación de esfuerzo de QA se calcula tomando como referencia los Story Points de cada microsprint y el riesgo funcional asociado.

### Microsprint 1 (2 días) — HU-01, HU-02

| HU | Story Points | Tareas QA principales | Esfuerzo estimado QA |
|---|---:|---|---|
| HU-01 | 5 | Matriz de datos, usuario autenticado, validaciones de campos, cobertura Colombia, casos origen/destino fuera de Colombia, límites de peso y API de pedido | Alto |
| HU-02 | 3 | Casos de selección de prioridad, validación de prioridad ausente y persistencia en estado del flujo | Medio |
| **Subtotal MS1** | **8** | | **Medio-Alto** |

### Microsprint 2 (2 días) — HU-03, HU-04, HU-05

| HU | Story Points | Tareas QA principales | Esfuerzo estimado QA |
|---|---:|---|---|
| HU-03 | 8 | Recomendación por prioridad, reglas de desempate, contratos de respuesta y escenarios con proveedores mock | Alto |
| HU-04 | 3 | Visualización de alternativas, ausencia de alternativas y consistencia con recomendación principal | Medio |
| HU-05 | 5 | Selección de proveedor, confirmación, persistencia en PostgreSQL y validación obligatoria de token | Alto |
| **Subtotal MS2** | **16** | | **Alto** |

### Microsprint 3 (2 días) — HU-06

| HU | Story Points | Tareas QA principales | Esfuerzo estimado QA |
|---|---:|---|---|
| HU-06 | 5 | Ruta en mapa, marcadores, ajuste automático, conversión de coordenadas y manejo de datos insuficientes | Alto |
| **Subtotal MS3** | **5** | | **Medio-Alto** |

### Microsprint 4 (2 días) — HU-07, HU-08, HU-09

| HU | Story Points | Tareas QA principales | Esfuerzo estimado QA |
|---|---:|---|---|
| HU-07 | 5 | Registro exitoso, correo duplicado, campos obligatorios y contraseña mínima | Alto |
| HU-08 | 3 | Login exitoso, credenciales inválidas, protección de funcionalidades operativas y logout | Medio |
| HU-09 | 5 | Historial por usuario, usuario sin pedidos y bloqueo de consulta de pedidos ajenos | Alto |
| **Subtotal MS4** | **13** | | **Alto** |

### Total MVP base + MVP v2

| | Story Points | Esfuerzo estimado QA |
|---|---:|---|
| **Total** | **42** | **Alto** |

---

## 10. Entregables de Prueba

| Artefacto | Descripción |
|---|---|
| `TEST_PLAN.md` | Este documento — plan de pruebas del MVP base y MVP v2 |
| Matrices de datos de prueba | Tablas con combinaciones de datos válidos e inválidos por HU |
| Casos de prueba documentados | Descripción, precondiciones, pasos, resultado esperado y resultado obtenido |
| Evidencias de pruebas manuales | Capturas, notas exploratorias, datos utilizados, resultado obtenido y defectos asociados |
| Escenarios Gherkin (`.feature`) | Escenarios BDD alineados a los criterios de aceptación de cada HU |
| Scripts de automatización Serenity Screenplay | Implementación de actores, tareas, preguntas e interacciones para los escenarios funcionales E2E |
| Colecciones Karate | Scripts de prueba de API para cada endpoint cubierto |
| Scripts k6 | Scripts de smoke, carga y rendimiento para recomendación, confirmación, login e historial autenticado |
| Reporte de ejecución Serenity Screenplay | Reporte HTML generado por Serenity con resultados de pruebas funcionales E2E |
| Reporte de ejecución Karate | Reporte de resultados de pruebas de API e integración |
| Reporte k6 | Métricas de rendimiento: p95, p99, throughput, tasa de error, checks y conclusión contra umbrales |
| Informe de defectos | Listado de defectos encontrados con severidad, prioridad y estado |
| Métricas de cobertura | % de casos ejecutados, % pasados, % fallidos, defectos por HU y cobertura PRD → HU |

---

## 11. Riesgos y Contingencias

### Riesgos de producto

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Fallos de lógica en el motor de recomendación que asigne la opción más cara en lugar de la más económica | Alto — el usuario recibe una recomendación incorrecta que genera mayores costos de envío | QA diseña casos exhaustivos para todas las combinaciones de prioridad y escenarios de desempate (Reglas 5–8) |
| Aplicación incorrecta de las reglas de desempate | Alto — ante empates, el sistema podría recomendar una opción subóptima | QA diseña casos específicos para empate en costo, empate en tiempo y empate total entre proveedores |
| Alternativas no visibles o inconsistentes con la recomendación | Medio — el usuario pierde capacidad de comparación entre proveedores | QA valida que las alternativas sean distintas a la recomendación principal y mantengan costo/tiempo consistentes |
| Inconsistencia entre datos del estado global y los persistidos en PostgreSQL | Alto — el usuario confirma datos que no coinciden con lo almacenado | QA valida en HU-05 y HU-09 que origen, destino, peso, prioridad y proveedor seleccionado coinciden entre frontend, API y base de datos |
| Error en la conversión de coordenadas de OpenRouteService a Leaflet | Alto — la ruta podría visualizarse en una ubicación incorrecta | QA cubre conversión `[lng, lat]` a `[lat, lng]`, marcadores y ajuste automático del mapa en HU-06 |
| Servicio de OpenRouteService no disponible o con límite de cuota | Medio — el autocompletado o la ruta pueden fallar aunque la lógica de recomendación funcione | QA prepara mocks/stubs de ubicaciones y rutas; valida manejo de error y que el sistema informe ausencia de datos suficientes sin bloquear indebidamente reglas no dependientes del mapa |
| Exposición de pedidos entre usuarios | Crítico — un usuario podría consultar información de otra cuenta | QA valida tokens, aislamiento por usuario autenticado y escenarios de intento de consulta de pedidos ajenos en HU-09 |
| Registro, login o logout sin validaciones suficientes | Alto — usuarios duplicados, contraseñas inválidas, accesos indebidos o sesión activa indebidamente | QA cubre correo único, contraseña mínima, credenciales inválidas, acceso a rutas protegidas sin autenticación y bloqueo posterior al logout |

### Riesgos de proyecto

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Entorno de QA inestable o no disponible | Alto — bloquea la ejecución de pruebas | Acordar criterios de entrada claros; QA no inicia ejecución sin build estable |
| Contratos de autenticación no alineados entre frontend, `user-service` y `shipment-service` | Alto — bloquea HU-08, HU-09 y endpoints protegidos | Revisar contratos antes de la ejecución, validar JWT compartido y documentar discrepancias como defectos bloqueantes |
| Servicio de OpenRouteService no disponible durante pruebas | Medio — bloquea validación de autocompletado y ruta real | Preparar datos mock/stub de ubicaciones y rutas como fallback para pruebas funcionales y de rendimiento |
| HU entregada por DEV sin criterios de aceptación completos | Medio — casos de prueba mal definidos | QA participa del refinamiento previo al desarrollo y valida trazabilidad con el PRD |
| Tiempo insuficiente para automatización completa en el ciclo | Medio — riesgo de automatización parcial sin afectar la cobertura total exigida | Priorizar automatización de flujos críticos y completar la cobertura requerida con pruebas manuales documentadas, sin reducir criterios de salida |
| Cambio de scope durante el microsprint | Alto — casos de prueba desactualizados | Cualquier cambio en HU debe pasar por refinamiento formal antes de ser testeado |
