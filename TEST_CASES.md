# Casos de Prueba — Optimizador de Envíos

## MVP base + MVP v2: HU-01 a HU-09

---

## Convenciones de ejecución

Cada caso de prueba está asignado a **una sola herramienta**, seleccionada según el foco principal del test:

| Herramienta | Cuándo se usa |
|---|---|
| **Serenity Screenplay** | Flujos E2E desde la perspectiva del usuario: happy paths, interacciones de UI, navegación y estados visuales. |
| **Karate DSL** | Validaciones de API e integración: contratos REST, códigos HTTP, reglas de negocio, seguridad, persistencia y stubs/mocks de OpenRouteService. |
| **Manual exploratoria** | Exploración libre de UX, mensajes, estados visuales, DevTools y comportamientos no cubiertos por scripts. |
| **k6** | Pruebas de smoke, carga y rendimiento sobre endpoints críticos autenticados. |

### Precondiciones transversales

- Ambiente QA estable con `frontend`, `shipment-service`, `user-service` y PostgreSQL disponibles.
- Proveedores mock configurados: FedEx, DHL y proveedor local.
- OpenRouteService disponible o mock/stub controlado para autocompletado y rutas.
- JWT configurado entre `user-service` y `shipment-service`.
- Para funcionalidades operativas, existe un usuario autenticado salvo en los casos que validan acceso sin sesión.
- Todos los casos quedan inicialmente en estado **Sin ejecutar** y el campo **Resultado de ejecución** se completa durante la ejecución formal de QA.

---

# Microsprint 1

---

## HU-01 — Registrar pedido de envío

### TC-HU01-01 · Registro exitoso con usuario autenticado

```gherkin
Dado que el usuario autenticado necesita enviar un producto
Cuando ingresa origen "Bogotá, Colombia", destino "Medellín, Colombia" y peso 5 Kg
Entonces el formulario avanza al cálculo conservando los datos
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario autenticado. Formulario disponible. |
| **Datos de prueba** | origen = "Bogotá, Colombia" · destino = "Medellín, Colombia" · peso = 5 Kg |
| **Pasos** | Login → ingresar datos → continuar → verificar que persisten. |
| **Resultado esperado** | Avanza al cálculo con origen, destino y peso conservados. |

---

### TC-HU01-02 · Autocompletado restringido a Colombia

```gherkin
Dado que el usuario autenticado registra un pedido
Cuando escribe texto parcial en origen o destino
Entonces el sistema muestra sugerencias de OpenRouteService restringidas a Colombia
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario autenticado. Endpoint o stub de OpenRouteService disponible. |
| **Datos de prueba** | búsqueda = "Bog" · país esperado = "Colombia" |
| **Pasos** | Escribir texto parcial → verificar sugerencias → seleccionar una válida. |
| **Resultado esperado** | Solo aparecen ubicaciones de Colombia. La selección llena el campo. |

---

### TC-HU01-03 · Registro fallido por campos vacíos

```gherkin
Dado que el usuario autenticado intenta registrar un pedido
Cuando envía origen, destino y peso vacíos
Entonces el sistema retorna HTTP 400 indicando campos obligatorios
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT válido. Endpoint de pedido disponible. |
| **Datos de prueba** | origen = "" · destino = "" · peso = vacío |
| **Pasos** | POST /api/v1/pedido sin campos → verificar HTTP 400 y mensaje. |
| **Resultado esperado** | HTTP 400. Mensaje indica campos obligatorios. No se crea pedido. |

---

### TC-HU01-04 · Registro fallido por origen fuera de Colombia

```gherkin
Dado que el usuario autenticado necesita enviar un producto
Cuando ingresa origen fuera de Colombia y destino dentro de Colombia
Entonces el sistema bloquea el cálculo e informa fuera de cobertura
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT válido. Validación de cobertura activa. |
| **Datos de prueba** | origen = "Buenos Aires, Argentina" · destino = "Medellín, Colombia" · peso = 5 Kg |
| **Pasos** | POST /api/v1/pedido con origen fuera → verificar HTTP 400. |
| **Resultado esperado** | HTTP 400. Fuera de cobertura. No se genera recomendación. |

---

### TC-HU01-05 · Registro fallido por destino fuera de Colombia

```gherkin
Dado que el usuario autenticado necesita enviar un producto
Cuando ingresa origen dentro de Colombia y destino fuera de Colombia
Entonces el sistema bloquea el cálculo e informa fuera de cobertura
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT válido. Validación de cobertura activa. |
| **Datos de prueba** | origen = "Bogotá, Colombia" · destino = "Buenos Aires, Argentina" · peso = 5 Kg |
| **Pasos** | POST /api/v1/pedido con destino fuera → verificar HTTP 400. |
| **Resultado esperado** | HTTP 400. Fuera de cobertura. No se genera recomendación. |

---

### TC-HU01-06 · Registro exitoso con peso mínimo (0.001 Kg)

```gherkin
Dado que el usuario autenticado ingresa datos válidos
Cuando el peso es 0.001 Kg
Entonces el sistema acepta el pedido
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Medio |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT válido. Origen y destino dentro de Colombia. |
| **Datos de prueba** | origen = "Bogotá" · destino = "Medellín" · peso = 0.001 Kg |
| **Pasos** | POST /api/v1/pedido con peso mínimo → verificar HTTP 201. |
| **Resultado esperado** | HTTP 201. Peso de 0.001 Kg aceptado. |

