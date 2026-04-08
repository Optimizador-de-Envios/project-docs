# Casos de Prueba — Optimizador de Envíos

## MVP base + MVP v2: HU-01 a HU-09

---

## 1. Convenciones de ejecución

Este documento define los casos de prueba para el MVP base y el MVP v2 del Optimizador de Envíos. La cobertura se organiza por historia de usuario y mantiene trazabilidad con el PRD, las subtareas QA, el plan de pruebas y los criterios de aceptación.

### Tipos de prueba y herramientas

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
- Los estados iniciales de base de datos se preparan por caso para evitar contaminación entre ejecuciones.

### Estados

Todos los casos quedan inicialmente en estado **Sin ejecutar** y el campo **Resultado de ejecución** se completa durante la ejecución formal de QA.

---

# Microsprint 1

---

## HU-01 — Registrar pedido de envío

### TC-HU01-01 · Registro exitoso con datos válidos y usuario autenticado

| Campo | Detalle |
|---|---|
| **ID** | TC-HU01-01 |
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado necesita enviar un producto
Cuando ingresa origen, destino y peso válidos dentro de Colombia
Entonces el sistema debe registrar los datos del pedido
Y debe permitir continuar con el cálculo del envío
```

**Precondiciones**

- Usuario registrado e iniciado en sesión.
- Formulario de pedido disponible.

**Datos de prueba**

- `origen = "Bogotá, Colombia"`
- `destino = "Medellín, Colombia"`
- `peso = 5 Kg`

**Pasos de ejecución**

1. Iniciar sesión desde la UI.
2. Completar origen, destino y peso en el formulario.
3. Verificar que el sistema permite continuar al cálculo de opciones.
4. Verificar que el estado del flujo conserva origen, destino y peso.

**Resultado esperado**

- El formulario avanza exitosamente al siguiente paso.
- Los datos ingresados se conservan en el flujo.
- El frontend permite continuar al cálculo de opciones.

---

### TC-HU01-02 · Autocompletado de origen y destino restringido a Colombia

| Campo | Detalle |
|---|---|
| **ID** | TC-HU01-02 |
| **Prioridad** | Alto |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado registra un pedido
Cuando escribe texto en origen o destino
Entonces el sistema debe consultar sugerencias mediante OpenRouteService
Y debe restringirlas a ubicaciones dentro de Colombia
```

**Precondiciones**

- Usuario autenticado.
- Endpoint o stub de OpenRouteService disponible.
- Campo de origen/destino visible.

**Datos de prueba**

- `búsqueda = "Bog"`
- `país esperado = "Colombia"`

**Pasos de ejecución**

1. Escribir texto parcial en origen y destino.
2. Verificar que aparecen sugerencias.
3. Revisar que las sugerencias pertenecen a Colombia.
4. Seleccionar una sugerencia válida.

**Resultado esperado**

- Las sugerencias pertenecen a Colombia.
- La selección de una sugerencia válida llena el campo correspondiente.
- No se ofrecen ubicaciones fuera de cobertura.

---

### TC-HU01-03 · Registro fallido por campos obligatorios vacíos

| Campo | Detalle |
|---|---|
| **ID** | TC-HU01-03 |
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado necesita enviar un producto
Cuando intenta registrar un pedido sin origen, destino ni peso
Entonces el sistema no debe permitir continuar
Y debe informar que todos los campos son obligatorios
```

**Precondiciones**

- Token JWT válido.
- Endpoint de pedido disponible.

**Datos de prueba**

- `origen = ""`
- `destino = ""`
- `peso = vacío`

**Pasos de ejecución**

1. Enviar `POST /api/v1/pedido` sin campos requeridos con JWT válido.
2. Verificar código HTTP.
3. Verificar estructura y mensaje del cuerpo de respuesta.

**Resultado esperado**

- La operación retorna HTTP 400.
- El mensaje indica que los campos obligatorios deben completarse.
- No se crea pedido ni recomendación.

---

### TC-HU01-04 · Registro fallido por origen fuera de Colombia

| Campo | Detalle |
|---|---|
| **ID** | TC-HU01-04 |
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado necesita enviar un producto
Cuando ingresa origen fuera de Colombia y destino dentro de Colombia
Entonces el sistema debe bloquear el cálculo
Y debe informar que el envío está fuera de cobertura
```

**Precondiciones**

- Token JWT válido.
- Validación de cobertura activa.

**Datos de prueba**

- `origen = "Buenos Aires, Argentina"`
- `destino = "Medellín, Colombia"`
- `peso = 5 Kg`

**Pasos de ejecución**

1. Enviar `POST /api/v1/pedido` con origen fuera de Colombia.
2. Verificar código HTTP y mensaje de error.

**Resultado esperado**

- La operación retorna HTTP 400.
- El sistema informa que el envío está fuera de cobertura.
- No se genera recomendación.

---

### TC-HU01-05 · Registro fallido por destino fuera de Colombia

| Campo | Detalle |
|---|---|
| **ID** | TC-HU01-05 |
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado necesita enviar un producto
Cuando ingresa origen dentro de Colombia y destino fuera de Colombia
Entonces el sistema debe bloquear el cálculo
Y debe informar que el envío está fuera de cobertura
```

**Precondiciones**

- Token JWT válido.
- Validación de cobertura activa.

**Datos de prueba**

- `origen = "Bogotá, Colombia"`
- `destino = "Buenos Aires, Argentina"`
- `peso = 5 Kg`

**Pasos de ejecución**

1. Enviar `POST /api/v1/pedido` con destino fuera de Colombia.
2. Verificar código HTTP y mensaje de error.

**Resultado esperado**

- La operación retorna HTTP 400.
- El sistema informa que el envío está fuera de cobertura.
- No se genera recomendación.

---

### TC-HU01-06 · Registro fallido por origen y destino fuera de Colombia

| Campo | Detalle |
|---|---|
| **ID** | TC-HU01-06 |
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado necesita enviar un producto
Cuando ingresa origen y destino fuera de Colombia
Entonces el sistema debe bloquear el cálculo
Y debe informar que el envío está fuera de cobertura
```

**Precondiciones**

- Token JWT válido.
- Validación de cobertura activa.

**Datos de prueba**

- `origen = "Montevideo, Uruguay"`
- `destino = "Buenos Aires, Argentina"`
- `peso = 5 Kg`

**Pasos de ejecución**

1. Enviar `POST /api/v1/pedido` con origen y destino fuera de Colombia.
2. Verificar código HTTP y mensaje de error.

**Resultado esperado**

- La operación retorna HTTP 400.
- El sistema informa que el envío está fuera de cobertura.
- No se genera recomendación.

---

### TC-HU01-07 · Registro exitoso con peso mínimo permitido

| Campo | Detalle |
|---|---|
| **ID** | TC-HU01-07 |
| **Prioridad** | Medio |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado ingresa datos válidos
Cuando el peso es 0.001 Kg
Entonces el sistema debe aceptar el pedido
```

**Precondiciones**

- Token JWT válido.
- Origen y destino dentro de Colombia.

**Datos de prueba**

- `origen = "Bogotá, Colombia"`
- `destino = "Medellín, Colombia"`
- `peso = 0.001 Kg`

**Pasos de ejecución**

1. Enviar `POST /api/v1/pedido` con peso mínimo permitido.
2. Verificar código HTTP.

**Resultado esperado**

- La operación retorna HTTP 201.
- El peso de 0.001 Kg es aceptado.

---

### TC-HU01-08 · Registro exitoso con peso máximo permitido

| Campo | Detalle |
|---|---|
| **ID** | TC-HU01-08 |
| **Prioridad** | Medio |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado ingresa datos válidos
Cuando el peso es 70 Kg
Entonces el sistema debe aceptar el pedido
```

**Precondiciones**

- Token JWT válido.
- Origen y destino dentro de Colombia.

**Datos de prueba**

- `origen = "Bogotá, Colombia"`
- `destino = "Medellín, Colombia"`
- `peso = 70 Kg`

**Pasos de ejecución**

1. Enviar `POST /api/v1/pedido` con peso máximo permitido.
2. Verificar código HTTP.

**Resultado esperado**

- La operación retorna HTTP 201.
- El peso de 70 Kg es aceptado.

---

### TC-HU01-09 · Registro fallido por peso inferior al mínimo

