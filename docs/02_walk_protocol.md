# 02 — Formal walking protocol

## 0. Goal of this document

The goal of this document is to define the mathematical protocol of walking.

The previous documents fixed:

- the rigid geometry of the body,
- the piecewise-linear terrain,
- the distinction between ankle $A$ and effective support point $Q$,
- the exact IK conditions for one and two rigid legs.

This document now defines:

- the state variables needed for walking,
- the hybrid walking modes,
- the nominal single-support manifold,
- the local coordinates of the CM during stance,
- the transition model in double support,
- the step-switching guard conditions,
- the way recovery must emerge from the same structure.

This document remains at the level of formal mathematical specification. It is not yet pseudocode.

## 1. Principle of the walking protocol

The central principle is the following.

A nominal walking step is composed of three conceptual phases:

1. single support on one leg,
2. support transfer through a finite double-support phase,
3. single support on the other leg.

This is not a cosmetic decomposition.

The literature on dynamic walking shows that the single-support phase behaves approximately like an inverted pendulum, while the step-to-step transition is energetically essential and cannot be reduced to a rendering trick.

Therefore the walking controller must be hybrid.

## 2. Walking modes

The minimal walking mode set is

$$
\mathcal{M}_{\mathrm{walk}} = \{SS_L,\; DS_{L\to R},\; SS_R,\; DS_{R\to L}\}.
$$

The meanings are:

- $SS_L$: left single support,
- $DS_{L\to R}$: transition from left support to right support,
- $SS_R$: right single support,
- $DS_{R\to L}$: transition from right support to left support.

At this stage, `Walk` is defined as motion inside this hybrid automaton.

`Run`, `Flight`, `Land`, and `Jump` will be treated in later documents.

## 3. State variables of the walking system

### 3.1. Authoritative dynamic state

The authoritative dynamic object remains the virtual center of mass

$$
C(t) \in \mathbb{R}^2.
$$

Its derivatives are

$$
V(t) = \dot C(t), \qquad A(t) = \ddot C(t).
$$

### 3.2. Rigid reconstruction variables

The reconstructed body also depends on a torso angle $\theta$.

The pelvis is

$$
P = C - 0.75L e_\theta,
$$

with

$$
e_\theta = (\sin\theta, \cos\theta).
$$

### 3.3. Contact state

At any time, the walking state must also contain:

- the current stance ankle,
- the current swing ankle,
- the current support point,
- the current support frame,
- the next planned foothold,
- the support-transfer variable in double support.

So the minimal walking state can be represented abstractly as

$$
X_{\mathrm{walk}} = (C, V, \theta, A_{\mathrm{st}}, A_{\mathrm{sw}}, Q, t_{\mathrm{sup}}, n_{\mathrm{sup}}, A_{\mathrm{next}}, \beta, \tau_{\mathrm{mode}}, \rho, \mathcal{M}).
$$

Here:

- $A_{\mathrm{st}}$ is the current stance ankle,
- $A_{\mathrm{sw}}$ is the current swing ankle,
- $Q$ is the effective support point,
- $t_{\mathrm{sup}}$ and $n_{\mathrm{sup}}$ are the effective support tangent and normal,
- $A_{\mathrm{next}}$ is the planned next ankle position,
- $\beta$ is the support-transfer scalar in double support,
- $\tau_{\mathrm{mode}}$ is the local time elapsed in the current mode,
- $\rho$ is the reactive intensity,
- $\mathcal{M}$ is the current hybrid mode.

## 4. Local support coordinates of the CM

During walking, the natural coordinates of the CM are not the global Cartesian coordinates alone.

They are the local coordinates relative to the effective support frame.

Given the effective support point $Q$ and support frame $(t_{\mathrm{sup}}, n_{\mathrm{sup}})$,

$$
C = Q + s_Q t_{\mathrm{sup}} + z_Q n_{\mathrm{sup}}.
$$

Thus

$$
s_Q = (C-Q) \cdot t_{\mathrm{sup}},
$$

and

$$
z_Q = (C-Q) \cdot n_{\mathrm{sup}}.
$$

