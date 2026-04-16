# 08 — Auxiliary variables, support progression, and reactive intensity

## 0. Goal of this document

The goal of this document is to define the auxiliary variables that must be propagated explicitly by the support-centered locomotion architecture.

This document fixes:

- the canonical support coordinate $\sigma$,
- the canonical bilateral support-allocation variable $\beta$,
- the canonical reactive-intensity variable $\rho$,
- their domains,
- their meanings,
- their state status,
- their evolution laws,
- their admissibility conditions,
- their reset behavior across hybrid events.

These variables are not decorative symbols.

If they appear in support geometry, planning, prediction, or hybrid transitions, then they must be defined explicitly and propagated coherently.

## 1. Classification of the three variables

The three variables do not have the same mathematical role.

### 1.1. Support coordinate

The quantity $\sigma$ is the canonical support coordinate inside one active foot.

It determines where the effective support point lies along the actual foot-terrain contact path.

### 1.2. Bilateral support-allocation variable

The quantity $\beta$ is the canonical support-allocation variable whenever both feet are simultaneously active.

It determines how effective support is distributed between the two feet.

### 1.3. Reactive-intensity variable

The quantity $\rho$ is the canonical regime-deformation variable.

It determines how far the system has moved away from nominal locomotion and therefore how strongly nominal laws should be deformed.

## 2. Canonical support coordinate $\sigma$

### 2.1. Contact-path definition

Let one active foot have ankle point $A$, heel length $L_{\mathrm{heel}} > 0$, toe length $L_{\mathrm{toe}} > 0$, and contact path

$$
\chi(\sigma), \qquad \sigma \in [-L_{\mathrm{heel}}, L_{\mathrm{toe}}],
$$

with

$$
\chi(0) = A.
$$

The effective support point is defined by

$$
Q = \chi(\sigma).
$$

The local support tangent is

$$
t_{\mathrm{sup}} = \frac{\partial_{\sigma} \chi(\sigma)}{\|\partial_{\sigma} \chi(\sigma)\|},
$$

and the associated support normal is

$$
n_{\mathrm{sup}} = (-t_{{\mathrm{sup}},y}, t_{{\mathrm{sup}},x}).
$$

Thus the scalar

$$
\sigma \in [-L_{\mathrm{heel}}, L_{\mathrm{toe}}]
$$

is the primary support variable, and the support point $Q$ is derived from it.

### 2.2. Why this replaces the old linear rule

The old linear relation

$$
Q = A + \sigma t_{\mathrm{sup}}
$$

is not canonical on general two-segment contact geometry.

When the foot spans two admissible terrain segments, the correct support progression follows the actual contact path under the foot. Therefore the support point must be defined by the path law

$$
Q = \chi(\sigma),
$$

not by a globally linearized expression.

### 2.3. Small positive heel length

The support interval is two-sided:

$$
[-L_{\mathrm{heel}}, L_{\mathrm{toe}}],
$$

with

$$
0 < L_{\mathrm{heel}} \ll L.
$$

Thus support progression is no longer purely forefoot-directed from a zero-heel interval. A small rear margin is part of the canonical model.

## 3. Support-local and ankle-local coordinates

Given the active support frame,

$$
C = Q + s_Q t_{\mathrm{sup}} + z_Q n_{\mathrm{sup}}.
$$

Relative to the ankle,

$$
C = A + s_A t_{\mathrm{sup}} + z_A n_{\mathrm{sup}}.
$$

Likewise, for the pelvis,

$$
P = A + p_A t_{\mathrm{sup}} + q_A n_{\mathrm{sup}},
$$

where

$$
p_A = (P-A) \cdot t_{\mathrm{sup}}.
$$

The quantity $p_A$ is the natural local progression coordinate for support evolution because using $s_Q$ directly would reintroduce a circular dependence through $Q = \chi(\sigma)$.

## 4. Canonical law for $\sigma$

### 4.1. State-based support phase

The canonical support law must be state-based.

A purely time-based support law is not acceptable as the canonical law because it does not adapt correctly to speed, slope, perturbation intensity, or support geometry.

Define protocol-dependent pelvis thresholds

$$
p_{\mathrm{heel}} < p_{\mathrm{toe}}.
$$

Then define the normalized support phase by

$$
\eta_{\sigma} = \mathrm{clamp}\left(\frac{p_A - p_{\mathrm{heel}}}{p_{\mathrm{toe}} - p_{\mathrm{heel}}}, 0, 1\right).
$$

### 4.2. Monotone support map

Let

$$
\Gamma_{\sigma} : [0,1] \to [-L_{\mathrm{heel}}, L_{\mathrm{toe}}]
$$

be a smooth monotone nondecreasing map satisfying

$$
\Gamma_{\sigma}(0) = -L_{\mathrm{heel}},
$$

