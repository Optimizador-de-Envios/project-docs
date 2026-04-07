# Subtasks - Optimizador de Envíos

---

# HU-01 | Registrar pedido de envío

## Objetivo de la historia
Permitirle al usuario agregar pedido para calcular y obtener una recomendación sobre cual proveedor de transporte es más adecuado de acuerdo a sus necesidades

## Subtareas DEV

### Backend
- **HU01-T01** | Definir el modelo Pedido y los DTOs necesarios para el registro del envío.
- **HU01-T02** | Implementar validaciones de campos obligatorios, rango de peso y cobertura en Colombia.
- **HU01-T03** | Implementar conversión de unidades de peso y normalización a kilogramos.
- **HU01-T04** | Integrar un servicio externo de geolocalización para autocompletado de ubicaciones y cálculo de distancia (*openRouteService*).
- **HU01-T05** | Implementar endpoints `GET /locations/autocomplete` y `POST /pedido`.
- **HU01-T06** | Realizar pruebas unitarias y de cobertura para las capas del backend.

### Frontend
- **HU01-T07** | Implementar formulario de registro con origen, destino, peso y unidad.
- **HU01-T08** | Implementar autocompletado para origen y destino consumiendo el endpoint correspondiente.
- **HU01-T09** | Validar campos obligatorios y formato del peso.
- **HU01-T10** | Consumir endpoint `POST /pedido` y almacenar la información en el estado global.
- **HU01-T11** | Permitir la navegación al siguiente paso del flujo.
- **HU01-T12** | Realizar pruebas unitarias y de cobertura.

## Subtareas QA

### Análisis y diseño
- **HU01-T13 (QA)** | Revisar que la HU, reglas de negocio y criterios de aceptación sean claros y testeables.
- **HU01-T14 (QA)** | Diseñar casos de prueba para registro exitoso, validación de campos, peso fuera de rango y cobertura geográfica.

### Validación técnica y funcional
- **HU01-T15 (QA)** | Verificar el funcionamiento de los endpoints `GET /locations/autocomplete` y `POST /pedido`.
- **HU01-T16 (QA)** | Validar el autocompletado y la restricción de ubicaciones dentro de Colombia.
- **HU01-T17 (QA)** | Verificar la conversión de unidades y cálculo de distancia.
- **HU01-T18 (QA)** | Validar el flujo completo desde frontend, incluyendo almacenamiento en estado global.
- **HU01-T19 (QA)** | Ejecutar pruebas funcionales y registrar hallazgos.

## Estimación: 5 puntos

### Justificación:
- **DEV:** Esfuerzo medio. Se implementa el registro del pedido con validaciones de datos (origen, destino y peso), construcción de DTOs y endpoint. La complejidad radica en asegurar la consistencia de los datos y su correcto guardado en el estado global. En el frontend, se desarrolla el formulario con sus correspondientes validaciones.
- **QA**: Esfuerzo medio. Se deben cubrir casos funcionales del registro del pedido, validar campos obligatorios y verificar la correcta respuesta del backend. Los escenarios son bastante simples, por lo que no requiere pruebas complejas.

---

# HU-02 | Definir prioridad del envío

## Objetivo de la historia
Permitir al usuario seleccionar el criterio de optimización del envío para que el sistema guarde su preferencia y la utilice en el cálculo de la recomendación.

## Subtareas DEV

### Backend
- **HU02-T01** | Extender el modelo Pedido para incluir el atributo prioridad.
- **HU02-T02** | Crear y ajustar los DTOs necesarios para manejar la prioridad.
- **HU02-T03** | Implementar validación de prioridad obligatoria.
- **HU02-T04** | Asociar la prioridad al pedido en memoria.
- **HU02-T05** | Exponer el endpoint POST /pedidos/prioridad.
- **HU02-T06** | Realizar pruebas unitarias y de cobertura.

### Frontend
- **HU02-T07** | Implementar pantalla de selección de prioridad (económico o rápido).
- **HU02-T08** | Validar que el usuario seleccione una opción antes de continuar.
- **HU02-T09** | Guardar la prioridad en el estado global.
- **HU02-T10** | Consumir el endpoint POST /pedidos/prioridad.
- **HU02-T11** | Manejar errores enviados por el backend.
- **HU02-T12** | Realizar pruebas unitarias y de cobertura.


