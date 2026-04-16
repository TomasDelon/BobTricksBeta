# 05 — Exact body reconstruction

## 0. Goal of this document

The goal of this document is to define the exact reconstruction of the visible body from the authoritative locomotion state.

This document fixes:

- the reconstruction forward axis,
- the torso-angle law,
- exact pelvis and torso reconstruction,
- exact stance-knee and swing-knee reconstruction,
- knee-branch selection,
- visible stance-foot and swing-foot orientation,
- exact frame validity conditions,
- the rule that impossible frames must be rejected upstream rather than faked in rendering.

## 1. Fundamental reconstruction principle

The visible body is reconstructed from the authoritative locomotion state.

The opposite causal order is forbidden.

Thus the reconstruction map has the form

$$
(C, V, \theta, \mathcal{M}, A_L, A_R, Q, t_{\mathrm{sup}}, n_{\mathrm{sup}})
\longrightarrow
\text{visible frame}.
$$

This means:

- the body must never contradict the CM state,
- the renderer must never invent geometry,
- no rendered frame may violate rigid segment lengths.

## 2. Reconstruction inputs

At a generic time, the reconstruction module receives at least:

- the virtual center of mass $C$,
- the CM velocity $V = \dot C$,
- the reactive intensity $\rho$,
- the current locomotion mode $\mathcal{M}$,
- the left and right ankle positions $A_L$ and $A_R$,
- the current or predicted reconstruction tangent,
- the current or predicted reconstruction normal,
- the current swing ankle if one foot is in swing,
- the relevant foot directions.

The key point is that body reconstruction is mode-aware, not walking-only.

## 3. Reconstruction forward axis

### 3.1. Why a reconstruction axis is needed

Several parts of the reconstruction require a coherent notion of forward:

- torso lean,
- knee-branch choice,
- visible swing-foot interpolation.

But in flight there may be no active support frame.

Therefore one must distinguish the support tangent from the reconstruction-forward tangent.

### 3.2. Definition of the reconstruction tangent

Define the reconstruction-forward tangent by

$$
t_{\mathrm{rec}}.
$$

It is the canonical forward unit tangent used by reconstruction.

When an active support frame exists, set

$$
t_{\mathrm{rec}} = f\, t_{\mathrm{sup}}.
$$

When no active support frame exists, as in flight, use either:

- the last active takeoff reconstruction tangent,
- or a blended tangent toward the predicted touchdown support tangent.

The corresponding reconstruction normal is

$$
n_{\mathrm{rec}} = (-t_{{\mathrm{rec}},y}, t_{{\mathrm{rec}},x}).
$$

This closes an important architectural gap: body reconstruction must remain well defined even when the support frame is temporarily absent.

## 4. Torso direction vector

Let the torso angle measured from the global vertical be denoted by $\theta$.

Define

$$
e_{\theta} = (\sin\theta, \cos\theta).
$$

This means:

- $\theta = 0$ is vertical torso,
- positive $\theta$ leans the torso toward the current reconstruction forward direction,
- negative $\theta$ leans it backward relative to that convention.

## 5. Target torso angle

### 5.1. Stance-based target law

When a support frame exists, the torso angle should depend on:

- forward tangential speed,
- support slope,
- support-normal CM height,
- reactive intensity.

Let

$$
v_f = V \cdot t_{\mathrm{rec}}.
$$

Let the support slope angle be

$$
\alpha_{\mathrm{sup}} = \mathrm{atan2}(t_{{\mathrm{sup}},y}, t_{{\mathrm{sup}},x}).
$$

Recall that if a support frame exists then

$$
C = Q + s_Q t_{\mathrm{sup}} + z_Q n_{\mathrm{sup}}.
$$

A first stance-based torso target law is

$$
\theta^{\star} = k_v \tanh\left(\frac{v_f}{v_{\mathrm{ref}}}\right) + k_{\alpha}\alpha_{\mathrm{sup}} + k_z \frac{z_Q - z_{\mathrm{ref}}}{L} + k_{\rho}\rho.
$$

### 5.2. Flight-based target law

When no support frame exists, the stance-based variables are unavailable.

In that case define a flight target law using the reconstruction frame:

$$
\theta_{\mathrm{flight}}^{\star} = k_v^{\mathrm{fl}} \tanh\left(\frac{v_f}{v_{\mathrm{ref}}^{\mathrm{fl}}}\right) + k_{\rho}^{\mathrm{fl}}\rho.
$$

So the torso target remains well defined even in flight.

### 5.3. Clamped and filtered torso angle

The target torso angle must remain bounded:

$$
\theta_{\mathrm{cmd}}^{\star} = \mathrm{clamp}(\theta^{\star}, \theta_{\min}, \theta_{\max}).
$$

The actual torso angle may then be filtered by