---

### TC-HU01-07 · Registro exitoso con peso máximo (70 Kg)

```gherkin
Dado que el usuario autenticado ingresa datos válidos
Cuando el peso es 70 Kg
Entonces el sistema acepta el pedido
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Medio |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT válido. Origen y destino dentro de Colombia. |
| **Datos de prueba** | origen = "Bogotá" · destino = "Medellín" · peso = 70 Kg |
| **Pasos** | POST /api/v1/pedido con peso máximo → verificar HTTP 201. |
| **Resultado esperado** | HTTP 201. Peso de 70 Kg aceptado. |

---

### TC-HU01-08 · Registro fallido por peso inferior al mínimo

```gherkin
Dado que el usuario autenticado ingresa origen y destino válidos
Cuando el peso es 0.0009 Kg
Entonces el sistema bloquea el pedido e informa peso fuera de rango
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT válido. Origen y destino dentro de Colombia. |
| **Datos de prueba** | peso = 0.0009 Kg |
| **Pasos** | POST /api/v1/pedido con peso < 0.001 → verificar HTTP 400. |
| **Resultado esperado** | HTTP 400. Peso fuera del rango permitido. |

---

### TC-HU01-09 · Registro fallido por peso superior al máximo

```gherkin
Dado que el usuario autenticado ingresa origen y destino válidos
Cuando el peso es 70.001 Kg
Entonces el sistema bloquea el pedido e informa peso fuera de rango
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT válido. Origen y destino dentro de Colombia. |
| **Datos de prueba** | peso = 70.001 Kg |
| **Pasos** | POST /api/v1/pedido con peso > 70 → verificar HTTP 400. |
| **Resultado esperado** | HTTP 400. Peso fuera del rango permitido. |

---

### TC-HU01-10 · Exploratoria del formulario, autocompletado y prioridad

```gherkin
Dado que el usuario autenticado usa el formulario de pedido y el selector de prioridad
Cuando interactúa libremente con campos, sugerencias, prioridades y navegación
Entonces no deben aparecer bloqueos, mensajes ambiguos ni inconsistencias visuales
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Medio |
| **Herramienta** | Manual exploratoria |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario autenticado. DevTools. Datos válidos e inválidos. |
| **Datos de prueba** | Búsquedas parciales · acentos · mayúsculas · separadores decimales · prioridades. |
| **Pasos** | Explorar escritura rápida, borrado, alternar prioridades → verificar consistencia → registrar hallazgos. |
| **Resultado esperado** | Formulario consistente. Mensajes claros. Prioridad sin estados ambiguos. |

---

### TC-HU01-11 · Smoke de API autenticada para pedido

```gherkin
Dado que existe un usuario autenticado y datos válidos
Cuando se ejecuta un smoke de baja carga sobre el endpoint de pedido
Entonces el servicio responde de forma estable
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | k6 |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario de prueba. JWT válido. ORS mockeado. |
| **Datos de prueba** | 1 VU · 1 min · POST /api/v1/pedido · peso = 5 Kg |
| **Pasos** | Ejecutar script k6 smoke → validar checks → revisar tasa de error. |
| **Resultado esperado** | http_req_failed < 1% · checks > 95% · sin errores de auth o contrato. |

---

## HU-02 — Definir prioridad del envío

### TC-HU02-01 · Selección de prioridad por menor costo

```gherkin
Dado que el usuario autenticado registró un pedido válido
Cuando selecciona prioridad de menor costo
Entonces el sistema usa el costo como criterio principal de recomendación
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario autenticado. Pedido válido en el flujo. |
| **Datos de prueba** | prioridad = "MENOR_COSTO" |
| **Pasos** | Preparar pedido → seleccionar menor costo → verificar que queda registrada. |
| **Resultado esperado** | Prioridad registrada como MENOR_COSTO. Sistema usa costo para recomendación. |

---

### TC-HU02-02 · Selección de prioridad por menor tiempo

```gherkin
Dado que el usuario autenticado registró un pedido válido
Cuando selecciona prioridad de menor tiempo de entrega
Entonces el sistema usa el tiempo como criterio principal de recomendación
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario autenticado. Pedido válido en el flujo. |
| **Datos de prueba** | prioridad = "MENOR_TIEMPO" |
| **Pasos** | Preparar pedido → seleccionar menor tiempo → verificar que queda registrada. |
| **Resultado esperado** | Prioridad registrada como MENOR_TIEMPO. Sistema usa tiempo para recomendación. |

---

### TC-HU02-03 · Registro fallido por no seleccionar prioridad

```gherkin
Dado que el usuario autenticado registró un pedido válido
Cuando no selecciona ninguna prioridad
Entonces el sistema retorna HTTP 400 indicando prioridad obligatoria
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT válido. Datos de pedido válidos. |
| **Datos de prueba** | prioridad = vacío |
| **Pasos** | POST /api/v1/pedido sin prioridad → verificar HTTP 400 y mensaje. |
| **Resultado esperado** | HTTP 400. Prioridad obligatoria. No se genera recomendación. |

