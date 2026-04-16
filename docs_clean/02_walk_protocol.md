# 02 — Formal walking protocol

## 0. Goal of this document

The goal of this document is to define the mathematical protocol of walking under the support-centered `CM-first` architecture.

This document fixes:

- the walk-related hybrid mode set,
- the walking state,
- the local support coordinates of the CM,
- the nominal single-support manifold,
- the role of the support coordinate $\sigma$ during stance,
- the role of the support-allocation variable $\beta$ in bilateral phases,
- gait initiation from standing,
- finite load acceptance after touchdown,
- the guard conditions for walk-related mode transitions,
- the reactive deformation of the same protocol through $\rho$.

This document does not redefine the full standing law, the full foothold planner, or the full swing generator. It states how walking uses those objects.

## 1. Walking as a support-centered hybrid protocol

Walking is not only an alternation of left and right single support.

Under the present architecture, a walking step must be understood through the following support logic:

1. a bilateral-support regime may exist before step release,
2. one foot becomes dominant support,
3. the other foot swings toward a future support acquisition,
4. touchdown creates contact but not yet full support dominance,
5. a finite load-acceptance phase transfers support dominance,
6. the next single-support phase begins.

Therefore the walk-related controller must be hybrid.

## 2. Walk-related mode set

The full walk-related support sequence must be compatible with standing and with finite load acceptance.

A minimal walk-related mode set is therefore

$$
\mathcal{M}_{\mathrm{walk}}
=
\{STAND_{LR},\; GI_{L\to R},\; SS_R,\; LA_{R\to L},\; SS_L,\; LA_{L\to R},\; GI_{R\to L}\}.
$$

The meanings are:

- $STAND_{LR}$: bilateral standing,
- $GI_{L\to R}$: gait initiation in which the left foot will swing first and the right foot becomes the first dominant stance foot,
- $SS_R$: right single support,
- $LA_{R\to L}$: left-foot touchdown followed by finite load acceptance from right support dominance toward left support dominance,
- $SS_L$: left single support,
- $LA_{L\to R}$: right-foot touchdown followed by finite load acceptance from left support dominance toward right support dominance,
- $GI_{R\to L}$: symmetric gait initiation in which the right foot will swing first.

This mode set makes explicit that bilateral phases are not all the same.

Standing, gait initiation, and load acceptance are distinct support regimes.

## 3. Walking state

The authoritative dynamic object remains the virtual center of mass

$$
C(t) \in \mathbb{R}^2.
$$

Its derivatives are

$$
V(t) = \dot C(t), \qquad A(t) = \ddot C(t).
$$

The walk-related state must also contain:

- the torso angle $\theta$,
- the left and right ankle positions $A_L$ and $A_R$,
- the foot-specific support coordinates $\sigma_L$ and $\sigma_R$ whenever the corresponding feet are active,
- the foot-specific support points $Q_L$ and $Q_R$ whenever they exist,
- the foot-specific support tangents $t_L$ and $t_R$ whenever they exist,
- the effective bilateral support-allocation variable $\beta$ whenever both feet are active,
- the current swing ankle trajectory when one foot is in swing,
- the next planned ankle target $A_{\mathrm{next}}$,
- the reactive intensity $\rho$,
- the local time in the current mode $\tau_{\mathrm{mode}}$,
- the current mode $\mathcal{M}$.

A minimal abstract state is therefore

$$
X_{\mathrm{walk}}
=
(C, V, \theta, A_L, A_R, \sigma_L, \sigma_R, Q_L, Q_R, t_L, t_R, \beta, A_{\mathrm{next}}, \rho, \tau_{\mathrm{mode}}, \mathcal{M}).
$$

The effective support frame used by the CM is derived from this state. It is not an independent object.

## 4. Effective support frame

### 4.1. Single support

During single support, one foot is dominant support.

Suppose the active stance foot is indexed by $i \in \{L,R\}$.

Then the effective support point is

$$
Q = Q_i,
$$

the effective support tangent is

$$
t_{\mathrm{sup}} = t_i,
$$

and the effective support normal is

$$
n_{\mathrm{sup}} = (-t_{i,y}, t_{i,x}).
$$

The support point $Q_i$ is not defined by the old rule

$$
A_i + \sigma_i t_i.
$$

It is defined from the support path of the active foot:

$$
Q_i = \chi_{A_i}(\sigma_i).
$$

### 4.2. Bilateral phases

During bilateral phases, both feet contribute to the effective support state.

Let the two active support points be $Q_L$ and $Q_R$, with tangents $t_L$ and $t_R$.

