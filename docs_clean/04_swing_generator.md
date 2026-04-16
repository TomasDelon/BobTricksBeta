# 04 — Formal swing generator

## 0. Goal of this document

The goal of this document is to define the mathematical generator that realizes swing motion under the support-centered `CM-first` architecture.

This document fixes:

- the interpretation of swing as the realization of a planned support acquisition,
- the swing state and the target data provided by the planner,
- the nominal ankle transfer law,
- the full-foot swing geometry with a small positive heel length,
- the clearance direction and clearance profile,
- full-foot collision avoidance on segmented terrain,
- touchdown validity,
- early landing,
- delayed landing,
- consistency with future support acquisition and future load acceptance.

This document does not redefine the planner or the support law. It states how the swing generator realizes a planned candidate without breaking support-centered coherence.

## 1. Principle of the swing phase

During locomotion, the stance side governs the support-aware CM dynamics.

The swing side has another role:

- realize the transfer of the swing ankle from its current position to a future support-acquisition candidate,
- preserve full-foot terrain clearance during the transfer,
- arrive in a touchdown pose compatible with admissible future support geometry,
- remain compatible with rigid morphology and with the post-touchdown support transition.

Therefore swing must not be treated as a purely visual arc.

It is a constrained geometric realization of a future support acquisition.

## 2. Swing target data

### 2.1. Future support-acquisition candidate

The planner provides at least the candidate output

$$
\mathcal{O}_{\mathrm{foot}} = \left(A_{\mathrm{next}}, \mathcal{W}_{\mathrm{td}}, \mathcal{V}_{\mathrm{td}}\right),
$$

where:

- $A_{\mathrm{next}}$ is the future ankle anchor,
- $\mathcal{W}_{\mathrm{td}}$ is the candidate touchdown window,
- $\mathcal{V}_{\mathrm{td}}$ is the viability certificate associated with that candidate.

The swing generator does not invent an unrelated touchdown target. It realizes this planned candidate.

### 2.2. Candidate-induced future foot geometry

The planned future ankle must induce a future contact path

$$
\chi_{\mathrm{next}}(\sigma), \qquad \sigma \in [-L_{\mathrm{heel}}, L_{\mathrm{toe}}],
$$

with

$$
\chi_{\mathrm{next}}(0) = A_{\mathrm{next}}.
$$

The future support acquisition is admissible only if this full contact path exists and is itself admissible on the segmented terrain.

Thus the swing target is not only the ankle point $A_{\mathrm{next}}$. It includes the future foot-support geometry induced by that ankle.

## 3. Swing variables

Let the current swing ankle be

$$
A_{\mathrm{sw}}(t).
$$

Let the swing start at time $t_0$.

Let the nominal touchdown time selected inside the touchdown window be

$$
t_1 \in \mathcal{W}_{\mathrm{td}}.
$$

Define the nominal swing duration

$$
T_{\mathrm{sw}} = t_1 - t_0.
$$

Define the normalized nominal swing parameter

$$
\tau = \frac{t-t_0}{T_{\mathrm{sw}}}.
$$

During nominal swing,

$$
\tau \in [0,1].
$$

Let

$$
A_0 = A_{\mathrm{sw}}(t_0),
$$

$$
A_1 = A_{\mathrm{next}}.
$$

The nominal swing ankle trajectory is a curve

$$
A_{\mathrm{sw}}(\tau), \qquad \tau \in [0,1],
$$

such that

$$
A_{\mathrm{sw}}(0) = A_0,
$$

$$
A_{\mathrm{sw}}(1) = A_1.
$$

## 4. Reference interpolation family

### 4.1. Quintic scalar blend

Use the quintic scalar blend

$$
h(\tau) = 10\tau^3 - 15\tau^4 + 6\tau^5.
$$

It satisfies

$$
h(0) = 0,
$$

$$
h(1) = 1,
$$

$$
h'(0) = h'(1) = 0,
$$

$$
h''(0) = h''(1) = 0.
$$

### 4.2. Base ankle transfer

Define the base ankle path by

$$
A_{\mathrm{base}}(\tau) = (1-h(\tau)) A_0 + h(\tau) A_1.
$$

This is the nominal horizontal or world-space transfer skeleton before adding clearance.

## 5. Clearance direction

### 5.1. Liftoff and touchdown support normals

Let the liftoff support normal under the swing foot be

$$
n_0.
$$

Let the future touchdown support normal induced by the planned candidate be

$$
n_1.
$$

### 5.2. Blended clearance direction

Define the blended support normal field by

$$
\widetilde n(\tau) = (1-h(\tau)) n_0 + h(\tau) n_1.
$$

Then define the normalized clearance direction

$$
n_{\mathrm{clr}}(\tau) = \frac{\widetilde n(\tau)}{\|\widetilde n(\tau)\|}.
$$

This keeps the lift compatible with both the liftoff support geometry and the planned touchdown support geometry.

## 6. Clearance profile

Let $H(\tau)$ be the scalar swing-clearance amplitude.

A first symmetric profile is

