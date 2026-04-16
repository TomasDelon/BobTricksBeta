# 03 — Formal foothold planner

## 0. Goal of this document

The goal of this document is to define the mathematical planner that selects the next support acquisition on a segmented terrain in a way consistent with the support-centered `CM-first` architecture.

This document fixes:

- the interpretation of the foothold planner as a support-acquisition planner,
- the exact planner output,
- the terrain-following candidate domain for the future ankle,
- the candidate-induced future foot and support geometry,
- the role of candidate-conditioned prediction,
- the exact admissibility conditions of a candidate,
- the distinction between touchdown viability and load-acceptance viability,
- the support-centered quantities used for candidate scoring,
- the cost structure used to compare admissible candidates,
- the reactive deformation of the same planner through $\rho$.

This document remains a formal mathematical specification. It does not impose a particular discrete search algorithm or a particular numerical optimizer.

## 1. Planner interpretation

The foothold planner must no longer be understood as an isolated module that chooses a single ankle point in front of the body.

Under the present architecture, standing, gait initiation, support progression, and load acceptance are explicit parts of the locomotion state. Therefore the planner must be interpreted as a support-acquisition planner.

Its role is not only to choose where the swing ankle should land. Its role is to propose a future contact that is simultaneously:

- geometrically admissible under the future foot,
- reachable by the swing leg at touchdown,
- compatible with the current support regime,
- acceptable through a finite load-transfer phase,
- coherent with the future support evolution of the same `CM-first` state.

Thus the planned object is a future support acquisition, not merely a target point in world space.

## 2. Planner output

### 2.1. The future ankle remains the geometric anchor

The future foothold remains anchored by a future ankle point

$$
A_{\mathrm{next}} \in \Gamma.
$$

The ankle remains the authoritative geometric target for the swing foot.

### 2.2. The planner output is richer than the ankle alone

The minimum planner output is the tuple

$$
\mathcal{O}_{\mathrm{foot}} = \left(A_{\mathrm{next}}, \mathcal{W}_{\mathrm{td}}, \mathcal{V}_{\mathrm{td}}\right),
$$

where $\mathcal{W}_{\mathrm{td}}$ is a candidate touchdown window and $\mathcal{V}_{\mathrm{td}}$ is a viability certificate associated with that candidate.

The meaning of these objects is the following.

- $A_{\mathrm{next}}$ is the future ankle anchor.
- $\mathcal{W}_{\mathrm{td}}$ encodes when touchdown is expected to occur.
- $\mathcal{V}_{\mathrm{td}}$ certifies that touchdown and the subsequent support transfer are both viable.

### 2.3. The planner does not choose the future support point directly

The planner does not directly choose the future effective support point $Q_{\mathrm{next}}$.

Instead, once $A_{\mathrm{next}}$ is proposed, one later constructs from it:

- the future contact path $\chi_{\mathrm{next}}(\sigma)$,
- the future visible foot direction $f_{\mathrm{foot,next}}$,
- the future support point $Q_{\mathrm{next}}$,
- the future support tangent $t_{\mathrm{sup,next}}$,
- the future support normal $n_{\mathrm{sup,next}}$.

This separation is essential. The leg anchors geometrically at the ankle, whereas balance, capture, and support transfer are better expressed relative to the effective support geometry.

## 3. Terrain-following candidate domain

### 3.1. Forward path from the current dominant support ankle

Let the current dominant support ankle be $A_{\mathrm{st}}$.

Let the terrain forward path in the current facing direction be parameterized by terrain arc-length:

$$
\gamma_{A_{\mathrm{st}}}(\ell), \qquad \ell \ge 0,
$$

with

$$
\gamma_{A_{\mathrm{st}}}(0) = A_{\mathrm{st}}.
$$

Every future ankle candidate is of the form

$$
A(\ell) = \gamma_{A_{\mathrm{st}}}(\ell).
$$

The variable $\ell$ is the terrain arc-length from the current dominant support ankle to the candidate ankle. On segmented terrain, this is the correct step-length variable.

### 3.2. Finite state-dependent search horizon

The planner must search only over a finite terrain-following interval. A purely fixed planning horizon is not acceptable as the canonical law because terrain compensation and reactive stepping require the horizon to deform with the locomotion state.

Define the candidate set of terrain abscissas by

$$
\Sigma_{\mathrm{plan}}(X, \rho, \mathcal{M}) = [\ell_{\min}^{\mathrm{plan}}, \ell_{\max}^{\mathrm{plan}}(X, \rho, \mathcal{M})].
$$