---

# Microsprint 2

---

## HU-03 — Obtener recomendación principal de proveedor

### TC-HU03-01 · Recomendación principal por menor costo

```gherkin
Dado que el usuario autenticado definió prioridad de menor costo
Cuando el sistema calcula las opciones disponibles
Entonces recomienda la opción de menor costo (Local = 28 000 COP)
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT válido. Proveedores mock con costos distintos. |
| **Datos de prueba** | FedEx 35 000/2d · DHL 42 000/1d · Local 28 000/3d · prioridad = MENOR_COSTO |
| **Pasos** | POST /api/v1/pedido → verificar recomendación principal = Local. |
| **Resultado esperado** | Recomendación = Local. Costo = 28 000 COP. |

---

### TC-HU03-02 · Recomendación principal por menor tiempo

```gherkin
Dado que el usuario autenticado definió prioridad de menor tiempo
Cuando el sistema calcula las opciones disponibles
Entonces recomienda la opción de menor tiempo (DHL = 1 día)
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT válido. Proveedores mock con tiempos distintos. |
| **Datos de prueba** | FedEx 35 000/2d · DHL 42 000/1d · Local 28 000/3d · prioridad = MENOR_TIEMPO |
| **Pasos** | POST /api/v1/pedido → verificar recomendación principal = DHL. |
| **Resultado esperado** | Recomendación = DHL. Tiempo = 1 día. |

---

### TC-HU03-03 · Desempate por tiempo ante empate en costo

```gherkin
Dado que la prioridad es menor costo y FedEx y Local empatan en 28 000 COP
Cuando el sistema calcula la recomendación
Entonces recomienda FedEx (menor tiempo entre empatados: 2d vs 3d)
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT válido. Datos mock con empate en costo. |
| **Datos de prueba** | FedEx 28 000/2d · DHL 42 000/1d · Local 28 000/3d · prioridad = MENOR_COSTO |
| **Pasos** | POST /api/v1/pedido → verificar recomendación = FedEx (desempate por tiempo). |
| **Resultado esperado** | Recomendación = FedEx. Desempate correcto por menor tiempo. |

---

### TC-HU03-04 · Desempate por costo ante empate en tiempo

```gherkin
Dado que la prioridad es menor tiempo y FedEx y DHL empatan en 1 día
Cuando el sistema calcula la recomendación
Entonces recomienda FedEx (menor costo entre empatados: 35 000 vs 42 000)
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT válido. Datos mock con empate en tiempo. |
| **Datos de prueba** | FedEx 35 000/1d · DHL 42 000/1d · Local 28 000/3d · prioridad = MENOR_TIEMPO |
| **Pasos** | POST /api/v1/pedido → verificar recomendación = FedEx (desempate por costo). |
| **Resultado esperado** | Recomendación = FedEx. Desempate correcto por menor costo. |

---

### TC-HU03-05 · Contrato de respuesta de recomendación

```gherkin
Dado que el usuario autenticado solicita una recomendación válida
Cuando la API responde
Entonces la estructura incluye proveedor, costo, tiempoEntrega, prioridad y alternativas
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT válido. Proveedores mock activos. |
| **Datos de prueba** | Campos esperados: proveedor, costo, tiempoEntrega, prioridad, alternativas. |
| **Pasos** | POST /api/v1/pedido → validar status → validar schema y tipos de datos. |
| **Resultado esperado** | Contrato cumplido. Tipos consistentes. Sin campos nulos críticos. |

---

### TC-HU03-06 · Carga del motor de recomendación

```gherkin
Dado que el flujo de recomendación es funcionalmente estable
Cuando se ejecuta carga concurrente autenticada (5→25 VU, 5 min)
Entonces el motor responde dentro del umbral acordado
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | k6 |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT de prueba. ORS mockeado. |
| **Datos de prueba** | 5→25 VU · 5 min · POST /api/v1/pedido |
| **Pasos** | Ejecutar k6 de carga → verificar checks y tasa de error. |
| **Resultado esperado** | p95 < 1200 ms · http_req_failed < 1% · checks > 95%. |

---

### TC-HU03-07 · Exploratoria de recomendación, alternativas y confirmación