Then the effective support point is

$$
Q = (1-\beta)Q_L + \beta Q_R.
$$

The effective support tangent is

$$
t_{\mathrm{sup}} = \frac{(1-\beta)t_L + \beta t_R}{\|(1-\beta)t_L + \beta t_R\|},
$$

provided the denominator is nonzero.

The effective support normal is

$$
n_{\mathrm{sup}} = (-t_{{\mathrm{sup}},y}, t_{{\mathrm{sup}},x}).
$$

Thus a single support frame still exists for CM decomposition even when both feet are active.

## 5. Local support coordinates of the CM

Given the effective support frame,

$$
C = Q + s_Q t_{\mathrm{sup}} + z_Q n_{\mathrm{sup}}.
$$

Thus

$$
s_Q = (C-Q)\cdot t_{\mathrm{sup}},
$$

$$
z_Q = (C-Q)\cdot n_{\mathrm{sup}}.
$$

Relative to a particular ankle $A_i$,

$$
C = A_i + s_{A_i} t_{\mathrm{sup}} + z_{A_i} n_{\mathrm{sup}}.
$$

Thus

$$
s_{A_i} = (C-A_i)\cdot t_{\mathrm{sup}},
$$

$$
z_{A_i} = (C-A_i)\cdot n_{\mathrm{sup}}.
$$

This distinction remains essential:

- rigid-leg geometry is written relative to ankles,
- support logic is written relative to the effective support frame.

Because the support point is now defined through the support path

$$
Q_i = \chi_{A_i}(\sigma_i),
$$

there is no longer a globally valid linear relation of the old form

$$
s_Q = s_A - \sigma.
$$

on arbitrary two-segment support geometry.

The relation between ankle-local and support-local coordinates is mediated by the support path itself.

## 6. Local support slope angle

Let

$$
t_{\mathrm{sup}} = (t_x, t_y).
$$

Define the support slope angle by

$$
\alpha_{\mathrm{sup}} = \mathrm{atan2}(t_y, t_x).
$$

This is the slope angle used in support-local walking dynamics.

## 7. Nominal single-support manifold

### 7.1. Target effective leg length

In nominal walking, the stance leg should be nearly extended.

Define

$$
\ell_w^{\star} = 2L - \varepsilon_w L,
$$

with

$$
0 < \varepsilon_w \ll 1.
$$

### 7.2. Nominal stance manifold

Suppose the active stance ankle is $A_{\mathrm{st}}$.

The nominal single-support manifold is

$$
\Phi_w(C,\theta,A_{\mathrm{st}})
=
\|P-A_{\mathrm{st}}\|^2 - (\ell_w^{\star})^2.
$$

Nominal walking single support corresponds to

$$
\Phi_w(C,\theta,A_{\mathrm{st}}) = 0.
$$

Since

$$
P = C - 0.75L e_{\theta},
$$

this can also be written as

$$
\Phi_w(C,\theta,A_{\mathrm{st}})
=
\|C - 0.75L e_{\theta} - A_{\mathrm{st}}\|^2 - (\ell_w^{\star})^2.
$$

### 7.3. Explicit nominal single-support path relative to the stance ankle

Write

$$
d_t = 0.75L (e_{\theta}\cdot t_{\mathrm{sup}}),
$$

$$
d_n = 0.75L (e_{\theta}\cdot n_{\mathrm{sup}}).
$$

Using

$$
C-A_{\mathrm{st}} = s_{A_{\mathrm{st}}} t_{\mathrm{sup}} + z_{A_{\mathrm{st}}} n_{\mathrm{sup}},
$$

we get

$$
P-A_{\mathrm{st}} = (s_{A_{\mathrm{st}}}-d_t)t_{\mathrm{sup}} + (z_{A_{\mathrm{st}}}-d_n)n_{\mathrm{sup}}.
$$

Therefore

$$
\|P-A_{\mathrm{st}}\|^2 = (s_{A_{\mathrm{st}}}-d_t)^2 + (z_{A_{\mathrm{st}}}-d_n)^2.
$$

The nominal manifold becomes

$$
(s_{A_{\mathrm{st}}}-d_t)^2 + (z_{A_{\mathrm{st}}}-d_n)^2 = (\ell_w^{\star})^2.
$$

So the upper branch is

$$
z_{A_{\mathrm{st}}}^{\star}(s_{A_{\mathrm{st}}},\theta)
=
d_n + \sqrt{(\ell_w^{\star})^2 - (s_{A_{\mathrm{st}}}-d_t)^2 }.
$$