## Subtareas QA

### Análisis y diseño
- **HU02-T13 (QA)** | Revisar que la HU, reglas de negocio y criterios de aceptación sean claros y testeables.
- **HU02-T14 (QA)** | Diseñar casos de prueba para selección de prioridad y validación de obligatoriedad.

### Validación técnica y funcional
- **HU02-T15 (QA)** | Verificar el funcionamiento del endpoint POST /pedidos/prioridad.
- **HU02-T16 (QA)** | Validar que la prioridad se guarde correctamente en memoria.
- **HU02-T17 (QA)** | Verificar la selección de prioridad en frontend y su almacenamiento en el estado global.
- **HU02-T18 (QA)** | Validar el flujo completo de selección de prioridad y manejo de errores.
- **HU02-T19 (QA)** | Ejecutar pruebas funcionales y registrar hallazgos.

## Estimación: 3 puntos

### Justificación:
- **DEV:** Esfuerzo bajo. Se actualiza el modelo del pedido para incluir la prioridad y se implementa un endpoint para actualizar esta información. En el frontend, se desarrolla una pantalla sencilla para seleccionar la prioridad, con validación de selección y almacenamiento en el estado global.
- **QA**: Esfuerzo bajo. Se deben validar casos funcionales de selección de prioridad y obligatoriedad. El proceso es bastante directo, por lo que no requiere pruebas complejas.

---

# HU-03 | Obtener recomendación principal de proveedor de envío

## Objetivo de la historia
Permitir al usuario obtener la mejor opción de envío según la prioridad seleccionada (menor costo o menor tiempo de entrega).

## Subtareas DEV

### Backend
- **HU03-T01** | Crear los DTOs necesarios para solicitar y devolver la recomendación.
- **HU03-T02** | Configurar los proveedores mock y sus valores de cotización.
- **HU03-T03** | Implementar la integración simulada con los proveedores mediante el patrón adapter.
- **HU03-T04** | Implementar el servicio de cotización para consultar todos los proveedores disponibles.
- **HU03-T05** | Implementar el motor de recomendación según prioridad y reglas de desempate utilizando los patrones strategy y factory.
- **HU03-T06** | Exponer el endpoint `POST /recomendacion`.
- **HU03-T07** | Realizar pruebas de cobertura e integración.

### Frontend
- **HU03-T08** | Crear la pantalla de resultados con la recomendación principal destacada.
- **HU03-T09** | Consumir el endpoint `POST /recomendacion` utilizando los datos almacenados en el estado global.
- **HU03-T10** | Mostrar la recomendación principal en la interfaz.
- **HU03-T11** | Agregar la navegación para seleccionar el proveedor recomendado.
- **HU03-T12** | Realizar pruebas unitarias y de cobertura.

## Subtareas QA

### Análisis y diseño
- **HU03-T13 (QA)** | Revisar que la HU, las reglas de negocio y los criterios de aceptación sean claros y testeables.
- **HU03-T14 (QA)** | Diseñar casos de prueba para recomendación por menor costo, menor tiempo y reglas de desempate.

### Validación técnica y funcional
- **HU03-T15 (QA)** | Verificar el funcionamiento del endpoint `POST /recomendacion`.
- **HU03-T16 (QA)** | Validar que la recomendación principal corresponda a la prioridad seleccionada.
- **HU03-T17 (QA)** | Verificar la correcta aplicación de las reglas de desempate.
- **HU03-T18 (QA)** | Validar la pantalla de resultados y la visualización de la recomendación principal.
- **HU03-T19 (QA)** | Ejecutar pruebas funcionales del flujo de recomendación y registrar hallazgos.

## Estimación: 8 puntos

### Justificación:
- **DEV:** Esfuerzo alto. Se implementa la lógica de integración con proveedores de envío, el motor de recomendación y el endpoint para obtener la recomendación principal, utilizando los patrones strategy y factory. En el frontend, se desarrolla la pantalla de resultados y se consume el endpoint para mostrar la recomendación al usuario.
- **QA**: Esfuerzo medio-alto. Se deben validar casos funcionales de recomendación según prioridad y reglas de desempate. El proceso es más complejo debido a la lógica de recomendación, por lo que requiere pruebas detalladas para asegurar la correcta aplicación de las reglas.