$$
\Gamma_{\sigma}(1) = L_{\mathrm{toe}}.
$$

Then define the support coordinate by

$$
\sigma = \Gamma_{\sigma}(\eta_{\sigma}).
$$

A normalized representation is

$$
\Gamma_{\sigma}(\eta) = -L_{\mathrm{heel}} + (L_{\mathrm{heel}} + L_{\mathrm{toe}}) \gamma_{\sigma}(\eta),
$$

where

$$
\gamma_{\sigma} : [0,1] \to [0,1]
$$

is smooth, monotone, and satisfies

$$
\gamma_{\sigma}(0) = 0,
$$

$$
\gamma_{\sigma}(1) = 1.
$$

### 4.3. Differential form

Whenever $\eta_{\sigma}$ is differentiable,

$$
\dot \sigma = \Gamma_{\sigma}'(\eta_{\sigma}) \dot \eta_{\sigma}.
$$

Whenever the support tangent varies explicitly in time,

$$
\dot p_A = \dot P \cdot t_{\mathrm{sup}} + (P-A) \cdot \dot t_{\mathrm{sup}}.
$$

In early approximations the $\dot t_{\mathrm{sup}}$ term may be simplified, but the canonical model must keep the dependence conceptually explicit.

## 5. Protocol dependence of $\sigma$

The support variable remains the same across protocols, but its law is protocol-dependent.

### 5.1. Standing

In bilateral standing, one should not force a full heel-to-toe rollover.

The support coordinate should remain inside a narrower standing-compatible band centered around an internal preferred standing value

$$
\sigma_{\mathrm{stand}}^{\star}.
$$

A canonical standing law is therefore a filtered support-target law of the form

$$
\dot \sigma = \frac{\sigma_{\mathrm{raw}}^{\mathrm{stand}} - \sigma}{\tau_{\sigma}^{\mathrm{stand}}},
$$

with saturation to

$$
[-L_{\mathrm{heel}}, L_{\mathrm{toe}}].
$$

### 5.2. Walking single support

In walking single support, the full support progression from rear-biased stance toward forefoot-biased late stance is canonical.

Therefore walking should use a dedicated phase law

$$
\eta_{\sigma}^{\mathrm{walk}}
$$

and a dedicated support map

$$
\sigma^{\mathrm{walk}} = \Gamma_{\sigma}^{\mathrm{walk}}\bigl(\eta_{\sigma}^{\mathrm{walk}}\bigr).
$$

### 5.3. Running stance

Running stance must not simply reuse the walking thresholds.

Running is shorter, more compressed, and typically more anteriorly biased.

Therefore running should use

$$
\eta_{\sigma}^{\mathrm{run}}
$$

and a distinct map

$$
\sigma^{\mathrm{run}} = \Gamma_{\sigma}^{\mathrm{run}}\bigl(\eta_{\sigma}^{\mathrm{run}}\bigr).
$$

The running map should be more forefoot-biased than the walking one.

## 6. Admissibility and resets of $\sigma$

The support coordinate is admissible only if

$$
-L_{\mathrm{heel}} \le \sigma \le L_{\mathrm{toe}}.
$$

Inside one continuous support phase, $\sigma$ must evolve continuously.

At support-acquisition events, a reset is allowed.

### 6.1. Walking stance entry

When a foot becomes the new active walking stance foot, a canonical initialization is

$$
\sigma^{+} = \sigma_{\mathrm{init}}^{\mathrm{walk}},
$$

with a rear-biased or mildly interior value inside the active interval.

### 6.2. Running stance entry

When a foot becomes the new active running stance foot, one may use

$$
\sigma^{+} = \sigma_{\mathrm{init}}^{\mathrm{run}},
$$

with a more anterior initialization if desired.

### 6.3. Standing entry or re-entry

When the architecture returns to a bilateral standing regime, each foot support coordinate should be reset or filtered toward a standing-compatible value rather than blindly continuing a late-walk or late-run support coordinate.

## 7. Canonical bilateral support-allocation variable $\beta$

### 7.1. General meaning

The variable

$$
\beta \in [0,1]
$$

is the canonical support-allocation variable whenever both feet are simultaneously active.

Its interpretation is general:

- $\beta = 0$ means full support dominance of the left or old support foot,
- $\beta = 1$ means full support dominance of the right or new support foot,
- intermediate values represent bilateral support.

This variable must not be restricted to one old-style double-support phase only.

### 7.2. Bilateral support geometry induced by $\beta$

Let the two active support points be

$$
Q_L, \qquad Q_R,
$$

with corresponding tangents

$$
t_L, \qquad t_R.
$$

Then the effective bilateral support point is

$$
Q = (1-\beta) Q_L + \beta Q_R.
$$

The effective bilateral support tangent is

