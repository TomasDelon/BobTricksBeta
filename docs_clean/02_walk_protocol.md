# 02 — Formal walking protocol

## 0. Goal of this document

The goal of this document is to define the mathematical protocol of walking.

This document fixes:

- the walking mode set,
- the walking state,
- the local support coordinates of the CM,
- the nominal single-support manifold,
- the support-rocker scalar $\sigma$,
- the finite double-support phase and its transfer scalar $\beta$,
- the state-based guard conditions for step switching,
- the reactive deformation of the same protocol through $\rho$.

## 1. Walking as a hybrid protocol

A nominal walking step is composed of:

1. single support on one leg,
2. finite support transfer,
3. single support on the other leg.

Therefore the walking controller must be hybrid.

The minimal walking mode set is

$$
\mathcal{M}_{\mathrm{walk}} = \{SS_L,\; DS_{L\to R},\; SS_R,\; DS_{R\to L}\}.
$$

## 2. Walking state

The authoritative dynamic object remains the virtual center of mass

$$
C(t) \in \mathbb{R}^2.
$$

Its derivatives are

$$
V(t) = \dot C(t), \qquad A(t) = \ddot C(t).
$$

The walking state must also contain:

- the torso angle $\theta$,
- the current stance ankle $A_{\mathrm{st}}$,
- the current swing ankle $A_{\mathrm{sw}}$,
- the effective support point $Q$,
- the effective support tangent $t_{\mathrm{sup}}$,
- the effective support normal $n_{\mathrm{sup}}$,
- the next ankle target $A_{\mathrm{next}}$,
- the rocker scalar $\sigma$,
- the double-support transfer scalar $\beta$,
- the local time in the current mode $\tau_{\mathrm{mode}}$,
- the reactive intensity $\rho$,
- the current mode $\mathcal{M}$.

A minimal abstract state is therefore

$$
X_{\mathrm{walk}}
=
(C, V, \theta, A_{\mathrm{st}}, A_{\mathrm{sw}}, Q, t_{\mathrm{sup}}, n_{\mathrm{sup}}, A_{\mathrm{next}}, \sigma, \beta, \tau_{\mathrm{mode}}, \rho, \mathcal{M}).
$$

## 3. Local support coordinates of the CM

Given the effective support point $Q$ and support frame $(t_{\mathrm{sup}}, n_{\mathrm{sup}})$,

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

Relative to the stance ankle,

$$
C = A_{\mathrm{st}} + s_A t_{\mathrm{sup}} + z_A n_{\mathrm{sup}}.
$$

Thus

$$
s_A = (C-A_{\mathrm{st}})\cdot t_{\mathrm{sup}},
$$

$$
z_A = (C-A_{\mathrm{st}})\cdot n_{\mathrm{sup}}.
$$

Since

$$
Q = A_{\mathrm{st}} + \sigma t_{\mathrm{sup}},
$$

we get

$$
s_Q = s_A - \sigma,
$$

$$
z_Q = z_A.
$$

This distinction is essential:

- the leg geometry is written with respect to the ankle,
- the support logic is written with respect to the effective support point.

## 4. Local support slope angle

Let

$$
t_{\mathrm{sup}} = (t_x, t_y).
$$

Define the support slope angle by

$$
\alpha_{\mathrm{sup}} = \mathrm{atan2}(t_y, t_x).
$$

This is the slope angle used in support-local walking dynamics.

## 5. Nominal single-support manifold

### 5.1. Target effective leg length

In nominal walking, the stance leg should be nearly extended.

Define

$$
\ell_w^{\star} = 2L - \varepsilon_w L,
$$

with

$$
0 < \varepsilon_w \ll 1.
$$

### 5.2. Nominal stance manifold

The nominal single-support manifold is

$$
\Phi_w(C,\theta,A_{\mathrm{st}})
=
\|P-A_{\mathrm{st}}\|^2 - (\ell_w^{\star})^2.
$$

Nominal single support corresponds to

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

### 5.3. Explicit nominal single-support path

Write

$$
d_t = 0.75L (e_{\theta}\cdot t_{\mathrm{sup}}),
$$

$$
d_n = 0.75L (e_{\theta}\cdot n_{\mathrm{sup}}).
$$

Using