The lower bound excludes degenerate tiny steps. The upper bound is allowed to depend on:

- the current support regime,
- the current support progression state,
- the reactive intensity $\rho$,
- protocol-specific anticipation.

Thus the search domain is

$$
A(\ell), \qquad \ell \in \Sigma_{\mathrm{plan}}(X, \rho, \mathcal{M}).
$$

## 4. Candidate-induced future foot geometry

### 4.1. Future contact path under the foot

For each candidate ankle $A_{\mathrm{next}}$, define the future local contact path under the foot by

$$
\chi_{\mathrm{next}}(\sigma), \qquad \sigma \in [-L_{\mathrm{heel}}, L_{\mathrm{toe}}],
$$

with

$$
\chi_{\mathrm{next}}(0) = A_{\mathrm{next}}.
$$

This object is the local support path induced by the terrain under the future foot.

### 4.2. Geometric admissibility of the future foot

A future ankle candidate is not admissible merely because it lies on the terrain. It must induce an admissible future foot geometry.

A candidate foot geometry is admissible only if all the following hold.

1. The full contact path exists over the active interval $[-L_{\mathrm{heel}}, L_{\mathrm{toe}}]$.

2. The path spans at most two admissible consecutive terrain segments.

3. If two terrain segments are covered, the angle variation along the active contact path satisfies

$$
\Delta \alpha_{\mathrm{foot}} \le \alpha_{\mathrm{foot}}^{\max}.
$$

4. The resulting visible foot direction is facing-compatible.

5. The future support frame can be constructed from the induced contact path.

### 4.3. Visible foot direction

When the induced contact path is admissible, the future visible foot direction is derived from the future foot geometry. It is not an independent decision variable.

The candidate remains admissible only if

$$
f\,(f_{\mathrm{foot,next}} \cdot e_X) > 0,
$$

where $f \in \{-1,+1\}$ is the facing sign.

## 5. Candidate-conditioned prediction

### 5.1. Prediction must be conditioned by the candidate

The planner must evaluate a candidate foothold through candidate-conditioned prediction, not through a prediction independent of the candidate.

Let the current locomotion state be $X_0$.

For each candidate $A_{\mathrm{next}}$, define the candidate-conditioned hybrid prediction by

$$
X_{\mathrm{td}}(A_{\mathrm{next}}) = \Pi\bigl(t_{\mathrm{td}}(A_{\mathrm{next}}); X_0, A_{\mathrm{next}}\bigr),
$$

where the touchdown time is itself generated by the candidate-conditioned swing and support evolution.

This prediction must propagate at least:

- the active support state,
- the current hybrid mode,
- the swing geometry,
- the future touchdown event,
- the upcoming load-acceptance phase.

### 5.2. Prediction failure rejects the candidate

The predictor is allowed to return failure.

If the candidate-conditioned predictor cannot produce a geometrically coherent touchdown state and a coherent post-touchdown support evolution, then the candidate must be rejected as non-viable.

Formally, a failure of the form

$$
\Pi\bigl(t_{\mathrm{td}}(A_{\mathrm{next}}); X_0, A_{\mathrm{next}}\bigr) = FAIL(\mathrm{reason})
$$

removes the candidate from the admissible set.

## 6. Exact touchdown viability

### 6.1. Predicted pelvis state

Let the candidate-conditioned predictor return a touchdown state containing in particular

$$
C_{\mathrm{td}}, \qquad \dot C_{\mathrm{td}}, \qquad \theta_{\mathrm{td}}.
$$

The predicted pelvis is then

$$
P_{\mathrm{td}} = C_{\mathrm{td}} - 0.75L e_{\theta_{\mathrm{td}}}.
$$

### 6.2. Exact rigid-leg reachability

A future candidate is exactly reachable at touchdown if and only if

$$
\|P_{\mathrm{td}} - A_{\mathrm{next}}\| \le 2L.
$$

This is a hard constraint. It is never softened by preference, by reactivity, or by the optimizer.

A candidate violating this bound must be rejected before scoring.

### 6.3. Soft reach penalty remains separate from exact reach

The planner may discourage candidates too close to the rigid limit, but that preference must remain distinct from exact reachability.

Define a preferred touchdown reach

$$
\ell_{\mathrm{pref}}(X, \rho, \mathcal{M}) < 2L.
$$

A soft reach penalty may then be built from the excess beyond $\ell_{\mathrm{pref}}$, while the exact bound $2L$ remains non-negotiable.

## 7. Touchdown viability is not enough

A candidate that is reachable at the touchdown instant is not automatically acceptable.