| Campo | Detalle |
|---|---|
| **ID** | TC-HU01-09 |
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado ingresa origen y destino válidos
Cuando el peso es menor a 0.001 Kg
Entonces el sistema debe bloquear el pedido
Y debe informar que el peso no está cubierto
```

**Precondiciones**

- Token JWT válido.
- Origen y destino dentro de Colombia.

**Datos de prueba**

- `origen = "Bogotá, Colombia"`
- `destino = "Medellín, Colombia"`
- `peso = 0.0009 Kg`

**Pasos de ejecución**

1. Enviar `POST /api/v1/pedido` con peso inferior al mínimo.
2. Verificar código HTTP y mensaje.

**Resultado esperado**

- La operación retorna HTTP 400.
- El mensaje indica que el peso está fuera del rango permitido.

---

### TC-HU01-10 · Registro fallido por peso superior al máximo

| Campo | Detalle |
|---|---|
| **ID** | TC-HU01-10 |
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado ingresa origen y destino válidos
Cuando el peso es mayor a 70 Kg
Entonces el sistema debe bloquear el pedido
Y debe informar que el peso no está cubierto
```

**Precondiciones**

- Token JWT válido.
- Origen y destino dentro de Colombia.

**Datos de prueba**

- `origen = "Bogotá, Colombia"`
- `destino = "Medellín, Colombia"`
- `peso = 70.001 Kg`

**Pasos de ejecución**

1. Enviar `POST /api/v1/pedido` con peso superior al máximo.
2. Verificar código HTTP y mensaje.

**Resultado esperado**

- La operación retorna HTTP 400.
- El mensaje indica que el peso está fuera del rango permitido.

---

### TC-HU01-11 · Intento de registrar pedido sin autenticación

| Campo | Detalle |
|---|---|
| **ID** | TC-HU01-11 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que una persona no ha iniciado sesión
Cuando intenta registrar un pedido de envío
Entonces el sistema no debe permitir el acceso
Y debe solicitar autenticación
```

**Precondiciones**

- No se envía JWT en la solicitud.

**Datos de prueba**

- `origen = "Bogotá, Colombia"`
- `destino = "Medellín, Colombia"`
- `peso = 5 Kg`

**Pasos de ejecución**

1. Llamar `POST /api/v1/pedido` sin token de autorización.
2. Verificar código HTTP.

**Resultado esperado**

- La API retorna HTTP 401 o 403.
- No se procesa el pedido.

---

### TC-HU01-12 · Exploratoria del formulario y autocompletado

| Campo | Detalle |
|---|---|
| **ID** | TC-HU01-12 |
| **Prioridad** | Medio |
| **Herramienta** | Manual exploratoria |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado usa el formulario de pedido
Cuando interactúa libremente con campos, sugerencias, errores y navegación
Entonces no deben aparecer bloqueos, mensajes ambiguos ni inconsistencias visuales
```

**Precondiciones**

- Usuario autenticado.
- DevTools disponible.
- Datos válidos e inválidos preparados.

**Datos de prueba**

- Búsquedas parciales, acentos, mayúsculas/minúsculas, edición posterior de ubicaciones, pesos con separador decimal.

**Pasos de ejecución**

1. Explorar escritura rápida, borrado, selección y reintento de origen/destino.
2. Probar pesos con formatos habituales.
3. Revisar mensajes y llamadas de red.
4. Registrar hallazgos con evidencia.

**Resultado esperado**

- El formulario mantiene consistencia visual y funcional.
- Los mensajes son claros.
- No se conservan sugerencias o estados inválidos luego de corregir datos.

---

### TC-HU01-13 · Smoke de API autenticada para pedido

| Campo | Detalle |
|---|---|
| **ID** | TC-HU01-13 |
| **Prioridad** | Alto |
| **Herramienta** | k6 |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que existe un usuario autenticado y datos válidos
Cuando se ejecuta un smoke de baja carga sobre el endpoint de pedido
Entonces el servicio debe responder de forma estable antes de iniciar carga mayor
```

**Precondiciones**

- Usuario de prueba precreado.
- JWT válido.
- OpenRouteService mockeado si aplica.

**Datos de prueba**

- 1 VU durante 1 minuto.
- `origen / destino = Colombia`
- `peso nominal = 5 Kg`

**Pasos de ejecución**

1. Ejecutar script k6 smoke sobre `POST /api/v1/pedido`.
2. Validar checks de status y estructura mínima.
3. Revisar tasa de error.

**Resultado esperado**

- `http_req_failed < 1%`
- `checks > 95%`
- No se observan errores funcionales de autenticación ni contrato.

---

## HU-02 — Definir prioridad del envío

### TC-HU02-01 · Selección exitosa de prioridad por menor costo

| Campo | Detalle |
|---|---|
| **ID** | TC-HU02-01 |
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado registró un pedido válido
Cuando selecciona prioridad de menor costo
Entonces el sistema debe usar el costo como criterio principal
```

**Precondiciones**

- Usuario autenticado.
- Pedido válido en el flujo.

**Datos de prueba**

- `prioridad = "MENOR_COSTO"`

**Pasos de ejecución**

1. Completar o preparar pedido válido desde la UI.
2. Seleccionar prioridad «menor costo».
3. Verificar que el flujo avanza y la prioridad queda registrada.

**Resultado esperado**

- La prioridad queda registrada como `MENOR_COSTO`.
- El sistema usa costo para la recomendación posterior.

---

### TC-HU02-02 · Selección exitosa de prioridad por menor tiempo

| Campo | Detalle |
|---|---|
| **ID** | TC-HU02-02 |
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado registró un pedido válido
Cuando selecciona prioridad de menor tiempo de entrega
Entonces el sistema debe usar el tiempo como criterio principal
```

**Precondiciones**

- Usuario autenticado.
- Pedido válido en el flujo.

**Datos de prueba**

- `prioridad = "MENOR_TIEMPO"`

**Pasos de ejecución**

1. Completar o preparar pedido válido desde la UI.
2. Seleccionar prioridad «menor tiempo».
3. Verificar que el flujo avanza y la prioridad queda registrada.

**Resultado esperado**

- La prioridad queda registrada como `MENOR_TIEMPO`.
- El sistema usa tiempo para la recomendación posterior.

---

### TC-HU02-03 · Registro fallido por no seleccionar prioridad

| Campo | Detalle |
|---|---|
| **ID** | TC-HU02-03 |
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado registró un pedido válido
Cuando no selecciona ninguna prioridad
Entonces el sistema no debe continuar con el cálculo
Y debe informar que se debe seleccionar prioridad
```

**Precondiciones**

- Token JWT válido.
- Datos de origen, destino y peso válidos.

**Datos de prueba**

- `prioridad = vacío`

**Pasos de ejecución**

1. Enviar `POST /api/v1/pedido` con datos válidos pero sin prioridad.
2. Verificar código HTTP y mensaje.

**Resultado esperado**

- La operación retorna HTTP 400.
- El mensaje indica que la prioridad es obligatoria.
- No se genera recomendación.

---

### TC-HU02-04 · Intento de seleccionar prioridad sin autenticación

| Campo | Detalle |
|---|---|
| **ID** | TC-HU02-04 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que una persona no ha iniciado sesión
Cuando intenta seleccionar la prioridad del envío
Entonces el sistema no debe permitir el acceso
Y debe solicitar autenticación
```

**Precondiciones**

- No se envía JWT en la solicitud.

**Datos de prueba**

- `prioridad = "MENOR_COSTO"`

**Pasos de ejecución**

1. Enviar solicitud protegida sin token de autorización.
2. Verificar código HTTP.

**Resultado esperado**

- La API retorna HTTP 401 o 403.
- No se guarda prioridad.

---

### TC-HU02-05 · Exploratoria del selector de prioridad

| Campo | Detalle |
|---|---|
| **ID** | TC-HU02-05 |
| **Prioridad** | Medio |
| **Herramienta** | Manual exploratoria |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado se encuentra en el paso de prioridad
Cuando alterna opciones, retrocede y corrige datos
Entonces el estado del flujo debe mantenerse consistente
```

**Precondiciones**

- Usuario autenticado.
- Pedido válido ya registrado.

**Datos de prueba**

- `prioridades disponibles = menor costo, menor tiempo`

**Pasos de ejecución**

1. Alternar entre ambas prioridades.
2. Volver al paso anterior y regresar.
3. Revisar que solo una prioridad quede seleccionada.
4. Registrar inconsistencias.

