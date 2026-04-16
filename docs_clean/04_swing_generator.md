# 04 — Formal swing generator

## 0. Goal of this document

The goal of this document is to define the mathematical generator of the swing foot.

This document fixes:

- the ankle swing path from current swing ankle to the selected target ankle,
- the clearance direction and clearance profile,
- collision avoidance over segmented terrain,
- full-foot clearance, not only ankle clearance,
- touchdown,
- early landing,
- delayed landing,
- consistency with the future support frame.

## 1. Principle of the swing phase

During locomotion, the stance foot supports the CM dynamics.

The swing foot has another role:

- move the ankle from its current position to the selected next foothold,
- avoid terrain collision during the transfer,
- arrive in a state compatible with touchdown and support transfer,
- remain compatible with rigid whole-body reconstruction.

Therefore the swing must be generated as a constrained geometric motion, not as a purely visual sinusoidal animation.

## 2. Swing variables

Let the current swing ankle be

$$
A_{\mathrm{sw}}(t).
$$

Let the swing start at time $t_0$ and have nominal touchdown time $t_1$.

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

The swing ankle trajectory is a curve

$$
A_{\mathrm{sw}}(\tau), \qquad \tau \in [0,1].
$$

Define the initial and target ankles by

$$
A_0 = A_{\mathrm{sw}}(t_0),
\qquad
A_1 = A_{\mathrm{next}}^{\star}.
$$

Then

$$
A_{\mathrm{sw}}(0)=A_0,
$$

$$
A_{\mathrm{sw}}(1)=A_1.
$$

## 3. Reference interpolation family

### 3.1. Quintic scalar blend

Use the quintic scalar blend

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

### 3.2. Base ankle trajectory

Define the base ankle path by

$$
A_{\mathrm{base}}(\tau) = (1-h(\tau))A_0 + h(\tau)A_1.
$$

This reference family imposes zero endpoint derivatives with respect to $\tau$.

If a later implementation wants prescribed endpoint world-time velocities or accelerations, it must replace or reparameterize this reference family consistently. In other words, the present document chooses the zero-endpoint-derivative family as the canonical first model.

## 4. Clearance direction

### 4.1. Liftoff and touchdown normals

Let the terrain normal under the swing foot at liftoff be

$$
n_0.
$$

Let the terrain normal at the selected touchdown foothold be

$$
n_1.
$$

### 4.2. Blended clearance direction

Define the blended normal field

$$
\widetilde n(\tau) = (1-h(\tau))n_0 + h(\tau)n_1.
$$

Then define the normalized clearance direction

$$
n_{\mathrm{clr}}(\tau) = \frac{\widetilde n(\tau)}{\|\widetilde n(\tau)\|}.
$$

This gives a continuous transition from liftoff terrain geometry to touchdown terrain geometry.

## 5. Clearance profile

Let $H(\tau)$ be the scalar clearance amplitude along the clearance direction.

A first symmetric profile is

$$
H(\tau) = H_{\max}\sin^2(\pi \tau).
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
\qquad
\text{for all } \tau \in (0,1).
$$

Its maximum is attained at

$$
\tau = \frac{1}{2},
$$

with value

$$
H\left(\frac{1}{2}\right) = H_{\max}.
$$

## 6. Full swing ankle trajectory

The full ankle swing path is

$$
A_{\mathrm{sw}}(\tau) = A_{\mathrm{base}}(\tau) + H(\tau)n_{\mathrm{clr}}(\tau).
$$

This construction has the following advantages.

1. Exact endpoint positions.
2. Positive mid-swing lift.
3. Continuous derivatives.
4. One scalar parameter $H_{\max}$ controls the lift amplitude.

## 7. Distance to the terrain

Let the Euclidean distance from a point $X$ to the terrain polyline be

$$
d_{\Gamma}(X) = \min_{Y \in \Gamma}\|X-Y\|.
$$

This is the basic terrain-distance function.

## 8. Full-foot swing geometry

The swing foot is not only the ankle.

At parameter value $\tau$, the visible swing foot is

$$
\mathcal{F}_{\mathrm{sw}}(\tau)
=
\{A_{\mathrm{sw}}(\tau) + \sigma f_{\mathrm{foot,sw}}(\tau) : \sigma \in [0, L_{\mathrm{toe}}]\}
$$

at the current stage, because $L_{\mathrm{heel}}=0$.

Define the full-foot terrain distance by

$$
d_{\Gamma}^{\mathrm{foot}}(\tau)
=
\min_{\sigma \in [0,L_{\mathrm{toe}}]}
d_{\Gamma}\bigl(A_{\mathrm{sw}}(\tau) + \sigma f_{\mathrm{foot,sw}}(\tau)\bigr).
$$

This is the correct object for clearance reasoning.

Checking only the ankle would be insufficient because the toe may collide even when the ankle does not.

## 9. Clearance constraint

Let $h_{\mathrm{safe}}$ be the minimum required swing-foot clearance.

The full-foot swing trajectory is admissible if

$$
d_{\Gamma}^{\mathrm{foot}}(\tau) \ge h_{\mathrm{safe}}
$$

for all

$$
\tau \in (0,1).
$$

At touchdown itself, this clearance is allowed to vanish.

## 10. Minimal admissible clearance amplitude

The clearance amplitude should not be arbitrary.

The mathematically correct minimal admissible amplitude is

