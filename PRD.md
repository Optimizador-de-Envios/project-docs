# Producto: Optimizador de envíos

## 1. Introducción:

### 1.1 Problema
A la hora de realizar el envío de un producto, las empresas/personas pueden encontrarse con el problema de no saber qué proveedor logístico elegir.
Esto puede resultar problemático ya que una selección inadecuada puede generar mayores costos de envío o tiempos de entrega que no cumplan con lo esperado por el cliente final.

### 1.2 Propuesta de valor
El Optimizador de Envíos propone una solución que resuelve este conflicto de forma rápida, automatizada y eficiente, brindando al cliente la posibilidad de elegir si desea priorizar menor costo o mayor velocidad.

---

## 2. Visión y Objetivos

### 2.1 Visión
Construir una herramienta logística que automatice la selección del proveedor de envíos más conveniente, ayudando a mejorar las condiciones de precio y tiempo de entrega para el cliente final.

### 2.2 Objetivos

| Objetivo | Descripción |
| --- | --- |
| **O1** | Automatizar la selección del proveedor de transporte evaluando las opciones entre FedEx, DHL y proveedores locales. |
| **O2** | Centralizar múltiples proveedores de transporte en una sola herramienta simplificando el proceso de toma de decisiones. |
| **O3** | Permitir la optimización del envío según la prioridad elegida por el cliente: menor costo o menor tiempo de entrega. |
| **O4** | Unificar y proveer estimaciones de precio y tiempo de entrega para entregar al cliente la opción más adecuada a sus requerimientos. |
| **O5** | Permitir la visualización de la ruta estimada del envío en un mapa para mejorar la comprensión del recorrido entre origen y destino. |
| **O6** | Asociar pedidos confirmados a usuarios autenticados para habilitar trazabilidad básica e historial de solicitudes. |
| **O7** | Proteger el uso de la aplicación mediante inicio de sesión obligatorio y cierre de sesión disponible para usuarios autenticados. |

---

## 3. Reglas de Negocio y Funcionales Generales

- **Regla 1:** El sistema **solo debe operar** para envíos cuyo origen y destino estén dentro de **Colombia**.
- **Regla 2:** Para calcular una cotización, el usuario autenticado debe ingresar **obligatoriamente origen, destino y peso del paquete**.
- **Regla 3:** El sistema no debe permitir calcular opciones si el paquete supera los límites admitidos por los proveedores disponibles; **el peso debe ser mayor o igual a 0,001 Kg y menor o igual a 70 Kg**.
- **Regla 4:** El usuario debe seleccionar una prioridad de envío (menor costo o menor tiempo) para que el sistema pueda generar la recomendación.
- **Regla 5:** Si la **prioridad es menor costo**, el **sistema** debe **recomendar la opción de menor costo** disponible.
- **Regla 6:** Si la **prioridad es menor tiempo de entrega**, el sistema **debe recomendar la opción de menor tiempo** disponible.
- **Regla 7:** Si **existe empate en el menor costo**, el sistema debe **recomendar la opción con menor tiempo de entrega** entre las empatadas.
- **Regla 8:** Si **existe empate en el menor tiempo de entrega**, el sistema debe **recomendar la opción con menor costo** entre las empatadas.
- **Regla 9:** El sistema debe **mostrar opciones alternativas** distintas a la recomendación principal para permitir la comparación si se da el caso.
- **Regla 10:** El usuario autenticado debe **seleccionar un proveedor** para poder continuar con el proceso del pedido.
- **Regla 11:** El sistema debe **persistir la información del pedido** cuando el usuario autenticado seleccione y confirme un proveedor para continuar con el proceso.
- **Regla 12:** El sistema debe **mostrar la ruta del envío únicamente si existen coordenadas válidas** de origen y destino.
- **Regla 13:** **OpenRouteService** debe utilizarse como servicio externo para el **autocompletado de ubicaciones** de origen/destino y para el **cálculo de la ruta estimada** del envío.
- **Regla 14:** Las coordenadas deben ser transformadas al formato requerido por la librería de mapas definida para el MVP v2, es decir, **Leaflet** en formato **latitud y longitud**.
- **Regla 15:** El mapa debe ajustarse automáticamente para mostrar toda la ruta calculada.
- **Regla 16:** El sistema debe mostrar marcadores para el origen y el destino del envío.
- **Regla 17:** El usuario debe ingresar obligatoriamente **nombre, correo electrónico y contraseña** para registrarse.
- **Regla 18:** El correo electrónico debe ser único por usuario dentro del sistema.
- **Regla 19:** La contraseña debe cumplir una longitud mínima de **8 caracteres**.
- **Regla 20:** Solo los usuarios previamente registrados pueden iniciar sesión en la plataforma.
- **Regla 21:** El sistema debe validar el correo electrónico y la contraseña antes de conceder acceso.
- **Regla 22:** El sistema no debe permitir el acceso si las credenciales son inválidas.
- **Regla 23:** El usuario debe iniciar sesión para acceder a la aplicación y utilizar sus funcionalidades operativas: registrar pedido, seleccionar prioridad, obtener recomendación, visualizar alternativas y ruta, confirmar proveedor y consultar historial.
- **Regla 24:** El sistema debe asociar cada pedido confirmado al usuario autenticado que realizó la operación.
- **Regla 25:** El usuario autenticado solo puede visualizar los pedidos asociados a su propia cuenta.
- **Regla 26:** El sistema debe mostrar como mínimo origen, destino, peso, prioridad y proveedor seleccionado por cada pedido.
- **Regla 27:** Si el usuario no tiene pedidos registrados, el sistema debe informar que no existen pedidos asociados a su cuenta.
- **Regla 28:** El sistema debe permitir que el usuario autenticado cierre sesión; luego del cierre de sesión, no debe permitir el uso de funcionalidades protegidas hasta que el usuario vuelva a iniciar sesión.

