# Reality Check - Optimizador de Envíos

Este documento relaciona las historias de usuario definidas en [USER_STORIES.md](USER_STORIES.md) con la estimación asignada en [SUBTASKS.md](SUBTASKS.md), contrastando lo previsto con la complejidad real observada durante la ejecución.

## Criterio de lectura

- **Bien estimada:** el esfuerzo real fue acorde a lo planificado.
- **Ligeramente subestimada:** hubo mayor complejidad que la prevista, pero sin romper la estimación de forma crítica.
- **Mal estimada:** el esfuerzo real fue claramente superior a lo previsto.

## Resumen general

En términos generales, las historias estuvieron bien estimadas. Las principales desviaciones aparecieron cuando fue necesario integrar servicios externos, resolver contratos de autenticación o modelar flujos con más dependencias de las que se veían en la fase de análisis.

- **Bien estimadas:** HU-02, HU-04, HU-05, HU-09.
- **Ligeramente subestimadas:** HU-01, HU-03, HU-06, HU-07, HU-08.
- **Mal estimadas:** ninguna de forma evidente.

## Análisis por historia

| HU | Estimación original | Valoración real | Comentario |
| --- | ---: | --- | --- |
| HU-01 | 5 | Ligeramente subestimada | La estimación estuvo cerca, pero el trabajo real fue un poco mayor por la integración con OpenRouteService para autocompletado y cálculo de distancia, además de las validaciones de cobertura en Colombia y normalización de peso. El alcance general era correcto, aunque el componente geográfico añadió más complejidad de la esperada. |
| HU-02 | 3 | Bien estimada | La definición de prioridad fue directa y el esfuerzo real coincidió con lo previsto. La historia se centró en extender el modelo, validar obligatoriedad y persistir la prioridad en memoria, sin dependencias técnicas relevantes. |
| HU-03 | 8 | Ligeramente subestimada | Aunque la historia estaba planteada como compleja, el esfuerzo real fue mayor por la dificultad de encontrar APIs reales de transportadoras con acceso simple. Al no existir una integración externa viable sin registro empresarial, se optó por datos mockeados. Además, el motor de recomendación requirió más trabajo por el uso combinado de Adapter, Strategy y Factory, junto con las reglas de desempate. |
| HU-04 | 3 | Bien estimada | El trabajo de alternativas fue consistente con la estimación. Se reutilizó la respuesta de recomendación para mostrar opciones adicionales y la complejidad fue principalmente de interfaz, sin introducir lógica de negocio nueva de peso significativo. |
| HU-05 | 5 | Bien estimada | La historia implicó confirmación, validaciones, persistencia y navegación, pero el tamaño esperado fue adecuado. El esfuerzo fue medio y se mantuvo dentro del rango previsto, sin sorpresas técnicas importantes. |
| HU-06 | 5 | Ligeramente subestimada | La integración del mapa con Leaflet y la visualización de la ruta demandaron más ajuste del esperado, sobre todo por la conversión de coordenadas, el manejo del formato de la geometría y la sincronización con los datos devueltos por el flujo de recomendación. El uso de OpenRouteService añadió fricción técnica adicional. |
| HU-07 | 5 | Ligeramente subestimada | El registro de usuario parecía un flujo estándar, pero la complejidad real subió por la necesidad de alinear validaciones, persistencia, hash de contraseña y contrato con el servicio de autenticación. El trabajo se extendió más de lo planeado por el encaje con la base de seguridad que luego consumiría el login. |
| HU-08 | 3 | Ligeramente subestimada | La estimación fue razonable para un login básico, pero el esfuerzo real creció por la generación y manejo del JWT, el control de claims, la gestión de errores de autenticación y la restauración de sesión en frontend. No llegó a ser una desviación grave, pero sí requirió más precisión de la prevista. |
| HU-09 | 5 | Bien estimada | La consulta de pedidos por usuario estuvo bien dimensionada. El trabajo principal fue asegurar el aislamiento por `userId`, exponer el historial y manejar estados de vacío o error en frontend. La complejidad funcional fue moderada y quedó alineada con la estimación. |

## Conclusión

La estimación global fue bastante acertada. Las historias con dependencia de servicios externos, contratos de autenticación y lógica de recomendación concentraron la mayor parte de la complejidad adicional, pero no hubo una desviación tan alta como para clasificar alguna historia como mal estimada.

En particular, los puntos que más impactaron el esfuerzo real fueron:

- la integración con OpenRouteService en las historias de registro y visualización de ruta;
- la búsqueda de fuentes reales para proveedores de transporte en la historia de recomendación;
- la implementación del flujo de autenticación con generación y uso de JWT;
- la coordinación entre frontend y backend en los flujos persistidos por usuario.

Esto confirma que las historias base quedaron bien dimensionadas, pero también que las dependencias externas y la seguridad suelen aumentar la complejidad real incluso cuando el alcance funcional parece acotado.