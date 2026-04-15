# 03 — Formal foothold planner

## 0. Goal of this document

The goal of this document is to define the mathematical planner that chooses the next stance ankle during walking.

The previous documents fixed:

- the rigid body geometry,
- the segmented terrain geometry,
- the distinction between ankle $A$ and effective support point $Q$,
- the exact rigid-leg IK conditions,
- the hybrid walking protocol and the nominal stance manifold.

This document now answers the following questions.

- What is a candidate next foothold?
- What makes a foothold geometrically admissible on a segmented terrain?
- How do we define the nominal step length?
- How do we enlarge the admissible step set under perturbation?
- How do we use support geometry, local CM state, and capture considerations to score candidates?
- How do we distinguish nominal stepping from recovery stepping without inventing a separate architecture?

This is still a formal mathematical document. It does not yet define a specific discrete search implementation.

## 1. Planning principle

The next foothold must not be chosen by a fixed time-based oscillation or by a fixed horizontal step length.

The next foothold must be chosen from:

- the current CM state,
- the current support state,
- the terrain geometry ahead of the body,
- the facing direction,
- the current reactive intensity $\rho$.

The planner must satisfy two requirements at once.

First requirement.

In nominal walking, it must produce economical, regular steps that preserve pendular motion and keep the CM well captured relative to the next support.

Second requirement.

Under perturbation, it must automatically allow larger or earlier steps, lower-CM recovery, and non-periodic stepping.

This means that the foothold planner must not be split into two unrelated systems called “normal stepping” and “recovery stepping”. Both must emerge from the same admissible set and the same cost structure, with parameters deformed continuously by $\rho$.

## 2. Geometric meaning of a foothold

### 2.1. Foothold as an ankle point

The next foothold is defined first of all by an ankle point

$$
A_{\mathrm{next}} \in \Gamma.
$$

This is the authoritative contact target for the swing foot.

The visible foot geometry and support geometry will later be derived from this ankle point and from the terrain immediately ahead of it.

### 2.2. Why the planner chooses the ankle and not the toe

The body reconstruction and rigid-leg IK use the ankle as the anchored node of the leg.

Therefore the natural controlled point is the ankle, not the toe.

The toe point and visible foot direction are then derived from the terrain path forward from the ankle.

This ensures compatibility between:

- the foothold planner,
- the stance manifold,
- rigid IK,
- the visible foot model.

## 3. Forward planning domain on the terrain

### 3.1. Terrain abscissa in the facing direction

Let the current stance ankle be $A_{\mathrm{st}}$ and the facing direction be $f \in \{-1,+1\}$.

Let the terrain be parameterized by arc-length.

We define the forward terrain ray from the current stance ankle by

$$
\gamma_{A_{\mathrm{st}}}(\sigma), \qquad \sigma \ge 0,
$$

where $\sigma$ is terrain arc-length measured forward in the facing direction.

Every future ankle candidate must lie on this forward terrain path.

So any candidate foothold can be written as

$$
A(\sigma) = \gamma_{A_{\mathrm{st}}}(\sigma), \qquad \sigma \ge 0.
$$

### 3.2. Finite planning horizon

The planner does not need to search arbitrarily far ahead.

Define a maximal terrain search distance

$$
\sigma_{\max}^{\mathrm{plan}}(\rho) > 0.
$$

The admissible search domain is

$$
\Sigma_{\mathrm{plan}}(\rho) = [\sigma_{\min}^{\mathrm{plan}}, \sigma_{\max}^{\mathrm{plan}}(\rho)].
$$

The lower bound $\sigma_{\min}^{\mathrm{plan}}$ prevents degenerate tiny steps.

The upper bound grows with reactivity:

$$
\sigma_{\max}^{\mathrm{plan}}(\rho) = \sigma_{\max,0}^{\mathrm{plan}} + \rho\,\Delta \sigma_{\max}^{\mathrm{plan}}.
$$

Thus strong perturbations automatically enlarge the terrain horizon accessible to the next step.

## 4. Exact rigid-leg reachability of the next foothold

### 4.1. Pelvis-based reachability condition

At touchdown, the new stance leg must admit rigid reconstruction.

Let $P_{\mathrm{td}}$ be the predicted pelvis position at touchdown.

Then a candidate ankle $A_{\mathrm{next}}$ is rigid-leg reachable if and only if

$$
\|P_{\mathrm{td}} - A_{\mathrm{next}}\| \le 2L.
$$

Since

$$
P_{\mathrm{td}} = C_{\mathrm{td}} - 0.75L e_{\theta_{\mathrm{td}}},
$$