### 7.4. Apex of the walking CM path

The nominal walking manifold still has a strict local apex when written relative to the stance ankle.

Differentiating with respect to $s_{A_{\mathrm{st}}}$ gives

$$
\frac{d z_{A_{\mathrm{st}}}^{\star}}{d s_{A_{\mathrm{st}}}}
=
-\frac{s_{A_{\mathrm{st}}}-d_t}{\sqrt{(\ell_w^{\star})^2-(s_{A_{\mathrm{st}}}-d_t)^2}}.
$$

Thus the apex occurs at

$$
s_{A_{\mathrm{st}}} = d_t.
$$

This shows that the vertical oscillation of nominal walking emerges from the rigid stance geometry rather than being imposed as an artificial CM sinusoid.

## 8. CM dynamics in single support

The CM always satisfies

$$
\ddot C = g_W + u.
$$

For walking single support, the virtual control is decomposed as

$$
u = u_{\mathrm{leg}} + u_{\mathrm{tan}} + u_{\mathrm{aux}}.
$$

### 8.1. Geometric stance term

Let

$$
r = P - A_{\mathrm{st}},
$$

$$
\hat r = \frac{r}{\|r\|}.
$$

A nominal geometric stance term is

$$
u_{\mathrm{leg}} = -k_{\Phi}\Phi_w \hat r - c_{\Phi}\dot \Phi_w \hat r.
$$

### 8.2. Tangential stabilization term

Define the tangential velocity along the active support tangent by

$$
\dot s_{A_{\mathrm{st}}} = \dot C \cdot t_{\mathrm{sup}}.
$$

Define a local pendular frequency

$$
\omega_0 = \sqrt{\frac{g\cos\alpha_{\mathrm{sup}}}{\bar z}},
$$

with $\bar z > 0$.

Define an XCoM-like quantity

$$
\xi = s_Q + \lambda \frac{\dot s_{A_{\mathrm{st}}}}{\omega_0},
$$

with

$$
0 < \lambda \le 1.
$$

Then a nominal tangential control term is

$$
u_{\mathrm{tan}}
=
k_v(v_t^{\star} - \dot s_{A_{\mathrm{st}}})t_{\mathrm{sup}}
+
k_{\xi}(\xi^{\star} - \xi)t_{\mathrm{sup}}.
$$

This quantity is useful for support viability and capture reasoning, but it is not the whole walking law.

## 9. Support progression during single support

The support coordinate of the active stance foot is a true state variable.

If the active stance foot is indexed by $i$, then its support point is

$$
Q_i = \chi_{A_i}(\sigma_i),
$$

with

$$
\sigma_i \in [-L_{\mathrm{heel}}, L_{\mathrm{toe}}].
$$

The canonical support progression law is state-based.

It must depend on the support state and on pelvis progression relative to the stance ankle, not only on elapsed time.

Therefore the stance foot support progression should be written abstractly as

$$
\dot \sigma_i = F_{\sigma}^{SS}(X),
$$

or equivalently through a support phase variable and a monotone support map, as developed in the support-centered working line.

The essential requirement is:

- early single support remains rear-biased,
- mid-stance places support in the foot interior,
- late stance advances support toward the forefoot,
- the support coordinate remains inside the true support interval.

## 10. Bilateral phases in walking

The present architecture distinguishes two kinds of bilateral phases relevant to walking.

### 10.1. Gait initiation

A walking step must not start by lifting a foot arbitrarily.

If the system starts from bilateral standing, then step release must be preceded by a gait-initiation phase in which:

- support allocation shifts toward the future stance foot,
- the future swing foot is unloaded,
- unilateral-support readiness is reached before swing release.

This bilateral phase is denoted

$$
GI_{L\to R}
\qquad \text{or} \qquad
GI_{R\to L}.
$$

The current walking protocol therefore interfaces explicitly with standing.

### 10.2. Load acceptance after touchdown

Touchdown does not immediately create full support dominance of the new foot.

Instead, touchdown begins a finite bilateral load-acceptance phase:

$$
LA_{L\to R}
\qquad \text{or} \qquad
LA_{R\to L}.
$$

During this phase:

- both feet are active,
- both foot-specific support paths are defined,
- the support-allocation variable $\beta$ transfers dominance from old to new support,
- the effective support frame is bilateral,
- single support begins only after sufficient support transfer has occurred.

## 11. Support-allocation variable in bilateral phases

During any bilateral phase, define

$$
\beta \in [0,1].
$$