> **Nota:** La numeración de las reglas del MVP v2 se normaliza dentro del PRD para mantener un único catálogo de reglas.

---

## 4. Alcance del MVP

> El alcance contempla el MVP base y las funcionalidades de crecimiento definidas para el **MVP v2**.

### 4.1 In Scope
- **Registro de datos del envío:** el sistema permite ingresar origen, destino y peso del pedido.
- **Autocompletado de ubicaciones:** el sistema permite buscar y seleccionar origen y destino mediante OpenRouteService, restringiendo las ubicaciones a Colombia.
- **Selector de prioridad del envío**: el sistema permite elegir si se desea priorizar **menor costo** o **menor tiempo de entrega**.
- **Motor de evaluación centralizado:** el sistema compara las opciones de transporte disponibles (*FedEx*, *DHL* y *proveedores locales*) y evalúa considerando costo y tiempo estimado.
- **Generación de recomendación principal:** el sistema devuelve la opción más adecuada según la prioridad seleccionada por el usuario.
- **Visualización de opciones alternativas:** el sistema muestra otras opciones disponibles para que el usuario pueda compararlas con la recomendación principal.
- **Selección de proveedor:** el usuario puede elegir una de las opciones disponibles para continuar con el proceso del pedido.
- **Visualización de ruta del envío:** el sistema muestra en un mapa la ruta estimada entre el origen y el destino, incluyendo marcadores y ajuste automático del mapa.
- **Registro de usuarios:** el sistema permite crear una cuenta con nombre, correo electrónico único y contraseña válida.
- **Inicio de sesión obligatorio:** el sistema permite autenticar usuarios registrados y protege todas las funcionalidades operativas de la aplicación.
- **Cierre de sesión:** el sistema permite cerrar la sesión activa y bloquea nuevamente el acceso a funcionalidades protegidas hasta un nuevo inicio de sesión.
- **Consulta de pedidos del usuario:** el sistema permite que un usuario autenticado consulte únicamente los pedidos asociados a su cuenta.

### 4.2 Out of Scope
- **Gestión avanzada de perfiles de usuario** como edición de perfil, roles, recuperación de contraseña o administración de cuentas.
- **Integración en tiempo real con APIs de proveedores logísticos.**
- **Módulo de pagos.**
- **Seguimiento del envío (tracking).**
- **Gestión de pedidos internacionales.**