```gherkin
Dado que el usuario visualiza recomendación, alternativas y confirma proveedor
Cuando compara opciones, selecciona, cambia de opinión y confirma
Entonces la pantalla comunica claramente cada paso sin inconsistencias
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Medio |
| **Herramienta** | Manual exploratoria |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario autenticado. Datos con/sin empate. 2-3 proveedores. |
| **Datos de prueba** | Prioridades ambas · proveedores FedEx/DHL/Local · escenarios de empate. |
| **Pasos** | Flujos de recomendación → comparar API/UI → cambiar proveedor → confirmar → volver atrás → reintentar. |
| **Resultado esperado** | Recomendación coincide con API. Sin duplicados. Sin confirmaciones dobles. |

---

## HU-04 — Obtener opciones alternativas de proveedores

### TC-HU04-01 · Visualización de alternativas disponibles

```gherkin
Dado que el usuario autenticado obtuvo una recomendación principal
Y existen otras opciones disponibles
Cuando el sistema muestra la recomendación
Entonces muestra automáticamente las alternativas con proveedor, costo y tiempo
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario autenticado. Respuesta con múltiples proveedores. |
| **Datos de prueba** | recomendado = Local · alternativas = FedEx, DHL |
| **Pasos** | Flujo de recomendación → verificar alternativas visibles con datos completos. |
| **Resultado esperado** | Alternativas visibles. Cada una incluye proveedor, costo y tiempo. |

---

### TC-HU04-02 · Ausencia de opciones alternativas

```gherkin
Dado que el usuario autenticado obtuvo una recomendación principal
Y no existen otras opciones disponibles
Cuando el sistema muestra la recomendación
Entonces notifica que no existen alternativas
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario autenticado. Mock con un solo proveedor. |
| **Datos de prueba** | recomendado = único proveedor · alternativas = [] |
| **Pasos** | Configurar un solo proveedor → recomendación → verificar mensaje. |
| **Resultado esperado** | UI informa sin alternativas. No se muestra lista vacía confusa. |

---

### TC-HU04-03 · Recomendación principal no se duplica como alternativa

```gherkin
Dado que existen recomendación y alternativas
Cuando se renderiza la lista de opciones
Entonces la recomendación principal no aparece dentro de alternativas
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT válido. Respuesta con recomendación y alternativas. |
| **Datos de prueba** | recomendado = Local · alternativas esperadas ≠ Local |
| **Pasos** | POST /api/v1/pedido → validar que alternativas no contienen al recomendado. |
| **Resultado esperado** | Ninguna alternativa tiene el mismo proveedor que la recomendación. |

---

## HU-05 — Seleccionar y confirmar proveedor

### TC-HU05-01 · Selección del proveedor recomendado

```gherkin
Dado que el usuario autenticado visualiza recomendación y opciones
Cuando selecciona el proveedor recomendado
Entonces el sistema permite continuar a confirmación final
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario autenticado. Recomendación generada. |
| **Datos de prueba** | proveedorSeleccionado = recomendado |
| **Pasos** | Flujo de pedido → seleccionar recomendado → verificar continuación. |
| **Resultado esperado** | Selección registrada. Se permite continuar a confirmación. |

---

### TC-HU05-02 · Selección de proveedor alternativo

```gherkin
Dado que el usuario autenticado visualiza recomendación y alternativas
Cuando selecciona un proveedor alternativo disponible
Entonces el sistema permite continuar a confirmación final
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario autenticado. Alternativas disponibles. |
| **Datos de prueba** | proveedorSeleccionado = alternativo ≠ recomendado |
| **Pasos** | Flujo de recomendación → seleccionar alternativa → confirmar. |
| **Resultado esperado** | Alternativa registrada. No se fuerza al usuario a confirmar solo el recomendado. |

---

### TC-HU05-03 · Confirmación fallida sin seleccionar proveedor

```gherkin
Dado que el usuario autenticado visualiza opciones disponibles
Cuando intenta continuar sin seleccionar proveedor
Entonces el sistema retorna HTTP 400 indicando proveedor obligatorio
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT válido. Recomendación generada. |
| **Datos de prueba** | proveedorSeleccionado = ninguno |
| **Pasos** | POST /api/v1/pedido/confirmar sin proveedor → verificar HTTP 400. |
| **Resultado esperado** | HTTP 400. Debe seleccionar proveedor. No se persiste pedido. |

---

### TC-HU05-04 · Confirmación fallida por proveedor inválido

```gherkin
Dado que el usuario autenticado tiene opciones disponibles
Cuando intenta confirmar un proveedor que no pertenece a las opciones calculadas
Entonces el sistema rechaza la confirmación
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT válido. Proveedor no incluido en la respuesta. |
| **Datos de prueba** | proveedorSeleccionado = "ProveedorInexistente" |
| **Pasos** | Generar recomendación → POST confirmar con proveedor inválido → verificar HTTP 400/422. |
| **Resultado esperado** | HTTP 400/422. Proveedor inválido. No se persiste pedido. |

---

### TC-HU05-05 · Persistencia del pedido asociada al usuario

