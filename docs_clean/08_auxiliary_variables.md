# 08 — Auxiliary variables, support progression, and reactive intensity

## 0. Goal of this document

The goal of this document is to define the auxiliary variables that appear across the locomotion architecture and that must be propagated explicitly by the prediction layer.

This document fixes:

- the rocker scalar $\sigma$,
- the double-support transfer scalar $\beta$,
- the reactive intensity $\rho$,
- their domains,
- their meanings,
- their evolution laws,
- their continuity requirements,
- their role in viability and planning.

A central principle of this document is the following.

These variables are not decorative symbols.

If they appear in support geometry, planning, or mode transitions, then they must be defined as true state variables or true derived variables with explicit laws.

## 1. Why these variables need a dedicated document

The current clean architecture already uses:

- the support point $Q$,
- the support tangent $t_{\mathrm{sup}}$,
- the support normal $n_{\mathrm{sup}}$,
- the local support coordinates of the CM,
- hybrid mode transitions,
- reactivity-dependent deformation of gait laws.

However, several key quantities are spread across the current documents and still require a single canonical treatment.

These quantities are:

- the progression of effective support inside the active foot,
- the transfer of effective support from one foot to the other in double support,
- the scalar intensity that continuously deforms nominal locomotion into reactive locomotion.

Those are precisely the roles of $\sigma$, $\beta$, and $\rho$.

## 2. Classification of the three variables

The three variables do not have the same mathematical status.

### 2.1. Rocker scalar

The quantity $\sigma$ is a support-geometry variable.

It determines where the effective support point lies inside the current foot support interval.

### 2.2. Transfer scalar

The quantity $\beta$ is a support-allocation variable.

It determines how support is transferred from the old support foot to the new support foot during double support.

### 2.3. Reactive intensity

The quantity $\rho$ is a regime-intensity variable.

It determines how far the system has moved away from nominal locomotion and therefore how strongly nominal laws should be deformed.

## 3. Rocker scalar $\sigma$

### 3.1. Definition

Let the active support foot have ankle point $A$ and effective support tangent $t_{\mathrm{sup}}$.

At the current stage of the project, the active support interval of the foot relative to the ankle is

$$
\mathcal{I}_{\mathrm{foot}} = [0, L_{\mathrm{toe}}]
$$

because

$$
L_{\mathrm{heel}} = 0.
$$

The rocker scalar is the unique scalar

$$
\sigma \in [0, L_{\mathrm{toe}}]
$$

such that the effective support point is

$$
Q = A + \sigma t_{\mathrm{sup}}.
$$

Thus $\sigma$ determines where the effective support lies inside the active foot.

### 3.2. Why $\sigma$ is necessary

If one writes support dynamics relative to $Q$ but reconstructs the stance geometry relative to $A$, then one must explicitly relate those two reference points.

The exact relation is

$$
s_Q = s_A - \sigma,
$$

$$
z_Q = z_A.
$$

Therefore, without $\sigma$, the support-local and ankle-local descriptions are not closed.

### 3.3. Interpretation

The scalar $\sigma$ is the canonical model variable representing the progression of effective support from the ankle toward the forefoot during stance.

It is inspired by the anterior progression of the center of pressure during stance and by the classical foot-rocker interpretation of gait.

But in this architecture, $\sigma$ is **not** defined as the exact physical center of pressure.

It is the effective support coordinate used by the `CM-first` model.

That distinction must remain explicit.

## 4. Canonical law for $\sigma$

### 4.1. Support-phase variable

Define a support-phase scalar

$$
\eta_{\sigma} \in [0,1].
$$

This scalar parametrizes the progression of the current support phase.

The exact meaning of $\eta_{\sigma}$ depends on the active gait protocol.

In walking single support, it is natural to define it from support-local progression.

In running stance, it is natural to define it from stance progression or compression-release progression.

### 4.2. Monotone rocker map

Let

$$
\gamma_{\sigma} : [0,1] \to [0,1]
$$

be a smooth monotone function satisfying

$$
\gamma_{\sigma}(0) = 0,
$$

$$
\gamma_{\sigma}(1) = 1.
$$

Then the canonical rocker law is

$$
\sigma = L_{\mathrm{toe}} \gamma_{\sigma}(\eta_{\sigma}).
$$