> 💡 **Nota sobre proveedores:** En el MVP, los proveedores (FedEx, DHL y un proveedor local representativo) 
> se modelarán con datos simulados (mock). El proveedor local representa una empresa de mensajería 
> regional colombiana con tarifas y tiempos predefinidos en el sistema.

### 4.3 Flujo Funcional Principal

1. El usuario debe registrarse o iniciar sesión para acceder a la aplicación.
2. Si no existe una sesión activa, el sistema no permite utilizar las funcionalidades protegidas y debe redirigir o solicitar autenticación.
3. Con sesión activa, el usuario ingresa origen y destino dentro de Colombia, apoyado por el autocompletado de ubicaciones de OpenRouteService.
4. El usuario ingresa el peso del paquete, selecciona la prioridad de envío y solicita el cálculo de opciones disponibles.
5. El sistema compara los proveedores mock definidos para el MVP, aplica las reglas de recomendación y desempate, y muestra la recomendación principal junto con alternativas si existen.
6. Si existen coordenadas válidas, el sistema calcula la ruta mediante OpenRouteService y la visualiza en Leaflet con marcadores de origen/destino y ajuste automático del mapa.
7. El usuario selecciona y confirma un proveedor; el sistema persiste el pedido asociado al usuario autenticado.
8. El usuario autenticado puede consultar su historial de pedidos y cerrar sesión cuando lo requiera.

---

## 5. Historias de Usuario del MVP v2

> Nota: Estas historias corresponden a la versión 2 del MVP, orientada al crecimiento post-MVP.

### 5.1 Resumen de Historias

| Historia de Usuario | Descripción | Estimación |
| --- | --- | ---: |
| **HU-06** | Visualizar ruta del envío en el mapa. | 5 SP |
| **HU-07** | Registrar usuario. | 5 SP |
| **HU-08** | Iniciar sesión. | 3 SP |
| **HU-09** | Consultar pedidos del usuario. | 5 SP |

### 5.2 HU-06 | Visualizar Ruta del Envío en el Mapa

**Descripción**

| Elemento | Detalle |
| --- | --- |
| **Como** | Usuario del sistema |
| **Quiero** | Visualizar en un mapa la ruta entre el origen y el destino del envío |
| **Para** | Entender gráficamente el recorrido estimado del pedido |

**Valor de Negocio**
- Permite al usuario comprender visualmente la ruta del envío.
- Mejora la experiencia de usuario al mostrar información geográfica clara.
- Facilita la validación de que el origen y destino seleccionados son correctos.
- Aumenta la confianza del usuario en el cálculo del envío.

**Reglas relacionadas**
- **Regla 12:** El sistema debe mostrar la ruta únicamente si existen coordenadas válidas de origen y destino.
- **Regla 13:** OpenRouteService debe utilizarse como servicio externo para autocompletado de ubicaciones y cálculo de ruta estimada.
- **Regla 14:** Las coordenadas deben transformarse al formato requerido por Leaflet.
- **Regla 15:** El mapa debe ajustarse automáticamente para mostrar toda la ruta.
- **Regla 16:** El sistema debe mostrar marcadores para el origen y el destino.

**Criterios de Aceptación**

| Escenario | Resultado esperado |
| --- | --- |
| Visualización de ruta en el mapa | El sistema muestra la ruta en el mapa y marca el origen y el destino. |
| Ajuste automático del mapa | El mapa se ajusta automáticamente para mostrar toda la ruta visible. |
| Conversión correcta de coordenadas | El sistema convierte coordenadas de formato `[lng, lat]` a `[lat, lng]` y visualiza la ruta correctamente. |
| Intento de visualización sin datos de ruta | El sistema no muestra la ruta e informa que no hay datos suficientes para visualizar el recorrido. |

**Definition of Done (DoD)**
- El mapa se visualiza correctamente en la aplicación.
- La ruta entre origen y destino se dibuja correctamente.
- Se muestran los marcadores de origen y destino.
- El mapa se ajusta automáticamente a la ruta.
- Se realiza correctamente la conversión de coordenadas.
- Se manejan los casos donde no hay datos suficientes.
- Se cumplen los criterios de aceptación definidos.
- La historia fue validada por QA.