$$
H_{\max}^{\star}
=
\inf\left\{
H \ge 0 :
d_{\Gamma}^{\mathrm{foot},(H)}(\tau) \ge h_{\mathrm{safe}}
\text{ for all } \tau \in (0,1)
\right\},
$$

where $d_{\Gamma}^{\mathrm{foot},(H)}$ is the full-foot terrain distance induced by the trajectory built with swing amplitude $H$.

## 11. Reactive clearance adaptation

Under perturbation or difficult terrain, the required clearance may increase.

This is naturally expressed by

$$
h_{\mathrm{safe}}(\rho) = h_{\mathrm{safe},0} + \rho\,\Delta h_{\mathrm{safe}}.
$$

Therefore reactivity can produce higher swing without changing the architecture.

## 12. Collision avoidance

### 12.1. Forbidden intersection condition

A swing trajectory is invalid if the swing foot intersects the terrain before touchdown.

A conservative condition is

$$
d_{\Gamma}^{\mathrm{foot}}(\tau) > 0
$$

for all

$$
\tau \in (0,1),
$$

except when an explicit early-landing event is triggered.

### 12.2. Why the conservative condition is acceptable

This condition is stronger than strictly necessary, but it is mathematically simple and robust on segmented terrain.

It is therefore a good canonical first model.

## 13. Touchdown condition

A touchdown is valid only if all the following hold.

1. The swing ankle reaches the target ankle up to tolerance:

$$
\|A_{\mathrm{sw}}(\tau_{\mathrm{td}}) - A_1\| \le \varepsilon_A.
$$

2. The ankle lies on the terrain support path of the selected foothold up to terrain tolerance.

3. The future foot geometry is admissible.

4. The new stance leg is rigidly reconstructible:

$$
\|P_{\mathrm{td}} - A_1\| \le 2L.
$$

5. The future support objects are defined:

$$
T_{c,1}, \qquad f_{\mathrm{foot},1}, \qquad Q_1, \qquad t_{\mathrm{sup},1}, \qquad n_{\mathrm{sup},1}.
$$

## 14. Early landing

### 14.1. Definition

Let

$$
\tau_{\mathrm{EL}} \in (0,1)
$$

be the first swing parameter such that

$$
d_{\Gamma}^{\mathrm{foot}}(\tau_{\mathrm{EL}}) \le \varepsilon_{\Gamma},
$$

where $\varepsilon_{\Gamma}$ is a terrain-contact tolerance.

### 14.2. Validity of early landing

Early landing is admissible only if:

1. the contacted terrain location defines an admissible future foot geometry,
2. the new stance leg is rigidly reconstructible:

$$
\|P(\tau_{\mathrm{EL}}) - A_{\mathrm{sw}}(\tau_{\mathrm{EL}})\| \le 2L,
$$

3. the hybrid locomotion controller allows the support transition.

If these conditions hold, the swing terminates early at $\tau_{\mathrm{EL}}$.

## 15. Delayed landing

### 15.1. Why delayed landing is needed

The opposite case may also occur.

At nominal swing completion, a valid touchdown may still be unavailable because of prediction error, terrain irregularity, or reactive deformation.

### 15.2. Delayed-landing approach phase

The delayed landing must **not** be modeled by naively extending the same sinusoidal clearance law beyond $\tau=1$.

Instead, one introduces a terminal approach phase with a new parameter

$$
\eta = \frac{t-t_1}{T_{\mathrm{DL}}},
\qquad
\eta \in [0,1].
$$

Let $A_{\mathrm{end}} = A_{\mathrm{sw}}(1)$ and let $H_{\mathrm{end}}$ be the remaining lift at the beginning of delayed landing.

Choose a monotone decay profile $g(\eta)$ such that

$$
g(0)=0,
\qquad
g(1)=1.
$$

Define a terminal approach path

$$
A_{\mathrm{DL}}(\eta)
=
(1-g(\eta))A_{\mathrm{end}} + g(\eta)A_1 + \widetilde H(\eta)n_1,
$$

where

$$
\widetilde H(0)=H_{\mathrm{end}},
\qquad
\widetilde H(1)=0,
$$

and $\widetilde H$ is monotone nonincreasing.

This fixes the previous logical inconsistency: delayed landing is a distinct terminal approach phase, not an uncontrolled extrapolation of the nominal lift formula.

## 16. Dependence on reactivity

The swing generator should depend continuously on $\rho$.

The natural parameters to deform are:

- swing duration,
- clearance,
- terminal approach duration.

Formally,

$$
T_{\mathrm{sw}}(\rho) = T_{\mathrm{sw},0} + \rho\,\Delta T_{\mathrm{sw}},
$$

$$
h_{\mathrm{safe}}(\rho) = h_{\mathrm{safe},0} + \rho\,\Delta h_{\mathrm{safe}},
$$

$$
T_{\mathrm{DL}}(\rho) = T_{\mathrm{DL},0} + \rho\,\Delta T_{\mathrm{DL}}.
$$

## 17. Main conclusions of this document

1. Swing must be treated as a constrained geometric motion of the ankle.
2. The reference swing family fixes exact endpoint positions and zero endpoint $\tau$-derivatives.
3. Terrain clearance must be checked on the full visible swing foot, not only on the ankle.
4. The clearance amplitude should be the minimal one that satisfies the full-foot constraint.
5. Early landing and delayed landing are mathematically necessary events on segmented terrain.
6. Delayed landing must use a dedicated terminal approach phase rather than blindly extending the nominal clearance formula.
7. The swing generator must remain consistent with the future support frame induced by the selected foothold.
