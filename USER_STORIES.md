# Historias de Usuario - Optimizador de Envíos

> Este documento consolida las historias de usuario del **MVP base** y del **MVP v2**. Las reglas relacionadas se referencian contra el catálogo unificado del PRD vigente.
>
> Las funcionalidades operativas del sistema requieren una sesión activa. El registro permite crear una cuenta, pero para registrar pedidos, seleccionar prioridad, obtener recomendación, visualizar alternativas y ruta, confirmar proveedor o consultar historial, el usuario debe iniciar sesión.

---

## HU-01 | Registrar Pedido de Envío

### Descripción

**Como** usuario autenticado del sistema  
**Quiero** registrar los datos de un pedido de envío  
**Para** poder calcular las opciones de transporte disponibles.

### Valor de Negocio
- Permite registrar un pedido con los datos necesarios para calcular el envío.
- Asegura que el sistema tenga la información correcta para generar recomendaciones de envío.
- Facilita la validación de datos de entrada para evitar errores en el proceso de cálculo.
- Restringe el flujo a envíos cuyo origen y destino estén dentro de Colombia.
- Aprovecha el autocompletado de ubicaciones para reducir errores de captura en origen y destino.

### Reglas relacionadas
- **Regla 1:** El sistema solo debe operar para envíos cuyo origen y destino estén dentro de Colombia.
- **Regla 2:** Para calcular una cotización, el usuario autenticado debe ingresar obligatoriamente origen, destino y peso del paquete.
- **Regla 3:** El peso debe ser mayor o igual a 0,001 Kg y menor o igual a 70 Kg.
- **Regla 13:** OpenRouteService debe utilizarse como servicio externo para el autocompletado de ubicaciones de origen/destino y para el cálculo de la ruta estimada del envío.
- **Regla 23:** El usuario debe iniciar sesión para acceder a la aplicación y utilizar sus funcionalidades operativas.

### Definition of Ready (DoR)
- La historia de usuario está redactada de forma clara.
- Están definidos los datos obligatorios del pedido: origen, destino y peso.
- Están claras las reglas de cobertura geográfica para Colombia.
- Están claros los límites de peso permitidos.
- Está definido el uso de OpenRouteService para el autocompletado de ubicaciones.
- Está definido que el flujo operativo requiere usuario autenticado.
- Los criterios de aceptación están definidos.
- La historia está revisada por DEV y QA.
- La historia puede ser estimada por el equipo técnico.

### Criterios de Aceptación

```gherkin
Escenario: Ingreso de datos válidos
    Dado que el usuario autenticado necesita enviar un producto
    Cuando ingresa un origen dentro de Colombia, un destino dentro de Colombia y un peso válido
    Entonces el sistema debe registrar los datos del pedido
    Y debe permitir continuar con el cálculo del envío
```

```gherkin
Escenario: Autocompletado de ubicaciones
    Dado que el usuario autenticado se encuentra registrando un pedido
    Cuando ingresa texto en los campos de origen o destino
    Entonces el sistema debe consultar sugerencias de ubicación mediante OpenRouteService
    Y debe restringir las sugerencias a ubicaciones dentro de Colombia
```

```gherkin
Escenario: Intento de registro con campos vacíos
    Dado que el usuario autenticado necesita enviar un producto
    Cuando intenta registrar un pedido sin ingresar origen, destino o peso
    Entonces el sistema no debe permitir continuar con el cálculo del envío
    Y debe informar que todos los campos son obligatorios
```

```gherkin
Escenario: Origen fuera de Colombia
    Dado que el usuario autenticado necesita enviar un producto
    Cuando ingresa un origen fuera de Colombia y un destino dentro de Colombia
    Entonces el sistema no debe permitir continuar con el cálculo del envío
    Y debe informar que el envío está fuera de la cobertura permitida
```

```gherkin
Escenario: Destino fuera de Colombia
    Dado que el usuario autenticado necesita enviar un producto
    Cuando ingresa un origen dentro de Colombia y un destino fuera de Colombia
    Entonces el sistema no debe permitir continuar con el cálculo del envío
    Y debe informar que el envío está fuera de la cobertura permitida
```

```gherkin
Escenario: Peso fuera del rango permitido
    Dado que el usuario autenticado necesita enviar un producto
    Cuando ingresa un peso menor a 0,001 Kg o mayor a 70 Kg
    Entonces el sistema no debe permitir continuar con el cálculo del envío
    Y debe informar que el peso ingresado no está cubierto por los proveedores disponibles
```