The variable $s_Q$ is the tangential progress of the CM relative to the support.

The variable $z_Q$ is the support-normal height of the CM.

These are the correct coordinates for support logic, balance, and capture.

However, the rigid leg constraint is written more naturally relative to the ankle $A_{\mathrm{st}}$, not relative to $Q$.

Therefore we also define

$$
C = A_{\mathrm{st}} + s_A t_{\mathrm{sup}} + z_A n_{\mathrm{sup}},
$$

with

$$
s_A = (C-A_{\mathrm{st}}) \cdot t_{\mathrm{sup}},
$$

and

$$
z_A = (C-A_{\mathrm{st}}) \cdot n_{\mathrm{sup}}.
$$

Since

$$
Q = A_{\mathrm{st}} + \sigma t_{\mathrm{sup}},
$$

we immediately obtain

$$
s_Q = s_A - \sigma,
$$

and

$$
z_Q = z_A.
$$

This identity is fundamental.

The same CM has one tangential coordinate relative to the ankle and another relative to the support point. The difference is exactly the foot-rocker variable $\sigma$.

## 5. Nominal single-support manifold

### 5.1. Target effective leg length in walking

In nominal walking, the stance leg should be nearly extended.

Define

$$
\ell_w^\star = 2L - \varepsilon_w L,
$$

with

$$
0 < \varepsilon_w \ll 1.
$$

The role of $\varepsilon_w$ is not to bend the leg visibly at all times. It simply avoids a permanently locked singular configuration.

### 5.2. Single-support stance manifold

The nominal single-support manifold is defined by

$$
\Phi_w(C, \theta, A_{\mathrm{st}}) = \|P - A_{\mathrm{st}}\|^2 - (\ell_w^\star)^2.
$$

Nominal walking single support corresponds to

$$
\Phi_w(C, \theta, A_{\mathrm{st}}) = 0.
$$

Since

$$
P = C - 0.75L e_\theta,
$$

this can also be written as

$$
\Phi_w(C, \theta, A_{\mathrm{st}}) = \|C - 0.75L e_\theta - A_{\mathrm{st}}\|^2 - (\ell_w^\star)^2.
$$

This is the core geometric equation of nominal walking.

### 5.3. Explicit local trajectory in support coordinates

Write the pelvis–CM offset in the support frame:

$$
d_t = 0.75L (e_\theta \cdot t_{\mathrm{sup}}),
$$

$$
d_n = 0.75L (e_\theta \cdot n_{\mathrm{sup}}).
$$

Using

$$
C - A_{\mathrm{st}} = s_A t_{\mathrm{sup}} + z_A n_{\mathrm{sup}},
$$

we get

$$
P - A_{\mathrm{st}} = (s_A-d_t)t_{\mathrm{sup}} + (z_A-d_n)n_{\mathrm{sup}}.
$$

Therefore

$$
\|P - A_{\mathrm{st}}\|^2 = (s_A-d_t)^2 + (z_A-d_n)^2.
$$

Hence the nominal single-support manifold becomes

$$
(s_A-d_t)^2 + (z_A-d_n)^2 = (\ell_w^\star)^2.
$$

Solving for the upper branch,

$$
z_A^\star(s_A, \theta) = d_n + \sqrt{(\ell_w^\star)^2 - (s_A-d_t)^2 }.
$$

This gives the nominal height of the CM relative to the support frame during walking.

## 6. Existence and location of the nominal walking apex

We now characterize the apex of the nominal CM path.

Differentiate with respect to $s_A$:

$$
\frac{d z_A^\star}{d s_A} = -\frac{s_A-d_t}{\sqrt{(\ell_w^\star)^2-(s_A-d_t)^2}}.
$$

Therefore

$$
\frac{d z_A^\star}{d s_A} = 0 \iff s_A = d_t.
$$

Differentiate again:

$$
\frac{d^2 z_A^\star}{d s_A^2} = -\frac{(\ell_w^\star)^2}{\left((\ell_w^\star)^2-(s_A-d_t)^2\right)^{3/2}}.
$$

As long as the square root is defined, the denominator is positive. Hence

