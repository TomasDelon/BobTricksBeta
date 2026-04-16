# Documentation Reorg Status

## Objetivo de esta reorganización

Dejar la documentación matemática y científica en un estado más limpio y trazable, sin borrar material, y separar con claridad:

- la línea de trabajo activa del rediseño actual,
- el material desplazado por la nueva dirección conceptual,
- la línea `docs_clean/` que deberá consolidarse más adelante.

## Archivos movidos a `docs/trash/`

Los siguientes documentos fueron movidos desde `docs/` a `docs/trash/` por haber quedado superados por la nueva dirección conceptual auditada:

- `docs/trash/00_overview.md`
- `docs/trash/01_geometry.md`
- `docs/trash/02_walk_protocol.md`
- `docs/trash/03_foothold_planner.md`
- `docs/trash/04_swing_generator.md`
- `docs/trash/05_body_reconstruction.md`
- `docs/trash/06_run_protocol.md`
- `docs/trash/00_overview_v2.md`
- `docs/trash/00_overview_v3.md`
- `docs/trash/00_overview_v4.md`

El movimiento se hizo de forma trazable, preservando contenido e historial mediante renames.

## Documentos de trabajo activos en `docs/`

La zona de trabajo activa en `docs/` queda centrada en:

- `docs/09_standing_support_load_acceptance.md`
- `docs/10_support_progression_sigma.md`
- `docs/11_stand_walk_transition_and_step_trigger.md`
- `docs/12_foothold_planner_under_support_framework.md`

Estos documentos reflejan la dirección actual del proyecto:

- standing bilateral explícito,
- support/load acceptance,
- progresión de soporte con sigma,
- transición stand `<->` walk,
- planner entendido como support acquisition planner.

## Estado de `docs_clean/`

Documentos relativamente más estables:

- `docs_clean/05_body_reconstruction.md`
- `docs_clean/06_run_protocol.md`

Documentos prioritarios para reescritura o consolidación futura:

- `docs_clean/README.md`
- `docs_clean/00_overview.md`
- `docs_clean/01_geometry.md`
- `docs_clean/02_walk_protocol.md`
- `docs_clean/03_foothold_planner.md`
- `docs_clean/04_swing_generator.md`
- `docs_clean/07_prediction_layer.md`
- `docs_clean/08_auxiliary_variables.md`

## Orden sugerido de consolidación futura

1. `docs_clean/01_geometry.md`
2. `docs_clean/02_walk_protocol.md`
3. `docs_clean/03_foothold_planner.md`
4. `docs_clean/04_swing_generator.md`
5. `docs_clean/07_prediction_layer.md`
6. `docs_clean/08_auxiliary_variables.md`
7. `docs_clean/00_overview.md`
8. `docs_clean/README.md`

## Notas de alcance

- Esta reorganización no reescribe teoría ni introduce teoría nueva.
- `docs_clean/` no fue enviado a `trash`.
