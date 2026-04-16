# 07 — Hybrid prediction layer

## 0. Goal of this document

The goal of this document is to define the mathematical prediction layer used by planning, swing realization, touchdown validation, load-acceptance reasoning, and gait transitions.

This document fixes:

- the formal meaning of the prediction operator,
- the relation between continuous dynamics and hybrid events,
- the mode-dependent predictors for standing, gait initiation, walking, running stance, and flight,
- the distinction between unconditional prediction and candidate-conditioned prediction,
- the prediction outputs required by support-acquisition planning,
- the consistency conditions linking prediction to support geometry, swing realization, and post-touchdown viability,
- viability failure as a first-class prediction output.

This document does not yet define the final numerical integrator or the final implementation-level optimization scheme. It defines the canonical mathematical object that later implementations must approximate.

## 1. Why a prediction layer is necessary

The architecture already contains:

- an authoritative center of mass $C$,
- support-centered geometry,
- standing and walking hybrid modes,
- a foothold planner understood as a support-acquisition planner,
- a swing generator,
- exact reconstruction constraints.

However, these modules cannot be fully closed unless one can predict, from a current state, what the locomotion state will be at a future time and at future hybrid events.

Without such a layer, expressions such as

$$
C_{\mathrm{td}}, \qquad \dot C_{\mathrm{td}}, \qquad \theta_{\mathrm{td}}, \qquad P_{\mathrm{td}}
$$

remain only symbolic.

Therefore the architecture needs an explicit prediction layer.

## 2. Prediction as a hybrid mode-dependent flow

### 2.1. Locomotion state

Let the full locomotion state be denoted by

$$
X.
$$

This state contains at least the variables needed by the active protocol, including:

- the authoritative CM state,
- the torso state,
- the active ankles,
- the active support variables when support exists,
- the swing state when swing exists,
- the reactive intensity $\rho$,
- the current hybrid mode $\mathcal{M}$.

### 2.2. Prediction operator

The unconditional prediction operator is denoted by

$$
\Pi(t; X_0).
$$

Its meaning is:

$$
\Pi(t; X_0) = X(t),
$$

where $X(t)$ is the state reached after hybrid propagation of the locomotion model starting from initial state $X_0$.

Thus $\Pi$ is not a purely continuous flow map. It is a hybrid prediction map obtained by alternating continuous propagation and event resets.

## 3. Continuous dynamics inside one mode

Inside a mode where no event occurs, the state evolves according to a mode-dependent differential law

$$
\dot X = F_{\mathcal{M}}(X).
$$

The CM part always satisfies

$$
\ddot C = g_W + u_{\mathcal{M}}(X).
$$

The exact expression of $u_{\mathcal{M}}$ depends on the current protocol.

### 3.1. Bilateral standing

If

$$
\mathcal{M} = STAND_{LR},
$$

then the predictor uses the bilateral standing law

$$
\dot X = F_{STAND}(X).
$$

This includes at least:

- the CM dynamics,
- the torso evolution,
- the bilateral support-allocation variable $\beta$,
- the foot-specific support coordinates $\sigma_L$ and $\sigma_R$ when both feet are active.

### 3.2. Gait initiation

If

$$
\mathcal{M} \in \{GI_{L\to R}, GI_{R\to L}\},
$$

then the predictor uses the gait-initiation law

$$
\dot X = F_{GI}(X).
$$

This includes at least:

- the CM dynamics,
- the torso evolution,
- bilateral support evolution,
- the state-based or mixed law for $\beta$,
- the support-state evolution that prepares unilateral-support readiness.

### 3.3. Walking single support

If

$$
\mathcal{M} \in \{SS_L, SS_R\},
$$

then the predictor uses the walking single-support law

$$
\dot X = F_{SS}(X).
$$

This includes at least:

- the CM dynamics,
- the torso evolution,
- the active support-coordinate law $\sigma$,
- the active swing evolution when a foot is in swing.

### 3.4. Walking load acceptance

If

