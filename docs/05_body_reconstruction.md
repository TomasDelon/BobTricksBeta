# 05 — Exact body reconstruction

## 0. Goal of this document

The goal of this document is to define the exact reconstruction of the visible body from the authoritative locomotion state.

The previous documents fixed:

- the rigid morphology,
- the segmented terrain,
- the foot geometry,
- exact rigid-leg IK,
- the walking protocol,
- the foothold planner,
- the swing generator.

This document now defines:

- how the torso angle is chosen,
- how the pelvis is reconstructed from the CM,
- how both knees are reconstructed exactly,
- how stance and swing feet are oriented,
- what geometric conditions every rendered frame must satisfy,
- what must happen when the nominal reconstruction becomes incompatible with the current locomotion state.

This document is still formal and mathematical. It does not yet prescribe a specific software architecture beyond the objects already introduced.

## 1. Fundamental principle of reconstruction

The body is not simulated as a set of masses whose center of mass is then observed.

The opposite principle is used.

The authoritative dynamic state contains the virtual CM $C$, the current contact state, the current support geometry, and the current locomotion mode.

The visible body must then be reconstructed from that state.

Thus the causal order is

$$
(C, \theta, A_L, A_R, Q, t_{\mathrm{sup}}, n_{\mathrm{sup}}, \mathcal{M}) \longrightarrow \text{visible body frame}.
$$

This means in particular:

- the body must never contradict the CM state,
- the renderer must never invent geometry,
- no rendered frame is allowed to violate rigid segment lengths.

## 2. Reconstruction inputs

At a generic time, the reconstruction module receives at least the following objects.

### 2.1. Authoritative dynamic inputs

- the virtual center of mass $C$,
- the CM velocity $V = \dot C$,
- the reactive intensity $\rho$,
- the current walking mode $\mathcal{M}$.

### 2.2. Contact inputs

- left ankle position $A_L$,
- right ankle position $A_R$,
- effective support point $Q$,
- effective support tangent $t_{\mathrm{sup}}$,
- effective support normal $n_{\mathrm{sup}}$.

### 2.3. Planning inputs

- the next foothold $A_{\mathrm{next}}^{\star}$,
- the current swing ankle trajectory $A_{\mathrm{sw}}(\tau)$.

### 2.4. Geometric constants

- body unit $L$,
- pelvis–CM offset $0.75L$,
- foot lengths $L_{\mathrm{heel}}$ and $L_{\mathrm{toe}}$.

The output of the reconstruction module must be a full set of visible nodes satisfying all geometric constraints.

## 3. Torso angle

### 3.1. Why the torso angle cannot be arbitrary

The torso angle is not a cosmetic variable.

It enters the rigid reconstruction directly because the pelvis is defined from the CM by the offset

$$
P = C - 0.75L e_\theta.
$$

Therefore any choice of $\theta$ changes:

- pelvis position,
- rigid-leg feasibility,
- knee geometry,
- nominal stance geometry,
- visible posture of the entire body.

So the torso angle must be part of the formal locomotion model.

### 3.2. Torso direction vector

Define

$$
e_\theta = (\sin\theta, \cos\theta).
$$

This convention means:

- $\theta = 0$ corresponds to a vertical torso,
- positive $\theta$ leans the torso forward in the facing direction when $f = +1$,
- negative $\theta$ leans the torso backward relative to the global vertical.

To encode facing symmetry cleanly, it is convenient to define a signed forward tangent along the current support direction:

$$
t_f = f\, t_{\mathrm{sup}}.
$$

Then $t_f$ is always the local forward direction of the character.

## 4. Target torso angle

The torso angle should depend on at least four quantities.

- support-relative speed,
- terrain slope,
- support-normal CM height,
- reactive intensity.

The reason is biomechanical and architectural.

- On flat terrain, a faster walker leans more forward.
- On uphill terrain, the torso tends to lean forward.
- On downhill terrain, the torso tends to lean backward relative to forward motion.
- Under perturbation, torso posture participates in whole-body balance regulation.

### 4.1. Local support slope angle

Let the effective support tangent be

$$
t_{\mathrm{sup}} = (t_x, t_y).
$$

Define the support slope angle by

$$
\alpha_{\mathrm{sup}} = \mathrm{atan2}(t_y, t_x).
$$

### 4.2. Forward tangential speed

Define the forward tangential speed by

$$
v_f = V \cdot t_f.
$$

This is the signed speed of the CM along the current local forward support direction.

### 4.3. Support-normal CM height

Recall that

$$
C = Q + s_Q t_{\mathrm{sup}} + z_Q n_{\mathrm{sup}}.
$$

The value $z_Q$ is the support-normal CM height.

### 4.4. Torso-angle target law

A first formal target law is