```gherkin
Dado que el usuario autenticado selecciona y confirma proveedor
Cuando la confirmación es exitosa
Entonces el sistema persiste el pedido asociado al usuario del JWT
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT válido. BD disponible. Pedido y recomendación generados. |
| **Datos de prueba** | origen = "Bogotá" · destino = "Medellín" · peso = 5 Kg · prioridad = MENOR_COSTO |
| **Pasos** | POST confirmar → verificar éxito → GET /mis-pedidos → verificar datos. |
| **Resultado esperado** | Pedido persistido con datos correctos, asociado al usuario del JWT. |

---

### TC-HU05-06 · Rendimiento de confirmación autenticada

```gherkin
Dado que la confirmación de proveedor es funcionalmente estable
Cuando se ejecuta carga moderada autenticada (10 VU, 3 min)
Entonces el endpoint responde dentro del umbral acordado
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | k6 |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT de prueba. Datos de pedido controlados. |
| **Datos de prueba** | 10 VU · 3 min · POST /api/v1/pedido/confirmar |
| **Pasos** | Ejecutar k6 de confirmación → validar checks → analizar p95. |
| **Resultado esperado** | p95 < 1500 ms · http_req_failed < 1% · checks > 95%. |

---

# Microsprint 3

---

## HU-06 — Visualizar ruta del envío en el mapa

### TC-HU06-01 · Visualización de ruta con marcadores

```gherkin
Dado que el usuario autenticado ingresó origen y destino válidos y el sistema calculó la ruta
Cuando visualiza el mapa
Entonces ve la ruta dibujada con marcadores de origen y destino
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario autenticado. Ruta de ORS o stub. Leaflet renderizando. |
| **Datos de prueba** | origen = "Bogotá" · destino = "Medellín" · coordenadas válidas |
| **Pasos** | Flujo de pedido/recomendación → visualizar mapa → verificar polyline y marcadores. |
| **Resultado esperado** | Polyline visible. Marcadores de origen y destino correctos. |

---

### TC-HU06-02 · Ajuste automático del mapa a la ruta

```gherkin
Dado que el sistema dibujó la ruta en el mapa
Cuando la ruta es visible
Entonces el mapa se ajusta automáticamente para mostrar toda la ruta
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario autenticado. Mapa con ruta dibujada. |
| **Datos de prueba** | Ruta Bogotá–Medellín |
| **Pasos** | Cargar pantalla con ruta → verificar bounds → verificar ruta completa visible. |
| **Resultado esperado** | Leaflet ajusta bounds. No se corta polyline ni marcadores. |

---

### TC-HU06-03 · Conversión de coordenadas ORS → Leaflet

```gherkin
Dado que el sistema recibe coordenadas en formato [lng, lat] de ORS
Cuando dibuja la ruta en Leaflet
Entonces las convierte a formato [lat, lng] y la ruta se visualiza correctamente
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Stub de ORS con respuesta controlada. |
| **Datos de prueba** | entrada ORS = [-74.0721, 4.7110] · salida Leaflet = [4.7110, -74.0721] |
| **Pasos** | Preparar respuesta ORS → validar transformación en respuesta del backend. |
| **Resultado esperado** | Coordenadas invertidas correctamente. Marcadores en posiciones esperadas. |

---

### TC-HU06-04 · Intento de visualización sin datos de ruta

```gherkin
Dado que el usuario autenticado no tiene origen/destino completo o no existe ruta calculada
Cuando intenta visualizar el mapa
Entonces el sistema no muestra ruta e informa datos insuficientes
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario autenticado. Flujo sin coordenadas completas. |
| **Datos de prueba** | origen = vacío o ruta = null |
| **Pasos** | Flujo sin datos suficientes → abrir mapa → verificar mensaje informativo. |
| **Resultado esperado** | No se renderiza ruta falsa. Mensaje de datos insuficientes. Sin ruptura de pantalla. |

---

### TC-HU06-05 · Manejo de error de OpenRouteService

```gherkin
Dado que OpenRouteService falla o responde sin ruta
Cuando el sistema intenta calcular la ruta estimada
Entonces maneja el error sin bloquear el flujo principal
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Stub de ORS con error 500, timeout o respuesta vacía. JWT válido. |
| **Datos de prueba** | respuesta ORS = error o features vacío |
| **Pasos** | Configurar stub con falla → POST pedido → verificar respuesta controlada. |
| **Resultado esperado** | Error controlado. Sin excepción no manejada. Cotización no falla por completo. |

---

### TC-HU06-06 · Exploratoria visual del mapa

```gherkin
Dado que el usuario autenticado visualiza el mapa
Cuando cambia resoluciones, hace zoom, panea o recarga
Entonces el mapa se mantiene usable y consistente
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Medio |
| **Herramienta** | Manual exploratoria |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario autenticado. Chrome con DevTools. Ruta válida y caso sin ruta. |
| **Datos de prueba** | Resoluciones desktop y mobile. Rutas corta y larga. |
| **Pasos** | Diferentes ventanas → zoom → desplazamiento → recargar → volver → registrar hallazgos. |
| **Resultado esperado** | Mapa visible y usable. Sin errores de consola. Mensaje claro sin ruta. |

---

# Microsprint 4

---

## HU-07 — Registrar usuario

### TC-HU07-01 · Registro exitoso de usuario