**Resultado esperado**

- El selector no permite estados ambiguos.
- La prioridad seleccionada coincide con el estado global y con la solicitud enviada.

---

# Microsprint 2

---

## HU-03 — Obtener recomendación principal de proveedor de envío

### TC-HU03-01 · Recomendación principal por menor costo

| Campo | Detalle |
|---|---|
| **ID** | TC-HU03-01 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado definió prioridad de menor costo
Cuando el sistema calcula las opciones disponibles
Entonces debe devolver como recomendada la opción de menor costo
```

**Precondiciones**

- Token JWT válido.
- Proveedores mock con costos distintos.
- Pedido válido en Colombia.

**Datos de prueba**

- `origen = "Bogotá, Colombia"`
- `destino = "Cali, Colombia"`
- `peso = 10 Kg`
- `prioridad = "MENOR_COSTO"`
- FedEx = 35 000 COP / 2 días
- DHL = 42 000 COP / 1 día
- Local = 28 000 COP / 3 días

**Pasos de ejecución**

1. Enviar `POST /api/v1/pedido` con prioridad menor costo.
2. Verificar que la recomendación principal tiene el menor costo.
3. Validar estructura de respuesta.

**Resultado esperado**

- La recomendación principal corresponde a **Local**.
- El costo recomendado es 28 000 COP.

---

### TC-HU03-02 · Recomendación principal por menor tiempo

| Campo | Detalle |
|---|---|
| **ID** | TC-HU03-02 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado definió prioridad de menor tiempo
Cuando el sistema calcula las opciones disponibles
Entonces debe devolver como recomendada la opción de menor tiempo
```

**Precondiciones**

- Token JWT válido.
- Proveedores mock con tiempos distintos.
- Pedido válido en Colombia.

**Datos de prueba**

- `origen = "Bogotá, Colombia"`
- `destino = "Cali, Colombia"`
- `peso = 10 Kg`
- `prioridad = "MENOR_TIEMPO"`
- FedEx = 35 000 COP / 2 días
- DHL = 42 000 COP / 1 día
- Local = 28 000 COP / 3 días

**Pasos de ejecución**

1. Enviar `POST /api/v1/pedido` con prioridad menor tiempo.
2. Verificar que la recomendación principal tiene el menor tiempo.
3. Validar estructura de respuesta.

**Resultado esperado**

- La recomendación principal corresponde a **DHL**.
- El tiempo recomendado es 1 día.

---

### TC-HU03-03 · Desempate por tiempo ante empate en menor costo

| Campo | Detalle |
|---|---|
| **ID** | TC-HU03-03 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que la prioridad es menor costo
Y existen proveedores empatados en el costo mínimo
Cuando el sistema calcula la recomendación
Entonces debe recomendar el proveedor con menor tiempo entre los empatados
```

**Precondiciones**

- Token JWT válido.
- Datos mock preparados para empate en costo.

**Datos de prueba**

- `prioridad = "MENOR_COSTO"`
- FedEx = 28 000 COP / 2 días
- DHL = 42 000 COP / 1 día
- Local = 28 000 COP / 3 días

**Pasos de ejecución**

1. Configurar o usar set de datos de empate en costo.
2. Enviar `POST /api/v1/pedido`.
3. Verificar recomendación y criterio de desempate.

**Resultado esperado**

- La recomendación principal corresponde a **FedEx**.
- El sistema desempata FedEx sobre Local por menor tiempo entre empatados.

---

### TC-HU03-04 · Desempate por costo ante empate en menor tiempo

| Campo | Detalle |
|---|---|
| **ID** | TC-HU03-04 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que la prioridad es menor tiempo
Y existen proveedores empatados en el menor tiempo
Cuando el sistema calcula la recomendación
Entonces debe recomendar el proveedor con menor costo entre los empatados
```

**Precondiciones**

- Token JWT válido.
- Datos mock preparados para empate en tiempo.

**Datos de prueba**

- `prioridad = "MENOR_TIEMPO"`
- FedEx = 35 000 COP / 1 día
- DHL = 42 000 COP / 1 día
- Local = 28 000 COP / 3 días

**Pasos de ejecución**

1. Configurar o usar set de datos de empate en tiempo.
2. Enviar `POST /api/v1/pedido`.
3. Verificar recomendación y criterio de desempate.

**Resultado esperado**

- La recomendación principal corresponde a **FedEx**.
- El sistema desempata FedEx sobre DHL por menor costo entre empatados.

---

### TC-HU03-05 · Intento de obtener recomendación sin autenticación

| Campo | Detalle |
|---|---|
| **ID** | TC-HU03-05 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que una persona no ha iniciado sesión
Cuando intenta obtener una recomendación de proveedor
Entonces el sistema debe bloquear el acceso
Y debe solicitar autenticación
```

**Precondiciones**

- No se envía JWT en la solicitud.

**Datos de prueba**

- Pedido válido con `prioridad = "MENOR_COSTO"`.

**Pasos de ejecución**

1. Llamar `POST /api/v1/pedido` sin token de autorización.
2. Verificar código HTTP.

**Resultado esperado**

- La API retorna HTTP 401 o 403.
- No se devuelve recomendación.

---

### TC-HU03-06 · Contrato de respuesta de recomendación

| Campo | Detalle |
|---|---|
| **ID** | TC-HU03-06 |
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado solicita una recomendación válida
Cuando la API responde
Entonces la estructura debe incluir recomendación principal y datos comparables
```

**Precondiciones**

- JWT válido.
- Pedido válido en Colombia.
- Proveedores mock activos.

**Datos de prueba**

- Campos esperados: `proveedor`, `costo`, `tiempoEntrega`, `prioridad`, `alternativas` (si existen).

**Pasos de ejecución**

1. Ejecutar `POST /api/v1/pedido`.
2. Validar status code.
3. Validar schema de respuesta.
4. Validar tipos de datos y ausencia de campos nulos críticos.

**Resultado esperado**

- La respuesta cumple el contrato definido.
- La recomendación incluye proveedor, costo y tiempo de entrega.
- Los tipos de datos son consistentes.

---

### TC-HU03-07 · Carga del motor de recomendación

| Campo | Detalle |
|---|---|
| **ID** | TC-HU03-07 |
| **Prioridad** | Alto |
| **Herramienta** | k6 |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el flujo de recomendación es funcionalmente estable
Cuando se ejecuta carga concurrente autenticada
Entonces el motor debe responder dentro del umbral acordado
```

**Precondiciones**

- Usuarios o JWT de prueba disponibles.
- OpenRouteService mockeado o desacoplado para carga.
- Datos dentro de Colombia.

**Datos de prueba**

- Rampa de 5 a 25 VU durante 5 minutos.
- `endpoint = POST /api/v1/pedido`

**Pasos de ejecución**

1. Ejecutar script k6 de carga.
2. Verificar checks de status, contrato básico y recomendación presente.
3. Analizar p95, p99 y tasa de error.

**Resultado esperado**

- `p95 < 1200 ms`
- `http_req_failed < 1%`
- Checks funcionales por encima de 95%.

---

### TC-HU03-08 · Exploratoria de claridad de recomendación

| Campo | Detalle |
|---|---|
| **ID** | TC-HU03-08 |
| **Prioridad** | Medio |
| **Herramienta** | Manual exploratoria |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario visualiza la recomendación
Cuando compara costo, tiempo y prioridad seleccionada
Entonces la pantalla debe comunicar claramente por qué se recomendó ese proveedor
```

**Precondiciones**

- Usuario autenticado.
- Datos de recomendación con y sin empate disponibles.

**Datos de prueba**

- `prioridad = menor costo y menor tiempo`
- Escenarios con empate.

**Pasos de ejecución**

1. Ejecutar distintos flujos de recomendación.
2. Revisar textos, orden visual y valores mostrados.
3. Comparar con respuesta de la API desde DevTools.
4. Registrar dudas o inconsistencias.

**Resultado esperado**

- La recomendación visible coincide con la API.
- El usuario puede entender el proveedor recomendado y sus valores principales.
- No hay datos contradictorios entre pantalla y backend.

---

## HU-04 — Obtener opciones alternativas de proveedores

### TC-HU04-01 · Visualización de alternativas disponibles