$$
\dot \theta = \frac{\theta_{\mathrm{cmd}}^{\star} - \theta}{\tau_{\theta}}.
$$

This keeps reconstruction continuous in time.

## 6. Exact pelvis and torso reconstruction

Once $C$ and $\theta$ are known, the pelvis is reconstructed exactly by

$$
P = C - 0.75L e_{\theta}.
$$

The torso center is

$$
T_c = P + L e_{\theta}.
$$

The upper torso node is

$$
T_t = P + 2L e_{\theta}.
$$

Therefore the torso constraints are exact:

$$
\|C-P\| = 0.75L,
$$

$$
\|P-T_c\| = L,
$$

$$
\|T_c-T_t\| = L.
$$

## 7. Exact stance-leg reconstruction

Let the current stance ankle be $A_{\mathrm{st}}$.

Rigid stance-leg reconstruction is possible if and only if

$$
\|P-A_{\mathrm{st}}\| \le 2L.
$$

Assume

$$
0 < \|P-A_{\mathrm{st}}\| < 2L.
$$

Define

$$
d_{\mathrm{st}} = \|P-A_{\mathrm{st}}\|,
$$

$$
M_{\mathrm{st}} = \frac{A_{\mathrm{st}} + P}{2},
$$

$$
\nu_{\mathrm{st}} = \frac{P-A_{\mathrm{st}}}{d_{\mathrm{st}}},
$$

$$
\nu_{\mathrm{st}}^{\perp} = (-\nu_{{\mathrm{st}},y}, \nu_{{\mathrm{st}},x}),
$$

$$
h_{\mathrm{st}} = \sqrt{L^2 - \left(\frac{d_{\mathrm{st}}}{2}\right)^2 }.
$$

Then the exact stance-knee candidates are

$$
K_{\mathrm{st}}^{\pm} = M_{\mathrm{st}} \pm h_{\mathrm{st}} \nu_{\mathrm{st}}^{\perp}.
$$

## 8. Exact swing-leg reconstruction

Let the current swing ankle be $A_{\mathrm{sw}}$.

Rigid swing-leg reconstruction is possible if and only if

$$
\|P-A_{\mathrm{sw}}\| \le 2L.
$$

Assume

$$
0 < \|P-A_{\mathrm{sw}}\| < 2L.
$$

Define

$$
d_{\mathrm{sw}} = \|P-A_{\mathrm{sw}}\|,
$$

$$
M_{\mathrm{sw}} = \frac{A_{\mathrm{sw}} + P}{2},
$$

$$
\nu_{\mathrm{sw}} = \frac{P-A_{\mathrm{sw}}}{d_{\mathrm{sw}}},
$$

$$
\nu_{\mathrm{sw}}^{\perp} = (-\nu_{{\mathrm{sw}},y}, \nu_{{\mathrm{sw}},x}),
$$

$$
h_{\mathrm{sw}} = \sqrt{L^2 - \left(\frac{d_{\mathrm{sw}}}{2}\right)^2 }.
$$

Then the exact swing-knee candidates are

$$
K_{\mathrm{sw}}^{\pm} = M_{\mathrm{sw}} \pm h_{\mathrm{sw}} \nu_{\mathrm{sw}}^{\perp}.
$$

## 9. Knee-branch selection

### 9.1. Need for a deterministic branch rule

Exact two-link IK yields two valid knees for each leg.

Only one should be selected for the visible body.

Therefore reconstruction requires a deterministic branch-selection rule.

### 9.2. Forward-side convention

Using the reconstruction frame, define

$$
n_{\mathrm{rec}} = (-t_{{\mathrm{rec}},y}, t_{{\mathrm{rec}},x}).
$$

For a candidate knee $K$ associated with ankle $A$ and pelvis $P$, define

$$
M = \frac{A+P}{2}.
$$

Then define the branch-sign quantity

$$
\chi(K;A,P) = (K-M)\cdot n_{\mathrm{rec}}.
$$

The chosen branch is the one whose sign matches the global visual convention of the project.

### 9.3. Temporal continuity

If both branch candidates satisfy the same sign convention because of a degenerate or nearly degenerate configuration, prefer the one minimizing discontinuity with respect to the previous frame.

If the previous knee position is $K_{\mathrm{prev}}$, select the admissible branch minimizing

$$
\|K^{\pm} - K_{\mathrm{prev}}\|.
$$

This prevents visible knee flipping.

## 10. Exact knee angle

For any leg with pelvis-to-ankle distance $d$, the internal knee angle satisfies

$$
d^2 = L^2 + L^2 - 2L^2 \cos \beta.
$$

Therefore

$$
\cos \beta = \frac{2L^2 - d^2}{2L^2}.
$$

Hence

$$
\beta = \arccos\left(\frac{2L^2 - d^2}{2L^2}\right).
$$

This formula gives an exact diagnostic of crouch.