```gherkin
Dado que una persona desea utilizar la plataforma
Cuando ingresa nombre "Usuario QA", correo único y contraseña "Password123"
Entonces el sistema registra al usuario y la cuenta queda disponible para login
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Correo no existente en base. Formulario disponible. |
| **Datos de prueba** | nombre = "Usuario QA" · email = "qa.unico@example.com" · password = "Password123" |
| **Pasos** | Abrir registro → completar datos → enviar → verificar éxito → login con cuenta creada. |
| **Resultado esperado** | Registro exitoso. Usuario persistido. Puede iniciar sesión. |

---

### TC-HU07-02 · Registro fallido por correo duplicado

```gherkin
Dado que ya existe un usuario con correo "usuario.existente@example.com"
Cuando otra persona intenta registrarse con ese mismo correo
Entonces el sistema retorna HTTP 409 e informa correo en uso
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario semilla existente. |
| **Datos de prueba** | email = "usuario.existente@example.com" |
| **Pasos** | Asegurar usuario semilla → POST /api/users/register con mismo email → verificar HTTP 409. |
| **Resultado esperado** | HTTP 409. Correo en uso. No se crea duplicado. |

---

### TC-HU07-03 · Registro fallido por campos obligatorios vacíos

```gherkin
Dado que una persona desea crear una cuenta
Cuando intenta registrarse sin nombre, correo o contraseña
Entonces el sistema retorna HTTP 400 indicando campos obligatorios
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Endpoint de registro disponible. |
| **Datos de prueba** | nombre = "" · email = "" · password = "" |
| **Pasos** | POST /api/users/register sin campos → verificar HTTP 400 y mensaje. |
| **Resultado esperado** | HTTP 400. Campos obligatorios. No se persiste usuario. |

---

### TC-HU07-04 · Registro fallido por contraseña < 8 caracteres

```gherkin
Dado que una persona desea crear una cuenta
Cuando ingresa una contraseña de 6 caracteres "abc123"
Entonces el sistema retorna HTTP 400 indicando longitud mínima de 8
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Endpoint disponible. Email único. |
| **Datos de prueba** | password = "abc123" (6 chars) |
| **Pasos** | POST /api/users/register con contraseña corta → verificar HTTP 400. |
| **Resultado esperado** | HTTP 400. Longitud mínima 8 caracteres. No se persiste usuario. |

---

### TC-HU07-05 · Persistencia segura de contraseña cifrada

```gherkin
Dado que una persona se registra correctamente
Cuando el sistema persiste el usuario
Entonces la contraseña no queda almacenada en texto plano
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Acceso controlado a BD QA o capa de integración. |
| **Datos de prueba** | password original = "Password123" |
| **Pasos** | Registrar usuario → consultar BD → verificar hash ≠ texto plano → login funciona. |
| **Resultado esperado** | Contraseña cifrada/hasheada. No coincide con texto plano. Login valida correctamente. |

---

### TC-HU07-06 · Exploratoria del formulario de registro

```gherkin
Dado que una persona usa el formulario de registro
Cuando corrige errores, cambia foco o reintenta
Entonces la experiencia es clara y no genera estados inconsistentes
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Medio |
| **Herramienta** | Manual exploratoria |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Navegador disponible. Correos válidos, duplicados e inválidos. |
| **Datos de prueba** | Nombres con espacios · emails con mayúsculas · password límite 8 chars. |
| **Pasos** | Explorar validaciones cliente/servidor → corregir campos → verificar redirección a login. |
| **Resultado esperado** | Mensajes claros. Redirección post-alta correcta. Sin filtración de contraseña. |

---

## HU-08 — Iniciar sesión

### TC-HU08-01 · Inicio de sesión exitoso con JWT

```gherkin
Dado que el usuario ya está registrado
Cuando ingresa correo y contraseña válidos
Entonces el sistema permite acceso y habilita funcionalidades protegidas
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario registrado. Formulario de login disponible. |
| **Datos de prueba** | email = "usuario.qa@example.com" · password = "Password123" |
| **Pasos** | Abrir login → enviar credenciales → verificar acceso → acceder a ruta protegida. |
| **Resultado esperado** | Login exitoso. Acceso a funcionalidades operativas. |

---

### TC-HU08-02 · Login fallido por credenciales inválidas

```gherkin
Dado que el usuario intenta acceder a la plataforma
Cuando ingresa correo o contraseña incorrectos
Entonces el sistema retorna HTTP 401 e informa credenciales inválidas
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario registrado. Endpoint de login disponible. |
| **Datos de prueba** | email = "usuario.qa@example.com" · password = "Incorrecta123" |
| **Pasos** | POST /api/users/login con credenciales inválidas → verificar HTTP 401. |
| **Resultado esperado** | HTTP 401. Credenciales inválidas. No se genera JWT. |

---

### TC-HU08-03 · Acceso sin autenticación a rutas protegidas