### Definition of Done (DoD)
- La funcionalidad de registro de pedido está implementada.
- El sistema permite registrar origen, destino y peso válidos para usuarios autenticados.
- El sistema bloquea origen y/o destino fuera de Colombia.
- El sistema bloquea pesos no permitidos.
- El sistema permite autocompletar ubicaciones mediante OpenRouteService con restricción a Colombia.
- Se cumplen los criterios de aceptación definidos.
- La historia fue validada por QA.

---

## HU-02 | Definir Prioridad del Envío

### Descripción

**Como** usuario autenticado del sistema  
**Quiero** seleccionar si prefiero menor costo o menor tiempo de entrega  
**Para** que el sistema realice una recomendación según mi prioridad.

### Valor de Negocio
- Permite seleccionar el criterio de optimización que responde a la necesidad del usuario.
- Asegura que el sistema tenga la información necesaria para generar recomendaciones de envío.
- Evita cálculos ambiguos cuando el usuario no define una prioridad.

### Reglas relacionadas
- **Regla 4:** El usuario debe seleccionar una prioridad de envío (menor costo o menor tiempo) para que el sistema pueda generar la recomendación.
- **Regla 23:** El usuario debe iniciar sesión para acceder a la aplicación y utilizar sus funcionalidades operativas.

### Definition of Ready (DoR)
- La historia de usuario está redactada de forma clara.
- Están definidas las prioridades disponibles: menor costo y menor tiempo de entrega.
- Está definido que la prioridad es obligatoria para generar recomendación.
- Está definido que el flujo operativo requiere usuario autenticado.
- Los criterios de aceptación están definidos.
- La historia está revisada por DEV y QA.
- La historia puede ser estimada por el equipo técnico.

### Criterios de Aceptación

```gherkin
Escenario: Selección de prioridad por menor costo
    Dado que el usuario autenticado registró un pedido válido
    Cuando selecciona prioridad de menor costo
    Entonces el sistema debe utilizar el costo como criterio principal para generar la recomendación
```

```gherkin
Escenario: Selección de prioridad por menor tiempo de entrega
    Dado que el usuario autenticado registró un pedido válido
    Cuando selecciona prioridad de menor tiempo de entrega
    Entonces el sistema debe utilizar el tiempo de entrega como criterio principal para generar la recomendación
```

```gherkin
Escenario: No selección de prioridad
    Dado que el usuario autenticado registró un pedido válido
    Cuando no selecciona ninguna prioridad de envío
    Entonces el sistema no debe permitir continuar con el cálculo del envío
    Y debe informar que se debe seleccionar una prioridad de envío
```

### Definition of Done (DoD)
- La funcionalidad de elegir la prioridad del envío está implementada.
- El sistema permite elegir entre menor costo y menor tiempo de entrega.
- El sistema no permite calcular recomendación sin una prioridad seleccionada.
- Se cumplen los criterios de aceptación definidos.
- La historia fue validada por QA.

---

## HU-03 | Obtener Recomendación Principal de Proveedor de Envío

### Descripción

**Como** usuario autenticado del sistema  
**Quiero** que el sistema me recomiende la mejor opción de envío  
**Para** elegir una alternativa que cumpla con mis necesidades.

### Valor de Negocio
- Proporciona una recomendación personalizada basada en la prioridad del usuario.
- Facilita la toma de decisiones al destacar la opción más relevante.
- Mejora la experiencia del usuario al ofrecer una recomendación clara y directa.
- Reduce el riesgo de seleccionar una opción subóptima frente a costo o tiempo.

### Reglas relacionadas
- **Regla 5:** Si la prioridad es menor costo, el sistema debe recomendar la opción de menor costo disponible.
- **Regla 6:** Si la prioridad es menor tiempo de entrega, el sistema debe recomendar la opción de menor tiempo disponible.
- **Regla 7:** Si existe empate en el menor costo, el sistema debe recomendar la opción con menor tiempo de entrega entre las empatadas.
- **Regla 8:** Si existe empate en el menor tiempo de entrega, el sistema debe recomendar la opción con menor costo entre las empatadas.
- **Regla 23:** El usuario debe iniciar sesión para acceder a la aplicación y utilizar sus funcionalidades operativas.