this can also be written as

$$
\|C_{\mathrm{td}} - 0.75L e_{\theta_{\mathrm{td}}} - A_{\mathrm{next}}\| \le 2L.
$$

This is an exact geometric condition.

### 4.2. Nominal and reactive effective reach

The exact rigid reach bound $2L$ is not the same thing as the nominal preferred reach.

In nominal walking, we do not want the system to choose arbitrary steps close to the exact boundary of rigid reach. We want a narrower nominal range.

Define the nominal preferred step interval in terrain arc-length by

$$
\Sigma_{\mathrm{step}}^{\mathrm{nom}} = [\sigma_{\min}^{\mathrm{nom}}, \sigma_{\max}^{\mathrm{nom}}].
$$

Under reactivity, this interval expands:

$$
\sigma_{\min}^{\mathrm{step}}(\rho) = \sigma_{\min}^{\mathrm{nom}} - \rho\,\Delta \sigma_{\min}^{\mathrm{step}},
$$

$$
\sigma_{\max}^{\mathrm{step}}(\rho) = \sigma_{\max}^{\mathrm{nom}} + \rho\,\Delta \sigma_{\max}^{\mathrm{step}}.
$$

Thus the exact rigid limit remains a hard geometric bound, but the planner deforms its preferred range continuously as reactivity increases.

## 5. Geometric admissibility of a candidate foothold

A candidate ankle is not admissible just because it is reachable.

It must also admit a valid foot geometry on the segmented terrain.

### 5.1. Candidate support path

Given a candidate ankle $A_{\mathrm{next}}$, define the forward terrain path from that ankle in the current facing direction:

$$
\gamma_{A_{\mathrm{next}}}(\sigma), \qquad \sigma \ge 0.
$$

The candidate foot uses this path to define its toe point.

### 5.2. Candidate toe point

If the terrain path exists up to toe length, define the candidate toe point by

$$
T_{c,\mathrm{next}} = \gamma_{A_{\mathrm{next}}}(L_{\mathrm{toe}}).
$$

### 5.3. Candidate visible foot direction

The visible foot direction is then

$$
f_{\mathrm{foot,next}} = \frac{T_{c,\mathrm{next}} - A_{\mathrm{next}}}{\|T_{c,\mathrm{next}} - A_{\mathrm{next}}\|}.
$$

### 5.4. Admissibility conditions

A candidate ankle $A_{\mathrm{next}}$ is geometrically admissible only if all of the following hold.

1. The forward terrain path from $A_{\mathrm{next}}$ exists at least up to arc-length $L_{\mathrm{toe}}$.

2. The visible foot spans at most two consecutive segments.

3. The terrain angle jump under the foot is bounded:

$$
\Delta \alpha_{\mathrm{foot}} \le \alpha_{\mathrm{foot}}^{\max}.
$$

4. The visible foot segment does not produce a forbidden terrain crossing.

5. The foot points in the facing direction:

$$
f\,(f_{\mathrm{foot,next}} \cdot e_X) > 0.
$$

6. The future support construction from this ankle admits a valid effective support point and tangent.

If one of these conditions fails, the candidate is rejected.

## 6. Predicted state at touchdown

The planner cannot score candidate footholds using only the current state. It must evaluate the predicted state at touchdown.

Let $t_{\mathrm{td}}$ be the predicted touchdown time of the swing foot.

Then we define the predicted touchdown state by

$$
X_{\mathrm{td}} = \Pi(t_{\mathrm{td}}; X_{\mathrm{now}}),
$$

where $\Pi$ is the forward prediction of the current walking dynamics.

At minimum, the planner needs:

$$
C_{\mathrm{td}}, \qquad \dot C_{\mathrm{td}}, \qquad \theta_{\mathrm{td}}.
$$

The exact prediction model can be simplified in early implementations, but the formal planner is defined in terms of the predicted touchdown state, not purely in terms of the instantaneous current state.

## 7. Local coordinates of the predicted CM relative to a candidate foothold

Let the candidate effective support frame induced by $A_{\mathrm{next}}$ be

$$
(t_{\mathrm{sup,next}}, n_{\mathrm{sup,next}}).
$$

Let the candidate support point be $Q_{\mathrm{next}}$.

Then the predicted touchdown CM has coordinates

$$
C_{\mathrm{td}} = Q_{\mathrm{next}} + s_{Q,\mathrm{next}} t_{\mathrm{sup,next}} + z_{Q,\mathrm{next}} n_{\mathrm{sup,next}}.
$$

