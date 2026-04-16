# 00 — Canonical overview of the `CM-first` locomotion model

## 0. Role of this document

This file is the canonical overview of the mathematical documentation.

Its role is deliberately limited.

It does **not** redevelop all derivations in detail. It only fixes:

- the global principles,
- the common notation,
- the current documentary map,
- the architectural commitments that all later files must respect,
- the relation between the stable clean line and the newer support-centered working line.

All detailed derivations belong to the specialized documents.

## 1. Current documentary state

The repository is intentionally split into three documentary zones.

### 1.1. `docs_clean/`

This folder is the canonical line intended for long-term consolidation.

At the present stage, it already contains the main mathematical backbone:

- `docs_clean/00_overview.md`
- `docs_clean/01_geometry.md`
- `docs_clean/02_walk_protocol.md`
- `docs_clean/03_foothold_planner.md`
- `docs_clean/04_swing_generator.md`
- `docs_clean/05_body_reconstruction.md`
- `docs_clean/06_run_protocol.md`
- `docs_clean/07_prediction_layer.md`
- `docs_clean/08_auxiliary_variables.md`

Most of these files are now reasonably aligned with the support-centered direction of the project.

The clean line should therefore no longer be read as mostly legacy material waiting for rewrite.

It should be read as a largely coherent canonical backbone that still leaves some open consolidation work on higher-level transitions, jump, landing, and future refinements.

### 1.2. `docs/`

This folder is the active working area for the current redesign.

The working documents currently carrying the strongest architectural direction are:

- `docs/09_standing_support_load_acceptance.md`
- `docs/10_support_progression_sigma.md`
- `docs/11_stand_walk_transition_and_step_trigger.md`
- `docs/12_foothold_planner_under_support_framework.md`

These files are not yet the final clean line, but they express the most recent project-level commitments on support, standing, gait initiation, and support acquisition.

### 1.3. `docs/trash/`

This folder stores displaced documents that were moved out of the active line for clarity, while preserving content and traceability.

## 1.1. Current alignment of `docs_clean/`

Most of these files are now reasonably aligned with the support-centered direction of the project.

The clean line should therefore no longer be read as mostly legacy material waiting for rewrite.

It should be read as a largely coherent canonical backbone that still leaves some open consolidation work on higher-level transitions, jump, landing, and future refinements.

## 2. Global principles

The whole project is organized around the following principles.

1. There exists exactly one authoritative center of mass, denoted $C$.

2. This $C$ is a virtual state variable. It is not recomputed from rendered body-part masses.

3. The visible body is reconstructed from the locomotion state. It never defines that state.

4. The terrain is a segmented polyline, not a globally smooth curve.

5. The ankle is the reference node of each leg and of each foot.

6. The body morphology is rigid: no rendered frame may violate segment lengths.

7. Walking, standing, and running are distinct hybrid protocols built on the same geometric and state architecture.

8. Recovery must emerge from the same formalism through continuous deformation of admissible sets, target lengths, timing, and costs.

9. A valid contact is not yet a dominant support. Contact acquisition, load acceptance, and support dominance are distinct notions.

10. The support state is primary. Rendering must follow support-aware locomotion state, not compensate for an incoherent support model.

## 3. Global frame and gravity

We work in the sagittal plane.

The world frame is

$$
W = (O, e_X, e_Y), \qquad e_X = (1,0), \qquad e_Y = (0,1).
$$

The $X$ axis is horizontal and the $Y$ axis is vertical upward.

Gravity is

$$
g_W = (0,-g), \qquad g > 0.
$$

We use virtual unit mass:

$$
m = 1.
$$

Therefore the authoritative CM dynamics are always written as

$$
\ddot C = g_W + u.
$$

The control term $u$ is interpreted as a virtual control acceleration.

## 4. Rigid morphology

The body is parameterized by a unique length unit $L$.

The current convention is

$$
H = 5L.
$$

The rigid segment lengths are

$$
\|A_i-K_i\| = L,
$$

$$
\|K_i-P\| = L,
$$

$$
\|P-T_c\| = L,
$$

$$
\|T_c-T_t\| = L,
$$

$$
\|C-P\| = 0.75L.
$$

Here:

- $A_i$ is an ankle,
- $K_i$ is a knee,
- $P$ is the pelvis,
- $T_c$ is the torso center,
- $T_t$ is the upper torso node,
- $C$ is the unique virtual center of mass.

