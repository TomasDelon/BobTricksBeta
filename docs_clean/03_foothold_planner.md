# 03 — Formal foothold planner

## 0. Goal of this document

The goal of this document is to define the mathematical planner that selects the next stance ankle on a segmented terrain in a way consistent with the `CM-first` architecture.

This document fixes:

- the candidate terrain domain for the next ankle,
- the exact distinction between ankle $A_{\mathrm{next}}$ and induced support objects,
- the admissibility conditions of a candidate foothold on a polyline terrain,
- the role of the predicted touchdown state,
- the nominal step-length objective,
- the capture-related quantities used to score candidates,
- the reactive deformation of the same planner through $\rho$.

This document remains a formal mathematical specification. It does not yet impose a particular discrete search algorithm or numerical implementation.

## 1. Planning principle

The next foothold must not be chosen by a purely time-based oscillation and must not be reduced to a fixed horizontal step length.

The next foothold must be selected from:

- the current locomotion state,
- the terrain geometry ahead in the facing direction,
- the predicted state at touchdown,
- the current reactive intensity $\rho$.

The planner must satisfy two requirements at once.

First, in nominal locomotion, it must produce economical, regular, geometrically coherent steps.

Second, under perturbation, it must automatically allow larger, earlier, or more terrain-adaptive steps without changing the architecture.

Therefore there is no separate planner called “normal stepping” and another called “recovery stepping”. Both must emerge from the same candidate set and the same cost structure, continuously deformed by $\rho$.

## 2. What the planner selects

### 2.1. The planned object is the next ankle

The planned foothold is first of all an ankle point on the terrain:

$$
A_{\mathrm{next}} \in \Gamma.
$$

This is the authoritative target for the swing ankle.

### 2.2. The planner does not choose the support point directly

The planner does not directly choose the effective support point $Q_{\mathrm{next}}$.

Instead, the planner chooses the ankle $A_{\mathrm{next}}$, and from that ankle one later constructs:

- the visible foot direction $f_{\mathrm{foot,next}}$,
- the toe point $T_{c,\mathrm{next}}$,
- the effective support point $Q_{\mathrm{next}}$,
- the effective support tangent $t_{\mathrm{sup,next}}$,
- the effective support normal $n_{\mathrm{sup,next}}$.

This separation is essential because the leg anchors geometrically at the ankle, whereas balance and capture are better expressed relative to the effective support.

## 3. Forward candidate domain on the terrain

### 3.1. Forward terrain ray from the current stance ankle

Let the current stance ankle be $A_{\mathrm{st}}$ and let the facing sign be $f \in \{-1,+1\}$.

Let the terrain be parameterized by arc-length.

Define the forward terrain path starting at the current stance ankle by

$$
\gamma_{A_{\mathrm{st}}}(\sigma), \qquad \sigma \ge 0,
$$

with

$$
\gamma_{A_{\mathrm{st}}}(0) = A_{\mathrm{st}}.
$$

Every candidate foothold is of the form

$$
A(\sigma) = \gamma_{A_{\mathrm{st}}}(\sigma), \qquad \sigma \ge 0.
$$

The scalar $\sigma$ is the terrain arc-length from the current stance ankle to the candidate ankle. On a segmented terrain, this is the natural step-length variable.

### 3.2. Finite planning horizon

The planner searches only over a finite forward terrain interval.

Define the planning set

$$
\Sigma_{\mathrm{plan}}(\rho) = [\sigma_{\min}^{\mathrm{plan}}, \sigma_{\max}^{\mathrm{plan}}(\rho)].
$$

The lower bound avoids degenerate tiny steps.

The upper bound grows with reactivity:

$$
\sigma_{\max}^{\mathrm{plan}}(\rho)
=
\sigma_{\max,0}^{\mathrm{plan}} + \rho\,\Delta \sigma_{\max}^{\mathrm{plan}}.
$$

Thus stronger perturbations enlarge the accessible terrain horizon.

## 4. Exact reachability at touchdown

### 4.1. Pelvis-based rigid reachability

A candidate ankle is not acceptable unless it will be rigidly reachable at touchdown.