---

# HU-04 | Consultar opciones alternativas

## Objetivo de la historia
Permitir al usuario visualizar opciones alternativas de proveedores de transporte para compararlas con la recomendación principal y elegir la opción más conveniente.

## Subtareas DEV

### Backend
- **HU04-T01** | Reutilizar `RecomendacionResponse` y `CotizacionResponse` para exponer las opciones alternativas junto con la recomendación principal.

> **Nota:** Esta HU no requiere cambios significativos en el backend, ya que se reutilizan los DTOs existentes para incluir las opciones alternativas en la respuesta del endpoint de recomendación.

### Frontend
- **HU04-T02** | Mostrar la recomendación principal destacada.
- **HU04-T03** | Mostrar la lista de opciones alternativas de forma separada de la recomendación principal.
- **HU04-T04** | Mostrar en cada alternativa la información relevante para la comparación:
  - proveedor
  - costo
  - tiempo de entrega
- **HU04-T05** | Permitir la selección de un proveedor alternativo desde la interfaz.
- **HU04-T06** | Agregar acción para seleccionar cualquiera de las opciones alternativas.
- **HU04-T07** | Realizar pruebas unitarias y de cobertura.

## Subtareas QA

### Análisis y diseño
- **HU04-T08 (QA)** | Revisar que la HU, las reglas de negocio y los criterios de aceptación sean claros y testeables.
- **HU04-T09 (QA)** | Diseñar casos de prueba para visualización de alternativas y ausencia de alternativas.

### Validación técnica y funcional
- **HU04-T10 (QA)** | Verificar que las opciones alternativas se muestren separadas de la recomendación principal.
- **HU04-T11 (QA)** | Validar que cada alternativa muestre proveedor, costo y tiempo de entrega.
- **HU04-T12 (QA)** | Verificar que la recomendación principal no se repita dentro de la lista de alternativas.
- **HU04-T13 (QA)** | Validar la selección de una opción alternativa desde la interfaz.
- **HU04-T14 (QA)** | Ejecutar pruebas funcionales y registrar hallazgos.

## Estimación: 3 puntos

### Justificación:
- **DEV:** Esfuerzo medio-bajo. Se reutilizan los DTOs existentes para mostrar las opciones alternativas en el frontend, con una separación clara de la recomendación principal. Se implementa la lógica para mostrar la información relevante de cada alternativa y permitir su selección.
- **QA**: Esfuerzo bajo. Se deben validar casos funcionales de visualización de alternativas y selección de una opción alternativa. El proceso es bastante directo, por lo que no requiere pruebas complejas.

---

# HU-05 | Seleccionar y confirmar proveedor

## Objetivo de la historia
Permitir al usuario seleccionar un proveedor de envío y confirmar su selección para completar el proceso de pedido.

## Subtareas DEV

### Backend
- **HU05-T01** | Crear el modelo y los DTOs necesarios para la confirmación del pedido.
- **HU05-T02** | Implementar las validaciones de campos obligatorios y proveedor seleccionado.
- **HU05-T03** | Recalcular cotizaciones y construir la entidad final del pedido.
- **HU05-T04** | Implementar la persistencia del pedido mediante repositorio.
- **HU05-T05** | Exponer el endpoint `POST /pedidos/confirmar`.
- **HU05-T06** | Realizar pruebas unitarias y de integración.

### Frontend
- **HU05-T07** | Implementar la opción para confirmar el proveedor seleccionado.
- **HU05-T08** | Consumir el endpoint `POST /pedidos/confirmar` enviando los datos necesarios.
- **HU05-T09** | Redirigir a la pantalla de confirmación cuando la operación sea exitosa.
- **HU05-T10** | Manejar errores enviados por el backend.
- **HU05-T11** | Realizar pruebas unitarias y de cobertura.

## Subtareas QA

### Análisis y diseño
- **HU05-T12 (QA)** | Revisar que la HU, las reglas de negocio y los criterios de aceptación sean claros y testeables.
- **HU05-T13 (QA)** | Diseñar casos de prueba para confirmación exitosa, ausencia de selección, proveedor inválido y persistencia del pedido.