| Campo | Detalle |
|---|---|
| **ID** | TC-HU04-01 |
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado obtuvo una recomendación principal
Y existen otras opciones disponibles
Cuando el sistema muestra la recomendación
Entonces debe mostrar automáticamente las alternativas disponibles
```

**Precondiciones**

- Usuario autenticado.
- Respuesta de recomendación con más de un proveedor disponible.

**Datos de prueba**

- `recomendado = "Local"`
- `alternativas = "FedEx", "DHL"`

**Pasos de ejecución**

1. Ejecutar flujo de recomendación desde la UI.
2. Verificar que se muestran alternativas separadas de la recomendación.
3. Verificar que cada alternativa incluye proveedor, costo y tiempo.

**Resultado esperado**

- El sistema muestra alternativas disponibles.
- Cada alternativa incluye proveedor, costo y tiempo de entrega.
- La sección de alternativas es visible y comprensible.

---

### TC-HU04-02 · Ausencia de opciones alternativas

| Campo | Detalle |
|---|---|
| **ID** | TC-HU04-02 |
| **Prioridad** | Alto |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado obtuvo una recomendación principal
Y no existen otras opciones disponibles
Cuando el sistema muestra la recomendación
Entonces debe notificar que no existen alternativas
```

**Precondiciones**

- Usuario autenticado.
- Datos mock o stub con un solo proveedor disponible.

**Datos de prueba**

- `recomendado = único proveedor disponible`
- `alternativas = []`

**Pasos de ejecución**

1. Configurar respuesta con una sola opción disponible.
2. Ejecutar recomendación desde la UI.
3. Verificar mensaje informativo en pantalla.

**Resultado esperado**

- La UI informa que no existen opciones alternativas.
- No se muestra una lista vacía confusa.

---

### TC-HU04-03 · La recomendación principal no se duplica como alternativa

| Campo | Detalle |
|---|---|
| **ID** | TC-HU04-03 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que existen recomendación y alternativas
Cuando se renderiza la lista de opciones
Entonces la recomendación principal no debe repetirse dentro de alternativas
```

**Precondiciones**

- Token JWT válido.
- Respuesta con recomendación principal y alternativas.

**Datos de prueba**

- `recomendado = "Local"`
- `alternativas esperadas = proveedores distintos a "Local"`

**Pasos de ejecución**

1. Enviar `POST /api/v1/pedido` con datos válidos.
2. Validar que la lista de alternativas no contiene el proveedor recomendado.
3. Comparar nombres de proveedores.

**Resultado esperado**

- Ninguna alternativa tiene el mismo proveedor que la recomendación principal.

---

### TC-HU04-04 · Intento de visualizar alternativas sin autenticación

| Campo | Detalle |
|---|---|
| **ID** | TC-HU04-04 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que una persona no ha iniciado sesión
Cuando intenta visualizar opciones alternativas
Entonces el sistema debe bloquear el acceso
Y debe solicitar autenticación
```

**Precondiciones**

- No se envía JWT en la solicitud.

**Datos de prueba**

- Pedido válido con `prioridad = "MENOR_COSTO"`.

**Pasos de ejecución**

1. Llamar endpoint protegido sin token de autorización.
2. Verificar código HTTP.

**Resultado esperado**

- La API retorna HTTP 401 o 403.
- No se devuelven alternativas.

---

### TC-HU04-05 · Exploratoria de comparación de alternativas

| Campo | Detalle |
|---|---|
| **ID** | TC-HU04-05 |
| **Prioridad** | Medio |
| **Herramienta** | Manual exploratoria |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario visualiza recomendación y alternativas
Cuando compara proveedores, costos y tiempos
Entonces la pantalla debe facilitar la comparación sin ambigüedades
```

**Precondiciones**

- Usuario autenticado.
- Escenarios con 2 o 3 proveedores disponibles.

**Datos de prueba**

- `proveedores = FedEx, DHL, Local`
- `prioridades = menor costo, menor tiempo`

**Pasos de ejecución**

1. Probar flujos con distintas prioridades.
2. Revisar orden, formato monetario, tiempos y selección.
3. Cambiar de proveedor alternativo y volver.
4. Registrar hallazgos.

**Resultado esperado**

- Los datos son consistentes con la API.
- La recomendación se diferencia de las alternativas.
- La selección de alternativa es clara y no genera estados duplicados.

---

## HU-05 — Seleccionar y confirmar proveedor

### TC-HU05-01 · Selección exitosa del proveedor recomendado

| Campo | Detalle |
|---|---|
| **ID** | TC-HU05-01 |
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado visualiza la recomendación y opciones disponibles
Cuando selecciona el proveedor recomendado
Entonces el sistema debe permitir continuar con el proceso
```

**Precondiciones**

- Usuario autenticado.
- Recomendación generada.
- Proveedor recomendado disponible.

**Datos de prueba**

- `proveedorSeleccionado = recomendado por el sistema`

**Pasos de ejecución**

1. Completar flujo de pedido y recomendación desde la UI.
2. Seleccionar proveedor recomendado.
3. Verificar que el sistema permite continuar a confirmación final.

**Resultado esperado**

- El sistema registra la selección del proveedor recomendado.
- Se permite continuar a confirmación final.

---

### TC-HU05-02 · Selección exitosa de proveedor alternativo

| Campo | Detalle |
|---|---|
| **ID** | TC-HU05-02 |
| **Prioridad** | Alto |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado visualiza recomendación y alternativas
Cuando selecciona un proveedor alternativo disponible
Entonces el sistema debe permitir continuar con el proceso
```

**Precondiciones**

- Usuario autenticado.
- Recomendación con alternativas disponibles.

**Datos de prueba**

- `proveedorSeleccionado = proveedor alternativo distinto al recomendado`

**Pasos de ejecución**

1. Completar flujo de recomendación desde la UI.
2. Seleccionar alternativa disponible.
3. Confirmar selección.

**Resultado esperado**

- El sistema registra la alternativa seleccionada.
- No se fuerza al usuario a confirmar solo el recomendado.

---

### TC-HU05-03 · Confirmación fallida por no seleccionar proveedor

| Campo | Detalle |
|---|---|
| **ID** | TC-HU05-03 |
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado visualiza opciones disponibles
Cuando intenta continuar sin seleccionar proveedor
Entonces el sistema no debe continuar
Y debe informar que debe seleccionar un proveedor
```

**Precondiciones**

- Token JWT válido.
- Recomendación generada.

**Datos de prueba**

- `proveedorSeleccionado = ninguno`

**Pasos de ejecución**

1. Enviar `POST /api/v1/pedido/confirmar` sin proveedor seleccionado.
2. Verificar código HTTP y mensaje.

**Resultado esperado**

- La API retorna HTTP 400.
- El mensaje indica que se debe seleccionar un proveedor.
- No se persiste pedido.

---

### TC-HU05-04 · Confirmación fallida por proveedor inválido o no disponible

| Campo | Detalle |
|---|---|
| **ID** | TC-HU05-04 |
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado tiene opciones disponibles
Cuando intenta confirmar un proveedor que no pertenece a las opciones calculadas
Entonces el sistema debe rechazar la confirmación
```

**Precondiciones**

- Token JWT válido.
- Recomendación generada.
- Proveedor no incluido en la respuesta.

**Datos de prueba**

- `proveedorSeleccionado = "ProveedorInexistente"`

**Pasos de ejecución**

1. Generar recomendación válida.
2. Enviar `POST /api/v1/pedido/confirmar` con proveedor inválido.
3. Verificar código y mensaje.

**Resultado esperado**

- La API retorna HTTP 400 o 422.
- El mensaje indica proveedor inválido o no disponible.
- No se persiste pedido.

---

### TC-HU05-05 · Persistencia del pedido asociada al usuario autenticado

| Campo | Detalle |
|---|---|
| **ID** | TC-HU05-05 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado selecciona y confirma proveedor
Cuando la confirmación es exitosa
Entonces el sistema debe persistir el pedido
Y debe asociarlo al usuario autenticado
```

**Precondiciones**

- Token JWT válido.
- Base de datos disponible.
- Pedido y recomendación generados.

**Datos de prueba**

- `origen = "Bogotá, Colombia"`
- `destino = "Medellín, Colombia"`
- `peso = 5 Kg`
- `prioridad = "MENOR_COSTO"`
- `proveedorSeleccionado = proveedor disponible`

**Pasos de ejecución**