The planner must also certify that the touchdown can be followed by a valid finite load-acceptance phase and by the transfer of support dominance toward the new foot.

Thus the correct support sequence is

$$
\text{swing realization} \to \text{touchdown} \to \text{load acceptance} \to \text{new support dominance}.
$$

A candidate that satisfies immediate touchdown reachability but cannot pass through a valid bilateral acceptance phase must still be rejected or heavily penalized.

## 8. Load-acceptance viability

### 8.1. Support-transfer requirement

Let the current dominant support foot be indexed by $b$ and the candidate future support foot by $f$.

After touchdown, the system must admit a finite bilateral phase over an interval

$$
[t_{\mathrm{td}}, t_{\mathrm{dom}}], \qquad t_{\mathrm{dom}} > t_{\mathrm{td}},
$$

in which support dominance can move continuously from the old support toward the new one.

### 8.2. Bilateral support frame during acceptance

During this predicted bilateral phase, let

$$
Q_b(t), \qquad t_b(t)
$$

be the old-foot support point and tangent, and let

$$
Q_f(t), \qquad t_f(t)
$$

be the new-foot support point and tangent.

The support-allocation law is represented by a variable

$$
\beta(t) \in [0,1].
$$

The effective bilateral support frame is then

$$
Q_{\beta}(t) = (1-\beta(t)) Q_b(t) + \beta(t) Q_f(t),
$$

$$
t_{\beta}(t) = \frac{(1-\beta(t)) t_b(t) + \beta(t) t_f(t)}{\|(1-\beta(t)) t_b(t) + \beta(t) t_f(t)\|},
$$

provided the denominator is nonzero.

A valid load-acceptance phase requires the existence of a continuous law $\beta(t)$ such that:

1. $\beta(t_{\mathrm{td}})$ remains near the old-support side,
2. $\beta(t_{\mathrm{dom}})$ reaches the new-support side,
3. the effective bilateral support frame remains well defined,
4. the CM/support evolution remains support-admissible throughout the interval.

### 8.3. Definition of load-acceptance viability

A candidate is load-acceptance viable if the predictor certifies the existence of such a finite bilateral transfer.

If no valid support-transfer law exists after touchdown, then the candidate is not fully viable, even if the touchdown itself is reachable.

## 9. Future support-centered quantities

### 9.1. Future support point and support frame

Suppose the candidate-conditioned prediction returns a valid future support coordinate initialization $\sigma_{\mathrm{next}}$ on the future contact path.

Then the future support point is

$$
Q_{\mathrm{next}} = \chi_{\mathrm{next}}(\sigma_{\mathrm{next}}).
$$

The future support tangent is

$$
t_{\mathrm{sup,next}} = \frac{\partial_{\sigma} \chi_{\mathrm{next}}(\sigma_{\mathrm{next}})}{\|\partial_{\sigma} \chi_{\mathrm{next}}(\sigma_{\mathrm{next}})\|}.
$$

The future support normal is

$$
n_{\mathrm{sup,next}} = (-t_{{\mathrm{sup,next}},y}, t_{{\mathrm{sup,next}},x}).
$$

### 9.2. Support-centered and ankle-centered coordinates at touchdown

The predicted CM decomposes in the future support frame as

$$
C_{\mathrm{td}} = Q_{\mathrm{next}} + s_{Q,\mathrm{next}} t_{\mathrm{sup,next}} + z_{Q,\mathrm{next}} n_{\mathrm{sup,next}}.
$$

It also decomposes relative to the future ankle as

$$
C_{\mathrm{td}} = A_{\mathrm{next}} + s_{A,\mathrm{next}} t_{\mathrm{sup,next}} + z_{A,\mathrm{next}} n_{\mathrm{sup,next}}.
$$

Therefore

$$
s_{A,\mathrm{next}} = (C_{\mathrm{td}} - A_{\mathrm{next}}) \cdot t_{\mathrm{sup,next}},
$$

$$
z_{A,\mathrm{next}} = (C_{\mathrm{td}} - A_{\mathrm{next}}) \cdot n_{\mathrm{sup,next}}.
$$

Since the ankle corresponds to $\sigma = 0$, one has

$$
s_{Q,\mathrm{next}} = s_{A,\mathrm{next}} - \sigma_{\mathrm{next}},
$$

$$
z_{Q,\mathrm{next}} = z_{A,\mathrm{next}}.
$$

This keeps explicit the same ankle-versus-support distinction used in the walking protocol.

### 9.3. Candidate local capture quantity

Define the tangential touchdown velocity by

$$
\dot s_{A,\mathrm{next}} = \dot C_{\mathrm{td}} \cdot t_{\mathrm{sup,next}}.
$$