$$
\mathcal{M} \in \{LA_{L\to R}, LA_{R\to L}\},
$$

then the predictor uses the load-acceptance law

$$
\dot X = F_{LA}(X).
$$

This includes at least:

- the CM dynamics,
- the torso evolution,
- the bilateral support-allocation law for $\beta$,
- both active support paths,
- the transfer from old support dominance toward new support dominance.

### 3.5. Running stance

If

$$
\mathcal{M} \in \{RS_L, RS_R\},
$$

then the predictor uses the running-stance law

$$
\dot X = F_{RS}(X).
$$

This includes at least:

- the CM dynamics,
- the torso evolution,
- the running support progression law for $\sigma$,
- the active swing evolution when applicable.

### 3.6. Flight

If

$$
\mathcal{M} = FLIGHT,
$$

then the simplest canonical predictor uses ballistic CM motion:

$$
\ddot C = g_W.
$$

More explicitly,

$$
C(t) = C_0 + t \dot C_0 + \frac{1}{2} t^2 g_W,
$$

$$
\dot C(t) = \dot C_0 + t g_W.
$$

The remaining variables are propagated by their own flight-compatible laws.

## 4. Event functions and hybrid times

A hybrid predictor must not only propagate continuous dynamics. It must also detect events.

Define event functions

$$
g_e(X)
$$

for each relevant event label $e$.

Typical events now include:

- standing-to-gait-initiation entry,
- gait-initiation swing release,
- single-support touchdown,
- load-acceptance completion,
- running takeoff,
- flight touchdown,
- early landing,
- delayed-landing entry,
- viability failure.

For each event label $e$, define the first event time by

$$
t_e(X_0) = \inf\left\{ t > 0 : g_e\bigl(\Pi^{-}(t;X_0)\bigr) = 0 \text{ and all event-side conditions hold} \right\}.
$$

Here $\Pi^{-}$ denotes the prediction just before the event reset.

## 5. Reset maps at events

When an event occurs, the hybrid state generally changes discontinuously.

Define a reset map for event $e$ by

$$
R_e.
$$

If event $e$ occurs at time $t_e$, then the post-event state is

$$
X^{+} = R_e(X^{-}).
$$

Examples include:

- entering gait initiation from standing,
- releasing the swing foot after unilateral-support readiness is reached,
- activating a new foot contact at touchdown,
- initializing load acceptance after touchdown,
- making a new stance foot active after flight touchdown,
- switching from running stance to flight,
- returning a structured failure state.

Therefore the hybrid prediction operator is obtained by concatenating mode-wise flows and reset maps.

## 6. Piecewise definition of the hybrid predictor

Let the initial state be

$$
X_0.
$$

Suppose the first event encountered after $t=0$ occurs at time $t_1$ with label $e_1$.

Then for

$$
0 \le t < t_1,
$$

we have

$$
\Pi(t;X_0) = \Phi_{\mathcal{M}_0}(t;X_0),
$$

where $\Phi_{\mathcal{M}_0}$ is the continuous flow inside the initial mode.

At the event time,

$$
X_1^{+} = R_{e_1}(X_1^{-}).
$$

The same construction is then repeated from $X_1^{+}$.

Thus the canonical predictor is a concatenation of mode-wise continuous flows and event resets.

## 7. Candidate-conditioned prediction

The unconditional prediction operator

$$
\Pi(t;X_0)
$$

is not always sufficient for planning, because some predictions depend on the candidate next support acquisition.

Therefore planning requires a second object:

$$
\Pi(t; X_0, A_{\mathrm{next}}).
$$

This candidate-conditioned predictor is the state obtained when:

- the current state is $X_0$,
- the future foothold is provisionally fixed to $A_{\mathrm{next}}$,
- the swing generator and support-transition logic are propagated consistently with that candidate.

The unconditional predictor describes how the system evolves without committing to a specific future foothold.

The candidate-conditioned predictor describes how the system evolves under the hypothesis that the future foothold is $A_{\mathrm{next}}$.