---

### 5.3 HU-07 | Registrar Usuario

**Descripción**

| Elemento | Detalle |
| --- | --- |
| **Como** | Usuario del sistema |
| **Quiero** | Registrarme en la plataforma |
| **Para** | Asociar mis pedidos a una cuenta y gestionar mi información en futuras sesiones |

**Valor de Negocio**
- Permite identificar a cada usuario dentro de la plataforma.
- Facilita la asociación de pedidos confirmados con un usuario específico.
- Habilita una base para futuras funcionalidades personalizadas.
- Mejora la trazabilidad de la operación por usuario.

**Reglas relacionadas**
- **Regla 17:** El usuario debe ingresar obligatoriamente nombre, correo electrónico y contraseña para registrarse.
- **Regla 18:** El correo electrónico debe ser único por usuario dentro del sistema.
- **Regla 19:** La contraseña debe cumplir una longitud mínima de 8 caracteres.

**Criterios de Aceptación**

| Escenario | Resultado esperado |
| --- | --- |
| Registro exitoso de usuario | El sistema registra al usuario con nombre, correo electrónico único y contraseña válida, dejando la cuenta disponible para iniciar sesión. |
| Intento de registro con correo ya existente | El sistema no permite el registro e informa que el correo electrónico ya se encuentra en uso. |
| Intento de registro con campos obligatorios vacíos | El sistema no permite el registro e informa que todos los campos son obligatorios. |
| Intento de registro con contraseña inválida | El sistema no permite el registro e informa que la contraseña no cumple las reglas mínimas de seguridad. |

**Definition of Done (DoD)**
- La funcionalidad de registro de usuarios está implementada.
- El sistema permite registrar usuarios con datos válidos.
- El sistema bloquea registros con correos duplicados.
- El sistema bloquea registros con campos obligatorios vacíos.
- El sistema valida la longitud mínima de la contraseña.
- Se cumplen los criterios de aceptación definidos.
- La historia fue validada por QA.

---

### 5.4 HU-08 | Iniciar Sesión

**Descripción**

| Elemento | Detalle |
| --- | --- |
| **Como** | Usuario registrado del sistema |
| **Quiero** | Iniciar sesión en la plataforma |
| **Para** | Acceder a la aplicación, utilizar sus funcionalidades y continuar con mi proceso de envío |

**Valor de Negocio**
- Permite autenticar de forma segura a los usuarios registrados.
- Habilita el acceso a funcionalidades personalizadas por usuario.
- Protege la información asociada a cada cuenta.
- Prepara la plataforma para la consulta de historial de pedidos.
- Permite finalizar la sesión activa cuando el usuario lo requiera.

**Reglas relacionadas**
- **Regla 20:** Solo los usuarios previamente registrados pueden iniciar sesión en la plataforma.
- **Regla 21:** El sistema debe validar el correo electrónico y la contraseña antes de conceder acceso.
- **Regla 22:** El sistema no debe permitir el acceso si las credenciales son inválidas.
- **Regla 23:** El usuario debe iniciar sesión para acceder a la aplicación y utilizar sus funcionalidades operativas.
- **Regla 28:** El sistema debe permitir que el usuario autenticado cierre sesión.

**Criterios de Aceptación**

| Escenario | Resultado esperado |
| --- | --- |
| Inicio de sesión exitoso | El sistema permite el acceso a la cuenta y habilita las funcionalidades asociadas al usuario autenticado. |
| Intento de inicio de sesión con credenciales inválidas | El sistema no permite el acceso e informa que las credenciales son inválidas. |
| Intento de acceso a funcionalidad protegida sin autenticación | El sistema no permite el acceso e informa que la persona debe autenticarse para continuar. |
| Cierre de sesión exitoso | El sistema finaliza la sesión activa y bloquea el acceso a funcionalidades protegidas hasta un nuevo inicio de sesión. |

