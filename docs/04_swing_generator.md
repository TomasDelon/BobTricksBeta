# 04 — Formal swing generator

## 0. Goal of this document

The goal of this document is to define the mathematical generator of the swing foot.

The previous documents fixed:

- the rigid geometry of the body,
- the segmented terrain geometry,
- the foot model,
- the exact rigid-leg IK conditions,
- the hybrid walking protocol,
- the foothold planner and the definition of the next ankle target $A_{\mathrm{next}}^{\star}$.

This document now defines:

- the swing state variables,
- the ankle trajectory from current swing ankle to the selected target ankle,
- the clearance constraints over segmented terrain,
- the collision-avoidance condition,
- the touchdown condition,
- early landing,
- delayed landing,
- the consistency requirements between swing generation and the future support geometry.

This is still a formal mathematical specification. It does not yet choose a concrete numerical interpolation scheme for the final implementation beyond the analytical families introduced here.

## 1. Principle of the swing phase

During walking, one foot is in stance and the other is in swing.

The stance foot supports the CM dynamics.

The swing foot has a different role:

- move the ankle from its current position to the selected next foothold,
- avoid terrain collision during the transfer,
- arrive with a touchdown state compatible with support transfer,
- remain consistent with rigid body reconstruction.

The swing trajectory must therefore be generated as a constrained geometric motion, not as a purely visual sinusoidal animation.

## 2. Swing variables

Let the current swing ankle be denoted by

$$
A_{\mathrm{sw}}(t).
$$

Let the swing phase start at time $t_0$ and nominally end at time $t_1$.

Define the swing duration

$$
T_{\mathrm{sw}} = t_1 - t_0.
$$

Define the normalized swing parameter

$$
\tau = \frac{t-t_0}{T_{\mathrm{sw}}}.
$$

When the swing proceeds nominally,

$$
\tau \in [0,1].
$$

The swing ankle trajectory is therefore a curve

$$
A_{\mathrm{sw}}(\tau), \qquad \tau \in [0,1].
$$

The trajectory starts at the current ankle position

$$
A_{\mathrm{sw}}(0) = A_0,
$$

and ends at the selected next foothold

$$
A_{\mathrm{sw}}(1) = A_1,
$$

where

$$
A_0 = A_{\mathrm{sw}}(t_0),
\qquad
A_1 = A_{\mathrm{next}}^{\star}.
$$

## 3. Required boundary conditions

A swing generator must satisfy at least the following boundary conditions.

### 3.1. Position conditions

$$
A_{\mathrm{sw}}(0) = A_0,
$$

$$
A_{\mathrm{sw}}(1) = A_1.
$$

### 3.2. Velocity conditions

For nominal swing, it is natural to require small endpoint vertical velocity and continuity with the pre-liftoff and post-touchdown gait state.

A minimal formulation is

$$
\dot A_{\mathrm{sw}}(0) = V_0,
$$

$$
\dot A_{\mathrm{sw}}(1) = V_1,
$$

where $V_0$ and $V_1$ are chosen target endpoint velocities.

At first approximation one may choose them tangent to the ground near liftoff and touchdown, but the formal model does not force a unique choice yet.

### 3.3. Optional acceleration conditions

If one wants higher smoothness, one may also prescribe endpoint accelerations:

$$
\ddot A_{\mathrm{sw}}(0) = a_0,
$$

$$
\ddot A_{\mathrm{sw}}(1) = a_1.
$$

This leads naturally to quintic interpolation families.

## 4. Reference interpolation family

### 4.1. Why a polynomial family is appropriate

The swing trajectory must satisfy endpoint constraints and remain easy to evaluate and differentiate.

For that reason, a polynomial interpolation family is a natural first formal choice.

A good default is the quintic blending function

$$
h(\tau) = 10\tau^3 - 15\tau^4 + 6\tau^5.
$$

It satisfies

$$
h(0)=0,
\qquad
h(1)=1,
$$

$$
h'(0)=0,
\qquad
h'(1)=0,
$$

$$
h''(0)=0,
\qquad
h''(1)=0.
$$

So it is appropriate for smooth endpoint interpolation.

### 4.2. Base ankle trajectory

A first base trajectory from $A_0$ to $A_1$ is

$$
A_{\mathrm{base}}(\tau) = (1-h(\tau))A_0 + h(\tau)A_1.
$$