$$
\theta^{\star} = k_v \tanh\left(\frac{v_f}{v_{\mathrm{ref}}}\right) + k_\alpha \alpha_{\mathrm{sup}} + k_z \frac{z_Q - z_{\mathrm{ref}}}{L} + k_\rho \rho.
$$

Interpretation of the terms.

The first term increases forward lean with speed.

The second term makes the torso follow the terrain inclination.

The third term lets the torso react to the support-normal CM height.

The fourth term allows a more protective or more balance-oriented torso posture under perturbation.

This law is not yet the final tuning law. It is the correct first mathematical structure.

### 4.5. Torso-angle bounds

The target torso angle must remain inside a biomechanically and visually admissible range.

Therefore define

$$
\theta^{\star}_{\mathrm{clamped}} = \mathrm{clamp}(\theta^{\star}, \theta_{\min}, \theta_{\max}).
$$

The actual torso angle may then be filtered dynamically, for example by a first-order law

$$
\dot \theta = \frac{\theta^{\star}_{\mathrm{clamped}} - \theta}{\tau_\theta}.
$$

This keeps the reconstruction continuous in time.

## 5. Exact pelvis reconstruction

Once the torso angle is known, the pelvis is reconstructed exactly from the CM.

The formula is

$$
P = C - 0.75L e_\theta.
$$

This is a rigid identity, not a soft preference.

Therefore the following quantity must be identically true in every rendered frame:

$$
\|C-P\| = 0.75L.
$$

## 6. Exact torso-node reconstruction

The torso center is reconstructed by

$$
T_c = P + L e_\theta.
$$

The upper torso node is reconstructed by

$$
T_t = P + 2L e_\theta.
$$

Hence the torso constraints are exactly satisfied:

$$
\|P-T_c\| = L,
$$

$$
\|T_c-T_t\| = L.
$$

So once $C$ and $\theta$ are known, the torso geometry is entirely determined.

## 7. Exact stance-leg reconstruction

### 7.1. Stance-leg feasibility condition

Let the current stance ankle be $A_{\mathrm{st}}$.

Rigid stance reconstruction is possible if and only if

$$
\|P-A_{\mathrm{st}}\| \le 2L.
$$

This is the exact two-link IK condition.

### 7.2. Midpoint construction of the stance knee

Assume

$$
0 < \|P-A_{\mathrm{st}}\| < 2L.
$$

Define

$$
d_{\mathrm{st}} = \|P-A_{\mathrm{st}}\|.
$$

Define the midpoint

$$
M_{\mathrm{st}} = \frac{A_{\mathrm{st}} + P}{2}.
$$

Define the unit direction from stance ankle to pelvis

$$
u_{\mathrm{st}} = \frac{P-A_{\mathrm{st}}}{d_{\mathrm{st}}}.
$$

Define its perpendicular unit vector

$$
\nu_{\mathrm{st}}^{\perp} = (-u_{{\mathrm{st}},y}, u_{{\mathrm{st}},x}).
$$

Then the stance knee candidates are

$$
K_{\mathrm{st}}^{\pm} = M_{\mathrm{st}} \pm h_{\mathrm{st}} \nu_{\mathrm{st}}^{\perp},
$$

where

$$
h_{\mathrm{st}} = \sqrt{L^2 - \left(\frac{d_{\mathrm{st}}}{2}\right)^2 }.
$$

Exactly one of these two branches must be chosen according to a global branch convention.

## 8. Exact swing-leg reconstruction

### 8.1. Swing ankle as a known input

The swing ankle is provided by the swing generator.

So at time $t$, the reconstruction module knows

$$
A_{\mathrm{sw}} = A_{\mathrm{sw}}(t).
$$

### 8.2. Swing-leg feasibility condition

Rigid swing-leg reconstruction is possible if and only if

$$
\|P-A_{\mathrm{sw}}\| \le 2L.
$$

This is the exact same condition as for the stance leg, but now applied to the moving swing ankle.

### 8.3. Midpoint construction of the swing knee

Assume

$$
0 < \|P-A_{\mathrm{sw}}\| < 2L.
$$

Define

$$
d_{\mathrm{sw}} = \|P-A_{\mathrm{sw}}\|.
$$

Define the midpoint

$$
M_{\mathrm{sw}} = \frac{A_{\mathrm{sw}} + P}{2}.
$$

Define the unit direction from swing ankle to pelvis

$$
u_{\mathrm{sw}} = \frac{P-A_{\mathrm{sw}}}{d_{\mathrm{sw}}}.
$$

Define its perpendicular unit vector

$$
\nu_{\mathrm{sw}}^{\perp} = (-u_{{\mathrm{sw}},y}, u_{{\mathrm{sw}},x}).
$$

Then the swing knee candidates are