$$
\frac{d^2 z_A^\star}{d s_A^2} < 0.
$$

So $s_A = d_t$ is a strict local maximum.

This proves that nominal walking has a natural apex of the CM.

The apex is not an animation artifact. It is a geometric consequence of the rigid body and nearly extended stance leg.

## 7. Dynamic equation in single support

The CM always satisfies

$$
\ddot C = g_W + u.
$$

For single support in walking, we decompose the virtual control as

$$
u = u_{\mathrm{leg}} + u_{\mathrm{tan}} + u_{\mathrm{torso}}.
$$

### 7.1. Geometric leg term

Let

$$
r = P - A_{\mathrm{st}},
$$

and

$$
\hat r = \frac{r}{\|r\|}.
$$

A first nominal geometric term is

$$
u_{\mathrm{leg}} = -k_\Phi \Phi_w \hat r - c_\Phi \dot\Phi_w \hat r.
$$

Its role is to keep the state close to the nominal stance geometry.

It must not be interpreted as an infinitely hard constraint.

### 7.2. Tangential stabilization term

Define the tangential velocity

$$
\dot s_A = \dot C \cdot t_{\mathrm{sup}}.
$$

Define a local pendular frequency

$$
\omega_0 = \sqrt{\frac{g\cos\alpha_{\mathrm{sup}}}{\bar z}},
$$

where $\bar z > 0$ is a reference support-normal height and $\alpha_{\mathrm{sup}}$ is the angle of $t_{\mathrm{sup}}$.

Define an XCoM-like variable

$$
\xi = s_Q + \lambda \frac{\dot s_A}{\omega_0},
$$

with

$$
0 < \lambda \le 1.
$$

Then a tangential control term is

$$
u_{\mathrm{tan}} = k_v(v_t^\star - \dot s_A)t_{\mathrm{sup}} + k_\xi(\xi^\star - \xi)t_{\mathrm{sup}}.
$$

The role of this term is not to replace the pendular geometry, but to regulate speed and dynamic capture relative to support.

## 8. Why the nominal manifold is not a prison

The existence of a nominal manifold does not imply that the CM must always remain tightly confined to it.

If a large perturbation acts on the body, then the following should be possible.

- The CM may depart from nominal walking geometry.
- The pelvis-to-ankle distance may become shorter.
- The body may lower the CM.
- A larger step may become admissible.
- Several recovery steps may follow.

Therefore the single-support manifold is a nominal geometric attractor, not an absolute prison.

We encode the departure from nominal walking by a reactive intensity variable

$$
\rho \in [0,1].
$$

A generic choice is

$$
\rho = \mathrm{clamp}\left(\max\left\{ \frac{|\Phi_w|}{\Delta_\Phi}, \frac{d(\xi,\mathcal S)-m_\xi}{\Delta_\xi}, \frac{|\dot s_A-v_t^\star|}{\Delta_v} \right\}, 0, 1\right).
$$

As $\rho$ increases, the controller must naturally allow:

$$
\ell_w^\star(\rho) = 2L - \varepsilon_w L - \rho\,\Delta\ell_w L,
$$

and

$$
\ell_{\max}^{\mathrm{step}}(\rho) = \ell_{\max,0}^{\mathrm{step}} + \rho\,\Delta\ell_{\max}^{\mathrm{step}}.
$$

So more reactivity means more crouch and larger recovery steps.

This is the correct way to let recovery emerge from the same mathematical structure.

## 9. Double support as a finite transition phase

### 9.1. Why double support must be explicit

Walking is not just alternating single-support arcs.

The transition from one stance leg to the other is mechanically essential. Therefore double support must be modeled as a real finite phase.

### 9.2. Transfer variable

During double support, define a transfer variable

$$
\beta \in [0,1].
$$

Interpretation:

- $\beta = 0$ means that support is still dominated by the previous stance foot,
- $\beta = 1$ means that support has fully transferred to the next foot.

### 9.3. Two-leg stance constraints

Let $A_b$ be the back ankle and $A_f$ the front ankle.

Define

$$
\Phi_b = \|P-A_b\|^2 - (\ell_b^\star)^2,
$$