### Definition of Ready (DoR)
- La historia de usuario está redactada de forma clara.
- Están definidas las prioridades de envío disponibles: menor costo y menor tiempo de entrega.
- Están definidas las reglas de negocio para generar la recomendación principal.
- Están definidas las reglas de desempate.
- Está definido que el flujo operativo requiere usuario autenticado.
- Los criterios de aceptación están definidos.
- La historia está revisada por DEV y QA.
- La historia puede ser estimada por el equipo técnico.

### Criterios de Aceptación

```gherkin
Escenario: Recomendación principal según prioridad de menor costo
    Dado que el usuario autenticado definió la prioridad de menor costo
    Cuando el sistema calcula las opciones disponibles
    Entonces el sistema debe devolver la opción recomendada
    Y la opción recomendada debe corresponder a la opción de menor costo disponible
```

```gherkin
Escenario: Recomendación principal según prioridad de menor tiempo
    Dado que el usuario autenticado definió la prioridad de menor tiempo
    Cuando el sistema calcula las opciones disponibles
    Entonces el sistema debe devolver la opción recomendada
    Y la opción recomendada debe corresponder a la opción de menor tiempo disponible
```

```gherkin
Escenario: Recomendación principal con empate en menor costo
    Dado que el usuario autenticado definió la prioridad de menor costo
    Y existen múltiples opciones con el mismo menor costo disponible
    Cuando el sistema calcula las opciones disponibles
    Entonces el sistema debe devolver una opción recomendada
    Y la opción recomendada debe corresponder a la de menor tiempo de entrega entre las empatadas
```

```gherkin
Escenario: Recomendación principal con empate en menor tiempo
    Dado que el usuario autenticado definió la prioridad de menor tiempo
    Y existen múltiples opciones con el mismo menor tiempo disponible
    Cuando el sistema calcula las opciones disponibles
    Entonces el sistema debe devolver una opción recomendada
    Y la opción recomendada debe corresponder a la de menor costo entre las empatadas
```

### Definition of Done (DoD)
- La funcionalidad de generar una recomendación principal está implementada.
- El sistema devuelve la opción recomendada de acuerdo con la prioridad seleccionada por el usuario autenticado.
- El sistema maneja correctamente los casos de empate en costo o tiempo.
- Se cumplen los criterios de aceptación definidos.
- La historia fue validada por QA.

---

## HU-04 | Obtener Opciones Alternativas de Proveedores

### Descripción

**Como** usuario autenticado del sistema  
**Quiero** ver otras opciones disponibles  
**Para** compararlas con la recomendación principal y evaluar la opción más conveniente.

### Valor de Negocio
- Permite al usuario tener control sobre la decisión de qué proveedor usar.
- Aporta transparencia al mostrar opciones adicionales a la recomendación principal.
- Facilita la comparación entre proveedores según costo y tiempo de entrega.

### Reglas relacionadas
- **Regla 9:** El sistema debe mostrar opciones alternativas distintas a la recomendación principal para permitir la comparación si se da el caso.
- **Regla 23:** El usuario debe iniciar sesión para acceder a la aplicación y utilizar sus funcionalidades operativas.

### Definition of Ready (DoR)
- La historia de usuario está redactada de forma clara.
- Está definida la regla de negocio para mostrar alternativas distintas a la recomendación principal.
- Están definidas las reglas de negocio relacionadas con proveedores, peso y cobertura.
- Está definido que el flujo operativo requiere usuario autenticado.
- Los criterios de aceptación están definidos.
- La historia está revisada por DEV y QA.
- La historia puede ser estimada por el equipo técnico.

### Criterios de Aceptación

```gherkin
Escenario: Visualización de alternativas
    Dado que el usuario autenticado obtuvo una recomendación principal
    Y existen otras opciones de envío disponibles
    Cuando el sistema muestra la recomendación principal
    Entonces el sistema debe mostrar automáticamente las opciones alternativas disponibles
    Y las opciones alternativas deben ser distintas a la recomendación principal
```

```gherkin
Escenario: Ausencia de opciones alternativas
    Dado que el usuario autenticado obtuvo una recomendación principal
    Y no existen otras opciones de envío disponibles
    Cuando el sistema muestra la recomendación principal
    Entonces el sistema debe notificar que no existen otras opciones alternativas
```

### Definition of Done (DoD)
- La funcionalidad de generar opciones alternativas de proveedores está implementada.
- El sistema devuelve opciones alternativas si existen para el caso evaluado.
- Las opciones alternativas no duplican la recomendación principal.
- Se cumplen los criterios de aceptación definidos.
- La historia fue validada por QA.