### Validación técnica y funcional
- **HU05-T14 (QA)** | Verificar el funcionamiento del endpoint `POST /pedidos/confirmar`.
- **HU05-T15 (QA)** | Validar la persistencia correcta del pedido y la respuesta devuelta por el sistema.
- **HU05-T16 (QA)** | Verificar la selección y confirmación del proveedor en frontend, incluyendo navegación y manejo de errores.
- **HU05-T17 (QA)** | Ejecutar pruebas funcionales del flujo completo de confirmación.
- **HU05-T18 (QA)** | Registrar hallazgos y validar el cumplimiento de los criterios de aceptación.

## Estimación: 5 puntos
### Justificación:
- **DEV:** Esfuerzo medio. Se implementa la lógica para confirmar el pedido, incluyendo validaciones, recalculo de cotizaciones y persistencia. En el frontend, se desarrolla la funcionalidad para confirmar la selección del proveedor y manejar la navegación y errores.
- **QA**: Esfuerzo medio. Se deben validar casos funcionales de confirmación del pedido, incluyendo validación de selección, manejo de errores y persistencia. El proceso es más complejo debido a la confirmación del pedido, por lo que requiere pruebas detalladas para asegurar la correcta aplicación de las reglas y la persistencia de los datos.

---

# HU-06 | Visualizar ruta del envío en el mapa

## Objetivo de la historia
Permitir al usuario visualizar en el mapa la ruta estimada entre el origen y el destino del envío, usando la geometría devuelta por el flujo de recomendación y respetando el formato requerido por Leaflet.

## Subtareas DEV

### Frontend
- **HU06-T07** | Implementar el componente de mapa con Leaflet para mostrar la ruta del envío.
- **HU06-T08** | Dibujar la polyline de la ruta y los marcadores de origen y destino.
- **HU06-T09** | Ajustar automáticamente los bounds y el zoom del mapa para mostrar toda la ruta.
- **HU06-T10** | Mostrar el estado vacío cuando no existan datos suficientes para renderizar el mapa.
- **HU06-T11** | Integrar el mapa dentro de la vista donde se muestra la recomendación o confirmación del envío.
- **HU06-T12** | Realizar pruebas unitarias y de cobertura para el componente y su orquestación.

## Subtareas QA

### Análisis y diseño
- **HU06-T13 (QA)** | Revisar que la historia, reglas de negocio y criterios de aceptación sean claros y testeables.
- **HU06-T14 (QA)** | Diseñar casos de prueba para visualización de ruta, ajuste automático, conversión de coordenadas y ausencia de datos.

### Validación técnica y funcional
- **HU06-T15 (QA)** | Verificar el contrato de ruta expuesto por el backend y su comportamiento ante coordenadas válidas e inválidas.
- **HU06-T16 (QA)** | Validar en frontend el dibujo correcto de la ruta, los marcadores y el ajuste automático del mapa.
- **HU06-T17 (QA)** | Verificar el manejo correcto de errores o escenarios sin datos suficientes.
- **HU06-T18 (QA)** | Validar la conversión de coordenadas y la consistencia visual en distintas resoluciones.
- **HU06-T19 (QA)** | Ejecutar pruebas funcionales y registrar hallazgos.

## Estimación: 5 puntos

### Justificación:
- **DEV:** Esfuerzo medio-alto. Se requiere integrar un servicio externo de rutas, normalizar la geometría y adaptar el frontend con Leaflet para mostrar la ruta y los marcadores. Aunque el alcance es acotado, depende de manejo correcto de coordenadas y de una respuesta consistente para el mapa.
- **QA**: Esfuerzo medio. Se deben validar escenarios de visualización correcta, ajuste del mapa, conversión de coordenadas y manejo de ausencia de datos. La complejidad está más en la validación visual y de integración que en la lógica de negocio.

---

# HU-07 | Registrar usuario

## Objetivo de la historia
Permitir que una persona cree una cuenta en el user-service con nombre, email y contraseña válida para habilitar el inicio de sesión y la asociación futura de pedidos.

## Subtareas DEV