and

$$
\Phi_f = \|P-A_f\|^2 - (\ell_f^\star)^2.
$$

A finite double-support geometric control can be written as

$$
u_{\mathrm{leg}}^{DS} = -(1-\beta)(k_b\Phi_b + c_b\dot\Phi_b)\hat r_b - \beta(k_f\Phi_f + c_f\dot\Phi_f)\hat r_f.
$$

This means that the geometric control authority gradually shifts from the old leg to the new leg.

### 9.4. Effective support interpolation in double support

During double support, the effective support point and support tangent must also transition.

A first interpolated support point is

$$
Q_{DS} = (1-\beta)Q_b + \beta Q_f.
$$

The corresponding effective support tangent can be defined by

$$
t_{DS} = \frac{(1-\beta)t_b + \beta t_f}{\|(1-\beta)t_b + \beta t_f\|}.
$$

And the normal is

$$
n_{DS} = (-t_{DS,y}, t_{DS,x}).
$$

This defines a continuous support frame during the transfer.

## 10. Swing foot during walking

The exact swing trajectory will be specified later in the dedicated foot-planning document.

For the walking protocol, we only need to state what the swing foot must satisfy.

During single support:

- one foot is the stance foot,
- the other is the swing foot,
- the next ankle target $A_{\mathrm{next}}$ must be geometrically feasible,
- the swing trajectory must not intersect forbidden terrain geometry,
- touchdown must produce a valid new stance ankle.

Thus the walking protocol depends on a future `FootstepPlanner` and `SwingGenerator`, but it already requires a mathematically explicit foothold target.

## 11. Guard conditions for step switching

### 11.1. Why switching must not be purely time-based

A purely time-based switch is too weak.

The switch to double support or to the next single-support phase must depend on the actual dynamic and geometric state.

Otherwise the model cannot properly react to perturbations, short terrain segments, or slope changes.

### 11.2. Single support to double support

The transition from $SS$ to $DS$ must occur when three things are simultaneously true.

1. A valid next foothold exists.
2. The support state indicates that the current stance phase is sufficiently advanced.
3. A minimum stance time has elapsed.

A formal example is:

$$
SS \to DS
$$

when

$$
A_{\mathrm{next}} \text{ exists},
$$

$$
s_Q \ge s_{\mathrm{exit}}
\quad \text{or} \quad
\xi \ge \xi_{\mathrm{exit}},
$$

and

$$
\tau_{\mathrm{mode}} \ge \tau_{\mathrm{stance}}^{\min}.
$$

This says that the current support must be sufficiently advanced, but that the decision is grounded in support geometry and capture rather than in timing alone.

### 11.3. Double support to next single support

The transition from $DS$ to the next $SS$ occurs when the support transfer is complete and the new stance is viable.

A formal example is:

$$
DS \to SS_{\mathrm{next}}
$$

when

$$
\beta = 1,
$$

and

$$
\|P-A_{\mathrm{new}}\| \le 2L,
$$

and

$$
A_{\mathrm{new}} \text{ is in valid terrain contact}.
$$

This ensures that the new stance is both dynamically and geometrically meaningful.

## 12. Nominal walking cycle versus reactive walking cycle

A very important design distinction must be made.

There is a difference between:

- the nominal walking cycle,
- the reactive walking cycle.

The nominal walking cycle corresponds to small $\rho$.

In that case:

- the leg is almost fully extended,
- the CM follows the nominal manifold closely,
- support transfer is regular,
- footholds remain near the nominal step length,
- the cycle is almost periodic.

The reactive walking cycle corresponds to larger $\rho$.

In that case:

- the leg may flex more,
- the CM may move away from the nominal manifold,
- support transfer may become earlier or later,
- step length may increase,
- the cycle may lose periodicity for one or several steps.

This distinction is essential because recovery must be part of the same protocol, not a completely separate architecture.

## 13. Exact single-support IK feasibility during walking

The exact rigid-leg IK condition for the current stance ankle is

$$
\|P-A_{\mathrm{st}}\| \le 2L.
$$

Equivalently, since