Its meaning depends on which foot is the current dominant support and which foot is becoming dominant, but in all cases it parameterizes support allocation between the two active feet.

The effective bilateral support point is

$$
Q = (1-\beta)Q_L + \beta Q_R.
$$

The effective bilateral support tangent is

$$
t_{\mathrm{sup}} = \frac{(1-\beta)t_L + \beta t_R}{\|(1-\beta)t_L + \beta t_R\|}.
$$

The corresponding normal is

$$
n_{\mathrm{sup}} = (-t_{{\mathrm{sup}},y}, t_{{\mathrm{sup}},x}).
$$

The canonical evolution law of $\beta$ during a bilateral phase is not purely temporal. It must be coupled to support state and support necessity.

Therefore the bilateral support-allocation variable should be written abstractly as

$$
\dot \beta = F_{\beta}^{B}(X),
$$

with the understanding that gait initiation and load acceptance may use different bilateral-support targets while sharing the same state variable.

## 12. Gait initiation and step release

A step can be triggered for two reasons:

- intentional locomotion onset,
- loss of viability of bilateral standing.

In both cases, the correct architecture is the same.

The system must enter gait initiation first, not release the swing foot immediately.

A swing-release event is allowed only when a unilateral-support-readiness condition has been satisfied. That readiness must include at least:

- sufficient support allocation toward the future stance foot,
- valid support path and support point for the future stance foot,
- exact rigid-leg feasibility of the future stance geometry,
- compatibility with the current reactive state.

Thus the transition

$$
GI \to SS
$$

is a genuine guard-based event, not a timer.

## 13. Touchdown and load acceptance

When the swing foot reaches a valid touchdown, the protocol must not jump directly to the next single-support mode.

Instead, it enters a finite load-acceptance phase.

A touchdown is valid only if:

- the contacting foot has an admissible support path,
- the contact geometry is valid over the full foot interval,
- the new stance leg is exactly IK-feasible,
- the bilateral support phase that follows remains viable.

This makes the chain explicit:

$$
\text{planner}
\to
\text{swing}
\to
\text{touchdown}
\to
\text{load acceptance}
\to
\text{new single support}.
$$

## 14. Return from walking to standing

Walking must not terminate by freezing the support state at an arbitrary instant.

The correct return to standing is a support-capture process in which:

- the walking speed target decreases,
- bilateral support becomes viable again,
- support allocation and support coordinates converge toward a quiet-standing-compatible state,
- the standing state is re-entered only after bilateral support has become a good solution again.

Thus standing is not the absence of stepping. It is a recovered bilateral-support regime.

## 15. Reactivity in walking

The nominal single-support manifold must not be interpreted as a hard prison.

Define the reactive intensity

$$
\rho \in [0,1].
$$

As $\rho$ increases, the controller must allow at least:

$$
\ell_w^{\star}(\rho)
=
2L - \varepsilon_w L - \rho\,\Delta \ell_w L,
$$

larger admissible step horizons, larger recovery steps, and modified support progression or timing compatible with a more reactive gait.

However, reactivity may never deactivate exact morphology or admissible contact geometry.

## 16. Exact IK feasibility in walk-related modes

Single-support rigid-leg feasibility requires

$$
\|P-A_{\mathrm{st}}\| \le 2L.
$$

Equivalently,

$$
\|C - 0.75L e_{\theta} - A_{\mathrm{st}}\| \le 2L.
$$

Bilateral-support rigid-leg feasibility requires

$$
\|P-A_L\| \le 2L,
$$

and

$$
\|P-A_R\| \le 2L.
$$

Equivalently,

$$
\|C - 0.75L e_{\theta} - A_L\| \le 2L,
$$

and

$$
\|C - 0.75L e_{\theta} - A_R\| \le 2L.
$$

These are exact geometric conditions. They are not proofs that the CM should remain near the midpoint between both feet at all times.

## 17. Main conclusions of this document

1. Walking must be modeled as a support-centered hybrid system.
2. Standing, gait initiation, single support, and load acceptance are distinct regimes.
3. The nominal single-support path of the CM follows from rigid geometry.
4. The ankle-local and support-local descriptions must both be kept explicitly.
5. The support coordinate $\sigma$ is a true state variable tied to the support path of the active foot.
6. The support-allocation variable $\beta$ is a true bilateral-support variable and must remain active in gait initiation and load acceptance.
7. Step release and support transfer must depend on state and viability, not only on time.
8. Recovery must emerge from the same formalism through the reactive intensity $\rho$.