$$
H(\tau) = H_{\max} \sin^2(\pi \tau).
$$

It satisfies

$$
H(0) = 0,
$$

$$
H(1) = 0,
$$

and

$$
H(\tau) > 0 \qquad \text{for } \tau \in (0,1).
$$

Its maximum is attained at

$$
\tau = \frac{1}{2},
$$

with value

$$
H\left(\frac{1}{2}\right) = H_{\max}.
$$

## 7. Full swing ankle trajectory

The full nominal ankle trajectory is

$$
A_{\mathrm{sw}}(\tau) = A_{\mathrm{base}}(\tau) + H(\tau) n_{\mathrm{clr}}(\tau).
$$

This construction gives:

1. exact endpoint positions,
2. positive mid-swing lift,
3. continuous derivatives,
4. compatibility with the touchdown support geometry through $n_1$.

## 8. Full visible swing-foot geometry

### 8.1. Small positive heel length

The swing foot must use the same canonical support interval as the rest of the architecture:

$$
[-L_{\mathrm{heel}}, L_{\mathrm{toe}}],
$$

with

$$
0 < L_{\mathrm{heel}} \ll L.
$$

Thus swing clearance must be checked on the full foot, not only on the ankle-to-toe part.

### 8.2. Visible swing-foot direction

Let the visible swing-foot direction at liftoff be

$$
f_{\mathrm{foot},0},
$$

and let the target touchdown foot direction induced by the candidate be

$$
f_{\mathrm{foot},1}.
$$

Define the blended foot-direction field by

$$
\widetilde f_{\mathrm{foot}}(\tau) = (1-h(\tau)) f_{\mathrm{foot},0} + h(\tau) f_{\mathrm{foot},1}.
$$

Then define the normalized visible swing-foot direction

$$
f_{\mathrm{foot,sw}}(\tau) = \frac{\widetilde f_{\mathrm{foot}}(\tau)}{\|\widetilde f_{\mathrm{foot}}(\tau)\|}.
$$

### 8.3. Swing-foot segment in world space

At parameter value $\tau$, define the visible swing-foot segment by

$$
\mathcal{F}_{\mathrm{sw}}(\tau) = \left\{ A_{\mathrm{sw}}(\tau) + \sigma f_{\mathrm{foot,sw}}(\tau) : \sigma \in [-L_{\mathrm{heel}}, L_{\mathrm{toe}}] \right\}.
$$

This is the correct object for swing clearance.

## 9. Distance to the terrain

Let the Euclidean distance from a point $X$ to the terrain polyline be

$$
d_{\Gamma}(X) = \min_{Y \in \Gamma} \|X-Y\|.
$$

Define the full-foot terrain distance during swing by

$$
d_{\Gamma}^{\mathrm{foot}}(\tau) = \min_{\sigma \in [-L_{\mathrm{heel}}, L_{\mathrm{toe}}]} d_{\Gamma}\bigl(A_{\mathrm{sw}}(\tau) + \sigma f_{\mathrm{foot,sw}}(\tau)\bigr).
$$

Checking only the ankle would be insufficient because either the heel-side or the toe-side of the visible foot may collide first.

## 10. Clearance constraint

Let the minimum required swing clearance be

$$
h_{\mathrm{safe}}.
$$

The nominal swing is clearance-admissible only if

$$
d_{\Gamma}^{\mathrm{foot}}(\tau) \ge h_{\mathrm{safe}}
$$

for all

$$
\tau \in (0,1).
$$

At touchdown itself, this clearance is allowed to vanish.

## 11. Minimal admissible clearance amplitude

The clearance amplitude should be the smallest one that preserves the full-foot clearance condition.

Define the minimal admissible amplitude by

$$
H_{\max}^{\star} = \inf\left\{ H \ge 0 : d_{\Gamma}^{\mathrm{foot},(H)}(\tau) \ge h_{\mathrm{safe}} \text{ for all } \tau \in (0,1) \right\},
$$

where $d_{\Gamma}^{\mathrm{foot},(H)}$ is the full-foot terrain distance induced by the trajectory built with amplitude $H$.

This quantity gives the canonical minimal swing lift compatible with the segmented terrain and with the full visible foot.

## 12. Touchdown validity

A touchdown is valid only if all the following hold.

1. The swing ankle reaches the planned future ankle up to tolerance:

$$
\|A_{\mathrm{sw}}(\tau_{\mathrm{td}}) - A_1\| \le \varepsilon_A.
$$

2. The full future foot geometry induced by the contacted ankle is admissible on the terrain.

3. The future contact path

$$
\chi_{\mathrm{next}}(\sigma), \qquad \sigma \in [-L_{\mathrm{heel}}, L_{\mathrm{toe}}]
$$

is well defined.

4. The predicted pelvis at touchdown satisfies exact rigid reachability:

$$
\|P_{\mathrm{td}} - A_1\| \le 2L.
$$

5. The future support objects are well defined:

$$
Q_{\mathrm{next}}, \qquad t_{\mathrm{sup,next}}, \qquad n_{\mathrm{sup,next}}.
$$