---

## HU-05 | Seleccionar y Confirmar Proveedor

### Descripción

**Como** usuario autenticado del sistema  
**Quiero** seleccionar el proveedor deseado  
**Para** confirmar mi elección y continuar con el proceso del pedido.

### Valor de Negocio
- Facilita la toma de decisiones al ofrecer una selección clara entre las opciones disponibles.
- Mejora la experiencia del usuario al permitirle confirmar su elección de proveedor.
- Permite persistir la decisión tomada para habilitar trazabilidad e historial.

### Reglas relacionadas
- **Regla 10:** El usuario autenticado debe seleccionar un proveedor para poder continuar con el proceso del pedido.
- **Regla 11:** El sistema debe persistir la información del pedido cuando el usuario autenticado seleccione y confirme un proveedor para continuar con el proceso.
- **Regla 24:** El sistema debe asociar cada pedido confirmado al usuario autenticado que realizó la operación.
- **Regla 23:** El usuario debe iniciar sesión para acceder a la aplicación y utilizar sus funcionalidades operativas.

### Definition of Ready (DoR)
- La historia de usuario está redactada de forma clara.
- Están definidas las reglas de negocio para la selección y confirmación del proveedor.
- Está definida la regla de negocio para continuar o bloquear el proceso de selección de proveedor.
- Está definida la regla de negocio para persistir la información del pedido al confirmar un proveedor.
- Está definido que el pedido confirmado se asocia al usuario autenticado.
- Los criterios de aceptación están definidos.
- La historia está revisada por DEV y QA.
- La historia puede ser estimada por el equipo técnico.

### Criterios de Aceptación

```gherkin
Escenario: Selección del proveedor recomendado
    Dado que el usuario autenticado visualiza la opción recomendada y las opciones disponibles
    Cuando selecciona el proveedor recomendado
    Entonces el sistema debe permitir continuar con el proceso del pedido
```

```gherkin
Escenario: Selección de un proveedor alternativo
    Dado que el usuario autenticado visualiza la opción recomendada y las opciones alternativas
    Cuando selecciona un proveedor alternativo disponible
    Entonces el sistema debe permitir continuar con el proceso del pedido
```

```gherkin
Escenario: Usuario no selecciona proveedor
    Dado que el usuario autenticado visualiza la opción recomendada y las opciones disponibles
    Cuando intenta continuar sin seleccionar un proveedor
    Entonces el sistema no debe continuar con el proceso de pedido
    Y debe informar que se debe seleccionar un proveedor para continuar
```

```gherkin
Escenario: Persistencia de información al confirmar proveedor
    Dado que el usuario autenticado visualiza la opción recomendada y las opciones disponibles
    Cuando selecciona un proveedor y confirma su selección
    Entonces el sistema debe persistir la información del pedido con el proveedor seleccionado
    Y debe asociar el pedido confirmado al usuario autenticado
    Y debe permitir continuar con el proceso del pedido
```

### Definition of Done (DoD)
- La funcionalidad de selección y confirmación del proveedor está implementada.
- El sistema permite seleccionar un proveedor de las opciones disponibles.
- El sistema bloquea el proceso de pedido si no se ha seleccionado un proveedor.
- El sistema persiste la información del pedido al seleccionar y confirmar un proveedor.
- El sistema asocia el pedido confirmado al usuario autenticado.
- Se cumplen los criterios de aceptación definidos.
- La historia fue validada por QA.

---

## HU-06 | Visualizar Ruta del Envío en el Mapa

### Descripción

**Como** usuario autenticado del sistema  
**Quiero** visualizar en un mapa la ruta entre el origen y el destino del envío  
**Para** entender gráficamente el recorrido estimado del pedido.

### Valor de Negocio
- Permite al usuario comprender visualmente la ruta del envío.
- Mejora la experiencia de usuario al mostrar información geográfica clara.
- Facilita la validación de que el origen y destino seleccionados son correctos.
- Aumenta la confianza del usuario en el cálculo del envío.

