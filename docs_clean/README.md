# docs_clean

Esta carpeta contiene la línea de documentación matemática y científica destinada a consolidación canónica.

## Estructura documental actual

- `docs/` contiene la zona de trabajo activa y el archivo documental operativo del rediseño actual.
- `docs/trash/` conserva documentación desplazada por la nueva dirección conceptual sin borrar contenido ni romper trazabilidad.
- `docs_clean/` contiene la línea canónica en proceso de consolidación, donde los documentos se van reescribiendo para reflejar de forma coherente la arquitectura `CM-first` centrada en soporte.

## Documentos activos en `docs/`

Los documentos de trabajo actualmente alineados con la dirección fuerte del proyecto son:

- `docs/09_standing_support_load_acceptance.md`
- `docs/10_support_progression_sigma.md`
- `docs/11_stand_walk_transition_and_step_trigger.md`
- `docs/12_foothold_planner_under_support_framework.md`

Estos documentos siguen siendo la referencia de trabajo viva cuando una decisión todavía no está completamente estabilizada en la línea clean.

## Estado actual de `docs_clean/`

### Núcleo actualmente bastante coherente

Los siguientes documentos ya están relativamente bien alineados entre sí bajo la arquitectura actual:

- `docs_clean/00_overview.md`
- `docs_clean/01_geometry.md`
- `docs_clean/02_walk_protocol.md`
- `docs_clean/03_foothold_planner.md`
- `docs_clean/04_swing_generator.md`
- `docs_clean/05_body_reconstruction.md`
- `docs_clean/06_run_protocol.md`
- `docs_clean/07_prediction_layer.md`
- `docs_clean/08_auxiliary_variables.md`

En particular, esta línea clean ya incorpora de manera bastante consistente:

- soporte definido por trayectoria de contacto,
- talón pequeño positivo,
- variable $\sigma$ basada en trayectoria,
- variable bilateral $\beta$ reutilizada en standing, gait initiation y load acceptance,
- planner entendido como support-acquisition planner,
- separación entre contacto, load acceptance y dominancia de soporte.

### Qué sigue abierto

Aunque la coherencia global ha mejorado mucho, `docs_clean/` no debe considerarse una línea totalmente cerrada.

Todavía quedan abiertas futuras consolidaciones sobre:

- transiciones walk-to-run y run-to-walk,
- jump, landing y protocolos de mayor energía,
- refinamiento de leyes concretas de predicción, soporte y reactividad,
- auditorías cruzadas adicionales entre documentos cuando cambien decisiones de alto nivel.

## Criterio editorial

- No mezclar reorganización con reescritura teórica profunda.
- Usar `docs/` para capturar la línea de trabajo viva.
- Usar `docs_clean/` como destino de consolidación canónica cuando la notación, las dependencias y la estructura conceptual estén suficientemente auditadas.
- Cuando haya conflicto entre una formulación vieja de `docs_clean/` y la nueva línea fuerte del proyecto, la consolidación debe mover `docs_clean/` hacia la arquitectura support-centered, no al revés.

## Regla práctica de lectura

La lectura recomendada del backbone actual es:

1. `00_overview.md`
2. `01_geometry.md`
3. `02_walk_protocol.md`
4. `03_foothold_planner.md`
5. `04_swing_generator.md`
6. `05_body_reconstruction.md`
7. `06_run_protocol.md`
8. `07_prediction_layer.md`
9. `08_auxiliary_variables.md`

Ese orden refleja bastante bien la dependencia conceptual actual del proyecto.