1. Enviar `POST /api/v1/pedido/confirmar` con proveedor válido.
2. Verificar respuesta exitosa.
3. Consultar `GET /api/v1/pedido/mis-pedidos` con el mismo JWT.
4. Verificar que el pedido aparece con los datos correctos.

**Resultado esperado**

- El pedido queda persistido con origen, destino, peso, prioridad y proveedor seleccionado.
- El pedido queda asociado al usuario del JWT.
- Los datos persistidos coinciden con el flujo.

---

### TC-HU05-06 · Intento de confirmar proveedor sin autenticación

| Campo | Detalle |
|---|---|
| **ID** | TC-HU05-06 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que una persona no ha iniciado sesión
Cuando intenta confirmar un proveedor
Entonces el sistema debe bloquear el acceso
Y debe solicitar autenticación
```

**Precondiciones**

- No se envía JWT en la solicitud.

**Datos de prueba**

- `proveedorSeleccionado = "Local"`

**Pasos de ejecución**

1. Llamar `POST /api/v1/pedido/confirmar` sin token de autorización.
2. Verificar código HTTP.

**Resultado esperado**

- La API retorna HTTP 401 o 403.
- No se persiste pedido.

---

### TC-HU05-07 · Rendimiento de confirmación autenticada

| Campo | Detalle |
|---|---|
| **ID** | TC-HU05-07 |
| **Prioridad** | Alto |
| **Herramienta** | k6 |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que la confirmación de proveedor es funcionalmente estable
Cuando se ejecuta carga moderada autenticada
Entonces el endpoint debe responder dentro del umbral acordado
```

**Precondiciones**

- Usuarios / JWT de prueba disponibles.
- Datos de pedido controlados.
- Base de datos aislada o con limpieza posterior.

**Datos de prueba**

- 10 VU durante 3 minutos.
- `endpoint = POST /api/v1/pedido/confirmar`

**Pasos de ejecución**

1. Ejecutar script k6 de confirmación.
2. Validar checks de status y respuesta.
3. Analizar p95, p99 y tasa de error.
4. Limpiar o aislar datos generados.

**Resultado esperado**

- `p95 < 1500 ms`
- `http_req_failed < 1%`
- Checks funcionales por encima de 95%.

---

### TC-HU05-08 · Exploratoria de confirmación y navegación posterior

| Campo | Detalle |
|---|---|
| **ID** | TC-HU05-08 |
| **Prioridad** | Medio |
| **Herramienta** | Manual exploratoria |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario selecciona un proveedor
Cuando confirma, vuelve, reintenta o cambia la selección
Entonces el flujo no debe duplicar confirmaciones ni perder datos
```

**Precondiciones**

- Usuario autenticado.
- Pedido y recomendación generados.

**Datos de prueba**

- Recomendado y alternativa disponibles.

**Pasos de ejecución**

1. Seleccionar recomendado y cambiar a alternativa.
2. Confirmar y revisar navegación posterior.
3. Usar «volver» del navegador y reintentar.
4. Verificar duplicados en historial o base.

**Resultado esperado**

- No se generan confirmaciones duplicadas por reintentos simples.
- El proveedor final coincide con la selección del usuario.
- La navegación posterior es consistente.

---

# Microsprint 3

---

## HU-06 — Visualizar ruta del envío en el mapa

### TC-HU06-01 · Visualización de ruta con marcadores de origen y destino

| Campo | Detalle |
|---|---|
| **ID** | TC-HU06-01 |
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado ingresó origen y destino válidos
Y el sistema calculó la ruta
Cuando visualiza el mapa
Entonces debe ver la ruta y los marcadores de origen y destino
```

**Precondiciones**

- Usuario autenticado.
- Respuesta de ruta disponible desde OpenRouteService o stub.
- Leaflet renderizando en navegador.

**Datos de prueba**

- `origen = "Bogotá, Colombia"`
- `destino = "Medellín, Colombia"`
- Coordenadas válidas.

**Pasos de ejecución**

1. Completar flujo de pedido/recomendación con coordenadas válidas desde la UI.
2. Visualizar mapa en frontend.
3. Verificar que se muestra la polyline de la ruta.
4. Verificar marcadores de origen y destino.

**Resultado esperado**

- El mapa muestra la polyline de la ruta.
- Existe marcador de origen y de destino.
- La ruta corresponde a los puntos seleccionados.

---

### TC-HU06-02 · Ajuste automático del mapa a la ruta

| Campo | Detalle |
|---|---|
| **ID** | TC-HU06-02 |
| **Prioridad** | Alto |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el sistema dibujó la ruta en el mapa
Cuando la ruta es visible
Entonces el mapa debe ajustarse automáticamente para mostrar toda la ruta
```

**Precondiciones**

- Usuario autenticado.
- Mapa con ruta dibujada.
- Viewport de navegador controlado.

**Datos de prueba**

- Ruta Bogotá – Medellín u otra ruta válida.

**Pasos de ejecución**

1. Cargar pantalla con ruta.
2. Observar bounds y zoom inicial.
3. Cambiar tamaño de ventana si aplica.
4. Verificar que la ruta completa queda visible.

**Resultado esperado**

- Leaflet ajusta los bounds para mostrar toda la ruta.
- No se corta la polyline ni quedan marcadores fuera de vista.

---

### TC-HU06-03 · Conversión correcta de coordenadas de OpenRouteService a Leaflet

| Campo | Detalle |
|---|---|
| **ID** | TC-HU06-03 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el sistema recibe coordenadas en formato [lng, lat]
Cuando dibuja la ruta en Leaflet
Entonces debe convertirlas a formato [lat, lng]
Y visualizar la ruta correctamente
```

**Precondiciones**

- Stub o respuesta controlada de OpenRouteService.

**Datos de prueba**

- `entrada ORS = [-74.0721, 4.7110]`
- `salida Leaflet esperada = [4.7110, -74.0721]`

**Pasos de ejecución**

1. Preparar respuesta de ruta con coordenadas conocidas.
2. Validar transformación en el contrato de respuesta del backend.
3. Verificar que las coordenadas se entregan en el formato esperado por el frontend.

**Resultado esperado**

- Las coordenadas se invierten correctamente para Leaflet.
- Los marcadores quedan en posiciones esperadas.
- No se dibuja la ruta en ubicaciones incorrectas.

---

### TC-HU06-04 · Intento de visualización sin datos suficientes de ruta

| Campo | Detalle |
|---|---|
| **ID** | TC-HU06-04 |
| **Prioridad** | Alto |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado no tiene origen/destino completo o no existe ruta calculada
Cuando intenta visualizar el mapa
Entonces el sistema no debe mostrar ruta
Y debe informar que no hay datos suficientes
```

**Precondiciones**

- Usuario autenticado.
- Flujo sin coordenadas completas o respuesta sin geometría.

**Datos de prueba**

- `origen = vacío` o `ruta = null`

**Pasos de ejecución**

1. Preparar flujo sin datos suficientes desde la UI.
2. Abrir sección de mapa.
3. Verificar mensaje informativo en pantalla.

**Resultado esperado**

- No se renderiza una ruta falsa.
- El sistema muestra mensaje de datos insuficientes.
- No se rompe la pantalla.

---

### TC-HU06-05 · Manejo de error o respuesta vacía de OpenRouteService

| Campo | Detalle |
|---|---|
| **ID** | TC-HU06-05 |
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que OpenRouteService falla o responde sin ruta
Cuando el sistema intenta calcular la ruta estimada
Entonces debe manejar el error sin bloquear indebidamente el flujo principal
```

**Precondiciones**

- Stub de OpenRouteService con error 500, timeout o respuesta vacía.
- Token JWT válido.

**Datos de prueba**

- `respuesta ORS = error o features vacío`

**Pasos de ejecución**

1. Configurar stub con falla o respuesta vacía.
2. Ejecutar `POST /api/v1/pedido` con datos válidos.
3. Verificar respuesta del backend.
4. Confirmar que el flujo de cotización no falla por completo.

**Resultado esperado**

- El sistema informa ausencia de datos suficientes o error controlado.
- No hay excepción no manejada.
- El flujo de cotización no falla por completo si solo falta la ruta.

---

### TC-HU06-06 · Intento de visualizar ruta sin autenticación

