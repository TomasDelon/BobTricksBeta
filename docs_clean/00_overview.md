# 00 — Canonical overview of the `CM-first` locomotion model

## 0. Role of this document

This file is the canonical overview of the mathematical documentation.

Its role is deliberately limited.

It does **not** redevelop all derivations in detail. It only fixes:

- the global principles,
- the common notation,
- the list of active documents,
- the architectural commitments that all later files must respect.

All detailed derivations belong to the specialized documents.

## 1. Active documentation map

The active clean documentation set is the following.

- `docs_clean/00_overview.md`
- `docs_clean/01_geometry.md`
- `docs_clean/02_walk_protocol.md`
- `docs_clean/04_swing_generator.md`
- `docs_clean/05_body_reconstruction.md`

The intended reading order is exactly the numbering order.

## 2. Global principles

The whole project is organized around the following principles.

1. There exists exactly one authoritative center of mass, denoted $C$.

2. This $C$ is a virtual state variable. It is not recomputed from rendered body-part masses.

3. The visible body is reconstructed from the locomotion state. It never defines that state.

4. The terrain is a segmented polyline, not a globally smooth curve.

5. The ankle is the reference node of each leg and of each foot.

6. The body morphology is rigid: no rendered frame may violate segment lengths.

7. Walking and running are two distinct hybrid protocols built on the same geometric and state architecture.

8. Recovery must emerge from the same formalism through continuous deformation of admissible sets, target lengths, timing, and costs.

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

## 5. Facing sign and local support geometry

The character facing direction is encoded by a sign

$$
f \in \{-1,+1\}.
$$

The value $f=+1$ means facing right, and $f=-1$ means facing left.

A stance or landing support defines:

- an ankle $A$,
- an effective support point $Q$,
- an effective support tangent $t_{\mathrm{sup}}$,
- an effective support normal $n_{\mathrm{sup}}$.

The CM may then be decomposed as

$$
C = Q + s_Q t_{\mathrm{sup}} + z_Q n_{\mathrm{sup}}.
$$

These local support coordinates are the natural coordinates for support logic, capture, and gait-phase reasoning.

## 6. Terrain model

The terrain is a finite ordered polyline

$$
\Gamma = \bigcup_{j=1}^{N} S_j,
$$

with

$$
S_j = [V_j, V_{j+1}].
$$

Each segment has a constant tangent $t_j$, a constant normal $n_j$, and a slope angle

$$
\alpha_j = \mathrm{atan2}(t_{j,y}, t_{j,x}).
$$

Vertices are genuine geometric events.

The model must explicitly handle:

- one-segment support,
- two-segment support,
- short support segments,
- slope discontinuities,
- step-like transitions.

## 7. Foot model

The ankle is the reference point of the foot.

A visible foot direction is denoted by $f_{\mathrm{foot}}$.

The current foot interval is

$$
\mathcal{F}(A, f_{\mathrm{foot}})
=
\{A + \sigma f_{\mathrm{foot}} : \sigma \in [0, L_{\mathrm{toe}}]\},
$$

because at the current stage

$$
L_{\mathrm{heel}} = 0.
$$

The visible foot direction and the effective support tangent are related but not identical objects.

- $f_{\mathrm{foot}}$ is used for visible reconstruction.
- $t_{\mathrm{sup}}$ is used for support dynamics and CM decomposition.

This distinction is structurally important.

## 8. Canonical document responsibilities

The specialized documents are responsible for the following.

### Geometry

`01_geometry.md` defines:

- the terrain polyline rigorously,
- one-segment and two-segment foot geometry,
- admissibility of a visible foot spanning two segments,
- effective support point and tangent,
- exact two-link leg IK,
- exact IK-feasible regions.

### Walk

`02_walk_protocol.md` defines:

- the hybrid walk modes,
- the nominal walking stance manifold,
- the local pendular geometry,
- the walking support transfer,
- the reactive walking deformation through $\rho$.

### Swing

`04_swing_generator.md` defines:

- the swing ankle path,
- clearance,
- collision avoidance,
- touchdown, early landing, delayed landing,
- consistency with the future support frame.

### Reconstruction

`05_body_reconstruction.md` defines:

- the torso-angle law,
- exact pelvis and torso reconstruction,
- exact knee reconstruction,
- visible foot orientation,
- frame validity conditions.

## 9. Main architectural commitments

The following commitments are binding for the whole documentation.

### 9.1. No fake geometry in rendering

If a locomotion state cannot be reconstructed with exact rigid segment lengths, the renderer must not cheat.

The correct response is to modify the locomotion state upstream.

### 9.2. Feasibility sets are geometric, not prisons

IK-feasible sets such as

$$
\|P-A\| \le 2L
$$

are exact geometric facts.

They must not be confused with tiny nominal boxes in which the CM should remain artificially trapped at all times.

Large perturbations must be able to drive the system away from nominal gait and trigger recovery.

### 9.3. Walk and run share architecture, not stance law

The following are shared across gait regimes:

- authoritative CM,
- rigid morphology,
- terrain geometry,
- ankle-based foot model,
- exact IK,
- foothold planning architecture,
- swing-generation architecture,
- body reconstruction architecture,
- reactive intensity $\rho$.

The following change with gait:

- the hybrid mode set,
- the stance law,
- the support-transfer law,
- the presence or absence of flight.

## 10. What is intentionally deferred

This overview does not fix the full mathematical treatment of:

- the exact evolution law of the foot-rocker scalar $\sigma$,
- the exact evolution law of the double-support transfer scalar $\beta$,
- the detailed prediction operator used by foothold planning,
- gait transitions,
- jump,
- landing.

Those points either already belong to specialized documents or should become dedicated documents in the next consolidation stage.

## 11. Main conclusion

This documentation is now organized around one stable backbone:

$$
\text{Terrain}
\longrightarrow
\text{Support geometry}
\longrightarrow
\text{CM dynamics}
\longrightarrow
\text{Foothold planning}
\longrightarrow
\text{Swing}
\longrightarrow
\text{Exact body reconstruction}.
$$

Everything else must remain compatible with this order.
