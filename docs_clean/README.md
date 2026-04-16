# docs_clean

Esta carpeta contiene la línea **canónica consolidada** de la documentación matemática y científica de BobTricksBeta.

## Regla de uso

- `docs/` = zona de trabajo, iteración, exploración y borradores.
- `docs_clean/` = versión auditada, corregida y coherente.

Un documento entra en `docs_clean/` solo cuando:

1. su notación es coherente con el resto,
2. no introduce duplicados estructurales innecesarios,
3. sus ecuaciones y dependencias están suficientemente estabilizadas,
4. pasó una auditoría conceptual razonable.

## Estado actual

Archivos ya consolidados en `docs_clean/`:

- `00_overview.md`
- `01_geometry.md`
- `02_walk_protocol.md`
- `04_swing_generator.md`
- `05_body_reconstruction.md`

Archivos todavía pendientes de consolidación completa:

- `03_foothold_planner.md`
- `06_run_protocol.md`

La idea es seguir iterando en `docs/`, hacer auditorías, y luego migrar la versión corregida a `docs_clean/`.