The support-acquisition planner needs the second one.

## 8. Predicted touchdown and post-touchdown state

For a candidate foothold $A_{\mathrm{next}}$, define the candidate touchdown time by

$$
t_{\mathrm{td}}(A_{\mathrm{next}}).
$$

Then define the predicted touchdown state by

$$
X_{\mathrm{td}}(A_{\mathrm{next}}) = \Pi\bigl(t_{\mathrm{td}}(A_{\mathrm{next}}); X_0, A_{\mathrm{next}}\bigr).
$$

The minimum required outputs include:

$$
C_{\mathrm{td}}, \qquad \dot C_{\mathrm{td}}, \qquad \theta_{\mathrm{td}}, \qquad P_{\mathrm{td}}.
$$

If the touchdown state is support-resolved, one also requires:

$$
Q_{\mathrm{next}}, \qquad t_{\mathrm{sup,next}}, \qquad n_{\mathrm{sup,next}}, \qquad \sigma_{\mathrm{next}}.
$$

In walking-like cases, touchdown is not the end of the relevant prediction chain. One must also be able to predict a post-touchdown support-transfer phase.

Thus a candidate-conditioned prediction may need to propagate not only to touchdown, but through a finite load-acceptance interval.

## 9. Prediction outputs required by support-acquisition planning

For each candidate foothold, the planner requires prediction outputs sufficient to evaluate:

- rigid reachability,
- local support coordinates,
- capture-related quantities,
- touchdown timing,
- touchdown viability,
- post-touchdown load-acceptance viability.

Therefore the prediction layer must provide at least:

$$
P_{\mathrm{td}} = C_{\mathrm{td}} - 0.75L e_{\theta_{\mathrm{td}}},
$$

$$
s_{A,\mathrm{next}} = (C_{\mathrm{td}} - A_{\mathrm{next}}) \cdot t_{\mathrm{sup,next}},
$$

$$
s_{Q,\mathrm{next}} = (C_{\mathrm{td}} - Q_{\mathrm{next}}) \cdot t_{\mathrm{sup,next}},
$$

$$
z_{Q,\mathrm{next}} = (C_{\mathrm{td}} - Q_{\mathrm{next}}) \cdot n_{\mathrm{sup,next}},
$$

$$
\dot s_{A,\mathrm{next}} = \dot C_{\mathrm{td}} \cdot t_{\mathrm{sup,next}}.
$$

These are outputs of the prediction layer composed with future support-geometry construction.

## 10. Prediction consistency with support geometry

The prediction layer must never predict only free CM variables while ignoring the support objects required later by planning and reconstruction.

Therefore every candidate-conditioned prediction must remain consistent with future support construction.

This means that when a candidate foothold is given, the predictor must also determine:

- whether the future foot geometry is admissible,
- whether the future contact path exists,
- whether the future support point exists,
- whether the future support tangent and normal exist,
- whether the touchdown frame is geometrically well defined.

If one of these objects fails to exist, then the candidate-conditioned prediction is incomplete and the associated foothold must be rejected as non-viable.

## 11. Prediction consistency with swing realization

The candidate-conditioned predictor must also remain consistent with the swing generator.

Therefore the following chain must be well defined:

$$
A_{\mathrm{next}} \longrightarrow A_{\mathrm{sw}}(t) \longrightarrow t_{\mathrm{td}}(A_{\mathrm{next}}) \longrightarrow X_{\mathrm{td}}(A_{\mathrm{next}}).
$$

In particular:

1. the swing law must be able to target $A_{\mathrm{next}}$,
2. the swing foot must remain admissible until touchdown or valid early landing,
3. touchdown time must be determined by the swing-support interaction, not by an unrelated clock,
4. the final touchdown state must be compatible with future support geometry.

Thus prediction is not only about the CM. It is about the coupled future of support, swing, and state.

## 12. Prediction consistency with load acceptance

In the current walking architecture, touchdown does not immediately imply new support dominance.

Therefore candidate-conditioned prediction must remain compatible with the chain