Thus

$$
s_{Q,\mathrm{next}} = (C_{\mathrm{td}} - Q_{\mathrm{next}}) \cdot t_{\mathrm{sup,next}},
$$

and

$$
z_{Q,\mathrm{next}} = (C_{\mathrm{td}} - Q_{\mathrm{next}}) \cdot n_{\mathrm{sup,next}}.
$$

Likewise the predicted tangential velocity relative to the candidate support is

$$
\dot s_{A,\mathrm{next}} = \dot C_{\mathrm{td}} \cdot t_{\mathrm{sup,next}}.
$$

These variables are needed to evaluate capture, support margin, and future pendular continuity.

## 8. Local XCoM-like capture quantity for a candidate foothold

Define the candidate local pendular frequency

$$
\omega_{0,\mathrm{next}} = \sqrt{\frac{g\cos\alpha_{\mathrm{sup,next}}}{\bar z_{\mathrm{next}}}},
$$

where $\alpha_{\mathrm{sup,next}}$ is the angle of the candidate support tangent and $\bar z_{\mathrm{next}} > 0$ is a positive reference support-normal height.

Then define the candidate XCoM-like variable by

$$
\xi_{\mathrm{next}} = s_{Q,\mathrm{next}} + \lambda \frac{\dot s_{A,\mathrm{next}}}{\omega_{0,\mathrm{next}}},
$$

with

$$
0 < \lambda \le 1.
$$

This quantity is not the whole foothold planner. It is one signal among others.

Its role is to quantify how well the predicted CM state can be captured by the future support.

## 9. Support margin for a candidate foot

The foot defines a finite support interval along the candidate support tangent.

If the candidate support point uses toe length $L_{\mathrm{toe}}$ and heel length $L_{\mathrm{heel}}$, then the support interval relative to the candidate ankle is

$$
\mathcal{I}_{\mathrm{foot,next}} = [-L_{\mathrm{heel}}, L_{\mathrm{toe}}].
$$

Relative to the candidate support point $Q_{\mathrm{next}} = A_{\mathrm{next}} + \sigma_{\mathrm{next}} t_{\mathrm{sup,next}}$, the corresponding support interval becomes

$$
\mathcal{I}_{Q,\mathrm{next}} = [-L_{\mathrm{heel}}-\sigma_{\mathrm{next}}, L_{\mathrm{toe}}-\sigma_{\mathrm{next}}].
$$

The capture margin of the candidate foothold is then the signed distance from $\xi_{\mathrm{next}}$ to this interval.

Define the distance-to-interval function by