$$
K_{\mathrm{sw}}^{\pm} = M_{\mathrm{sw}} \pm h_{\mathrm{sw}} \nu_{\mathrm{sw}}^{\perp},
$$

where

$$
h_{\mathrm{sw}} = \sqrt{L^2 - \left(\frac{d_{\mathrm{sw}}}{2}\right)^2 }.
$$

Again, exactly one branch must be selected.

## 9. Knee-branch selection

### 9.1. Why the branch matters

For each leg, exact IK yields two mathematically valid knees.

Only one of them corresponds to the chosen anatomical and visual convention.

Therefore body reconstruction requires a deterministic branch-selection rule.

### 9.2. Forward-knee convention

A simple and coherent convention is to choose the knee branch so that the knee points toward the character's local forward side.

Let

$$
t_f = f\, t_{\mathrm{sup}}.
$$

For a candidate knee $K$, define the forward signed area-like quantity

$$
\chi(K; A, P) = (K - M) \cdot n_f,
$$

where

$$
M = \frac{A+P}{2},
$$

and $n_f$ is the normal obtained from $t_f$ by a quarter-turn.

Then the chosen branch is the one whose sign matches the global visual convention.

This is not a theorem of geometry. It is a model convention that turns the exact IK into a unique visible knee.

### 9.3. Continuity constraint

The chosen knee branch must also remain temporally continuous.

Therefore, if both branches are geometrically valid, one should prefer the branch that minimizes discontinuity relative to the knee position at the previous frame.

Let $K_{\mathrm{prev}}$ be the knee position at the previous frame.

Then one may choose the branch that minimizes

$$
\|K^{\pm} - K_{\mathrm{prev}}\|.
$$

subject to the branch-sign convention.

This prevents sudden knee flipping.

## 10. Exact knee angle

The internal knee angle of a leg with pelvis–ankle distance $d$ is given by the law of cosines:

$$
d^2 = L^2 + L^2 - 2L^2 \cos \beta.
$$

Hence

$$
\cos \beta = \frac{2L^2 - d^2}{2L^2}.
$$

Therefore

$$
\beta = \arccos\left(\frac{2L^2 - d^2}{2L^2}\right).
$$

This formula is important for reconstruction quality.

If a rendered walking pose systematically yields a knee angle far below $\pi$ in nominal stance, then the body is too crouched and the pendular walking geometry is not being respected.

## 11. Stance foot orientation

### 11.1. Visible stance foot direction

The visible stance foot direction should be derived from terrain support geometry.

If the stance foot lies on a single segment, then

$$
f_{\mathrm{foot,st}} = f\, t_i.
$$

If the stance foot spans two admissible segments, then

$$
f_{\mathrm{foot,st}} = \frac{T_{c,\mathrm{st}} - A_{\mathrm{st}}}{\|T_{c,\mathrm{st}} - A_{\mathrm{st}}\|}.
$$

Thus the visible stance foot direction is always defined from the current terrain support of that foot.

### 11.2. Effective support versus visible direction

The visible stance foot direction need not coincide exactly with the effective support tangent $t_{\mathrm{sup}}$.

Both vectors may be close, but they play different roles.

- $f_{\mathrm{foot,st}}$ is used for visible reconstruction.
- $t_{\mathrm{sup}}$ is used for support geometry, CM decomposition, and support transfer logic.

This distinction must remain explicit.

## 12. Swing foot orientation

The swing foot must also have a visible orientation during flight.

A natural construction is to blend its liftoff and touchdown visible directions.

Let the visible liftoff foot direction be

$$
f_{\mathrm{foot},0},
$$

and the target touchdown foot direction be

$$
f_{\mathrm{foot},1}.
$$

Using the same quintic blending function $h(\tau)$ as in the swing generator, define

$$
\widetilde f_{\mathrm{foot}}(\tau) = (1-h(\tau)) f_{\mathrm{foot},0} + h(\tau) f_{\mathrm{foot},1}.
$$

Then define the normalized swing-foot direction by

$$
f_{\mathrm{foot,sw}}(\tau) = \frac{\widetilde f_{\mathrm{foot}}(\tau)}{\|\widetilde f_{\mathrm{foot}}(\tau)\|}.
$$

This gives a continuous visible foot orientation during swing.

## 13. Exact visible toe points

Once the visible foot directions are known, the visible toe points are determined exactly.

For the stance foot,

$$
T_{\mathrm{st}} = A_{\mathrm{st}} + L_{\mathrm{toe}} f_{\mathrm{foot,st}}.
$$

For the swing foot,

$$
T_{\mathrm{sw}} = A_{\mathrm{sw}} + L_{\mathrm{toe}} f_{\mathrm{foot,sw}}.
$$

If later a heel segment is introduced, the visible heel points would be