$$
t_{\mathrm{sup}} = \frac{(1-\beta) t_L + \beta t_R}{\|(1-\beta) t_L + \beta t_R\|},
$$

provided the denominator is nonzero.

The associated support normal is

$$
n_{\mathrm{sup}} = (-t_{{\mathrm{sup}},y}, t_{{\mathrm{sup}},x}).
$$

Thus $\beta$ determines not only an abstract support split, but the effective support frame seen by the CM.

## 8. Protocol dependence of $\beta$

The same variable $\beta$ is reused across different bilateral phases.

### 8.1. Quiet or adaptive standing

In bilateral standing, $\beta$ remains an interior support-allocation variable.

A canonical standing target can be defined from the relative CM position between the two active support points.

Let

$$
e_{LR} = \frac{Q_R - Q_L}{\|Q_R - Q_L\|}
$$

when the support points are distinct.

Define the pairwise support coordinate of the CM by

$$
\nu = (C - Q_L) \cdot e_{LR}.
$$

Let

$$
D = \|Q_R - Q_L\|.
$$

Then a canonical raw standing target is

$$
\beta_{\mathrm{raw}}^{\mathrm{stand}} = \mathrm{clamp}\left(\frac{\nu}{D}, 0, 1\right).
$$

The propagated variable may then satisfy

$$
\dot \beta = \frac{\beta_{\mathrm{raw}}^{\mathrm{stand}} - \beta}{\tau_{\beta}^{\mathrm{stand}}}.
$$

### 8.2. Gait initiation

In gait initiation, $\beta$ remains the same bilateral variable, but its target must move toward the future stance foot.

The canonical gait-initiation law must not be purely temporal. It should be mixed time-state based:

$$
\dot \beta = F_{\beta}^{\mathrm{GI}}(X).
$$

### 8.3. Load acceptance

In load acceptance, $\beta$ governs the transfer from old support dominance toward new support dominance after touchdown.

Again, the canonical law should be mixed time-state based:

$$
\dot \beta = F_{\beta}^{\mathrm{LA}}(X).
$$

The essential point is that $\beta$ is one general bilateral-support variable reused across bilateral support regimes, not a narrow special-purpose timer.

## 9. Admissibility and resets of $\beta$

The support-allocation variable is admissible only if

$$
0 \le \beta \le 1.
$$

Inside one continuous bilateral-support phase, $\beta$ must evolve continuously.

At bilateral-phase entry, an initialization is allowed.

### 9.1. Standing entry

When bilateral standing is entered or re-entered, $\beta$ should be initialized or filtered toward a standing-compatible interior value.

### 9.2. Gait-initiation entry

When gait initiation begins, $\beta$ should start from the current standing-support allocation and then evolve toward the future stance foot.

### 9.3. Load-acceptance entry

When touchdown begins a load-acceptance phase, a canonical initialization is

$$
\beta^{+} = \beta_{\mathrm{init}}^{\mathrm{acc}},
$$

with a value still near the old-support side.

### 9.4. Bilateral-phase exit

When bilateral support ends, $\beta$ becomes inactive rather than remaining as a meaningless scalar outside bilateral-support regimes.

## 10. Canonical reactive-intensity variable $\rho$

### 10.1. Meaning

The reactive intensity is a scalar

$$
\rho \in [0,1].
$$

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

### 10.2. Status

Unlike $\sigma$ and $\beta$, the variable $\rho$ is not directly a support-geometry coordinate.

It is a regime-deformation variable that modulates:

- target lengths,
- planning horizons,
- support-law parameters,
- swing clearances,
- touchdown timings,
- planning cost weights,
- viability margins.

## 11. Canonical construction of $\rho$

To avoid making $\rho$ a vague symbol, define normalized deficit measures.

### 11.1. Standing-support deficit

A canonical standing-support deficit is

$$
\delta_{\mathrm{stand}} = \frac{E_{\mathrm{stand}}}{\Delta_{\mathrm{stand}}},
$$

where $E_{\mathrm{stand}}$ is a standing-support error and $\Delta_{\mathrm{stand}} > 0$ is a normalization scale.

### 11.2. Capture deficit

If the candidate or current support interval is denoted by $\mathcal{I}_{Q}$ and the capture quantity by $\xi$, define

$$
\delta_{\xi} = \frac{d_{\mathcal{I}}(\xi, \mathcal{I}_{Q})}{\Delta_{\xi}}.
$$

### 11.3. Velocity deficit

If the tangential velocity target is $v_t^{\star}$, define

$$
\delta_v = \frac{|\dot s_A - v_t^{\star}|}{\Delta_v}.
$$

### 11.4. Reach deficit

If the pelvis-to-ankle geometry approaches the rigid limit, define