This formula guarantees:

- support starts near the ankle,
- support progresses toward the forefoot,
- $Q$ always remains inside the support interval.

### 4.3. Differential form

Whenever $\eta_{\sigma}$ is differentiable in time, one may write

$$
\dot \sigma = L_{\mathrm{toe}} \gamma_{\sigma}'(\eta_{\sigma}) \dot \eta_{\sigma}.
$$

This makes $\sigma$ an explicitly propagated variable in the prediction layer.

## 5. Admissibility and continuity of $\sigma$

The rocker scalar is admissible only if

$$
0 \le \sigma \le L_{\mathrm{toe}}.
$$

If a numerical propagation attempts to produce a value outside this interval, then either:

- the value must be saturated,
- or the predicted state must be marked as non-viable,

depending on whether the overshoot is interpreted as numerical approximation error or as genuine model inconsistency.

Inside one continuous support phase, $\sigma$ must evolve continuously in time.

At a support change event, a reset is allowed because the active support foot may change.

## 6. Initial and reset values of $\sigma$

### 6.1. Walking stance entry

When a foot becomes the new active stance foot, a canonical initialization is

$$
\sigma^{+} = \sigma_{\mathrm{init}}^{\mathrm{st}},
$$

with

$$
0 \le \sigma_{\mathrm{init}}^{\mathrm{st}} \ll L_{\mathrm{toe}}.
$$

This expresses that newly acquired support begins near the rear part of the active support interval.

### 6.2. Running stance entry

When a foot becomes the new active running stance foot, the initialization may differ:

$$
\sigma^{+} = \sigma_{\mathrm{init}}^{\mathrm{run}}.
$$

The architecture allows this value to be gait-dependent.

What is required is only that it remain inside the active support interval and that the chosen reset be consistent with the support geometry induced by touchdown.

## 7. Double-support transfer scalar $\beta$

### 7.1. Definition

During double support, the old support foot and the new support foot coexist.

The transfer scalar is

$$
\beta \in [0,1].
$$

Its interpretation is:

- $\beta = 0$ means support still fully assigned to the old support foot,
- $\beta = 1$ means support fully assigned to the new support foot,
- intermediate values mean partial transfer.

### 7.2. Why $\beta$ is necessary

Without $\beta$, double support would remain only a verbal phase label.

But in the present architecture, double support affects:

- the support point,
- the support tangent,
- the support normal,
- the stance law,
- the transition conditions.

Therefore a scalar transfer coordinate is required.

## 8. Canonical law for $\beta$

### 8.1. Double-support phase variable

Define a normalized double-support phase scalar

$$
\eta_{\beta} \in [0,1].
$$

A canonical time-based form is

$$
\eta_{\beta} = \mathrm{clamp}\left(\frac{t - t_{\mathrm{DS},0}}{T_{\mathrm{DS}}}, 0, 1\right).
$$

### 8.2. Monotone transfer map

Let

$$
\gamma_{\beta} : [0,1] \to [0,1]
$$

be a smooth monotone function satisfying

$$
\gamma_{\beta}(0) = 0,
$$

$$
\gamma_{\beta}(1) = 1.
$$

Then define

$$
\beta = \gamma_{\beta}(\eta_{\beta}).
$$

### 8.3. Differential form

Whenever $\eta_{\beta}$ is differentiable, one has

$$
\dot \beta = \gamma_{\beta}'(\eta_{\beta}) \dot \eta_{\beta}.
$$

Thus $\beta$ is also an explicitly propagated variable.

## 9. Support objects induced by $\beta$

During double support, let:

- $Q_b$ be the old-foot support point,
- $Q_f$ be the new-foot support point,
- $t_b$ be the old support tangent,
- $t_f$ be the new support tangent.

Then the canonical interpolated support point is

$$
Q_{DS} = (1-\beta) Q_b + \beta Q_f.
$$

The canonical interpolated support tangent is

$$
t_{DS} = \frac{(1-\beta) t_b + \beta t_f}{\|(1-\beta) t_b + \beta t_f\|},
$$

provided the denominator is nonzero.

The associated normal is

$$
n_{DS} = (-t_{DS,y}, t_{DS,x}).
$$

Thus $\beta$ determines not only support transfer in an abstract sense, but the actual support geometry seen by the CM.