### Reglas relacionadas
- **Regla 12:** El sistema debe mostrar la ruta del envío únicamente si existen coordenadas válidas de origen y destino.
- **Regla 13:** OpenRouteService debe utilizarse como servicio externo para el autocompletado de ubicaciones de origen/destino y para el cálculo de la ruta estimada del envío.
- **Regla 14:** Las coordenadas deben ser transformadas al formato requerido por Leaflet en formato latitud y longitud.
- **Regla 15:** El mapa debe ajustarse automáticamente para mostrar toda la ruta calculada.
- **Regla 16:** El sistema debe mostrar marcadores para el origen y el destino del envío.
- **Regla 23:** El usuario debe iniciar sesión para acceder a la aplicación y utilizar sus funcionalidades operativas.

### Definition of Ready (DoR)
- La historia de usuario está redactada de forma clara.
- Se cuenta con latitud y longitud de origen y destino.
- Está definido el uso de OpenRouteService para cálculo de ruta.
- Está definida la librería de visualización de mapas: Leaflet.
- Está definida la conversión de coordenadas desde formato `[lng, lat]` hacia `[lat, lng]`.
- Está definido que el flujo operativo requiere usuario autenticado.
- Los criterios de aceptación están definidos.
- La historia está revisada por DEV y QA.
- La historia puede ser estimada por el equipo técnico.

### Criterios de Aceptación

```gherkin
Escenario: Visualización de ruta en el mapa
    Dado que el usuario autenticado ha ingresado un origen y un destino válidos
    Y el sistema ha calculado la ruta entre ambos puntos
    Cuando el usuario visualiza el mapa
    Entonces el sistema debe mostrar la ruta en el mapa
    Y debe mostrar un marcador en el origen
    Y debe mostrar un marcador en el destino
```

```gherkin
Escenario: Ajuste automático del mapa
    Dado que el sistema ha dibujado la ruta en el mapa
    Cuando la ruta es visible
    Entonces el mapa debe ajustarse automáticamente para mostrar toda la ruta
```

```gherkin
Escenario: Conversión correcta de coordenadas
    Dado que el sistema recibe coordenadas en formato [lng, lat]
    Cuando se dibuja la ruta en el mapa
    Entonces el sistema debe convertirlas a formato [lat, lng]
    Y la ruta debe visualizarse correctamente
```

```gherkin
Escenario: Intento de visualización sin datos de ruta
    Dado que el usuario autenticado no ha ingresado origen o destino
    O el sistema no ha podido calcular la ruta
    Cuando intenta visualizar el mapa
    Entonces el sistema no debe mostrar la ruta
    Y debe informar que no hay datos suficientes para visualizar el recorrido
```

### Definition of Done (DoD)
- El mapa se visualiza correctamente en la aplicación.
- La ruta entre origen y destino se dibuja correctamente.
- Se muestran los marcadores de origen y destino.
- El mapa se ajusta automáticamente a la ruta.
- Se realiza correctamente la conversión de coordenadas.
- Se manejan los casos donde no hay datos suficientes.
- Se cumplen los criterios de aceptación definidos.
- La historia fue validada por QA.

---

## HU-07 | Registrar Usuario

### Descripción

**Como** usuario del sistema  
**Quiero** registrarme en la plataforma  
**Para** asociar mis pedidos a una cuenta y gestionar mi información en futuras sesiones.

### Valor de Negocio
- Permite identificar a cada usuario dentro de la plataforma.
- Facilita la asociación de pedidos confirmados con un usuario específico.
- Habilita una base para futuras funcionalidades personalizadas.
- Mejora la trazabilidad de la operación por usuario.

### Reglas relacionadas
- **Regla 17:** El usuario debe ingresar obligatoriamente nombre, correo electrónico y contraseña para registrarse.
- **Regla 18:** El correo electrónico debe ser único por usuario dentro del sistema.
- **Regla 19:** La contraseña debe cumplir una longitud mínima de 8 caracteres.

### Definition of Ready (DoR)
- La historia de usuario está redactada de forma clara.
- Están definidos los datos obligatorios para el registro del usuario.
- Está definida la validación de unicidad del correo electrónico.
- Está definida la regla mínima de seguridad para la contraseña.
- Los criterios de aceptación están definidos.
- La historia está revisada por DEV y QA.
- La historia puede ser estimada por el equipo técnico.

### Criterios de Aceptación

```gherkin
Escenario: Registro exitoso de usuario
    Dado que una persona desea utilizar la plataforma
    Cuando ingresa un nombre, un correo electrónico único y una contraseña válida
    Entonces el sistema debe registrar al usuario correctamente
    Y debe dejar disponible la cuenta para iniciar sesión
```