A direct consequence is

$$
\|P-A_i\| \le 2L.
$$

So rigid two-link leg IK is possible if and only if the pelvis-to-ankle distance stays below or at $2L$.

## 5. Terrain model

The terrain is a finite ordered polyline

$$
\Gamma = \bigcup_{j=1}^{N} S_j,
$$

with

$$
S_j = [V_j, V_{j+1}].
$$

Each segment has a constant tangent $t_j$, a constant normal $n_j$, and a slope angle determined by that tangent.

Vertices are genuine geometric events.

The model must explicitly handle:

- one-segment support,
- two-segment support,
- short support segments,
- slope discontinuities,
- steep but still admissible hills.

At the current project stage, the main terrain target is uneven piecewise-linear hills with segment angles below ninety degrees. Stairs are not yet a defining design constraint.

## 6. Foot and support model

The ankle is the reference point of the foot.

A visible foot direction is denoted by $f_{\mathrm{foot}}$.

The current project-level direction is to use a **small but positive heel length**, not a strictly zero heel length. Therefore the intended active foot interval is

$$
[-L_{\mathrm{heel}}, L_{\mathrm{toe}}],
$$

with

$$
0 < L_{\mathrm{heel}} \ll L.
$$

This is a project-level commitment already adopted in the active working line, even though some older clean files still need to be updated to reflect it.

The key support-centered decision is the following.

The effective support point must not be defined by a purely world-space average. Instead, each active foot induces a contact path

$$
\chi(\sigma), \qquad \sigma \in [-L_{\mathrm{heel}}, L_{\mathrm{toe}}],
$$

and the effective support point is defined from the support coordinate itself:

$$
Q = \chi(\sigma).
$$

Thus the support coordinate $\sigma$ is primary, and $Q$ is derived from it.

This is the correct architectural interpretation of support progression in the current model.

## 7. Facing sign and local support geometry

The character facing direction is encoded by a sign

$$
f \in \{-1,+1\}.
$$

The value $f=+1$ means facing right, and $f=-1$ means facing left.

A support state defines:

- one or two active ankles,
- one or two foot-specific support points,
- an effective support point $Q$,
- an effective support tangent $t_{\mathrm{sup}}$,
- an effective support normal $n_{\mathrm{sup}}$.

The CM may then be decomposed as

$$
C = Q + s_Q t_{\mathrm{sup}} + z_Q n_{\mathrm{sup}}.
$$

These local support coordinates are the natural coordinates for support logic, capture, and gait-phase reasoning.

## 8. Standing, walking, and running

The architecture must now be read in three layers.

### 8.1. Standing

Standing is **not** walking with zero speed.

Standing is an explicit bilateral-support regime in which both feet remain in ground contact and support allocation remains internal to a bilateral support state.

The current working line models this through:

- explicit bilateral support,
- foot-specific support coordinates,
- a support-allocation scalar $\beta$,
- viability-based step triggering.

### 8.2. Walking

Walking remains a hybrid gait with alternating support phases, but it must now be interpreted through the support-centered framework.

This means that walking is no longer just:

- single support,
- simple double support,
- step switch.

It must instead be understood together with:

- explicit support progression through $\sigma$,
- finite support transfer through $\beta$,
- standing-to-walking gait initiation,
- load acceptance after touchdown,
- planner outputs interpreted as support acquisition rather than only ankle placement.

### 8.3. Running

Running remains a distinct hybrid protocol from walking.

The same authoritative state architecture is preserved, but the contact protocol changes:

- spring-like stance,
- explicit takeoff,
- explicit flight,
- touchdown into a new stance regime.

## 9. Prediction and auxiliary variables

The clean line now already contains a prediction layer and an auxiliary-variable layer.

### Prediction

`docs_clean/07_prediction_layer.md` defines the prediction operator as a hybrid, event-aware, candidate-conditioned map used by planning and touchdown reasoning.

### Auxiliary variables

`docs_clean/08_auxiliary_variables.md` defines:

- the support progression variable $\sigma$,
- the transfer variable $\beta$,
- the reactive intensity $\rho$.

The newer working documents in `docs/09`–`12` were the source of several support-centered refinements that have now largely been merged back into the clean line, especially for:

- small positive heel length,
- support-path-based definition of $Q$,
- bilateral standing,
- gait initiation,
- finite load acceptance,
- support-acquisition planning.