**Definition of Done (DoD)**
- La funcionalidad de inicio de sesión está implementada.
- El sistema permite autenticarse con credenciales válidas.
- El sistema bloquea el acceso con credenciales inválidas.
- El sistema protege las funcionalidades operativas para usuarios no autenticados.
- El sistema permite cerrar sesión y revoca el acceso a funcionalidades protegidas hasta un nuevo inicio de sesión.
- Se cumplen los criterios de aceptación definidos.
- La historia fue validada por QA.

---

### 5.5 HU-09 | Consultar Pedidos del Usuario

**Descripción**

| Elemento | Detalle |
| --- | --- |
| **Como** | Usuario autenticado del sistema |
| **Quiero** | Consultar los pedidos asociados a mi cuenta |
| **Para** | Hacer seguimiento a mis solicitudes y revisar mis decisiones anteriores |

**Valor de Negocio**
- Permite al usuario revisar el historial de pedidos realizados.
- Mejora la trazabilidad de la información persistida en el sistema.
- Aporta continuidad al proceso luego de la selección y confirmación del proveedor.
- Incrementa el valor del producto al ofrecer gestión básica de cuenta.

**Reglas relacionadas**
- **Regla 24:** El sistema debe asociar cada pedido confirmado al usuario autenticado que realizó la operación.
- **Regla 25:** El usuario autenticado solo puede visualizar los pedidos asociados a su propia cuenta.
- **Regla 26:** El sistema debe mostrar como mínimo origen, destino, peso, prioridad y proveedor seleccionado por cada pedido.
- **Regla 27:** Si el usuario no tiene pedidos registrados, el sistema debe informar que no existen pedidos asociados a su cuenta.

**Criterios de Aceptación**

| Escenario | Resultado esperado |
| --- | --- |
| Visualización de pedidos del usuario autenticado | El sistema muestra únicamente los pedidos del usuario autenticado, incluyendo origen, destino, peso, prioridad y proveedor seleccionado. |
| Usuario autenticado sin pedidos registrados | El sistema informa que no existen pedidos registrados para ese usuario. |
| Intento de consultar pedidos de otro usuario | El sistema no muestra pedidos de otras cuentas y restringe la información a los pedidos asociados al usuario autenticado. |

**Definition of Done (DoD)**
- La funcionalidad de consulta de pedidos por usuario está implementada.
- El sistema asocia los pedidos confirmados al usuario autenticado.
- El sistema muestra únicamente los pedidos correspondientes a la cuenta autenticada.
- El sistema informa cuando un usuario no tiene pedidos registrados.
- Se muestran los datos mínimos definidos para cada pedido.
- Se cumplen los criterios de aceptación definidos.
- La historia fue validada por QA.

---

### 5.6 Estimación de las Historias de Usuario

| Historia de Usuario | Estimación (Story Points) |
| --- | ---: |
| **HU-06** | 5 |
| **HU-07** | 5 |
| **HU-08** | 3 |
| **HU-09** | 5 |

> 💡 **Nota sobre estimaciones:** Las estimaciones consideran complejidad, esfuerzo y riesgos asociados a cada historia para la versión 2 del MVP.

**Notación:**
- Las estimaciones se realizaron utilizando la técnica de **Story Points**, considerando la complejidad, el esfuerzo y el riesgo asociado a cada historia de usuario.

**Rúbrica de estimación:**
- **1 SP:** Historia sencilla, sin dependencias y sin incertidumbres importantes.
- **2 SP:** Historia con baja complejidad, con pocas dependencias o incertidumbres menores.
- **3 SP:** Historia de complejidad media, con algunas dependencias o incertidumbres moderadas.
- **5 SP:** Historia compleja, con múltiples dependencias o incertidumbres significativas.
- **8 SP:** Historia muy compleja, con muchas dependencias o incertidumbres altas.
- **13 SP:** Historia extremadamente compleja, con numerosas dependencias o incertidumbres muy altas; en algunos casos, puede ser dividida en historias más pequeñas.

---

## 6. Riesgos técnicos y de negocio