```gherkin
Dado que existe una funcionalidad operativa protegida (pedido, recomendación, confirmación, historial)
Cuando una persona intenta acceder sin JWT
Entonces el sistema bloquea el acceso y solicita autenticación
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | No se envía JWT. |
| **Datos de prueba** | rutas protegidas = pedido, recomendación, confirmación, historial |
| **Pasos** | Llamar cada endpoint protegido sin token → verificar HTTP 401/403. |
| **Resultado esperado** | HTTP 401/403. No se exponen datos operativos. |

---

### TC-HU08-04 · Cierre de sesión bloquea acceso y limpia historial

```gherkin
Dado que el usuario A ha iniciado sesión y tiene historial visible
Cuando cierra sesión
Entonces el sistema bloquea rutas protegidas y limpia estado de historial
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario A autenticado con pedidos. Usuario B disponible. |
| **Datos de prueba** | usuarioA con pedidos · usuarioB distinto |
| **Pasos** | Login A → abrir historial → logout → verificar bloqueo → login B → verificar aislamiento. |
| **Resultado esperado** | Token limpio. Rutas bloqueadas. Pedidos de A no visibles para B. |

---

### TC-HU08-05 · Rechazo de token inválido o expirado

```gherkin
Dado que el endpoint protegido requiere JWT válido
Cuando se envía un token inválido, alterado o expirado
Entonces el sistema rechaza la solicitud
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Endpoint protegido disponible. Token inválido preparado. |
| **Datos de prueba** | Authorization = "Bearer token-invalido" |
| **Pasos** | Llamar endpoint protegido con token inválido → verificar HTTP 401/403. |
| **Resultado esperado** | HTTP 401/403. No se procesa solicitud. Sin datos expuestos. |

---

### TC-HU08-06 · Restauración de sesión al recargar

```gherkin
Dado que el usuario inició sesión correctamente
Cuando recarga el navegador
Entonces el sistema restaura la sesión o solicita login si no existe token válido
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Medio |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario autenticado. Persistencia de sesión frontend configurada. |
| **Datos de prueba** | JWT válido almacenado |
| **Pasos** | Login → recargar → verificar sesión → logout → recargar → verificar bloqueo. |
| **Resultado esperado** | Con token: sesión restaurada. Sin token: solicita login. |

---

### TC-HU08-07 · Rendimiento de login controlado

```gherkin
Dado que existen usuarios de prueba precreados
Cuando se ejecuta carga moderada de login (5-10 VU, 3 min)
Entonces el servicio responde dentro del umbral
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Medio |
| **Herramienta** | k6 |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuarios precreados. BD estable. |
| **Datos de prueba** | 5-10 VU · 3 min · POST /api/users/login |
| **Pasos** | Ejecutar k6 de login → validar checks y presencia de token. |
| **Resultado esperado** | p95 < 1000 ms · http_req_failed < 1% · checks > 95%. |

---

### TC-HU08-08 · Exploratoria de autenticación, logout e historial

```gherkin
Dado que el usuario interactúa con login, rutas protegidas, logout e historial
Cuando alterna sesiones, recarga, usa múltiples pestañas y consulta historial
Entonces la aplicación mantiene seguridad, claridad de estado y aislamiento de datos
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Medio |
| **Herramienta** | Manual exploratoria |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuarios con/sin pedidos. Múltiples pestañas. Simular error de API. |
| **Datos de prueba** | Sesiones válida/cerrada · historial con/sin pedidos · error de API. |
| **Pasos** | Múltiples pestañas → logout parcial → atrás/adelante → validar historial → cambiar sesión → registrar hallazgos. |
| **Resultado esperado** | Rutas bloqueadas tras logout. Estados claros. Sin mezcla de datos entre usuarios. |

---

## HU-09 — Consultar pedidos del usuario

### TC-HU09-01 · Visualización de pedidos del usuario autenticado

```gherkin
Dado que el usuario ha iniciado sesión y existen pedidos asociados a su cuenta
Cuando consulta su listado de pedidos
Entonces el sistema muestra solo sus pedidos con origen, destino, peso, prioridad y proveedor
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario autenticado con pedidos confirmados. |
| **Datos de prueba** | Pedidos con datos completos (origen, destino, peso, prioridad, proveedor). |
| **Pasos** | Confirmar pedidos → abrir historial → verificar datos completos. |
| **Resultado esperado** | Solo pedidos del usuario. Cada pedido con datos mínimos requeridos. |

---

### TC-HU09-02 · Usuario sin pedidos registrados

```gherkin
Dado que el usuario ha iniciado sesión y no existen pedidos asociados
Cuando consulta su listado de pedidos
Entonces el sistema informa que no existen pedidos registrados
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario autenticado sin pedidos. |
| **Datos de prueba** | usuarioSinPedidos = true |
| **Pasos** | Login con usuario sin pedidos → abrir historial → verificar mensaje. |
| **Resultado esperado** | UI informa sin pedidos. Sin error técnico. |

---

### TC-HU09-03 · Aislamiento de pedidos entre usuarios