Those working documents should still be treated as the live design reference when a newer project-level decision has not yet been canonically restated in `docs_clean/`.

## 10. Canonical document responsibilities

The specialized documents are responsible for the following.

### Geometry

`01_geometry.md` should define:

- the terrain polyline rigorously,
- foot geometry with the current heel-to-toe interval,
- support-path construction,
- effective support point and tangent,
- exact two-link leg IK,
- exact IK-feasible regions.

### Walk

`02_walk_protocol.md` should define:

- the hybrid walk modes,
- the nominal walking stance manifold,
- the walking support law,
- support transfer,
- gait initiation and return to standing,
- the reactive walking deformation through $\rho$.

### Foothold planning

`03_foothold_planner.md` should define:

- the support-acquisition planner,
- admissible future contact paths,
- candidate-conditioned prediction at touchdown,
- load-acceptance viability,
- support-centered scoring.

### Swing

`04_swing_generator.md` should define:

- the swing ankle path,
- clearance,
- collision avoidance,
- touchdown,
- early landing and delayed landing,
- consistency with support acquisition and load acceptance.

### Reconstruction

`05_body_reconstruction.md` should define:

- the torso-angle law,
- exact pelvis and torso reconstruction,
- exact knee reconstruction,
- visible foot orientation,
- frame validity conditions.

### Run

`06_run_protocol.md` should define:

- the running stance law,
- takeoff,
- flight,
- touchdown into stance,
- continuity with the shared `CM-first` support architecture.

### Prediction

`07_prediction_layer.md` should define:

- hybrid event-aware prediction,
- candidate-conditioned prediction,
- prediction consistency with support and swing,
- viability failure as a first-class prediction output.

### Auxiliary variables

`08_auxiliary_variables.md` should define:

- the canonical meaning of $\sigma$,
- the canonical meaning of $\beta$,
- the canonical meaning of $\rho$,
- their continuity, resets, and admissibility conditions.

## 11. Main architectural commitments

The following commitments are binding for the whole documentation.

### 11.1. No fake geometry in rendering

If a locomotion state cannot be reconstructed with exact rigid segment lengths, the renderer must not cheat.

The correct response is to modify the locomotion state upstream.

### 11.2. Feasibility sets are geometric, not prisons

IK-feasible sets such as

$$
\|P-A\| \le 2L
$$

are exact geometric facts.

They must not be confused with tiny nominal boxes in which the CM should remain artificially trapped at all times.

Large perturbations must be able to drive the system away from nominal gait and trigger recovery.

### 11.3. Contact, support, and dominance are distinct notions

A valid contact event does not automatically imply dominant support.

The architecture must distinguish at least:

- contact acquisition,
- load acceptance,
- support dominance.

### 11.4. Walk and run share architecture, not stance law

The following are shared across gait regimes:

- authoritative CM,
- rigid morphology,
- terrain geometry,
- ankle-based foot model,
- exact IK,
- foothold planning architecture,
- swing-generation architecture,
- body reconstruction architecture,
- prediction architecture,
- reactive intensity $\rho$.

The following change with gait:

- the hybrid mode set,
- the stance law,
- the support-transfer law,
- the presence or absence of flight.

## 12. Present consolidation status

The current situation is now the following.

- The clean line contains a mostly coherent backbone from overview and geometry through walk, planner, swing, reconstruction, run, prediction, and auxiliary variables.
- The active working line in `docs/09`–`12` remains the live source for the strongest recent support-centered decisions and for future extensions not yet fully closed in the clean line.
- The next major clean consolidations should no longer focus on basic support geometry or on basic walking structure, which are now largely aligned.
- The next major clean consolidations should instead focus on higher-level protocol extensions such as walk-to-run transition, run-to-walk transition, jump, landing, and higher-energy capture.

This overview intentionally records that status rather than pretending that the project is already fully closed.

## 13. Main conclusion

The documentation is now organized around one stable backbone:

$$
\text{Terrain}
\longrightarrow
\text{Support geometry}
\longrightarrow
\text{CM dynamics}
\longrightarrow
\text{Prediction and support viability}
\longrightarrow
\text{Foothold planning}
\longrightarrow
\text{Swing and contact acquisition}
\longrightarrow
\text{Load acceptance and support transfer}
\longrightarrow
\text{Exact body reconstruction}.
$$

Everything else must remain compatible with this order.

At the present stage, this backbone is already much closer to a coherent canonical line than to a preliminary draft skeleton.