This curve ensures exact endpoint positions and zero endpoint derivatives with respect to $\tau$.

However, this base curve alone is not enough, because it does not ensure terrain clearance.

## 5. Clearance direction

### 5.1. Need for a clearance direction

A swing ankle trajectory must leave the terrain.

To do so, we need a direction along which the ankle can be lifted away from the ground.

The simplest option would be to use the global vertical direction $e_Y$.

But on a segmented sloped terrain, it is better to use a blended support-normal direction.

### 5.2. Liftoff and touchdown normals

Let the terrain normal under the swing foot at liftoff be denoted by

$$
n_0.
$$

Let the terrain normal at the selected touchdown foothold be denoted by

$$
n_1.
$$

### 5.3. Blended clearance direction

Define the blended normal field by

$$
\widetilde n(\tau) = (1-h(\tau))n_0 + h(\tau)n_1.
$$

Then define the normalized clearance direction by

$$
n_{\mathrm{clr}}(\tau) = \frac{\widetilde n(\tau)}{\|\widetilde n(\tau)\|}.
$$

This vector provides a continuous transition between the liftoff and touchdown terrain normals.

## 6. Clearance profile

### 6.1. Scalar clearance function

Let $H(\tau)$ be the scalar clearance height added along the clearance direction.

A simple symmetric profile is

$$
H(\tau) = H_{\max} \sin^2(\pi \tau).
$$

This satisfies

$$
H(0)=0,
\qquad
H(1)=0,
$$

and

$$
H(\tau) > 0
$$

for all $\tau \in (0,1)$.

Its maximum is reached at

$$
\tau = \frac{1}{2},
$$

with value

$$
H\left(\frac{1}{2}\right) = H_{\max}.
$$

### 6.2. Full swing trajectory

The full ankle trajectory is then

$$
A_{\mathrm{sw}}(\tau) = A_{\mathrm{base}}(\tau) + H(\tau) n_{\mathrm{clr}}(\tau).
$$

This construction has the following properties.

1. It preserves the exact endpoints.
2. It gives a positive mid-swing lift.
3. It remains easy to differentiate.
4. It can be adapted by changing only the scalar $H_{\max}$.

## 7. Distance to the terrain

### 7.1. Need for a distance function

The swing ankle must remain above the terrain during the swing, except at touchdown.

For this purpose we define a terrain-distance function.

Let the Euclidean distance from a point $X$ to the terrain polyline be

$$
d_{\Gamma}(X) = \min_{Y \in \Gamma} \|X-Y\|.
$$

This is a nonnegative function.

It measures the minimum Euclidean distance from the point $X$ to the terrain.

### 7.2. Signed support-side distance

In practice, it is useful to distinguish whether the point lies above or below the local terrain support.

For the swing constraints, a sufficient first formulation is to require a strictly positive safety margin relative to the terrain itself.

Thus we do not yet need a fully signed distance everywhere. The unsigned distance is enough to formulate a first conservative clearance condition.

## 8. Clearance constraint

### 8.1. Global swing clearance condition

Let $h_{\mathrm{safe}}$ be the minimum required ankle clearance above the terrain during swing.

The swing trajectory is admissible if

$$
d_{\Gamma}(A_{\mathrm{sw}}(\tau)) \ge h_{\mathrm{safe}}
$$

for all

$$
\tau \in (0,1).
$$

At touchdown itself, the distance is allowed to go to zero.

### 8.2. Minimal clearance amplitude problem

The clearance amplitude $H_{\max}$ should not be arbitrary.

A principled choice is to take the smallest value that makes the trajectory admissible.

Formally,

$$
H_{\max}^{\star} = \inf \left\{ H \ge 0 : d_{\Gamma}(A_{\mathrm{sw}}^{(H)}(\tau)) \ge h_{\mathrm{safe}} \text{ for all } \tau \in (0,1) \right\},
$$

where $A_{\mathrm{sw}}^{(H)}$ denotes the swing trajectory built with clearance amplitude $H$.

This is the mathematically correct way to define a minimal admissible swing lift.

### 8.3. Reactive clearance adaptation

Under perturbation or difficult terrain transitions, the required clearance may need to increase.

This is naturally expressed by making the safe clearance depend on reactivity and terrain class:

$$
h_{\mathrm{safe}}(\rho) = h_{\mathrm{safe},0} + \rho\,\Delta h_{\mathrm{safe}}.
$$