| Campo | Detalle |
|---|---|
| **ID** | TC-HU06-06 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que una persona no ha iniciado sesión
Cuando intenta visualizar la ruta del envío
Entonces el sistema debe bloquear el acceso
Y debe solicitar autenticación
```

**Precondiciones**

- No se envía JWT en la solicitud.

**Datos de prueba**

- Ruta válida previamente conocida.

**Pasos de ejecución**

1. Llamar endpoint protegido sin token de autorización.
2. Verificar código HTTP.

**Resultado esperado**

- La API retorna HTTP 401 o 403.
- No se muestra ruta protegida.

---

### TC-HU06-07 · Exploratoria visual del mapa en navegador real

| Campo | Detalle |
|---|---|
| **ID** | TC-HU06-07 |
| **Prioridad** | Medio |
| **Herramienta** | Manual exploratoria |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado visualiza el mapa
Cuando cambia resoluciones, hace zoom, panea o recarga
Entonces el mapa debe mantenerse usable y consistente
```

**Precondiciones**

- Usuario autenticado.
- Navegador Chrome con DevTools.
- Ruta válida y caso sin ruta disponibles.

**Datos de prueba**

- Resoluciones desktop y mobile.
- Rutas corta y larga.

**Pasos de ejecución**

1. Validar mapa con diferentes tamaños de ventana.
2. Usar controles de zoom y desplazamiento.
3. Recargar la pantalla y volver desde historial.
4. Registrar artefactos visuales o errores de consola.

**Resultado esperado**

- El mapa se mantiene visible y usable.
- No aparecen errores de consola relevantes.
- El mensaje de datos insuficientes se mantiene claro y no ocupa el lugar de una ruta válida.

---

# Microsprint 4

---

## HU-07 — Registrar usuario

### TC-HU07-01 · Registro exitoso de usuario

| Campo | Detalle |
|---|---|
| **ID** | TC-HU07-01 |
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que una persona desea utilizar la plataforma
Cuando ingresa nombre, correo único y contraseña válida
Entonces el sistema debe registrar al usuario
Y debe dejar la cuenta disponible para iniciar sesión
```

**Precondiciones**

- Correo no existente en base.
- Formulario de registro disponible.

**Datos de prueba**

- `nombre = "Usuario QA"`
- `email = "qa.usuario.unico@example.com"`
- `password = "Password123"`

**Pasos de ejecución**

1. Abrir formulario de registro desde la UI.
2. Completar nombre, correo y contraseña.
3. Enviar formulario.
4. Verificar mensaje de éxito.
5. Intentar iniciar sesión con la cuenta creada.

**Resultado esperado**

- El registro se completa exitosamente.
- El usuario queda persistido.
- La cuenta puede iniciar sesión posteriormente.

---

### TC-HU07-02 · Registro fallido por correo duplicado

| Campo | Detalle |
|---|---|
| **ID** | TC-HU07-02 |
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que ya existe un usuario con un correo electrónico
Cuando otra persona intenta registrarse con ese mismo correo
Entonces el sistema no debe permitir el registro
Y debe informar que el correo está en uso
```

**Precondiciones**

- Usuario semilla existente.
- Endpoint de registro disponible.

**Datos de prueba**

- `email existente = "usuario.existente@example.com"`

**Pasos de ejecución**

1. Crear o asegurar usuario semilla vía `POST /api/users/register`.
2. Intentar registrar otro usuario con el mismo email.
3. Verificar código y mensaje.

**Resultado esperado**

- La API retorna HTTP 409 o validación equivalente.
- El mensaje informa que el correo ya se encuentra en uso.
- No se crea un segundo usuario con el mismo email.

---

### TC-HU07-03 · Registro fallido por campos obligatorios vacíos

| Campo | Detalle |
|---|---|
| **ID** | TC-HU07-03 |
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que una persona desea crear una cuenta
Cuando intenta registrarse sin nombre, correo o contraseña
Entonces el sistema no debe permitir el registro
Y debe informar que todos los campos son obligatorios
```

**Precondiciones**

- Endpoint de registro disponible.

**Datos de prueba**

- `nombre = ""`
- `email = ""`
- `password = ""`

**Pasos de ejecución**

1. Enviar `POST /api/users/register` sin campos requeridos.
2. Verificar código y mensajes.
3. Confirmar que no se crea usuario.

**Resultado esperado**

- La API retorna HTTP 400.
- El mensaje indica campos obligatorios.
- No se persiste usuario.

---

### TC-HU07-04 · Registro fallido por contraseña menor a 8 caracteres

| Campo | Detalle |
|---|---|
| **ID** | TC-HU07-04 |
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que una persona desea crear una cuenta
Cuando ingresa una contraseña con menos de 8 caracteres
Entonces el sistema no debe permitir el registro
Y debe informar que no cumple la regla mínima
```

**Precondiciones**

- Endpoint de registro disponible.
- Email único.

**Datos de prueba**

- `nombre = "Usuario QA"`
- `email = "qa.password.invalida@example.com"`
- `password = "abc123"`

**Pasos de ejecución**

1. Enviar `POST /api/users/register` con contraseña corta.
2. Verificar código y mensaje.
3. Confirmar que no se crea usuario.

**Resultado esperado**

- La API retorna HTTP 400.
- El mensaje indica longitud mínima de 8 caracteres.
- No se persiste usuario.

---

### TC-HU07-05 · Persistencia segura de contraseña cifrada

| Campo | Detalle |
|---|---|
| **ID** | TC-HU07-05 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que una persona se registra correctamente
Cuando el sistema persiste el usuario
Entonces la contraseña no debe quedar almacenada en texto plano
```

**Precondiciones**

- Acceso controlado a base de datos QA o verificación de capa de integración.
- Registro exitoso disponible.

**Datos de prueba**

- `password original = "Password123"`

**Pasos de ejecución**

1. Registrar usuario válido vía `POST /api/users/register`.
2. Consultar registro persistido de forma controlada o validar vía prueba de integración.
3. Verificar que el valor almacenado no coincide con la contraseña original.
4. Verificar que el login funciona con la contraseña original.

**Resultado esperado**

- La contraseña se almacena cifrada/hasheada.
- No coincide con el texto plano enviado.
- El login sigue validando correctamente las credenciales.

---

### TC-HU07-06 · Exploratoria del formulario de registro

| Campo | Detalle |
|---|---|
| **ID** | TC-HU07-06 |
| **Prioridad** | Medio |
| **Herramienta** | Manual exploratoria |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que una persona usa el formulario de registro
Cuando corrige errores, cambia foco o reintenta
Entonces la experiencia debe ser clara y no generar estados inconsistentes
```

**Precondiciones**

- Navegador disponible.
- Correos válidos, duplicados e inválidos preparados.

**Datos de prueba**

- Nombres con espacios, emails con mayúsculas, password límite de 8 caracteres.

**Pasos de ejecución**

1. Explorar validaciones cliente y servidor.
2. Probar corrección de campos tras errores.
3. Revisar feedback de éxito y redirección a login.
4. Registrar hallazgos.

**Resultado esperado**

- Los mensajes son claros.
- La redirección posterior al alta exitosa funciona.
- No se filtra la contraseña ni se mantienen errores corregidos.

---

## HU-08 — Iniciar sesión

### TC-HU08-01 · Inicio de sesión exitoso con JWT válido

| Campo | Detalle |
|---|---|
| **ID** | TC-HU08-01 |
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario ya está registrado
Cuando ingresa correo y contraseña válidos
Entonces el sistema debe permitir acceso
Y debe habilitar funcionalidades protegidas
```

**Precondiciones**

- Usuario registrado existente.
- Formulario de login disponible.

**Datos de prueba**

- `email = "usuario.qa@example.com"`
- `password = "Password123"`

**Pasos de ejecución**

1. Abrir formulario de login desde la UI.
2. Enviar credenciales válidas.
3. Verificar que se accede a la aplicación.
4. Acceder a una funcionalidad protegida.

**Resultado esperado**

- Login exitoso.
- El usuario accede a funcionalidades operativas.

---

### TC-HU08-02 · Inicio de sesión fallido por credenciales inválidas

| Campo | Detalle |
|---|---|
| **ID** | TC-HU08-02 |
| **Prioridad** | Alto |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario intenta acceder a la plataforma
Cuando ingresa correo o contraseña incorrectos
Entonces el sistema no debe permitir el acceso
Y debe informar credenciales inválidas
```

**Precondiciones**

- Usuario registrado existente.
- Endpoint de login disponible.