```gherkin
Dado que existen pedidos de usuario A y usuario B
Cuando usuario A consulta su historial
Entonces no ve pedidos de usuario B
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario A y B con pedidos diferenciados. JWT de cada uno. |
| **Datos de prueba** | usuarioA · usuarioB · pedidos diferenciados |
| **Pasos** | Crear pedidos A y B → GET /mis-pedidos con JWT A → GET con JWT B → comparar. |
| **Resultado esperado** | A solo ve pedidos de A. B solo ve pedidos de B. Sin filtración. |

---

### TC-HU09-04 · La consulta no acepta userId desde frontend

```gherkin
Dado que la propiedad del pedido se obtiene desde el JWT
Cuando el cliente intenta forzar un userId distinto en la consulta
Entonces el sistema ignora o rechaza el userId y filtra por el JWT
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | Usuario A autenticado. Usuario B con pedidos. |
| **Datos de prueba** | JWT = usuario A · query/body userId = usuario B |
| **Pasos** | GET /mis-pedidos con JWT A + userId B → verificar que no muestra pedidos de B. |
| **Resultado esperado** | Sistema filtra por JWT. No expone datos de B. |

---

### TC-HU09-05 · Rendimiento de historial autenticado

```gherkin
Dado que existen usuarios autenticados con datos de prueba
Cuando se ejecuta carga sobre la consulta de historial (10-20 VU, 5 min)
Entonces el endpoint responde dentro del umbral acordado
```

| Campo | Detalle |
|---|---|
| **Prioridad** | Alto |
| **Herramienta** | k6 |
| **Estado** | Sin ejecutar |
| **Resultado de ejecución** | — |
| **Precondiciones** | JWT precreados. Pedidos semilla. BD estable. |
| **Datos de prueba** | 10-20 VU · 5 min · GET /api/v1/pedido/mis-pedidos |
| **Pasos** | Ejecutar k6 de historial → validar checks y estructura de lista. |
| **Resultado esperado** | p95 < 800 ms · http_req_failed < 1% · checks > 95%. |

---

# Resumen de cobertura

## Cobertura por microsprint

| Microsprint | Historias | Casos | Foco principal |
|---|---|---:|---|
| Microsprint 1 | HU-01, HU-02 | 14 | Registro de pedido, autocompletado, cobertura Colombia, peso y prioridad. |
| Microsprint 2 | HU-03, HU-04, HU-05 | 15 | Recomendación, desempate, alternativas, selección, confirmación y rendimiento. |
| Microsprint 3 | HU-06 | 6 | Ruta en mapa, ORS, conversión de coordenadas, marcadores y manejo de errores. |
| Microsprint 4 | HU-07, HU-08, HU-09 | 20 | Registro, login, JWT, logout, rutas protegidas, historial y aislamiento. |
| **Total** | **HU-01 a HU-09** | **55** | **MVP base + MVP v2 completo** |

## Cobertura por historia de usuario

| ID | Historia de Usuario | Casos | Serenity | Karate | Exploratoria | k6 |
|---|---|---:|---:|---:|---:|---:|
| HU-01 | Registrar pedido de envío | 11 | 2 | 7 | 1 | 1 |
| HU-02 | Definir prioridad del envío | 3 | 2 | 1 | 0 | 0 |
| HU-03 | Obtener recomendación principal | 7 | 0 | 5 | 1 | 1 |
| HU-04 | Obtener alternativas de proveedores | 3 | 2 | 1 | 0 | 0 |
| HU-05 | Seleccionar y confirmar proveedor | 6 | 2 | 3 | 0 | 1 |
| HU-06 | Visualizar ruta en el mapa | 6 | 3 | 2 | 1 | 0 |
| HU-07 | Registrar usuario | 6 | 1 | 4 | 1 | 0 |
| HU-08 | Iniciar sesión | 8 | 3 | 2 | 1 | 1 |
| HU-09 | Consultar pedidos del usuario | 5 | 2 | 2 | 0 | 1 |
| **Total** | **MVP base + MVP v2** | **55** | **17** | **27** | **5** | **5** |

## Cobertura de reglas del PRD

| Reglas PRD | HU cubiertas | Casos principales |
|---|---|---|
| Reglas 1–4 | HU-01, HU-02 | TC-HU01-01 a TC-HU01-11, TC-HU02-01 a TC-HU02-03 |
| Reglas 5–8 | HU-03 | TC-HU03-01 a TC-HU03-07 |
| Regla 9 | HU-04 | TC-HU04-01 a TC-HU04-03 |
| Reglas 10–11, 24 | HU-05, HU-09 | TC-HU05-01 a TC-HU05-06, TC-HU09-01, TC-HU09-03, TC-HU09-04 |
| Reglas 12–16 | HU-06 | TC-HU06-01 a TC-HU06-06 |
| Reglas 17–23, 28 | HU-07, HU-08 | TC-HU07-01 a TC-HU07-06, TC-HU08-01 a TC-HU08-08 |
| Reglas 25–27 | HU-09 | TC-HU09-01 a TC-HU09-05 |