```gherkin
Escenario: Intento de registro con correo ya existente
    Dado que ya existe un usuario registrado con un correo electrónico
    Cuando otra persona intenta registrarse con ese mismo correo
    Entonces el sistema no debe permitir el registro
    Y debe informar que el correo electrónico ya se encuentra en uso
```

```gherkin
Escenario: Intento de registro con campos obligatorios vacíos
    Dado que una persona desea crear una cuenta
    Cuando intenta registrarse sin completar nombre, correo electrónico o contraseña
    Entonces el sistema no debe permitir el registro
    Y debe informar que todos los campos son obligatorios
```

```gherkin
Escenario: Intento de registro con contraseña inválida
    Dado que una persona desea crear una cuenta
    Cuando ingresa una contraseña con menos de 8 caracteres
    Entonces el sistema no debe permitir el registro
    Y debe informar que la contraseña no cumple las reglas mínimas de seguridad
```

### Definition of Done (DoD)
- La funcionalidad de registro de usuarios está implementada.
- El sistema permite registrar usuarios con datos válidos.
- El sistema bloquea registros con correos duplicados.
- El sistema bloquea registros con campos obligatorios vacíos.
- El sistema valida la longitud mínima de la contraseña.
- Se cumplen los criterios de aceptación definidos.
- La historia fue validada por QA.

---

## HU-08 | Iniciar Sesión

### Descripción

**Como** usuario registrado del sistema  
**Quiero** iniciar sesión en la plataforma  
**Para** acceder a la aplicación, utilizar sus funcionalidades y continuar con mi proceso de envío.

### Valor de Negocio
- Permite autenticar de forma segura a los usuarios registrados.
- Habilita el acceso a funcionalidades personalizadas por usuario.
- Protege la información asociada a cada cuenta.
- Prepara la plataforma para la consulta de historial de pedidos.
- Permite finalizar la sesión activa cuando el usuario lo requiera.

### Reglas relacionadas
- **Regla 20:** Solo los usuarios previamente registrados pueden iniciar sesión en la plataforma.
- **Regla 21:** El sistema debe validar el correo electrónico y la contraseña antes de conceder acceso.
- **Regla 22:** El sistema no debe permitir el acceso si las credenciales son inválidas.
- **Regla 23:** El usuario debe iniciar sesión para acceder a la aplicación y utilizar sus funcionalidades operativas.
- **Regla 28:** El sistema debe permitir que el usuario autenticado cierre sesión; luego del cierre de sesión, no debe permitir el uso de funcionalidades protegidas hasta que el usuario vuelva a iniciar sesión.

### Definition of Ready (DoR)
- La historia de usuario está redactada de forma clara.
- Están definidas las credenciales necesarias para el inicio de sesión.
- Está definida la validación de autenticación.
- Está definido el comportamiento ante credenciales inválidas.
- Está definido que el login es obligatorio para utilizar funcionalidades operativas.
- Está definido el comportamiento de cierre de sesión.
- Los criterios de aceptación están definidos.
- La historia está revisada por DEV y QA.
- La historia puede ser estimada por el equipo técnico.

### Criterios de Aceptación

```gherkin
Escenario: Inicio de sesión exitoso
    Dado que el usuario ya se encuentra registrado en la plataforma
    Cuando ingresa un correo electrónico y una contraseña válidos
    Entonces el sistema debe permitir el acceso a su cuenta
    Y debe habilitar las funcionalidades asociadas al usuario autenticado
```

```gherkin
Escenario: Intento de inicio de sesión con credenciales inválidas
    Dado que el usuario intenta acceder a la plataforma
    Cuando ingresa un correo electrónico o una contraseña incorrectos
    Entonces el sistema no debe permitir el acceso
    Y debe informar que las credenciales son inválidas
```

```gherkin
Escenario: Intento de acceso a funcionalidad protegida sin autenticación
    Dado que existe una funcionalidad operativa protegida
    Cuando una persona intenta acceder sin haber iniciado sesión
    Entonces el sistema no debe permitir el acceso
    Y debe informar que debe autenticarse para continuar
```

```gherkin
Escenario: Cierre de sesión exitoso
    Dado que el usuario ha iniciado sesión en la plataforma
    Cuando cierra sesión
    Entonces el sistema debe finalizar la sesión activa
    Y debe bloquear el acceso a funcionalidades protegidas hasta un nuevo inicio de sesión
```