## 10. Admissibility and continuity of $\beta$

The transfer scalar is admissible only if

$$
0 \le \beta \le 1.
$$

During one continuous double-support phase, $\beta$ must evolve continuously.

At double-support entry,

$$
\beta^{+} = 0.
$$

At double-support exit,

$$
\beta^{-} = 1.
$$

If a transition out of double support occurs before $\beta$ reaches values compatible with the planned support transfer, then the corresponding prediction must be considered non-viable unless another explicitly allowed event explains that interruption.

## 11. Reactive intensity $\rho$

### 11.1. Definition

The reactive intensity is a scalar

$$
\rho \in [0,1].
$$

Its meaning is not “stability” in an abstract global sense.

Its canonical meaning is:

$$
\rho = 0
$$

corresponds to nominal locomotion,

and

$$
\rho = 1
$$

corresponds to strongly reactive locomotion at the edge of the admissible recovery regime.

### 11.2. Status of $\rho$

Unlike $\sigma$ and $\beta$, the variable $\rho$ is not directly a support-geometry coordinate.

It is a regime-deformation variable.

Its role is to modulate:

- effective target lengths,
- planning horizons,
- swing clearances,
- timing targets,
- relative weights in planning costs,
- viability margins.

Thus $\rho$ is a canonical architectural variable of BobTricksBeta.

It is not claimed to be a direct standard biomechanical observable.

## 12. Construction of $\rho$ from normalized deficits

To avoid making $\rho$ a vague symbol, define a family of normalized deficit measures.

### 12.1. Stance-manifold deficit

In walking, define the nominal stance deficit by

$$
\delta_{\Phi} = \frac{|\Phi_w|}{\Delta_{\Phi}}.
$$

### 12.2. Capture deficit

If the local support interval is denoted by $\mathcal{I}_{Q}$ and the capture quantity by $\xi$, define

$$
\delta_{\xi} = \frac{d_{\mathcal{I}}(\xi, \mathcal{I}_{Q})}{\Delta_{\xi}}.
$$

### 12.3. Velocity deficit

If the tangential velocity target is $v_t^{\star}$, define

$$
\delta_v = \frac{|\dot s_A - v_t^{\star}|}{\Delta_v}.
$$

### 12.4. Reach deficit

If the pelvis-to-ankle distance is near the rigid limit, define

$$
\delta_{\ell} = \frac{\max\{0, \|P-A\| - \ell_{\mathrm{pref}}\}}{2L - \ell_{\mathrm{pref}}}.
$$

### 12.5. Aggregation law

Define an aggregated reactivity raw score by

$$
\widetilde \rho = \max\{\delta_{\Phi}, \delta_{\xi}, \delta_v, \delta_{\ell}\}.
$$

Then define

$$
\rho = \mathrm{clamp}(\widetilde \rho, 0, 1).
$$

This gives a mathematically explicit construction of $\rho$ from normalized deficits.

## 13. Dynamic law for $\rho$

A purely instantaneous definition of $\rho$ may lead to undesirable jitter.

Therefore the propagated variable should be a filtered version of the raw score.

Let

$$
\rho_{\mathrm{raw}} = \mathrm{clamp}(\widetilde \rho, 0, 1).
$$

Then define the canonical filtered law

$$
\dot \rho = \frac{\rho_{\mathrm{raw}} - \rho}{\tau_{\rho}}
$$

with

$$
\tau_{\rho} > 0.
$$

This keeps $\rho$ continuous in time while still allowing strong reactive growth when needed.

## 14. Role of $\rho$ in the architecture

The reactive intensity deforms nominal laws continuously.

### 14.1. Walking length deformation

A canonical example is

$$
\ell_w^{\star}(\rho) = 2L - \varepsilon_w L - \rho \, \Delta \ell_w L.
$$

### 14.2. Planning-horizon deformation

A canonical example is

$$
\sigma_{\max}^{\mathrm{plan}}(\rho)
=
\sigma_{\max,0}^{\mathrm{plan}} + \rho \, \Delta \sigma_{\max}^{\mathrm{plan}}.
$$

### 14.3. Step-length deformation

A canonical example is

$$
\sigma_{\mathrm{step}}^{\star}(\rho)
=
\sigma_{\mathrm{step},0}^{\star} + \rho \, \Delta \sigma_{\mathrm{step}}^{\star}.
$$