### Backend
- **HU07-T01** | Crear los DTOs necesarios para el registro de usuario con validaciones de entrada.
- **HU07-T02** | Implementar la entidad Usuario con unicidad por email y persistencia en la base de datos del user-service.
- **HU07-T03** | Validar campos obligatorios y la longitud mínima de la contraseña.
- **HU07-T04** | Implementar el hash de contraseña con BCrypt antes de persistir el usuario.
- **HU07-T05** | Exponer el endpoint `POST /api/users/register` conforme al contrato definido.
- **HU07-T06** | Mapear errores de validación y conflicto de email duplicado a respuestas `400` y `409`.
- **HU07-T07** | Realizar pruebas unitarias e integración sobre el caso de uso y el endpoint.

### Frontend
- **HU07-T08** | Implementar el formulario de registro con nombre, correo y contraseña.
- **HU07-T09** | Validar en UI los campos obligatorios y la regla mínima de la contraseña.
- **HU07-T10** | Consumir el `authApiService` para registrar el usuario.
- **HU07-T11** | Mostrar feedback de éxito y redirigir al login después de registrar la cuenta.
- **HU07-T12** | Realizar pruebas unitarias y de cobertura para el formulario y su flujo.

## Subtareas QA

### Análisis y diseño
- **HU07-T13 (QA)** | Revisar que la historia, reglas de negocio y criterios de aceptación sean claros y testeables.
- **HU07-T14 (QA)** | Diseñar casos de prueba para registro exitoso, email duplicado, campos vacíos y contraseña inválida.

### Validación técnica y funcional
- **HU07-T15 (QA)** | Verificar el funcionamiento del endpoint `POST /api/users/register`.
- **HU07-T16 (QA)** | Validar que el sistema rechace emails duplicados y campos obligatorios vacíos.
- **HU07-T17 (QA)** | Verificar que la contraseña cumpla la longitud mínima y se persista cifrada.
- **HU07-T18 (QA)** | Validar el formulario de registro y la navegación posterior al alta exitosa.
- **HU07-T19 (QA)** | Ejecutar pruebas funcionales y registrar hallazgos.

## Estimación: 5 puntos

### Justificación:
- **DEV:** Esfuerzo medio. Aunque el flujo es directo, implica modelar el dominio de usuario, aplicar validaciones, cifrar la contraseña y exponer un contrato estable para registro. En frontend, el formulario y sus validaciones también requieren coordinación con el contrato del backend.
- **QA**: Esfuerzo medio. Los escenarios son claros, pero hay varios puntos a validar: unicidad del email, obligatorios, longitud mínima y respuesta del sistema ante errores. No es una historia trivial porque toca persistencia y seguridad básica.

---

# HU-08 | Iniciar sesión

## Objetivo de la historia
Permitir que un usuario registrado inicie sesión y reciba un JWT compatible con el shipment-service para acceder a funcionalidades privadas.

## Subtareas DEV

### Backend
- **HU08-T01** | Crear los DTOs necesarios para el login y la respuesta de autenticación.
- **HU08-T02** | Implementar el caso de uso de inicio de sesión validando credenciales contra el usuario persistido.
- **HU08-T03** | Implementar la emisión del JWT con los claims mínimos definidos en el contrato compartido.
- **HU08-T04** | Exponer el endpoint `POST /api/users/login` conforme al contrato definido.
- **HU08-T05** | Mapear errores de payload inválido y credenciales inválidas a respuestas `400` y `401`.
- **HU08-T06** | Realizar pruebas unitarias e integración sobre autenticación y emisión del token.

### Frontend
- **HU08-T07** | Implementar el formulario de login con correo y contraseña.
- **HU08-T08** | Persistir la sesión autenticada en el auth store y restaurarla al recargar la página.
- **HU08-T09** | Consumir el `authApiService` para iniciar sesión y manejar la respuesta JWT.
- **HU08-T10** | Proteger rutas y redirigir al usuario cuando no exista una sesión válida.
- **HU08-T11** | Manejar el estado de error para credenciales inválidas y limpiar la sesión al logout.
- **HU08-T12** | Realizar pruebas unitarias y de cobertura para el flujo de login y protección de rutas.

## Subtareas QA

### Análisis y diseño
- **HU08-T13 (QA)** | Revisar que la historia, reglas de negocio y criterios de aceptación sean claros y testeables.
- **HU08-T14 (QA)** | Diseñar casos de prueba para login exitoso, credenciales inválidas y acceso a rutas protegidas.