$$
H_{\mathrm{st}} = A_{\mathrm{st}} - L_{\mathrm{heel}} f_{\mathrm{foot,st}},
$$

and

$$
H_{\mathrm{sw}} = A_{\mathrm{sw}} - L_{\mathrm{heel}} f_{\mathrm{foot,sw}}.
$$

At the current stage, $L_{\mathrm{heel}} = 0$, so the ankle is also the heel endpoint.

## 14. Full reconstructed frame

A full visible frame is defined by the node set

$$
\mathcal{N}_{\mathrm{frame}} = \{ C, P, T_c, T_t, A_L, A_R, K_L, K_R, T_L, T_R \},
$$

where $T_L$ and $T_R$ are the toe points of the left and right feet.

This frame is valid only if all rigid constraints and contact constraints hold.

## 15. Exact frame validity conditions

A rendered frame is valid only if the following conditions hold.

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

### 15.3. Stance contact consistency

If a foot is in stance, then its ankle must lie on the terrain support path of that foot, and its visible foot geometry must be admissible on the terrain.

### 15.4. Swing consistency

If a foot is in swing, then its ankle must lie on the swing trajectory, and the swing reconstruction must remain collision-free except at valid touchdown events.

No frame may violate these conditions.

## 16. What to do when exact reconstruction becomes impossible

Exact rigid reconstruction may fail in three different ways.

### 16.1. One-leg IK failure

If for one leg

$$
\|P-A_i\| > 2L,
$$

then exact rigid IK for that leg does not exist.

This is not a rendering problem.

It means the locomotion state itself has become incompatible with the rigid morphology.

### 16.2. Contact-geometry failure

A stance foot may become geometrically incompatible with the terrain, for example if the support interpolation becomes non-admissible.

### 16.3. Swing-terrain failure

A swing foot may intersect forbidden terrain before a valid touchdown.

### 16.4. Architectural consequence

In all these cases, the correct response is **not** to render an impossible frame by stretching geometry.

The correct response is to propagate the failure back to the locomotion controller and force a change in:

- CM target,
- step timing,
- foothold selection,
- support-transfer policy,
- or mode transition.

This principle is essential.

The renderer must never repair a physically impossible locomotion state by cheating on lengths.

## 17. Reconstruction in nominal walking and reactive walking

### 17.1. Nominal walking reconstruction

In nominal walking, one expects:

- stance leg nearly extended,
- swing leg flexed enough to clear the ground,
- torso angle close to the nominal target,
- foot orientations smoothly aligned with terrain support.

This means in particular that, for the stance leg,

$$
\|P-A_{\mathrm{st}}\| \approx \ell_w^\star.
$$

So the stance knee angle should remain close to full extension.

### 17.2. Reactive walking reconstruction

In reactive walking, one expects:

- more stance-leg flexion,
- larger or earlier recovery steps,
- possibly stronger torso compensation,
- larger swing clearance.

This appears geometrically as a smaller pelvis–ankle distance and a larger deviation from the nominal walking manifold.

But the exact rigid segment lengths remain unchanged.

The body changes posture. It does not change morphology.

## 18. Summary of exact body reconstruction

The reconstruction process is therefore the following.

1. Read the authoritative state $(C, V, \theta, \mathcal{M}, A_L, A_R, Q, t_{\mathrm{sup}}, n_{\mathrm{sup}})$.

2. Reconstruct the pelvis from the CM:

$$
P = C - 0.75L e_\theta.
$$

3. Reconstruct the torso nodes:

$$
T_c = P + L e_\theta,
$$

$$
T_t = P + 2L e_\theta.
$$

4. Reconstruct the stance knee by exact two-link IK.

5. Reconstruct the swing knee by exact two-link IK.

6. Reconstruct the visible stance foot direction from terrain support.

7. Reconstruct the visible swing foot direction from the swing interpolation.

8. Reconstruct the visible toe points.

9. Verify all exact frame-validity conditions.

If one of them fails, the locomotion state must be corrected upstream.

## 19. Main conclusions of this document

1. Body reconstruction must be a deterministic consequence of the authoritative locomotion state.
2. The torso angle is a structural variable, not a cosmetic variable.
3. The pelvis and torso nodes are reconstructed exactly from $C$ and $\theta$.
4. Both knees are reconstructed by exact closed-form two-link IK.
5. Knee-branch selection must satisfy both a global convention and temporal continuity.
6. Visible foot directions must remain distinct from effective support tangents.
7. No rendered frame may violate any rigid segment length.
8. If exact reconstruction fails, the correct response is to change the locomotion state upstream, not to cheat in rendering.

## 20. What comes next

The next document should formalize running.

In particular, it should define:

- the hybrid running modes,
- the effective spring-mass stance model,
- flight,
- touchdown after flight,
- the relation between walking-to-running transition and the common CM-first architecture.