Let $C_{\mathrm{td}}$ and $\theta_{\mathrm{td}}$ be the predicted CM and torso angle at touchdown.

Define the predicted pelvis position by

$$
P_{\mathrm{td}} = C_{\mathrm{td}} - 0.75L e_{\theta_{\mathrm{td}}}.
$$

Then a candidate ankle $A_{\mathrm{next}}$ is exactly reachable if and only if

$$
\|P_{\mathrm{td}} - A_{\mathrm{next}}\| \le 2L.
$$

This is a hard geometric condition.

### 4.2. Preferred reach versus exact reach

The exact rigid bound $2L$ must not be confused with the nominal preferred operating range.

The hard condition is

$$
\|P_{\mathrm{td}} - A_{\mathrm{next}}\| \le 2L.
$$

But the planner may also prefer a narrower soft range. Define a preferred touchdown leg length by

$$
\ell_{\mathrm{pref}}(\rho) < 2L.
$$

Then a soft reach penalty can later be built from the excess beyond $\ell_{\mathrm{pref}}(\rho)$ while still keeping $2L$ as the non-negotiable exact bound.

## 5. Support objects induced by a candidate ankle

Given a candidate ankle $A_{\mathrm{next}}$, the terrain geometry ahead of that ankle in the facing direction induces the future foot and support objects.

### 5.1. Candidate toe point

If the forward terrain path from $A_{\mathrm{next}}$ exists up to $L_{\mathrm{toe}}$, define

$$
T_{c,\mathrm{next}} = \gamma_{A_{\mathrm{next}}}(L_{\mathrm{toe}}).
$$

### 5.2. Candidate visible foot direction

Define the future visible foot direction by

$$
f_{\mathrm{foot,next}} = \frac{T_{c,\mathrm{next}} - A_{\mathrm{next}}}{\|T_{c,\mathrm{next}} - A_{\mathrm{next}}\|}.
$$

This uses the ankle-to-toe mini-segment and not an artificial average of terrain angles.

### 5.3. Candidate effective support geometry

From the candidate foot geometry one then constructs:

$$
Q_{\mathrm{next}}, \qquad t_{\mathrm{sup,next}}, \qquad n_{\mathrm{sup,next}}.
$$

These are not independent decision variables. They are derived geometric objects induced by $A_{\mathrm{next}}$ and the local terrain configuration under the future foot.

## 6. Geometric admissibility of a candidate foothold

A candidate ankle is geometrically admissible only if the future foot geometry is itself admissible on the segmented terrain.

A candidate ankle $A_{\mathrm{next}}$ is admissible only if all the following hold.

1. The forward terrain path from $A_{\mathrm{next}}$ exists at least up to arc-length $L_{\mathrm{toe}}$.

2. The future visible foot spans at most two consecutive terrain segments.

3. If two segments are covered, the terrain angle jump under the foot satisfies

$$
\Delta \alpha_{\mathrm{foot}} \le \alpha_{\mathrm{foot}}^{\max}.
$$

4. The visible foot segment does not generate a forbidden terrain crossing.

5. The visible foot is compatible with the facing direction:

$$
f\,(f_{\mathrm{foot,next}} \cdot e_X) > 0.
$$

6. The induced support objects

$$
Q_{\mathrm{next}}, \qquad t_{\mathrm{sup,next}}, \qquad n_{\mathrm{sup,next}}
$$

are well defined.

If one of these conditions fails, the candidate is rejected before any dynamic score is computed.

## 7. Predicted touchdown state

The planner must evaluate candidates at predicted touchdown, not only from the instantaneous current state.

Let $X_{\mathrm{now}}$ be the current locomotion state and let $t_{\mathrm{td}}$ be the predicted touchdown time associated with a candidate.

Define the prediction operator by

$$
X_{\mathrm{td}} = \Pi(t_{\mathrm{td}}; X_{\mathrm{now}}).
$$

At minimum, the planner needs the predicted quantities

$$
C_{\mathrm{td}}, \qquad \dot C_{\mathrm{td}}, \qquad \theta_{\mathrm{td}}.
$$