### Definition of Done (DoD)
- La funcionalidad de inicio de sesión está implementada.
- El sistema permite autenticarse con credenciales válidas.
- El sistema bloquea el acceso con credenciales inválidas.
- El sistema protege las funcionalidades operativas para usuarios no autenticados.
- El sistema permite cerrar sesión y revoca el acceso a funcionalidades protegidas hasta un nuevo inicio de sesión.
- Se cumplen los criterios de aceptación definidos.
- La historia fue validada por QA.

---

## HU-09 | Consultar Pedidos del Usuario

### Descripción

**Como** usuario autenticado del sistema  
**Quiero** consultar los pedidos asociados a mi cuenta  
**Para** hacer seguimiento a mis solicitudes y revisar mis decisiones anteriores.

### Valor de Negocio
- Permite al usuario revisar el historial de pedidos realizados.
- Mejora la trazabilidad de la información persistida en el sistema.
- Aporta continuidad al proceso luego de la selección y confirmación del proveedor.
- Incrementa el valor del producto al ofrecer gestión básica de cuenta.

### Reglas relacionadas
- **Regla 24:** El sistema debe asociar cada pedido confirmado al usuario autenticado que realizó la operación.
- **Regla 25:** El usuario autenticado solo puede visualizar los pedidos asociados a su propia cuenta.
- **Regla 26:** El sistema debe mostrar como mínimo origen, destino, peso, prioridad y proveedor seleccionado por cada pedido.
- **Regla 27:** Si el usuario no tiene pedidos registrados, el sistema debe informar que no existen pedidos asociados a su cuenta.

### Definition of Ready (DoR)
- La historia de usuario está redactada de forma clara.
- Está definida la relación entre usuario y pedido persistido.
- Están definidos los datos mínimos que se mostrarán por pedido.
- Está definido el criterio de seguridad para restringir la consulta a la cuenta autenticada.
- Los criterios de aceptación están definidos.
- La historia está revisada por DEV y QA.
- La historia puede ser estimada por el equipo técnico.

### Criterios de Aceptación

```gherkin
Escenario: Visualización de pedidos del usuario autenticado
    Dado que el usuario ha iniciado sesión en la plataforma
    Y existen pedidos asociados a su cuenta
    Cuando consulta su listado de pedidos
    Entonces el sistema debe mostrar únicamente los pedidos del usuario autenticado
    Y cada pedido debe incluir origen, destino, peso, prioridad y proveedor seleccionado
```

```gherkin
Escenario: Usuario autenticado sin pedidos registrados
    Dado que el usuario ha iniciado sesión en la plataforma
    Y no existen pedidos asociados a su cuenta
    Cuando consulta su listado de pedidos
    Entonces el sistema debe informar que no existen pedidos registrados para ese usuario
```

```gherkin
Escenario: Intento de consultar pedidos de otro usuario
    Dado que existen pedidos asociados a diferentes usuarios
    Cuando un usuario autenticado consulta su información
    Entonces el sistema no debe mostrar pedidos de otras cuentas
    Y debe restringir la información a los pedidos asociados al usuario autenticado
```

### Definition of Done (DoD)
- La funcionalidad de consulta de pedidos por usuario está implementada.
- El sistema asocia los pedidos confirmados al usuario autenticado.
- El sistema muestra únicamente los pedidos correspondientes a la cuenta autenticada.
- El sistema informa cuando un usuario no tiene pedidos registrados.
- Se muestran los datos mínimos definidos para cada pedido.
- Se cumplen los criterios de aceptación definidos.
- La historia fue validada por QA.

---

## Estimación de las Historias de Usuario

| Historia de Usuario | Alcance | Estimación (Story Points) |
| --- | --- | ---: |
| HU-01 | MVP base | 5 |
| HU-02 | MVP base | 3 |
| HU-03 | MVP base | 8 |
| HU-04 | MVP base | 3 |
| HU-05 | MVP base | 5 |
| HU-06 | MVP v2 | 5 |
| HU-07 | MVP v2 | 5 |
| HU-08 | MVP v2 | 3 |
| HU-09 | MVP v2 | 5 |
| **Total** | **MVP base + MVP v2** | **42** |

> Nota: Las estimaciones del MVP base se mantienen según el documento original de historias y subtareas. Las estimaciones del MVP v2 se incorporan desde `USER_STORIES_V2.md` y se mantienen alineadas con el PRD vigente.

---

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

Autores: **Nahuel Lemes** y **Santiago Angarita**.