> **Notación:**  
> El impacto y la probabilidad de cada riesgo se clasifican de 1 a 3, donde 1 es bajo, 2 es medio y 3 es alto.  
> El riesgo se calcula como el producto del impacto y la probabilidad, y se clasifica como bajo (1-3), medio (4-6) o alto (7-9).

### 6.1 Riesgos de Negocio

| Riesgo | Descripción | Impacto | Probabilidad | Riesgo | Mitigación |
| --- | --- | --- | --- | --- | --- |
| **R1: Pérdidas financieras por fallos de lógica** | Si el algoritmo calcula mal el cruce entre peso y distancia desde el origen al destino, podría asignar sistemáticamente la opción más cara cuando el usuario prioriza el costo. | 3 | 2 | Medio (6) | Realizar pruebas exhaustivas del algoritmo de recomendación, incluyendo casos límite y validación con datos reales. |
| **R2: Incumplimiento de la propuesta de valor** | Si el usuario prioriza “más rápido” y el sistema falla en estimar los tiempos, el producto puede llegar tarde, afectando la experiencia de usuario del cliente. | 3 | 2 | Medio (6) | Implementar un sistema de retroalimentación para que los usuarios puedan reportar discrepancias en los tiempos estimados y ajustar el algoritmo en consecuencia. |
| **R3: Incompatibilidad del modelo del “Proveedor local”** | Empresas como DHL y FedEx tienen sistemas y matrices de precios estandarizados y muy estructurados. Un proveedor local en cambio, puede ser más informal, y el sistema correría riesgos tales como cambios de precio sin aviso o información desactualizada. | 2 | 2 | Medio (4) | Establecer acuerdos claros con los proveedores locales para garantizar la actualización periódica de sus tarifas y condiciones, así como implementar un sistema de monitoreo para detectar cambios en tiempo real. |

### 6.2 Riesgos Técnicos

| Riesgo | Descripción | Impacto | Probabilidad | Riesgo | Mitigación |
| --- | --- | --- | --- | --- | --- |
| **R4: Acoplamiento fuerte a proveedores específicos** | Una implementación poco flexible puede dificultar la escalabilidad (ejemplo: agregar un nuevo proveedor) en futuras versiones. | 2 | 3 | Medio (6) | Diseñar el sistema con una arquitectura modular y orientada a servicios, utilizando interfaces y abstracciones para facilitar la integración de nuevos proveedores sin afectar la lógica central. |
| **R5: Inconsistencia en los datos de entrada** | Si el sistema no gestiona los tipos de datos de forma adecuada (ejemplo: tipos de medidas en peso, origen y destino correctos, tipo de moneda) podría verse afectado el cálculo de las opciones disponibles. | 3 | 2 | Medio (6) | Implementar validaciones estrictas en la entrada de datos, incluyendo formatos, rangos permitidos y tipos de datos, para asegurar que la información ingresada sea consistente y adecuada para el procesamiento. |
| **R6: Manejo insuficiente de casos límite** | Destinos sin cobertura o pesos no admitidos por los proveedores pueden generar fallos si no se controlan adecuadamente. | 3 | 1 | Bajo (3) | Desarrollar un sistema de manejo de errores robusto que identifique y gestione adecuadamente los casos límite, proporcionando mensajes claros al usuario y evitando que el sistema falle ante situaciones no contempladas. |
| **R7: Dependencia del servicio externo OpenRouteService** | Si OpenRouteService no responde, responde con errores o cambia sus límites de uso, el autocompletado de ubicaciones o la visualización de ruta pueden fallar aunque la lógica de recomendación funcione correctamente. | 2 | 2 | Medio (4) | Encapsular la integración con el servicio externo, manejar errores con mensajes claros, usar mocks/stubs cuando corresponda y permitir que el flujo principal de cotización continúe sin visualización de ruta cuando no haya datos suficientes. |
| **R8: Exposición de información entre usuarios** | Una implementación deficiente de autenticación o filtrado de pedidos podría permitir que un usuario consulte pedidos de otra cuenta. | 3 | 2 | Medio (6) | Validar autorización en backend, asociar pedidos al usuario autenticado, cubrir casos de acceso no autorizado con pruebas y evitar depender únicamente de controles del frontend. |