More refined implementations may include further predicted variables, but the formal planner is defined in terms of the touchdown state produced by $\Pi$.

## 8. Candidate-local support coordinates at touchdown

Once the candidate support objects are defined, the predicted CM can be written in the future support frame as

$$
C_{\mathrm{td}} = Q_{\mathrm{next}} + s_{Q,\mathrm{next}} t_{\mathrm{sup,next}} + z_{Q,\mathrm{next}} n_{\mathrm{sup,next}}.
$$

Thus

$$
s_{Q,\mathrm{next}} = (C_{\mathrm{td}} - Q_{\mathrm{next}}) \cdot t_{\mathrm{sup,next}},
$$

$$
z_{Q,\mathrm{next}} = (C_{\mathrm{td}} - Q_{\mathrm{next}}) \cdot n_{\mathrm{sup,next}}.
$$

Relative to the candidate ankle,

$$
C_{\mathrm{td}} = A_{\mathrm{next}} + s_{A,\mathrm{next}} t_{\mathrm{sup,next}} + z_{A,\mathrm{next}} n_{\mathrm{sup,next}}.
$$

Thus

$$
s_{A,\mathrm{next}} = (C_{\mathrm{td}} - A_{\mathrm{next}}) \cdot t_{\mathrm{sup,next}},
$$

$$
z_{A,\mathrm{next}} = (C_{\mathrm{td}} - A_{\mathrm{next}}) \cdot n_{\mathrm{sup,next}}.
$$

If the induced support point is written as

$$
Q_{\mathrm{next}} = A_{\mathrm{next}} + \sigma_{\mathrm{next}} t_{\mathrm{sup,next}},
$$

then

$$
s_{Q,\mathrm{next}} = s_{A,\mathrm{next}} - \sigma_{\mathrm{next}},
$$

$$
z_{Q,\mathrm{next}} = z_{A,\mathrm{next}}.
$$

This keeps explicit the same ankle-versus-support distinction already fixed in the walking document.

## 9. Candidate local capture quantity

Define the candidate local tangential touchdown velocity by

$$
\dot s_{A,\mathrm{next}} = \dot C_{\mathrm{td}} \cdot t_{\mathrm{sup,next}}.
$$

Define the candidate local pendular frequency by

$$
\omega_{0,\mathrm{next}} = \sqrt{\frac{g\cos\alpha_{\mathrm{sup,next}}}{\bar z_{\mathrm{next}}}},
$$

where $\alpha_{\mathrm{sup,next}}$ is the angle of $t_{\mathrm{sup,next}}$ and $\bar z_{\mathrm{next}} > 0$ is a positive reference support-normal height.

Then define the candidate XCoM-like quantity by

$$
\xi_{\mathrm{next}} = s_{Q,\mathrm{next}} + \lambda \frac{\dot s_{A,\mathrm{next}}}{\omega_{0,\mathrm{next}}},
$$

with

$$
0 < \lambda \le 1.
$$

This quantity is not the whole planner. It is one useful signal that estimates how well the predicted state is captured by the future support.

## 10. Candidate support interval and capture defect

At the current stage of the project, the support interval of the future foot relative to the ankle is

$$
\mathcal{I}_{\mathrm{foot,next}} = [0, L_{\mathrm{toe}}],
$$

because $L_{\mathrm{heel}} = 0$.

Relative to the induced support point

$$
Q_{\mathrm{next}} = A_{\mathrm{next}} + \sigma_{\mathrm{next}} t_{\mathrm{sup,next}},
$$

the same interval becomes

$$
\mathcal{I}_{Q,\mathrm{next}} = [-\sigma_{\mathrm{next}}, L_{\mathrm{toe}} - \sigma_{\mathrm{next}}].
$$

Define the distance-to-interval function by