$$
\text{touchdown} \to \text{load acceptance} \to \text{new support dominance}.
$$

A candidate-conditioned prediction that reaches touchdown but cannot propagate through a viable finite support-transfer phase is incomplete for walking support-acquisition planning.

Thus prediction must be able to certify not only touchdown viability, but also load-acceptance viability when the active protocol requires it.

## 13. Nominal and reactive prediction

The prediction layer must depend continuously on the reactive intensity

$$
\rho \in [0,1].
$$

This dependence may enter through:

- support-law parameters,
- stance-length targets,
- bilateral-support timing,
- swing duration,
- touchdown tolerances,
- viability margins.

Therefore the canonical predictor may be written more explicitly as

$$
\Pi(t; X_0, A_{\mathrm{next}}, \rho).
$$

In nominal motion, $\rho$ is small and prediction remains close to the nominal gait laws.

Under stronger perturbation, the same prediction layer remains valid but propagates a deformed dynamics.

This is critical because recovery must emerge from the same architecture rather than from a disconnected fallback predictor.

## 14. Multi-stage prediction hierarchy

The final implementation may use several prediction fidelities, but they must all approximate the same canonical object.

A mathematically clean hierarchy is:

### Level 1

A mode-local predictor with frozen or slowly varying support quantities.

### Level 2

A hybrid predictor with explicit event detection and reset maps.

### Level 3

A candidate-conditioned hybrid predictor coupled to swing realization, touchdown viability, and post-touchdown support viability.

The clean documentation must define Level 3 as the canonical object, even if the first implementation temporarily uses a lower-order approximation.

## 15. Admissible approximations

Approximation is acceptable only if the approximated predictor still preserves the logical dependencies of the exact architecture.

A reduced predictor is admissible only if:

1. it predicts the same state variables or an explicit subset of them,
2. it preserves mode dependence,
3. it preserves event dependence,
4. it preserves candidate dependence when planning is involved,
5. it does not silently discard a geometric or support-transition constraint later assumed by planning or reconstruction.

A cheap predictor is allowed.

A logically inconsistent predictor is not.

## 16. Viability failure as a prediction outcome

Prediction must not assume success by default.

For some initial states or some candidate footholds, the predicted evolution may fail to remain viable.

Typical failure cases are:

- no admissible future foot geometry,
- impossible touchdown IK,
- unreachable swing timing,
- impossible support transition,
- violation of rigid morphology.

Therefore the output of the prediction layer is not only a future state. It may also be a structured viability failure.

Formally, the predictor should be understood as returning either

$$
X_{\mathrm{pred}}
$$

or

$$
\text{FAIL}(\text{reason}).
$$

This prevents later modules from treating an undefined prediction as a valid state.

## 17. Minimal event set for the current architecture

At the present stage of the project, the minimal event set that the canonical predictor should support is:

- $STAND_{LR} \to GI$ entry,
- $GI \to SS$ swing-release event,
- $SS \to LA$ touchdown event,
- $LA \to SS$ support-dominance completion,
- $RS \to FLIGHT$ takeoff,
- $FLIGHT \to RS$ touchdown,
- nominal swing touchdown,
- early landing,
- delayed-landing entry,
- prediction failure.

A later document may enlarge this set for jump and landing.

## 18. Main conclusions of this document

1. The prediction operator must be a hybrid map, not only a continuous flow.

2. Planning requires a candidate-conditioned prediction operator, not only an unconditional future-state map.

3. The predictor must propagate continuous dynamics, detect events, and apply reset maps.

4. The touchdown state is an output of the predictor, not an external guess.

5. Prediction must remain consistent with support geometry, swing realization, and post-touchdown support logic.

6. Prediction may return a structured failure, not only a state.

7. Approximate predictors are acceptable only if they preserve the logical structure of the canonical hybrid predictor.

8. This prediction layer is the mathematical bridge linking locomotion dynamics, support-acquisition planning, swing realization, and hybrid mode transitions.