Define the local support angle $\alpha_{\mathrm{sup,next}}$ from the future support tangent.

Let $\bar z_{\mathrm{next}} > 0$ be a positive reference support-normal height.

Then define

$$
\omega_{0,\mathrm{next}} = \sqrt{\frac{g \cos \alpha_{\mathrm{sup,next}}}{\bar z_{\mathrm{next}}}}.
$$

A support-centered XCoM-like quantity is then

$$
\xi_{\mathrm{next}} = s_{Q,\mathrm{next}} + \lambda \frac{\dot s_{A,\mathrm{next}}}{\omega_{0,\mathrm{next}}},
$$

with

$$
0 < \lambda \le 1.
$$

This quantity is useful, but it is not the whole planner.

### 9.4. Future support interval and capture defect

Relative to the future ankle, the active foot interval is

$$
[-L_{\mathrm{heel}}, L_{\mathrm{toe}}].
$$

Relative to the future support point $Q_{\mathrm{next}} = \chi_{\mathrm{next}}(\sigma_{\mathrm{next}})$, the same interval becomes

$$
\mathcal{I}_{Q,\mathrm{next}} = [-L_{\mathrm{heel}} - \sigma_{\mathrm{next}}, L_{\mathrm{toe}} - \sigma_{\mathrm{next}}].
$$

Let $d_{\mathcal{I}}(x, I)$ denote the distance from a scalar $x$ to an interval $I$.

The candidate capture defect is then

$$
D_{\mathrm{cap}}(A_{\mathrm{next}}) = d_{\mathcal{I}}\bigl(\xi_{\mathrm{next}}, \mathcal{I}_{Q,\mathrm{next}}\bigr).
$$

## 10. Cost terms

### 10.1. Step-length objective

The natural terrain-based step length is

$$
\ell(A_{\mathrm{next}}).
$$

The planner should prefer a protocol-dependent target step length

$$
\ell_{\mathrm{step}}^{\star}(X, \rho, \mathcal{M}).
$$

Define the step-length deviation cost by

$$
D_{\ell}(A_{\mathrm{next}}; X, \rho, \mathcal{M}) = \bigl(\ell(A_{\mathrm{next}}) - \ell_{\mathrm{step}}^{\star}(X, \rho, \mathcal{M})\bigr)^2.
$$

### 10.2. Terrain height-change cost

Define the candidate height change by

$$
\Delta h(A_{\mathrm{next}}) = (A_{\mathrm{next}} - A_{\mathrm{st}}) \cdot e_Y.
$$

Then define

$$
D_h(A_{\mathrm{next}}) = \Delta h(A_{\mathrm{next}})^2.
$$

### 10.3. Support-angle transition cost

Let $\alpha_{\mathrm{sup,current}}$ be the current dominant-support angle and let $\alpha_{\mathrm{sup,next}}$ be the future support angle induced by the candidate.

Define

$$
D_{\alpha}(A_{\mathrm{next}}) = \bigl(\alpha_{\mathrm{sup,next}} - \alpha_{\mathrm{sup,current}}\bigr)^2.
$$

### 10.4. Timing mismatch cost

Let the candidate-conditioned predictor return a touchdown time $t_{\mathrm{td}}(A_{\mathrm{next}})$.

Let the protocol-dependent target touchdown time be

$$
t_{\mathrm{td}}^{\star}(X, \rho, \mathcal{M}).
$$

Define

$$
D_t(A_{\mathrm{next}}; X, \rho, \mathcal{M}) = \bigl(t_{\mathrm{td}}(A_{\mathrm{next}}) - t_{\mathrm{td}}^{\star}(X, \rho, \mathcal{M})\bigr)^2.
$$

### 10.5. Soft reach cost

Define the soft reach penalty by

$$
D_{\mathrm{IK}}(A_{\mathrm{next}}; X, \rho, \mathcal{M}) = \Bigl(\max\bigl\{0, \|P_{\mathrm{td}} - A_{\mathrm{next}}\| - \ell_{\mathrm{pref}}(X, \rho, \mathcal{M})\bigr\}\Bigr)^2.
$$

### 10.6. Load-acceptance difficulty cost

The support-centered framework requires a specific cost term associated with post-touchdown support transfer.

Define

$$
D_{\mathrm{acc}}(A_{\mathrm{next}})
$$

as the predicted difficulty of passing from old support dominance to new support dominance after touchdown.

This term must depend on the predicted bilateral support geometry and may increase, for example, when:

- the old and new support points are badly arranged,
- the support-angle discontinuity is too large,
- the required progression of $\beta$ is too abrupt,
- the candidate produces an excessive support-reorganization burden.

If the predicted load-acceptance phase is impossible, then the candidate is not admissible. If it is possible but poor, then $D_{\mathrm{acc}}$ must be large.

## 11. Admissible candidate set

A candidate ankle $A_{\mathrm{next}}$ is admissible only if all the following hold.

1. It lies in the terrain-forward search set.

2. It induces an admissible future contact path over the full heel-to-toe interval.

3. The candidate-conditioned predictor returns a coherent touchdown state.

4. Exact rigid-leg reachability holds:

$$
\|P_{\mathrm{td}} - A_{\mathrm{next}}\| \le 2L.
$$

5. A valid finite load-acceptance phase exists after touchdown.

6. The candidate is compatible with the current support regime and side choice.

Define the admissible set by

$$
\mathcal{A}(X, \rho, \mathcal{M}) = \left\{A(\ell) : \ell \in \Sigma_{\mathrm{plan}}(X, \rho, \mathcal{M}), \ A(\ell) \text{ satisfies all admissibility conditions above} \right\}.
$$

## 12. Full planner cost

Among admissible candidates, define the support-centered planner cost by

$$
J(A_{\mathrm{next}}) = w_{\mathrm{cap}} D_{\mathrm{cap}} + w_{\ell} D_{\ell} + w_h D_h + w_{\alpha} D_{\alpha} + w_{\mathrm{acc}} D_{\mathrm{acc}} + w_t D_t + w_{\mathrm{IK}} D_{\mathrm{IK}}.
$$

The selected future support acquisition is then

$$
A_{\mathrm{next}}^{\star} = \arg\min_{A \in \mathcal{A}(X, \rho, \mathcal{M})} J(A).
$$

The weights may depend on $X$, $\rho$, and $\mathcal{M}$, but the hard admissibility constraints do not.

## 13. Coupling with gait initiation and side choice

The planner must remain compatible with standing and gait initiation.

If the gait-initiation phase has already selected the future swing side, then the foothold search must be restricted to that side.

If the gait-initiation phase is not yet committed, then side choice and foothold choice must not be treated as unrelated decisions. They must be coupled either jointly or through a clear sequential logic.

Thus the planner cannot be interpreted as a side-blind ankle selector.

## 14. Protocol awareness and reactivity

The planner is one and the same across nominal and reactive conditions, but its horizon, targets, and weights may deform with the locomotion state.

### 14.1. Nominal regime

In nominal walking, the planner should favor:

- regular terrain-following step lengths,
- smooth support progression,
- economical terrain transitions,
- easy load acceptance.

### 14.2. Reactive regime

When $\rho$ increases, the planner may allow:

- a larger search horizon,
- larger target step lengths,
- different touchdown timing preferences,
- stronger emphasis on capture and support acquisition.

However, reactivity may never legalize a candidate that violates exact morphology or admissible foot-contact geometry.

## 15. Compatibility with the rest of the architecture

### 15.1. Compatibility with the walking protocol

The present planner is compatible with the walking protocol because it uses the same distinction between ankle and support variables, the same support-centered interpretation of touchdown, and the same explicit role of load acceptance.

### 15.2. Compatibility with the swing generator

The planner does not itself realize the motion. It proposes a future support acquisition. The swing generator must then realize a collision-free ankle transfer toward the planned candidate while preserving the viability assumptions used at planning time.

### 15.3. Compatibility with later protocols

The same planner architecture can later be reused in running, landing, crouching, or jump-related protocols. What changes is not the existence of the planner, but mainly:

- the contact protocol,
- the timing laws,
- the admissible horizon,
- the relative importance of the cost terms.

## 16. Main conclusions of this document

1. The foothold planner must be interpreted as a support-acquisition planner.

2. The future ankle remains the geometric anchor, but the planner output must be richer than the ankle alone.

3. A candidate must induce an admissible future contact path over the full heel-to-toe interval.

4. Candidate evaluation must be performed through candidate-conditioned prediction.

5. Exact rigid-leg reachability at touchdown remains a hard constraint.

6. Touchdown viability and load-acceptance viability are distinct notions.

7. The support-centered quantities used for scoring must be defined relative to the future support state, not only relative to the ankle.

8. The planner cost must include a specific load-acceptance difficulty term $D_{\mathrm{acc}}$.

9. The planner must remain compatible with standing, gait initiation, walking, and later protocol extensions.

10. Recovery stepping must emerge from continuous deformation of the same planner, not from a separate architecture.