$$
d_{\mathcal{I}}(x,[a,b]) =
\left\{
\begin{array}{ll}
a-x & \text{if } x<a, \\
0 & \text{if } a \le x \le b, \\
x-b & \text{if } x>b.
\end{array}
\right.
$$

Then define the candidate capture defect by

$$
D_{\mathrm{cap}}(A_{\mathrm{next}})
=
d_{\mathcal{I}}\bigl(\xi_{\mathrm{next}}, \mathcal{I}_{Q,\mathrm{next}}\bigr).
$$

This quantity vanishes when the local capture quantity lies inside the future support interval.

## 11. Nominal terrain-based step length objective

### 11.1. Terrain step length

For a candidate ankle $A_{\mathrm{next}} = \gamma_{A_{\mathrm{st}}}(\sigma)$, the natural step length is exactly the terrain abscissa

$$
\sigma(A_{\mathrm{next}}).
$$

This is preferable to world-horizontal distance because the terrain is segmented and may contain slope changes or steps.

### 11.2. Nominal target step length

Let the nominal target step length be

$$
\sigma_{\mathrm{step}}^{\star}(\rho).
$$

A simple reactive law is

$$
\sigma_{\mathrm{step}}^{\star}(\rho)
=
\sigma_{\mathrm{step},0}^{\star} + \rho\,\Delta \sigma_{\mathrm{step}}^{\star}.
$$

Then define the step-length penalty by

$$
D_{\mathrm{step}}(A_{\mathrm{next}};\rho)
=
\left(\sigma(A_{\mathrm{next}}) - \sigma_{\mathrm{step}}^{\star}(\rho)\right)^2.
$$

Thus longer recovery steps emerge continuously as $\rho$ increases.

## 12. Terrain transition costs

### 12.1. Height change cost

Define the ankle height change by

$$
\Delta h(A_{\mathrm{next}}) = (A_{\mathrm{next}} - A_{\mathrm{st}}) \cdot e_Y.
$$

A simple nominal penalty is

$$
D_h(A_{\mathrm{next}}) = \left(\Delta h(A_{\mathrm{next}})\right)^2.
$$

This does not forbid step-up or step-down events. It only penalizes unnecessary vertical excursions in nominal locomotion.

### 12.2. Slope transition cost

Let $\alpha_{\mathrm{sup}}$ be the current support slope angle.

Let $\alpha_{\mathrm{sup,next}}$ be the future support slope angle induced by the candidate foothold.

Define the slope-transition penalty

$$
D_{\alpha}(A_{\mathrm{next}})
=
\left(\alpha_{\mathrm{sup,next}} - \alpha_{\mathrm{sup}}\right)^2.
$$

Again, this is a preference, not a prohibition.

## 13. Touchdown-time compatibility

The planner must remain compatible with the swing generator.

Let the touchdown time associated with a candidate be

$$
t_{\mathrm{td}}(A_{\mathrm{next}}).
$$

Let the nominal touchdown-time target be

$$
t_{\mathrm{td}}^{\star}(\rho).
$$

Then a timing penalty is

$$
D_t(A_{\mathrm{next}};\rho)
=
\left(t_{\mathrm{td}}(A_{\mathrm{next}}) - t_{\mathrm{td}}^{\star}(\rho)\right)^2.
$$

This term expresses compatibility with the current swing evolution without splitting planning and swing into unrelated systems.

## 14. Soft reach penalty

Besides the exact hard bound, the planner may discourage candidates that require a touchdown leg too close to the geometric limit.

Define the soft reach penalty by

$$
D_{\mathrm{IK}}(A_{\mathrm{next}};\rho)
=
\left(
\max\left\{0, \|P_{\mathrm{td}} - A_{\mathrm{next}}\| - \ell_{\mathrm{pref}}(\rho)\right\}
\right)^2.
$$

This term is zero in the preferred operating range and positive only when the candidate pushes the touchdown geometry toward the exact rigid boundary.

## 15. Admissible candidate set

Define the admissible candidate set by

$$
\mathcal{A}(\rho)
=
\left\{
A(\sigma) : \sigma \in \Sigma_{\mathrm{plan}}(\rho), \ A(\sigma) \text{ is geometrically admissible}, \ \|P_{\mathrm{td}} - A(\sigma)\| \le 2L
\right\}.
$$

This expression separates three logically different layers:

1. the forward terrain search domain,
2. the foot and support admissibility constraints,
3. the exact rigid reachability constraint.

That separation is important both mathematically and for future implementation.

## 16. Full candidate cost

Among admissible candidates, define the total foothold cost by

$$
J(A_{\mathrm{next}};\rho)
=
w_{\mathrm{cap}}(\rho) D_{\mathrm{cap}}
+
w_{\mathrm{step}}(\rho) D_{\mathrm{step}}
+
w_h(\rho) D_h
+
w_{\alpha}(\rho) D_{\alpha}
+
w_t(\rho) D_t
+
w_{\mathrm{IK}}(\rho) D_{\mathrm{IK}}.
$$

The selected foothold is then

$$
A_{\mathrm{next}}^{\star}
=
\arg\min_{A \in \mathcal{A}(\rho)} J(A;\rho).
$$

## 17. Nominal versus reactive planning

The planner is one and the same, but its horizon, targets, and weights vary continuously with $\rho$.

### 17.1. Nominal regime

When $\rho$ is small:

- the planning horizon is moderate,
- the preferred step length remains near its nominal value,
- terrain elegance matters,
- the planner favors regular capture-compatible steps.

### 17.2. Reactive regime

When $\rho$ increases:

- the planning horizon expands,
- the preferred step length increases,
- timing flexibility increases,
- capture and reachability become more dominant,
- terrain elegance may be relatively de-emphasized.

A simple formal example is

$$
w_{\mathrm{cap}}(\rho) = w_{\mathrm{cap},0} + \rho\,\Delta w_{\mathrm{cap}},
$$

$$
w_h(\rho) = w_{h,0} - \rho\,\Delta w_h.
$$

Thus recovery stepping is not a separate architecture. It is the same planner under a different reactive deformation.

## 18. Compatibility with walk, swing, and run

### 18.1. Compatibility with walking

The present foothold planner is directly compatible with the walking protocol because it uses the same explicit distinction between ankle and effective support, the same local support coordinates, and the same reactive intensity $\rho$.

### 18.2. Compatibility with the swing generator

The planner does not itself realize the step. It proposes

$$
A_{\mathrm{next}}^{\star}.
$$

The swing generator must then realize a full-foot collision-free ankle transfer toward that target while preserving touchdown viability.

### 18.3. Compatibility with running

The planner is also compatible with the running protocol at the architectural level.

What changes in running is not the existence of the planner as such, but mainly:

- the contact protocol,
- the touchdown timing law,
- the relative importance of the cost terms,
- the admissible horizon and target step length under the running regime.

Thus the same `CM-first` foothold-planning structure can later be reused across gaits.

## 19. Touchdown viability after planning

Planning a foothold is not enough. The touchdown event must still be realizable.

At touchdown, one must have at least:

$$
\|A_{\mathrm{sw}}(t_{\mathrm{td}}) - A_{\mathrm{next}}^{\star}\| \le \varepsilon_A,
$$

$$
\|P_{\mathrm{td}} - A_{\mathrm{next}}^{\star}\| \le 2L,
$$

and the future foot geometry induced by $A_{\mathrm{next}}^{\star}$ must remain admissible.

This makes explicit the distinction between:

- planning,
- swing realization,
- touchdown validation,
- support transition.

## 20. Main conclusions of this document

1. The foothold planner must select the next ankle on the terrain polyline, not the toe and not the support point directly.
2. A candidate foothold induces future foot and support objects that must be explicitly admissible.
3. Candidate evaluation must be performed at predicted touchdown through the operator $\Pi$.
4. Terrain arc-length is the natural step-length variable on segmented terrain.
5. A local XCoM-like quantity is useful but is only one term of the full foothold score.
6. Exact rigid reachability remains a hard geometric constraint.
7. Recovery stepping must emerge from continuous deformation of the same planner through $\rho$.
8. The planner must stay compatible with walking, swing realization, and later running reuse.

## 21. What comes next

The next clean documents should formalize at least one of the following extensions.

- walk-to-run and run-to-walk transitions,
- jump and intentional entry into flight,
- landing and high-energy capture,
- a more explicit common prediction layer shared across planners and gait protocols.