If a nominal walking frame systematically produces a stance knee angle far below $\pi$, then the stance is too flexed and the nominal pendular geometry is not being respected.

## 11. Visible stance-foot orientation

The visible stance-foot direction must be determined from terrain-supported foot geometry.

If the stance foot lies on one segment, the visible foot direction follows the admissible terrain-supported stance orientation.

If the stance foot spans two admissible segments, the visible stance-foot direction must be derived from the effective heel and toe points of the support path.

Thus the visible stance-foot direction need not coincide exactly with $t_{\mathrm{sup}}$.

## 12. Visible swing-foot orientation

Let the visible liftoff swing-foot direction be

$$
f_{\mathrm{foot},0},
$$

and the target touchdown foot direction be

$$
f_{\mathrm{foot},1}.
$$

Using the same quintic blend $h(\tau)$ as in the swing generator, define

$$
\widetilde f_{\mathrm{foot}}(\tau) = (1-h(\tau))f_{\mathrm{foot},0} + h(\tau)f_{\mathrm{foot},1}.
$$

Then define the normalized swing-foot direction

$$
f_{\mathrm{foot,sw}}(\tau) = \frac{\widetilde f_{\mathrm{foot}}(\tau)}{\|\widetilde f_{\mathrm{foot}}(\tau)\|}.
$$

This gives a continuous visible swing-foot orientation.

## 13. Visible heel and toe points

The canonical visible foot interval is

$$
[-L_{\mathrm{heel}}, L_{\mathrm{toe}}],
$$

with small positive heel length.

Once the foot directions are known, the visible stance-foot endpoints are

$$
H_{\mathrm{st}} = A_{\mathrm{st}} - L_{\mathrm{heel}} f_{\mathrm{foot,st}},
$$

$$
T_{\mathrm{st}} = A_{\mathrm{st}} + L_{\mathrm{toe}} f_{\mathrm{foot,st}}.
$$

Likewise the visible swing-foot endpoints are

$$
H_{\mathrm{sw}} = A_{\mathrm{sw}} - L_{\mathrm{heel}} f_{\mathrm{foot,sw}},
$$

$$
T_{\mathrm{sw}} = A_{\mathrm{sw}} + L_{\mathrm{toe}} f_{\mathrm{foot,sw}}.
$$

Thus visible foot geometry is consistent with the canonical small-positive-heel model already adopted by the clean support geometry.

## 14. Full visible frame

A full visible frame is the node set

$$
\mathcal{N}_{\mathrm{frame}} = \{ C, P, T_c, T_t, A_L, A_R, K_L, K_R, T_L, T_R \}.
$$

It is valid only if all rigid constraints and contact constraints hold.

## 15. Exact frame validity conditions

A rendered frame is valid only if all the following hold.

### 15.1. Torso constraints

$$
\|C-P\| = 0.75L,
$$

$$
\|P-T_c\| = L,
$$

$$
\|T_c-T_t\| = L.
$$

### 15.2. Leg constraints

For each leg $i \in \{L,R\}$,

$$
\|A_i-K_i\| = L,
$$

$$
\|K_i-P\| = L.
$$

### 15.3. Stance-contact consistency

If a foot is in stance, its ankle must lie on its support terrain path and its visible foot geometry must be admissible on the terrain.

### 15.4. Swing consistency

If a foot is in swing, its ankle must lie on the swing path and the visible swing foot must satisfy the swing-geometry constraints.

No frame may violate these conditions.

## 16. What to do when exact reconstruction becomes impossible

If for one leg

$$
\|P-A_i\| > 2L,
$$

then exact rigid IK for that leg does not exist.

This is not a rendering problem.

It means the locomotion state itself is incompatible with the rigid morphology.

Likewise, if stance support or swing-foot geometry becomes inadmissible, the correct response is not to stretch the body visually.

The correct response is to propagate the failure upstream and change:

- CM target,
- step timing,
- foothold selection,
- support-transfer policy,
- or mode transition.

The renderer must never repair an impossible locomotion state by cheating on lengths.

## 17. Main conclusions of this document

1. Body reconstruction must be a deterministic consequence of the authoritative locomotion state.

2. The torso angle is a structural variable, not a cosmetic variable.

3. A reconstruction-forward tangent $t_{\mathrm{rec}}$ must exist even when the support frame is inactive.

4. The pelvis and torso nodes are reconstructed exactly from $C$ and $\theta$.

5. Both knees are reconstructed by exact closed-form two-link IK.

6. Knee-branch selection must satisfy both a global convention and temporal continuity.

7. Visible foot directions must remain distinct from effective support tangents.

8. Visible foot geometry must be consistent with the canonical small-positive-heel model.

9. No rendered frame may violate any rigid segment length.

10. If exact reconstruction fails, the locomotion state must be corrected upstream rather than repaired in rendering.