Thus recovery can produce larger foot lift without changing the architecture.

## 9. Collision avoidance with segmented terrain

### 9.1. Forbidden intersection condition

A swing trajectory is invalid if it intersects terrain before touchdown.

Formally, if there exists some

$$
\tau \in (0,1)
$$

such that

$$
A_{\mathrm{sw}}(\tau) \in \Gamma,
$$

then the trajectory is invalid unless that intersection corresponds to an allowed early landing event.

### 9.2. Conservative admissibility condition

A sufficient conservative condition is:

$$
d_{\Gamma}(A_{\mathrm{sw}}(\tau)) > 0
$$

for all

$$
\tau \in (0,1),
$$

except when the early-landing rule is explicitly triggered.

This condition is stronger than strictly necessary, but it is easy to reason about mathematically.

## 10. Touchdown condition

### 10.1. Geometric touchdown condition

Touchdown occurs when the swing ankle reaches the planned ankle target and the corresponding foot geometry becomes support-admissible.

The most basic positional condition is

$$
\|A_{\mathrm{sw}}(\tau_{\mathrm{td}}) - A_1\| \le \varepsilon_A,
$$

where $\varepsilon_A$ is a small numerical tolerance.

### 10.2. Terrain contact consistency

At touchdown, the ankle must also lie on the terrain support path associated with the selected foothold. That is,

$$
A_{\mathrm{sw}}(\tau_{\mathrm{td}}) \in \Gamma
$$

up to numerical tolerance.

### 10.3. Future rigid-leg feasibility

The touchdown is not valid unless the new stance can support rigid reconstruction.

Therefore we also require

$$
\|P_{\mathrm{td}} - A_1\| \le 2L,
$$

where $P_{\mathrm{td}}$ is the pelvis at touchdown.

This ensures that the new stance leg can exist geometrically.

## 11. Early landing

### 11.1. Why early landing is needed

On segmented terrain, especially when the terrain rises or when the selected foothold sits on a short segment transition, the nominal swing curve may touch the terrain before nominal swing completion.

If that contact is physically valid, the trajectory should terminate early instead of forcing the foot to pass through the ground or to continue above it unnaturally.

### 11.2. Early-landing condition

Let

$$
\tau_{\mathrm{EL}} \in (0,1)
$$

be the first swing parameter such that

$$
d_{\Gamma}(A_{\mathrm{sw}}(\tau_{\mathrm{EL}})) \le \varepsilon_{\Gamma},
$$

where $\varepsilon_{\Gamma}$ is a terrain-contact tolerance.

Early landing is valid only if the contacted terrain point is compatible with a future support foot geometry and rigid stance reconstruction.

So early landing requires:

$$
\|P(\tau_{\mathrm{EL}}) - A_{\mathrm{sw}}(\tau_{\mathrm{EL}})\| \le 2L,
$$

and the foot admissibility conditions at that contact location.

If these conditions are met, the swing may terminate at $\tau_{\mathrm{EL}}$ instead of continuing to $\tau = 1$.

## 12. Delayed landing

### 12.1. Why delayed landing is needed

The opposite situation can also occur.

Because of dynamics, prediction error, or reactive stepping, the ankle may not yet be in a valid support configuration exactly at the nominal endpoint time.

Therefore the model must allow delayed landing.

### 12.2. Delayed-landing extension

If touchdown is not yet valid at nominal swing completion, the trajectory may be continued for a short extension interval

$$
\tau \in [1, 1+\Delta \tau_{\mathrm{DL}}].
$$

In that case, the nominal endpoint $A_1$ remains the target, but the trajectory is reparameterized or extended while preserving collision constraints.

A delayed landing remains admissible only if:

1. the swing ankle still avoids terrain collision during the extension,
2. a valid touchdown occurs before the maximal extension is exhausted,
3. the hybrid walking controller still allows the support switch.

This is mathematically necessary to avoid brittle time-locked transitions.

## 13. Consistency with the future support frame

The swing generator cannot stop at delivering the ankle point $A_1$.

It must also be consistent with the future support geometry derived from $A_1$.

Therefore, at touchdown, the following future support objects must be defined and admissible:

$$
T_{c,1}, \qquad f_{\mathrm{foot},1}, \qquad Q_1, \qquad t_{\mathrm{sup},1}, \qquad n_{\mathrm{sup},1}.
$$