$$
d_{\mathcal{I}}(x, [a,b]) =
\left\{
\begin{array}{ll}
a-x & \text{if } x<a, \\
0 & \text{if } a \le x \le b, \\
x-b & \text{if } x>b.
\end{array}
\right.
$$

Then the capture defect of the candidate foothold is

$$
D_{\mathrm{cap}}(A_{\mathrm{next}}) = d_{\mathcal{I}}\bigl(\xi_{\mathrm{next}}, \mathcal{I}_{Q,\mathrm{next}}\bigr).
$$

This quantity is zero when the XCoM-like capture point lies inside the support interval.

It is positive when the predicted support would fail to capture the current motion well.

## 10. Nominal step length objective

### 10.1. Terrain-based step length

The natural step length on a segmented terrain is not the horizontal difference in world coordinates. It is the terrain arc-length between the current stance ankle and the candidate next ankle.

Define

$$
\sigma(A_{\mathrm{next}})
$$

as the forward terrain arc-length from $A_{\mathrm{st}}$ to $A_{\mathrm{next}}$.

This is the natural scalar step length variable on a polyline terrain.

### 10.2. Nominal step length

Let the nominal step length target be

$$
\sigma^\star_{\mathrm{step}}.
$$

This target may depend on mode, preferred walking speed, and coarse terrain class, but it is still a single scalar target in the nominal regime.

A nominal step length penalty is therefore

$$
D_{\mathrm{step}}(A_{\mathrm{next}}) = \left(\sigma(A_{\mathrm{next}}) - \sigma^\star_{\mathrm{step}}\right)^2.
$$

### 10.3. Reactive deformation of the target step length

Under perturbation, the preferred step length must be allowed to increase.

A simple reactive target is

$$
\sigma^\star_{\mathrm{step}}(\rho) = \sigma^\star_{\mathrm{step},0} + \rho\,\Delta \sigma^\star_{\mathrm{step}}.
$$

Then the step penalty becomes

$$
D_{\mathrm{step}}(A_{\mathrm{next}}; \rho) = \left(\sigma(A_{\mathrm{next}}) - \sigma^\star_{\mathrm{step}}(\rho)\right)^2.
$$

Thus longer recovery steps emerge continuously as $\rho$ increases.

## 11. Height and slope transition costs

The foothold planner must also account for terrain transitions.

### 11.1. Height change

Define the height change of the candidate ankle relative to the current stance ankle by

$$
\Delta h(A_{\mathrm{next}}) = (A_{\mathrm{next}} - A_{\mathrm{st}}) \cdot e_Y.
$$

A simple penalty is

$$
D_h(A_{\mathrm{next}}) = \left(\Delta h(A_{\mathrm{next}})\right)^2.
$$

This does not prohibit step-up or step-down events. It only penalizes unnecessary vertical excursions in nominal walking.

### 11.2. Slope jump

Let the support slope angle at the current stance be $\alpha_{\mathrm{sup}}$.

Let the candidate support slope angle be $\alpha_{\mathrm{sup,next}}$.

Define the slope-jump penalty

$$
D_\alpha(A_{\mathrm{next}}) = \left(\alpha_{\mathrm{sup,next}} - \alpha_{\mathrm{sup}}\right)^2.
$$

Again, this is not a prohibition. It simply expresses that large slope changes are less desirable in nominal gait unless the terrain forces them.

## 12. Time-of-touchdown cost

The future foothold must also be compatible with the swing phase.

Let the predicted touchdown time associated with candidate $A_{\mathrm{next}}$ be

$$
t_{\mathrm{td}}(A_{\mathrm{next}}).
$$

In a first planner, a simple cost is

$$
D_t(A_{\mathrm{next}}) = \left(t_{\mathrm{td}}(A_{\mathrm{next}}) - t_{\mathrm{td}}^\star\right)^2.
$$

where $t_{\mathrm{td}}^\star$ is a nominal swing-duration target.

As with the step length target, this time target may itself be deformed by $\rho$.

This allows earlier or later steps under perturbation without switching to a wholly different planner.

## 13. Full candidate cost function

We now combine the previous components into one formal foothold score.

A candidate foothold $A_{\mathrm{next}}$ is first rejected if it is not geometrically admissible.

Among admissible candidates, define the total cost

$$
J(A_{\mathrm{next}}; \rho) = w_{\mathrm{cap}} D_{\mathrm{cap}} + w_{\mathrm{step}} D_{\mathrm{step}} + w_h D_h + w_\alpha D_\alpha + w_t D_t + w_{\mathrm{IK}} D_{\mathrm{IK}}.
$$

Here $D_{\mathrm{IK}}$ is a soft reach penalty, for example

$$
D_{\mathrm{IK}}(A_{\mathrm{next}}) = \left(\max\left\{0, \|P_{\mathrm{td}}-A_{\mathrm{next}}\| - \ell_{\mathrm{pref,next}}\right\}\right)^2,
$$

where $\ell_{\mathrm{pref,next}}$ is a preferred, not maximal, pelvis-to-ankle distance.

The selected foothold is then

$$
A_{\mathrm{next}}^{\star} = \arg\min_{A \in \mathcal{A}(\rho)} J(A; \rho),
$$

where $\mathcal{A}(\rho)$ is the admissible candidate set.

## 14. Candidate set

The admissible candidate set is defined by

$$
\mathcal{A}(\rho) = \left\{ A(\sigma) : \sigma \in \Sigma_{\mathrm{plan}}(\rho), \ A(\sigma) \text{ is geometrically admissible}, \ \|P_{\mathrm{td}}-A(\sigma)\| \le 2L \right\}.
$$

This formula is important because it separates three logically different layers.

1. The search horizon on terrain.
2. The foot admissibility constraints.
3. The exact rigid-leg reachability constraint.

This separation will later make the implementation cleaner and easier to test.

## 15. Nominal versus reactive foothold planning

The planner is one and the same, but the parameters vary with $\rho$.

### 15.1. Nominal regime

When $\rho$ is small:

- the search horizon is moderate,
- the preferred step length is near its nominal value,
- the timing target is regular,
- the support-capture term keeps the CM well centered relative to the new support,
- unusual terrain transitions are penalized strongly.

### 15.2. Reactive regime

When $\rho$ increases:

- the search horizon expands,
- the preferred step length grows,
- the step timing may change,
- the capture term becomes more dominant,
- terrain penalties may be relatively de-emphasized if necessary for recovery.

A simple formal way to express that last point is to make the weights themselves depend on $\rho$.

For example,

$$
w_{\mathrm{cap}}(\rho) = w_{\mathrm{cap},0} + \rho\,\Delta w_{\mathrm{cap}},
$$

and

$$
w_h(\rho) = w_{h,0} - \rho\,\Delta w_h.
$$

Thus dynamic capture becomes more important and terrain elegance becomes less important under strong perturbation.

This is exactly the right qualitative behavior for recovery stepping.

## 16. First necessary condition for nominal capture on the next step

Suppose the candidate support interval relative to $Q_{\mathrm{next}}$ is

$$
[a_{\mathrm{next}}, b_{\mathrm{next}}].
$$

If

$$
\xi_{\mathrm{next}} \in [a_{\mathrm{next}}, b_{\mathrm{next}}],
$$

then

$$
D_{\mathrm{cap}}(A_{\mathrm{next}}) = 0.
$$

This is a necessary condition for a candidate foothold to be nominally well-capturing in the local support sense.

It is not by itself sufficient for the whole gait to succeed, because other conditions still matter:

- rigid-leg reachability,
- foot admissibility,
- swing timing,
- global torso and CM evolution.

But it is a necessary local support criterion.

## 17. Touchdown viability

A planned foothold is not enough. The touchdown event must also be viable.

At touchdown, the following must hold.

1. The swing ankle coincides with the target ankle within tolerance.

2. The future foot geometry is admissible.

3. The rigid-leg IK condition holds for the new stance ankle.

4. The support-transfer phase can begin with valid old and new support frames.

Formally, touchdown viability requires at least

$$
\|A_{\mathrm{sw}}(t_{\mathrm{td}}) - A_{\mathrm{next}}^\star\| \le \varepsilon_A,
$$

and

$$
\|P_{\mathrm{td}} - A_{\mathrm{next}}^\star\| \le 2L.
$$

This clarifies the separation between planning and realization.

The planner proposes a foothold. The swing generator and the hybrid mode transition must then realize it.

## 18. Summary of the foothold planner

The formal foothold planner is now defined by the following elements.

### Candidate terrain domain

$$
A(\sigma) = \gamma_{A_{\mathrm{st}}}(\sigma), \qquad \sigma \in \Sigma_{\mathrm{plan}}(\rho).
$$

### Exact rigid reachability

$$
\|P_{\mathrm{td}} - A_{\mathrm{next}}\| \le 2L.
$$

### Geometric foot admissibility

- valid forward terrain path,
- at most two consecutive support segments,
- bounded angle jump,
- no forbidden terrain crossing,
- facing-compatible visible foot direction.

### Candidate support state

$$
(Q_{\mathrm{next}}, t_{\mathrm{sup,next}}, n_{\mathrm{sup,next}}).
$$

### Candidate capture quantity

$$
\xi_{\mathrm{next}} = s_{Q,\mathrm{next}} + \lambda \frac{\dot s_{A,\mathrm{next}}}{\omega_{0,\mathrm{next}}}.
$$

### Candidate cost

$$
J(A_{\mathrm{next}}; \rho) = w_{\mathrm{cap}} D_{\mathrm{cap}} + w_{\mathrm{step}} D_{\mathrm{step}} + w_h D_h + w_\alpha D_\alpha + w_t D_t + w_{\mathrm{IK}} D_{\mathrm{IK}}.
$$

### Selected foothold

$$
A_{\mathrm{next}}^{\star} = \arg\min_{A \in \mathcal{A}(\rho)} J(A; \rho).
$$

## 19. Main conclusions of this document

1. The foothold planner must choose the next ankle on the terrain polyline, not the toe.
2. The candidate set must be constrained by both exact rigid reachability and exact foot admissibility.
3. The planner must evaluate candidates at predicted touchdown, not only from the instantaneous state.
4. Terrain arc-length is the natural step-length variable on a segmented terrain.
5. A capture quantity of XCoM type is useful, but it is only one component of the full foothold score.
6. Recovery stepping must emerge by continuous deformation of the same candidate set and the same cost function through $\rho$.
7. The planner must remain compatible with the swing generator and the hybrid support transfer model.

## 20. What comes next

The next document should define the swing generator itself.

In particular, it should specify:

- the ankle swing trajectory from current ankle to $A_{\mathrm{next}}^{\star}$,
- toe clearance constraints,
- collision avoidance with segmented terrain,
- touchdown timing,
- early landing and delayed landing conditions,
- consistency with the future support frame.