6. The post-touchdown support transition remains viable according to the candidate viability certificate or its updated equivalent.

Thus a valid touchdown is not merely geometric contact. It is a support-acquisition event compatible with the future support state.

## 13. Early landing

### 13.1. Definition

Let

$$
\tau_{\mathrm{EL}} \in (0,1)
$$

be the first swing parameter such that

$$
d_{\Gamma}^{\mathrm{foot}}(\tau_{\mathrm{EL}}) \le \varepsilon_{\Gamma},
$$

where $\varepsilon_{\Gamma}$ is a terrain-contact tolerance.

### 13.2. Validity of early landing

Early landing is admissible only if the contacted configuration already defines a valid support acquisition.

This requires at least:

1. the contacted ankle and full foot define an admissible future contact path,
2. the rigid geometry remains viable:

$$
\|P(\tau_{\mathrm{EL}}) - A_{\mathrm{sw}}(\tau_{\mathrm{EL}})\| \le 2L,
$$

3. the future support objects are well defined,
4. the post-contact support transition remains viable.

If these conditions hold, the swing may terminate early at $\tau_{\mathrm{EL}}$.

Otherwise the contact is not accepted as a valid touchdown.

## 14. Delayed landing

### 14.1. Why delayed landing is needed

The opposite case may also occur.

At nominal swing completion, a valid touchdown may still be unavailable because of prediction error, terrain irregularity, or reactive deformation.

The correct response is not to extrapolate the nominal clearance law blindly beyond $\tau = 1$.

### 14.2. Terminal approach phase

Instead, define a separate delayed-landing approach phase with parameter

$$
\eta = \frac{t-t_1}{T_{\mathrm{DL}}}, \qquad \eta \in [0,1].
$$

Let

$$
A_{\mathrm{end}} = A_{\mathrm{sw}}(1).
$$

Let the remaining lift at delayed-landing entry be

$$
H_{\mathrm{end}}.
$$

Choose a monotone scalar blend $g(\eta)$ such that

$$
g(0) = 0,
$$

$$
g(1) = 1.
$$

Define the delayed-landing ankle path by

$$
A_{\mathrm{DL}}(\eta) = (1-g(\eta)) A_{\mathrm{end}} + g(\eta) A_1 + \widetilde H(\eta) n_1,
$$

where

$$
\widetilde H(0) = H_{\mathrm{end}},
$$

$$
\widetilde H(1) = 0,
$$

and $\widetilde H$ is monotone nonincreasing.

This delayed-landing phase is a separate terminal approach, not an uncontrolled extension of the nominal swing arc.

## 15. Consistency with future support acquisition

The swing generator must remain logically consistent with the planner and with the future support law.

Therefore the chain

$$
\mathcal{O}_{\mathrm{foot}} \longrightarrow A_{\mathrm{sw}}(t) \longrightarrow \text{touchdown} \longrightarrow \text{future support acquisition}
$$

must stay well defined.

In particular:

1. the swing target must remain the future ankle selected by the planner,
2. the touchdown pose must induce the future contact path used later by support logic,
3. touchdown must not be declared valid if the resulting support state is undefined.

## 16. Consistency with load acceptance

In walking, touchdown does not immediately imply new support dominance.

Therefore the swing generator must terminate in a configuration compatible with the subsequent finite load-acceptance phase.

A valid swing realization is therefore one that leads not only to contact, but to a viable chain

$$
\text{swing realization} \to \text{touchdown} \to \text{load acceptance} \to \text{new support dominance}.
$$

If the contacted configuration cannot support such a continuation, the swing realization must be considered unsuccessful as a support-acquisition event.

## 17. Reactivity dependence

The swing generator may depend continuously on the reactive intensity

$$
\rho \in [0,1].
$$

The natural parameters to deform are:

- swing duration,
- required clearance,
- terminal approach duration.

For example,

$$
T_{\mathrm{sw}}(\rho) = T_{\mathrm{sw},0} + \rho \, \Delta T_{\mathrm{sw}},
$$

$$
h_{\mathrm{safe}}(\rho) = h_{\mathrm{safe},0} + \rho \, \Delta h_{\mathrm{safe}},
$$

$$
T_{\mathrm{DL}}(\rho) = T_{\mathrm{DL},0} + \rho \, \Delta T_{\mathrm{DL}}.
$$

Thus stronger perturbations may produce longer, higher, or more cautious swing realizations without changing the architecture.

## 18. Main conclusions of this document

1. Swing must be treated as the realization of a planned support acquisition.

2. The swing target is richer than a mere ankle point because it includes future support geometry.

3. Full-foot clearance must be checked on the entire interval

$$
[-L_{\mathrm{heel}}, L_{\mathrm{toe}}].
$$

4. A small positive heel length is therefore part of the canonical swing geometry.

5. Touchdown validity requires future support admissibility, not only ankle contact.

6. Early landing and delayed landing are mathematically necessary events on segmented terrain.

7. The swing generator must remain compatible with the future support-acquisition and load-acceptance logic.

8. Reactivity may deform swing timing and clearance, but not the hard geometric constraints.