$$
\delta_{\ell} = \frac{\max\{0, \|P-A\| - \ell_{\mathrm{pref}}\}}{2L - \ell_{\mathrm{pref}}}.
$$

### 11.5. Aggregation law

Define the raw reactive score by

$$
\widetilde \rho = \max\{\delta_{\mathrm{stand}}, \delta_{\xi}, \delta_v, \delta_{\ell}\}.
$$

Then define

$$
\rho_{\mathrm{raw}} = \mathrm{clamp}(\widetilde \rho, 0, 1).
$$

## 12. Dynamic law for $\rho$

To avoid jitter, the propagated variable should be filtered.

Define

$$
\dot \rho = \frac{\rho_{\mathrm{raw}} - \rho}{\tau_{\rho}},
$$

with

$$
\tau_{\rho} > 0.
$$

Thus $\rho$ remains continuous in time while still reacting to changes in viability.

## 13. Role of $\rho$ in the architecture

The variable $\rho$ deforms nominal laws continuously.

### 13.1. Walking-length deformation

A canonical example is

$$
\ell_w^{\star}(\rho) = 2L - \varepsilon_w L - \rho \, \Delta \ell_w L.
$$

### 13.2. Planning-horizon deformation

A canonical example is

$$
\ell_{\max}^{\mathrm{plan}}(\rho) = \ell_{\max,0}^{\mathrm{plan}} + \rho \, \Delta \ell_{\max}^{\mathrm{plan}}.
$$

### 13.3. Step-length deformation

A canonical example is

$$
\ell_{\mathrm{step}}^{\star}(\rho) = \ell_{\mathrm{step},0}^{\star} + \rho \, \Delta \ell_{\mathrm{step}}^{\star}.
$$

### 13.4. Swing-clearance deformation

A canonical example is

$$
h_{\mathrm{safe}}(\rho) = h_{\mathrm{safe},0} + \rho \, \Delta h_{\mathrm{safe}}.
$$

Thus the same architecture deforms continuously from nominal locomotion into recovery locomotion.

## 14. Viability conditions involving $\sigma$, $\beta$, and $\rho$

### 14.1. Support viability

The support state is viable only if

$$
-L_{\mathrm{heel}} \le \sigma \le L_{\mathrm{toe}}
$$

and the corresponding contact path and support tangent remain well defined.

### 14.2. Bilateral-support viability

The bilateral-support state is viable only if

$$
0 \le \beta \le 1
$$

and the interpolated support geometry remains well defined.

### 14.3. Reactive admissibility

The reactive regime remains admissible only if the deformations generated by $\rho$ do not violate hard geometric constraints.

For example,

$$
\|P-A\| \le 2L
$$

remains non-negotiable even when $\rho$ is large.

## 15. State status of the three variables

### 15.1. $\sigma$

The variable $\sigma$ should be treated as an explicit propagated support-state variable.

### 15.2. $\beta$

The variable $\beta$ should be treated as an explicit propagated bilateral-support variable whenever both feet are active.

### 15.3. $\rho$

The variable $\rho$ may be treated either as:

- a propagated filtered state variable,
- or a filtered derived variable explicitly recomputed from normalized deficits.

What is not allowed is to let $\rho$ appear in equations without fixing one of these two statuses.

## 16. Reset behavior across hybrid events

### 16.1. Support acquisition

When a new foot becomes active support, the corresponding $\sigma$ is initialized by a support-entry reset and the bilateral variable $\beta$ either becomes inactive or is reinterpreted according to the new bilateral state.

### 16.2. Bilateral-support entry

When a bilateral phase begins, both feet become simultaneously active and $\beta$ becomes active with a protocol-dependent initialization.

### 16.3. Bilateral-support exit

When support dominance becomes unilateral again, $\beta$ becomes inactive and the surviving active foot keeps its own support-state evolution.

### 16.4. Takeoff and flight

When support disappears, the corresponding support-based $\sigma$ becomes inactive. The variable $\beta$ is inactive outside bilateral phases. The variable $\rho$ remains part of the predictive and reactive state.

## 17. Main conclusions of this document

1. $\sigma$ is the canonical support coordinate along the actual foot-terrain contact path.

2. The canonical support interval is

$$
[-L_{\mathrm{heel}}, L_{\mathrm{toe}}].
$$

3. $\beta$ is the canonical bilateral support-allocation variable across standing, gait initiation, and load acceptance.

4. $\rho$ is the canonical regime-deformation variable that continuously deforms nominal locomotion into reactive locomotion.

5. None of these variables may remain implicit if they appear in support geometry, planning, prediction, or hybrid transitions.

6. $\sigma$ and $\beta$ must remain bounded inside their natural intervals.

7. $\rho$ may deform nominal laws, but it may never deactivate hard geometry.

8. These variables are part of the state architecture and must therefore be handled explicitly by the prediction layer.