### 14.4. Swing-clearance deformation

A canonical example is

$$
h_{\mathrm{safe}}(\rho) = h_{\mathrm{safe},0} + \rho \, \Delta h_{\mathrm{safe}}.
$$

Thus $\rho$ is the canonical scalar through which the same architecture deforms from nominal locomotion into recovery locomotion.

## 15. State or derived variable status

The three variables need not all have identical implementation status.

### 15.1. $\sigma$

The variable $\sigma$ should be treated as an explicit propagated support-state variable.

### 15.2. $\beta$

The variable $\beta$ should be treated as an explicit propagated transition-state variable during double support.

### 15.3. $\rho$

The variable $\rho$ may be treated either as:

- a derived filtered variable, or
- a propagated state variable with its own evolution law.

What is not allowed is to let it appear in equations without one of these two statuses being fixed.

## 16. Viability conditions involving $\sigma$, $\beta$, and $\rho$

### 16.1. Support viability

The support state is viable only if

$$
0 \le \sigma \le L_{\mathrm{toe}}.
$$

### 16.2. Transfer viability

The double-support transfer state is viable only if

$$
0 \le \beta \le 1
$$

and the interpolated support geometry is well defined.

### 16.3. Reactive admissibility

The reactive regime remains admissible only if the deformations generated by $\rho$ do not violate hard geometric bounds.

For example, a deformed preferred length may change, but the exact rigid bound

$$
\|P-A\| \le 2L
$$

remains non-negotiable.

Thus $\rho$ may deform nominal laws, but it may never deactivate exact morphology.

## 17. Reset behavior across events

A clean hybrid architecture must specify what happens to these variables at mode changes.

### 17.1. Support acquisition

When a new stance foot becomes active:

- $\sigma$ is initialized by a support-entry reset,
- $\beta$ is either set to zero or becomes inactive,
- $\rho$ remains continuous unless the event definition explicitly includes a reset.

### 17.2. Double-support entry

When double support begins:

- the old-foot and new-foot support objects become simultaneously active,
- $\beta$ is initialized to zero,
- the current $\sigma$ values of both feet become relevant for support interpolation if the chosen model uses them.

### 17.3. Double-support exit

When the new support becomes fully active:

- $\beta$ reaches one and then becomes inactive,
- the new active foot retains its own support-state initialization or continuation,
- $\rho$ remains continuous.

### 17.4. Takeoff and flight

When a foot loses active support:

- the corresponding support-based $\sigma$ becomes inactive,
- $\beta$ is inactive outside double support,
- $\rho$ remains part of the predictive and reactive state.

## 18. Scientific status of the three variables

The three variables do not claim the same scientific status.

### 18.1. $\sigma$

The idea behind $\sigma$ is strongly motivated by support progression and center-of-pressure progression during stance.

### 18.2. $\beta$

The idea behind $\beta$ is strongly motivated by the fact that double support is a finite transfer phase and that support redistribution affects whole-body balance.

### 18.3. $\rho$

The variable $\rho$ is not introduced as a standard measured biomechanical quantity.

It is introduced as a mathematically explicit regime-deformation scalar, canonically built from normalized viability deficits.

This distinction must be preserved in all later documents.

## 19. Main conclusions of this document

1. $\sigma$ is the canonical effective-support coordinate inside the active foot.
2. $\beta$ is the canonical support-transfer coordinate during double support.
3. $\rho$ is the canonical reactive-intensity coordinate deforming nominal locomotion into recovery locomotion.
4. None of these variables may remain implicit if they appear in equations used by planning, prediction, or transitions.
5. $\sigma$ and $\beta$ must remain bounded inside their natural intervals.
6. $\rho$ must be built from explicit normalized deficits and propagated continuously.
7. Hard geometric constraints remain hard even when $\rho$ is large.
8. These variables are part of the state architecture and must therefore be handled explicitly by the prediction layer.

## 20. What comes next

The next clean document should use the prediction layer and the auxiliary variables together to close the chain from foothold planning to actual touchdown realization.

The next natural target is therefore a document on:

- touchdown viability,
- candidate realization,
- event-consistent contact acquisition,
- failure cases linking planner, swing, and support transition.