Thus the swing generator is coupled to the foothold planner not only through the target point $A_1$, but through the entire future support structure induced by that point.

## 14. Coupling with the current reactive intensity

The swing generator should also respond continuously to the reactive intensity $\rho$.

The most natural parameters to deform are:

- swing duration,
- clearance height,
- endpoint velocity targets.

Formally, one may write

$$
T_{\mathrm{sw}}(\rho) = T_{\mathrm{sw},0} + \rho\,\Delta T_{\mathrm{sw}},
$$

$$
h_{\mathrm{safe}}(\rho) = h_{\mathrm{safe},0} + \rho\,\Delta h_{\mathrm{safe}},
$$

$$
V_1(\rho) = V_{1,0} + \rho\,\Delta V_1.
$$

This allows the same formal generator to produce nominal swing, reactive high-clearance swing, and faster or slower touchdown preparation depending on the situation.

## 15. Exact relation to the walking protocol

The swing generator fits into the walking protocol as follows.

1. The foothold planner selects

$$
A_1 = A_{\mathrm{next}}^{\star}.
$$

2. The swing generator builds

$$
A_{\mathrm{sw}}(\tau)
$$

from $A_0$ to $A_1$.

3. During the swing, the stance foot remains the active support foot.

4. At valid touchdown, the hybrid controller enters double support.

5. The future support frame built from $A_1$ becomes active progressively through the transfer variable $\beta$.

This makes the swing generator a mathematically distinct object, but not an isolated one.

It is tightly coupled to foothold planning and support transfer.

## 16. Summary of the swing generator

The formal swing generator is now defined by the following objects.

### Swing parameter

$$
\tau = \frac{t-t_0}{T_{\mathrm{sw}}}.
$$

### Boundary conditions

$$
A_{\mathrm{sw}}(0)=A_0,
$$

$$
A_{\mathrm{sw}}(1)=A_1.
$$

### Base interpolation

$$
h(\tau)=10\tau^3 - 15\tau^4 + 6\tau^5,
$$

$$
A_{\mathrm{base}}(\tau) = (1-h(\tau))A_0 + h(\tau)A_1.
$$

### Clearance direction

$$
\widetilde n(\tau) = (1-h(\tau))n_0 + h(\tau)n_1,
$$

$$
n_{\mathrm{clr}}(\tau) = \frac{\widetilde n(\tau)}{\|\widetilde n(\tau)\|}.
$$

### Clearance profile

$$
H(\tau) = H_{\max} \sin^2(\pi \tau).
$$

### Full trajectory

$$
A_{\mathrm{sw}}(\tau) = A_{\mathrm{base}}(\tau) + H(\tau) n_{\mathrm{clr}}(\tau).
$$

### Clearance constraint

$$
d_{\Gamma}(A_{\mathrm{sw}}(\tau)) \ge h_{\mathrm{safe}}
$$

for all intermediate swing values of $\tau$.

### Minimal admissible clearance

$$
H_{\max}^{\star} = \inf \left\{ H \ge 0 : d_{\Gamma}(A_{\mathrm{sw}}^{(H)}(\tau)) \ge h_{\mathrm{safe}} \text{ for all } \tau \in (0,1) \right\}.
$$

### Touchdown condition

$$
\|A_{\mathrm{sw}}(\tau_{\mathrm{td}}) - A_1\| \le \varepsilon_A,
$$

with rigid-leg feasibility and foot admissibility at touchdown.

## 17. Main conclusions of this document

1. Swing must be treated as a constrained geometric motion of the ankle.
2. The swing trajectory must satisfy exact endpoint constraints.
3. Terrain clearance must be expressed through an explicit distance-to-terrain condition.
4. The clearance amplitude should be the minimal one that satisfies the constraint.
5. Early landing and delayed landing are not optional hacks. They are mathematically necessary events on segmented terrain.
6. The swing generator must remain consistent with the future support frame induced by the selected foothold.
7. The same formal generator must serve both nominal and reactive gait through continuous dependence on $\rho$.

## 18. What comes next

The next document should formalize torso orientation and whole-body reconstruction during walking.

In particular, it should specify:

- the target torso angle as a function of speed, slope, CM state, and reactivity,
- the exact reconstruction of pelvis, knees, torso nodes, and foot direction at each frame,
- the consistency conditions that every rendered frame must satisfy.