### Validación técnica y funcional
- **HU08-T15 (QA)** | Verificar el funcionamiento del endpoint `POST /api/users/login`.
- **HU08-T16 (QA)** | Validar que el JWT devuelto cumpla el contrato de claims y expiración definida.
- **HU08-T17 (QA)** | Verificar la protección de rutas privadas y el comportamiento ante ausencia de sesión.
- **HU08-T18 (QA)** | Validar el manejo de errores y la restauración de sesión en frontend.
- **HU08-T19 (QA)** | Ejecutar pruebas funcionales y registrar hallazgos.

## Estimación: 3 puntos

### Justificación:
- **DEV:** Esfuerzo bajo-medio. El flujo de login reutiliza gran parte de la base creada para registro, añadiendo validación de credenciales y emisión de JWT. En frontend, el trabajo principal está en persistir la sesión y proteger rutas, que es más acotado que el registro completo.
- **QA**: Esfuerzo bajo-medio. Los escenarios son pocos y bien delimitados: login correcto, login fallido y acceso protegido. La validación principal se concentra en el contrato del token y en el comportamiento de la sesión.

---

# HU-09 | Consultar pedidos del usuario

## Objetivo de la historia
Permitir que un usuario autenticado consulte únicamente los pedidos confirmados asociados a su propia cuenta, sin exponer datos de otras personas.

## Subtareas DEV

### Backend
- **HU09-T01** | Adaptar la persistencia del pedido para asociar cada confirmación al `userId` obtenido desde el JWT.
- **HU09-T02** | Implementar la consulta de pedidos filtrada por usuario autenticado en el repositorio y el caso de uso.
- **HU09-T03** | Exponer el endpoint `GET /api/v1/pedido/mis-pedidos` conforme al contrato definido.
- **HU09-T04** | Asegurar que la confirmación y el historial respeten la propiedad del pedido y no acepten `userId` desde el frontend.
- **HU09-T05** | Ajustar los mapeos de respuesta para devolver origen, destino, peso, prioridad y proveedor seleccionado.
- **HU09-T06** | Realizar pruebas unitarias e integración para la consulta del historial autenticado.

### Frontend
- **HU09-T07** | Implementar la pantalla o componente de historial de pedidos del usuario.
- **HU09-T08** | Consumir el `userOrdersApiService` para obtener los pedidos del usuario autenticado.
- **HU09-T09** | Mostrar estados de carga, vacío y error de forma clara en la vista de historial.
- **HU09-T10** | Integrar la navegación al historial desde la aplicación autenticada.
- **HU09-T11** | Limpiar el estado del historial al cerrar sesión para evitar datos cruzados.
- **HU09-T12** | Realizar pruebas unitarias y de cobertura para la vista y sus estados.

## Subtareas QA

### Análisis y diseño
- **HU09-T13 (QA)** | Revisar que la historia, reglas de negocio y criterios de aceptación sean claros y testeables.
- **HU09-T14 (QA)** | Diseñar casos de prueba para historial con pedidos, historial vacío y aislamiento por usuario.

### Validación técnica y funcional
- **HU09-T15 (QA)** | Verificar el funcionamiento del endpoint `GET /api/v1/pedido/mis-pedidos`.
- **HU09-T16 (QA)** | Validar que el sistema muestre únicamente pedidos del usuario autenticado.
- **HU09-T17 (QA)** | Verificar el comportamiento ante usuarios sin pedidos registrados.
- **HU09-T18 (QA)** | Validar la navegación al historial y el renderizado correcto de los datos mínimos requeridos.
- **HU09-T19 (QA)** | Ejecutar pruebas funcionales y registrar hallazgos.

## Estimación: 5 puntos

### Justificación:
- **DEV:** Esfuerzo medio. La historia requiere tocar persistencia, seguridad y consulta filtrada por `userId`, además de un componente de historial en frontend. No es compleja por lógica algorítmica, pero sí por la necesidad de mantener aislamiento de datos y contratos consistentes.
- **QA**: Esfuerzo medio. Hay que validar la asociación correcta de pedidos al usuario autenticado, el historial vacío y la imposibilidad de ver pedidos ajenos. La validación funcional exige cubrir seguridad de datos, no solo renderizado.

---
    