$$
P = C - 0.75L e_\theta,
$$

we get the exact CM feasibility condition

$$
\|C - 0.75L e_\theta - A_{\mathrm{st}}\| \le 2L.
$$

This is an exact geometric statement.

It does not mean that the nominal controller should hug the boundary. It only means that any physically reconstructed walking state must satisfy it.

## 14. Exact double-support IK feasibility during walking

If both feet are simultaneously in contact, then exact rigid-leg feasibility requires

$$
\|P-A_L\| \le 2L,
$$

and

$$
\|P-A_R\| \le 2L.
$$

In terms of the CM,

$$
\|C - 0.75L e_\theta - A_L\| \le 2L,
$$

and

$$
\|C - 0.75L e_\theta - A_R\| \le 2L.
$$

Thus the exact double-support feasible set is

$$
\mathcal{V}_{LR}(\theta) = \mathcal{V}_{A_L}(\theta) \cap \mathcal{V}_{A_R}(\theta).
$$

This set matters geometrically, but again it is not a proof that the CM should always stay near the middle between both feet.

A perturbation may momentarily drive the body away from the nominal center of that region, and the correct response may then be a recovery step.

## 15. Summary of the formal walking protocol

The formal walking protocol is now defined by the following objects.

### Modes

$$
\mathcal{M}_{\mathrm{walk}} = \{SS_L,\; DS_{L\to R},\; SS_R,\; DS_{R\to L}\}.
$$

### State

$$
X_{\mathrm{walk}} = (C, V, \theta, A_{\mathrm{st}}, A_{\mathrm{sw}}, Q, t_{\mathrm{sup}}, n_{\mathrm{sup}}, A_{\mathrm{next}}, \beta, \tau_{\mathrm{mode}}, \rho, \mathcal{M}).
$$

### Nominal stance manifold

$$
\Phi_w(C, \theta, A_{\mathrm{st}}) = \|P-A_{\mathrm{st}}\|^2 - (\ell_w^\star)^2.
$$

### Nominal single-support path

$$
z_A^\star(s_A, \theta) = d_n + \sqrt{(\ell_w^\star)^2 - (s_A-d_t)^2 }.
$$

### Walking dynamics

$$
\ddot C = g_W + u,
$$

with

$$
u = u_{\mathrm{leg}} + u_{\mathrm{tan}} + u_{\mathrm{torso}}.
$$

### Double-support dynamics

$$
u_{\mathrm{leg}}^{DS} = -(1-\beta)(k_b\Phi_b + c_b\dot\Phi_b)\hat r_b - \beta(k_f\Phi_f + c_f\dot\Phi_f)\hat r_f.
$$

### Support interpolation

$$
Q_{DS} = (1-\beta)Q_b + \beta Q_f.
$$

$$
t_{DS} = \frac{(1-\beta)t_b + \beta t_f}{\|(1-\beta)t_b + \beta t_f\|}.
$$

### Reactivity

$$
\rho = \mathrm{clamp}\left(\max\left\{ \frac{|\Phi_w|}{\Delta_\Phi}, \frac{d(\xi,\mathcal S)-m_\xi}{\Delta_\xi}, \frac{|\dot s_A-v_t^\star|}{\Delta_v} \right\}, 0, 1\right).
$$

## 16. Main conclusions of this document

1. Walking must be modeled as a hybrid system.
2. Single support and double support have distinct mathematical roles.
3. The nominal single-support path of the CM follows from rigid geometry.
4. The support point $Q$ and ankle $A$ play different roles and must both be kept in the model.
5. Step switching must depend on state, not only on time.
6. Recovery must emerge from the same formalism through the reactive intensity $\rho$.
7. Exact IK feasibility remains a non-negotiable geometric constraint at every reconstructed frame.

## 17. What comes next

The next document should define the mathematical foothold planner.

In particular, it should answer:

- how to choose the next ankle $A_{\mathrm{next}}$ on a segmented terrain,
- how to enforce geometric admissibility of the foot,
- how to use the local CM state and support state to generate nominal and reactive step lengths,
- how to generate swing trajectories that remain collision-free.
