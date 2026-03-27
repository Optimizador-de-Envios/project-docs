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
    