$$
C-A_{\mathrm{st}} = s_A t_{\mathrm{sup}} + z_A n_{\mathrm{sup}},
$$

we get

$$
P-A_{\mathrm{st}} = (s_A-d_t)t_{\mathrm{sup}} + (z_A-d_n)n_{\mathrm{sup}}.
$$

Therefore

$$
\|P-A_{\mathrm{st}}\|^2 = (s_A-d_t)^2 + (z_A-d_n)^2.
$$

The nominal manifold becomes

$$
(s_A-d_t)^2 + (z_A-d_n)^2 = (\ell_w^{\star})^2.
$$

So the upper branch is

$$
z_A^{\star}(s_A,\theta)
=
d_n + \sqrt{(\ell_w^{\star})^2 - (s_A-d_t)^2 }.
$$

### 5.4. Apex of the walking CM path

Differentiate with respect to $s_A$:

$$
\frac{d z_A^{\star}}{d s_A}
=
-\frac{s_A-d_t}{\sqrt{(\ell_w^{\star})^2-(s_A-d_t)^2}}.
$$

Thus

$$
\frac{d z_A^{\star}}{d s_A} = 0
\iff
s_A = d_t.
$$

Differentiating again gives

$$
\frac{d^2 z_A^{\star}}{d s_A^2}
=
-\frac{(\ell_w^{\star})^2}{\left((\ell_w^{\star})^2-(s_A-d_t)^2\right)^{3/2}} < 0
$$

whenever the square root is defined.

So nominal walking has a strict local CM apex.

## 6. Walking dynamics in single support

The CM always satisfies

$$
\ddot C = g_W + u.
$$

For walking single support, we decompose the virtual control as

$$
u = u_{\mathrm{leg}} + u_{\mathrm{tan}} + u_{\mathrm{aux}}.
$$

The term $u_{\mathrm{aux}}$ groups all supplementary regularization terms that are not part of the core stance geometry or tangential capture law.

### 6.1. Geometric stance term

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

### 6.2. Tangential stabilization term

Define the tangential velocity

$$
\dot s_A = \dot C \cdot t_{\mathrm{sup}}.
$$

Define a local pendular frequency

$$
\omega_0 = \sqrt{\frac{g\cos\alpha_{\mathrm{sup}}}{\bar z}},
$$

with $\bar z > 0$.

Define an XCoM-like quantity

$$
\xi = s_Q + \lambda \frac{\dot s_A}{\omega_0},
$$

with

$$
0 < \lambda \le 1.
$$

Then a nominal tangential control term is

$$
u_{\mathrm{tan}}
=
k_v(v_t^{\star} - \dot s_A)t_{\mathrm{sup}}
+
k_{\xi}(\xi^{\star} - \xi)t_{\mathrm{sup}}.
$$

## 7. Rocker variable of the support foot

### 7.1. Why $\sigma$ needs its own law

The scalar $\sigma$ is not decorative.

It determines where the effective support lies inside the stance foot, and therefore shifts the relation between $s_A$ and $s_Q$.

So $\sigma$ must be a modeled variable.

### 7.2. Stance-phase scalar

Define a stance-phase scalar

$$
\eta_{\mathrm{st}} \in [0,1].
$$

A first geometrically natural definition is

$$
\eta_{\mathrm{st}}
=
\mathrm{clamp}\left(
\frac{s_Q - s_{\mathrm{heel}}}{s_{\mathrm{toeoff}} - s_{\mathrm{heel}}},
0,
1
\right).
$$

### 7.3. Rocker law

Let $\gamma_{\sigma}$ be a smooth monotone function on $[0,1]$ such that

$$
\gamma_{\sigma}(0)=0,
\qquad
\gamma_{\sigma}(1)=1.
$$

Then define

$$
\sigma = L_{\mathrm{toe}} \, \gamma_{\sigma}(\eta_{\mathrm{st}}).
$$

This gives a simple formal law for progression of effective support from ankle toward forefoot.

## 8. Reactivity in walking

The nominal manifold must not be interpreted as a hard prison.

Define the reactive intensity

$$
\rho \in [0,1].
$$

A generic choice is

$$
\rho
=
\mathrm{clamp}\left(
\max\left\{
\frac{|\Phi_w|}{\Delta_{\Phi}},
\frac{d(\xi,\mathcal{S})-m_{\xi}}{\Delta_{\xi}},
\frac{|\dot s_A-v_t^{\star}|}{\Delta_v}
\right\},
0,
1
\right).
$$