**Datos de prueba**

- `email = "usuario.qa@example.com"`
- `password = "Incorrecta123"`

**Pasos de ejecución**

1. Enviar `POST /api/users/login` con credenciales inválidas.
2. Verificar código y mensaje.
3. Verificar que no se genera token.

**Resultado esperado**

- La API retorna HTTP 401.
- El mensaje informa credenciales inválidas.
- No se genera JWT.

---

### TC-HU08-03 · Acceso a funcionalidad protegida sin autenticación

| Campo | Detalle |
|---|---|
| **ID** | TC-HU08-03 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que existe una funcionalidad operativa protegida
Cuando una persona intenta acceder sin sesión
Entonces el sistema debe bloquear el acceso
Y debe solicitar autenticación
```

**Precondiciones**

- No se envía JWT en la solicitud.

**Datos de prueba**

- `ruta protegida = pedido, recomendación, confirmación o historial`

**Pasos de ejecución**

1. Llamar endpoint protegido sin token de autorización.
2. Verificar código HTTP.

**Resultado esperado**

- La API retorna HTTP 401 o 403.
- No se exponen datos operativos.

---

### TC-HU08-04 · Cierre de sesión exitoso bloquea funcionalidades protegidas

| Campo | Detalle |
|---|---|
| **ID** | TC-HU08-04 |
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario ha iniciado sesión
Cuando cierra sesión
Entonces el sistema debe finalizar la sesión activa
Y bloquear funcionalidades protegidas hasta un nuevo login
```

**Precondiciones**

- Usuario autenticado.
- Opción de logout visible.

**Datos de prueba**

- Sesión activa con JWT válido.

**Pasos de ejecución**

1. Iniciar sesión desde la UI.
2. Ejecutar logout.
3. Intentar acceder a una ruta protegida.
4. Verificar que se solicita login nuevamente.

**Resultado esperado**

- El token o estado local se limpia.
- El usuario queda fuera de la aplicación protegida.
- La ruta protegida solicita login nuevamente.

---

### TC-HU08-05 · Rechazo de token inválido o expirado

| Campo | Detalle |
|---|---|
| **ID** | TC-HU08-05 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el endpoint protegido requiere JWT válido
Cuando se envía un token inválido, alterado o expirado
Entonces el sistema debe rechazar la solicitud
```

**Precondiciones**

- Endpoint protegido disponible.
- Token inválido preparado.

**Datos de prueba**

- `Authorization = "Bearer token-invalido"`

**Pasos de ejecución**

1. Llamar endpoint protegido con token inválido.
2. Repetir con token ausente si aplica.
3. Verificar código y mensaje.

**Resultado esperado**

- La API retorna HTTP 401 o 403.
- No se procesa la solicitud.
- No se exponen datos protegidos.

---

### TC-HU08-06 · Restauración de sesión al recargar la página

| Campo | Detalle |
|---|---|
| **ID** | TC-HU08-06 |
| **Prioridad** | Medio |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario inició sesión correctamente
Cuando recarga el navegador
Entonces el sistema debe restaurar la sesión válida o solicitar login si no existe token válido
```

**Precondiciones**

- Usuario autenticado.
- Persistencia de sesión frontend configurada.

**Datos de prueba**

- JWT válido almacenado por el frontend.

**Pasos de ejecución**

1. Iniciar sesión desde la UI.
2. Recargar página.
3. Verificar que la ruta protegida sigue disponible.
4. Repetir tras logout para validar bloqueo.

**Resultado esperado**

- Con token válido, la sesión se restaura correctamente.
- Tras logout, la sesión no se restaura y se solicita login.

---

### TC-HU08-07 · Rendimiento de login controlado

| Campo | Detalle |
|---|---|
| **ID** | TC-HU08-07 |
| **Prioridad** | Medio |
| **Herramienta** | k6 |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que existen usuarios de prueba precreados
Cuando se ejecuta carga moderada de login
Entonces el servicio debe responder dentro del umbral sin crear usuarios masivos
```

**Precondiciones**

- Usuarios de prueba precreados.
- Base de datos estable.

**Datos de prueba**

- 5 a 10 VU durante 3 minutos.
- `endpoint = POST /api/users/login`

**Pasos de ejecución**

1. Ejecutar script k6 de login.
2. Validar checks de status y presencia de token.
3. Analizar p95 y tasa de error.

**Resultado esperado**

- `p95 < 1000 ms`
- `http_req_failed < 1%`
- Checks funcionales por encima de 95%.

---

### TC-HU08-08 · Exploratoria de autenticación y logout

| Campo | Detalle |
|---|---|
| **ID** | TC-HU08-08 |
| **Prioridad** | Medio |
| **Herramienta** | Manual exploratoria |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario interactúa con login, rutas protegidas y logout
Cuando alterna sesiones, recarga o usa múltiples pestañas
Entonces la aplicación debe mantener seguridad y claridad de estado
```

**Precondiciones**

- Usuario registrado.
- Navegador con múltiples pestañas disponible.

**Datos de prueba**

- Sesión válida, sesión cerrada, credenciales inválidas.

**Pasos de ejecución**

1. Abrir múltiples pestañas autenticadas.
2. Cerrar sesión en una pestaña y validar el resto.
3. Probar botones atrás/adelante tras logout.
4. Registrar hallazgos.

**Resultado esperado**

- No se permite acceder a rutas protegidas tras logout.
- Los mensajes de sesión son claros.
- No quedan datos sensibles visibles indebidamente.

---

## HU-09 — Consultar pedidos del usuario

### TC-HU09-01 · Visualización de pedidos del usuario autenticado

| Campo | Detalle |
|---|---|
| **ID** | TC-HU09-01 |
| **Prioridad** | Crítico |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario ha iniciado sesión
Y existen pedidos asociados a su cuenta
Cuando consulta su listado de pedidos
Entonces el sistema debe mostrar solo sus pedidos con datos mínimos requeridos
```

**Precondiciones**

- Usuario autenticado con pedidos confirmados.
- Pantalla de historial disponible.

**Datos de prueba**

- Pedidos con origen, destino, peso, prioridad, proveedor seleccionado.

**Pasos de ejecución**

1. Confirmar uno o más pedidos para el usuario.
2. Abrir historial desde la UI.
3. Verificar que se muestran los pedidos con datos completos.

**Resultado esperado**

- La pantalla muestra solo pedidos del usuario autenticado.
- Cada pedido incluye origen, destino, peso, prioridad y proveedor seleccionado.

---

### TC-HU09-02 · Usuario autenticado sin pedidos registrados

| Campo | Detalle |
|---|---|
| **ID** | TC-HU09-02 |
| **Prioridad** | Alto |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario ha iniciado sesión
Y no existen pedidos asociados a su cuenta
Cuando consulta su listado de pedidos
Entonces el sistema debe informar que no existen pedidos registrados
```

**Precondiciones**

- Usuario autenticado nuevo o sin pedidos.
- Pantalla de historial disponible.

**Datos de prueba**

- `usuarioSinPedidos = true`

**Pasos de ejecución**

1. Iniciar sesión con usuario sin pedidos.
2. Abrir historial desde la UI.
3. Verificar mensaje informativo.

**Resultado esperado**

- La UI informa que no existen pedidos registrados para el usuario.
- No se muestra error técnico.

---

### TC-HU09-03 · Aislamiento de pedidos entre usuarios

| Campo | Detalle |
|---|---|
| **ID** | TC-HU09-03 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que existen pedidos asociados a diferentes usuarios
Cuando un usuario autenticado consulta su historial
Entonces no debe ver pedidos de otras cuentas
```

**Precondiciones**

- Usuario A con pedidos.
- Usuario B con pedidos.
- JWT de cada usuario disponible.

**Datos de prueba**

- `usuarioA`, `usuarioB`, pedidos diferenciados.

**Pasos de ejecución**

1. Crear pedidos para usuario A y usuario B.
2. Consultar `GET /api/v1/pedido/mis-pedidos` con JWT de usuario A.
3. Consultar `GET /api/v1/pedido/mis-pedidos` con JWT de usuario B.
4. Comparar resultados.

**Resultado esperado**

- Usuario A solo ve pedidos de A.
- Usuario B solo ve pedidos de B.
- No hay filtración de datos entre cuentas.

---

### TC-HU09-04 · Intento de consultar pedidos sin autenticación

| Campo | Detalle |
|---|---|
| **ID** | TC-HU09-04 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que una persona no ha iniciado sesión
Cuando intenta consultar pedidos de usuario
Entonces el sistema debe bloquear el acceso
Y debe solicitar autenticación
```

**Precondiciones**

- No se envía JWT en la solicitud.

**Datos de prueba**

- `endpoint = GET /api/v1/pedido/mis-pedidos`

**Pasos de ejecución**

1. Llamar `GET /api/v1/pedido/mis-pedidos` sin token de autorización.
2. Verificar código HTTP.

**Resultado esperado**

- La API retorna HTTP 401 o 403.
- No se devuelven pedidos.

---

### TC-HU09-05 · La consulta no acepta userId enviado desde frontend

| Campo | Detalle |
|---|---|
| **ID** | TC-HU09-05 |
| **Prioridad** | Crítico |
| **Herramienta** | Karate DSL |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que la propiedad del pedido se obtiene desde el JWT
Cuando el cliente intenta forzar un userId distinto en la consulta
Entonces el sistema debe ignorarlo o rechazarlo
Y debe filtrar por el usuario autenticado
```

**Precondiciones**

- Usuario A autenticado.
- Usuario B con pedidos existentes.
- Endpoint protegido disponible.

**Datos de prueba**

- `JWT = usuario A`
- `query/body userId = usuario B`

**Pasos de ejecución**

1. Llamar `GET /api/v1/pedido/mis-pedidos` con JWT de usuario A.
2. Agregar parámetro o payload con userId de usuario B.
3. Verificar respuesta.

**Resultado esperado**

- El sistema no muestra pedidos de usuario B.
- El filtro efectivo se basa en el JWT.
- Si el contrato no admite userId, la solicitud se rechaza o se ignora sin exponer datos.

---

### TC-HU09-06 · Limpieza de historial al cerrar sesión

| Campo | Detalle |
|---|---|
| **ID** | TC-HU09-06 |
| **Prioridad** | Alto |
| **Herramienta** | Serenity Screenplay |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario autenticado visualiza su historial
Cuando cierra sesión
Entonces el estado de historial debe limpiarse
Y no debe quedar visible para otra sesión
```

**Precondiciones**

- Usuario A autenticado con historial visible.
- Logout disponible.

**Datos de prueba**

- `usuarioA` con pedidos.
- `usuarioB` sin pedidos o con pedidos distintos.

**Pasos de ejecución**

1. Iniciar sesión con usuario A y abrir historial.
2. Cerrar sesión.
3. Iniciar sesión con usuario B.
4. Revisar historial visible.

**Resultado esperado**

- Los pedidos de usuario A no quedan en pantalla ni en estado compartido.
- Usuario B ve solo sus propios pedidos o estado vacío.

---

### TC-HU09-07 · Rendimiento de historial autenticado

| Campo | Detalle |
|---|---|
| **ID** | TC-HU09-07 |
| **Prioridad** | Alto |
| **Herramienta** | k6 |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que existen usuarios autenticados con datos de prueba
Cuando se ejecuta carga sobre la consulta de historial
Entonces el endpoint debe responder dentro del umbral acordado
```

**Precondiciones**

- Usuarios / JWT precreados.
- Pedidos semilla controlados.
- Base de datos estable.

**Datos de prueba**

- 10 a 20 VU durante 5 minutos.
- `endpoint = GET /api/v1/pedido/mis-pedidos`

**Pasos de ejecución**

1. Ejecutar script k6 de historial.
2. Validar checks de status y estructura de lista.
3. Analizar p95, p99 y tasa de error.

**Resultado esperado**

- `p95 < 800 ms`
- `http_req_failed < 1%`
- Checks funcionales por encima de 95%.

---

### TC-HU09-08 · Exploratoria de historial y estados de pantalla

| Campo | Detalle |
|---|---|
| **ID** | TC-HU09-08 |
| **Prioridad** | Medio |
| **Herramienta** | Manual exploratoria |
| **Estado** | Sin ejecutar |

**Escenario**

```gherkin
Dado que el usuario consulta su historial
Cuando navega entre estados con pedidos, sin pedidos, error y recarga
Entonces la pantalla debe comunicar claramente cada estado
```

**Precondiciones**

- Usuarios con y sin pedidos.
- Capacidad de simular error de API o desconexión temporal.

**Datos de prueba**

- Historial con 1 pedido, varios pedidos, vacío y error controlado.

**Pasos de ejecución**

1. Revisar estado de carga.
2. Validar historial con uno y varios pedidos.
3. Validar estado vacío.
4. Simular error de API y recargar.
5. Registrar hallazgos.

**Resultado esperado**

- Los estados de carga, vacío y error son claros.
- Los datos mínimos se muestran completos.
- No se mezclan pedidos entre usuarios tras recarga o cambio de sesión.

---

# Resumen de cobertura

## Cobertura por microsprint

| Microsprint | Historias | Casos | Foco principal |
|---|---|---:|---|
| Microsprint 1 | HU-01, HU-02 | 18 | Registro de pedido, autocompletado, cobertura Colombia, peso, prioridad y autenticación obligatoria. |
| Microsprint 2 | HU-03, HU-04, HU-05 | 21 | Recomendación, reglas de desempate, alternativas, selección, confirmación, persistencia y rendimiento. |
| Microsprint 3 | HU-06 | 7 | Ruta en mapa, OpenRouteService, conversión de coordenadas, marcadores, ajuste automático y manejo de ausencia de datos. |
| Microsprint 4 | HU-07, HU-08, HU-09 | 22 | Registro, login, JWT, logout, rutas protegidas, historial propio, aislamiento entre usuarios y rendimiento. |
| **Total** | **HU-01 a HU-09** | **68** | **MVP base + MVP v2 completo** |

## Cobertura por historia de usuario

| ID | Historia de Usuario | Casos | Serenity Screenplay | Karate DSL | Manual exploratoria | k6 |
|---|---|---:|---:|---:|---:|---:|
| HU-01 | Registrar pedido de envío | 13 | 2 | 9 | 1 | 1 |
| HU-02 | Definir prioridad del envío | 5 | 2 | 2 | 1 | 0 |
| HU-03 | Obtener recomendación principal | 8 | 0 | 6 | 1 | 1 |
| HU-04 | Obtener alternativas de proveedores | 5 | 2 | 2 | 1 | 0 |
| HU-05 | Seleccionar y confirmar proveedor | 8 | 2 | 4 | 1 | 1 |
| HU-06 | Visualizar ruta en el mapa | 7 | 3 | 3 | 1 | 0 |
| HU-07 | Registrar usuario | 6 | 1 | 4 | 1 | 0 |
| HU-08 | Iniciar sesión | 8 | 3 | 3 | 1 | 1 |
| HU-09 | Consultar pedidos del usuario | 8 | 3 | 3 | 1 | 1 |
| **Total** | **MVP base + MVP v2** | **68** | **18** | **36** | **9** | **5** |

> **Nota:** Cada caso de prueba está asignado a una sola herramienta según su foco principal. Los flujos E2E de usuario se validan con Serenity Screenplay. Las validaciones de API, contratos, reglas de negocio y seguridad se automatizan con Karate DSL. Las pruebas exploratorias cubren UX y estados no cubiertos por scripts. k6 se reserva para escenarios de rendimiento: smoke de pedido, carga de recomendación, confirmación, login e historial autenticado.

## Cobertura de reglas del PRD

| Reglas PRD | HU cubiertas | Casos principales |
|---|---|---|
| Reglas 1–4 | HU-01, HU-02 | TC-HU01-01 a TC-HU01-13, TC-HU02-01 a TC-HU02-05 |
| Reglas 5–8 | HU-03 | TC-HU03-01 a TC-HU03-08 |
| Regla 9 | HU-04 | TC-HU04-01 a TC-HU04-05 |
| Reglas 10–11 y 24 | HU-05, HU-09 | TC-HU05-01 a TC-HU05-08, TC-HU09-01, TC-HU09-03, TC-HU09-05 |
| Reglas 12–16 | HU-06 | TC-HU06-01 a TC-HU06-07 |
| Reglas 17–23 y 28 | HU-07, HU-08 | TC-HU07-01 a TC-HU07-06, TC-HU08-01 a TC-HU08-08 |
| Reglas 25–27 | HU-09 | TC-HU09-01 a TC-HU09-08 |