As $\rho$ increases, the controller must allow:

$$
\ell_w^{\star}(\rho)
=
2L - \varepsilon_w L - \rho\,\Delta \ell_w L,
$$

and

$$
\sigma_{\max}^{\mathrm{step}}(\rho)
=
\sigma_{\max,0}^{\mathrm{step}} + \rho\,\Delta \sigma_{\max}^{\mathrm{step}}.
$$

So stronger perturbations naturally allow more crouch and larger recovery steps.

## 9. Double support as a finite transition phase

### 9.1. Transfer scalar

During double support, define a support-transfer scalar

$$
\beta \in [0,1].
$$

Interpretation:

- $\beta = 0$ means support still dominated by the old stance foot,
- $\beta = 1$ means support fully transferred to the new stance foot.

### 9.2. Transfer law

Let $t_{\mathrm{DS},0}$ be the instant when double support begins.

Let $T_{\mathrm{DS}} > 0$ be the nominal duration of double support.

Let $\gamma_{\beta}$ be a smooth monotone map on $[0,1]$ such that

$$
\gamma_{\beta}(0)=0,
\qquad
\gamma_{\beta}(1)=1.
$$

Define the normalized double-support time

$$
\eta_{\mathrm{DS}}
=
\mathrm{clamp}\left(
\frac{t-t_{\mathrm{DS},0}}{T_{\mathrm{DS}}},
0,
1
\right).
$$

Then define

$$
\beta = \gamma_{\beta}(\eta_{\mathrm{DS}}).
$$

This gives a mathematically explicit transfer law.

### 9.3. Two-leg stance constraints

Let $A_b$ be the back ankle and $A_f$ the front ankle.

Define

$$
\Phi_b = \|P-A_b\|^2 - (\ell_b^{\star})^2,
$$

$$
\Phi_f = \|P-A_f\|^2 - (\ell_f^{\star})^2.
$$

Then a finite double-support stance law is

$$
u_{\mathrm{leg}}^{DS}
=
-(1-\beta)(k_b\Phi_b + c_b\dot\Phi_b)\hat r_b
-
\beta(k_f\Phi_f + c_f\dot\Phi_f)\hat r_f.
$$

### 9.4. Support interpolation in double support

During double support, define the interpolated support point

$$
Q_{DS} = (1-\beta)Q_b + \beta Q_f.
$$

Define the old and new support tangents by $t_{\mathrm{old}}$ and $t_{\mathrm{new}}$.

Then define

$$
t_{DS}
=
\frac{(1-\beta)t_{\mathrm{old}} + \beta t_{\mathrm{new}}}{\|(1-\beta)t_{\mathrm{old}} + \beta t_{\mathrm{new}}\|}.
$$

The corresponding normal is

$$
n_{DS} = (-t_{DS,y}, t_{DS,x}).
$$

This avoids the previous notation clash where $t_f$ could mean either “front” or “forward”.

## 10. Guard conditions for step switching

### 10.1. Single support to double support

The transition

$$
SS \to DS
$$

must occur only if all the following hold.

1. A valid next foothold exists.
2. The current support state is sufficiently advanced.
3. A minimum stance time has elapsed.

A representative condition is

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

### 10.2. Double support to next single support

The transition

$$
DS \to SS_{\mathrm{next}}
$$

must occur only if

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

## 11. Exact IK feasibility in walking

Single-support rigid-leg feasibility requires

$$
\|P-A_{\mathrm{st}}\| \le 2L.
$$

Equivalently,

$$
\|C - 0.75L e_{\theta} - A_{\mathrm{st}}\| \le 2L.
$$

Double-support rigid-leg feasibility requires

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

## 12. Main conclusions of this document

1. Walking must be modeled as a hybrid system.
2. Single support and double support have different mathematical roles.
3. The nominal single-support path of the CM follows from rigid geometry.
4. The ankle $A$ and support point $Q$ must both be kept explicitly.
5. The rocker scalar $\sigma$ needs its own law.
6. The transfer scalar $\beta$ also needs its own law.
7. Step switching must depend on state, not only on time.
8. Recovery must emerge from the same formalism through the reactive intensity $\rho$.
