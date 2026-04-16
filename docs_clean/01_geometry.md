# 01 — Exact geometry of terrain, foot support, and rigid leg reconstruction

## 0. Goal of this document

The goal of this document is to define, with mathematical precision, the geometric objects manipulated by the locomotion model.

This document answers the following questions.

- What exactly is the terrain from a mathematical point of view?
- What exactly is a foot if the ankle is the reference point?
- How do we define the visible direction of a foot that spans two terrain segments?
- What exactly is the support point used by balance and CM dynamics?
- Given a pelvis and an ankle, when does a rigid two-segment leg exist?
- If it exists, how do we reconstruct the knee exactly?

This document is purely geometric. It does not define the walking or running control laws themselves.

## 1. Global conventions

We work in the sagittal plane.

The global frame is

$$
W = (O, e_X, e_Y), \qquad e_X = (1,0), \qquad e_Y = (0,1).
$$

The vertical axis is $e_Y$ and points upward.

The character facing direction is encoded by a sign

$$
f \in \{-1,+1\}.
$$

The value $f=+1$ means that the character faces right. The value $f=-1$ means that the character faces left.

The basic body unit is $L$.

The rigid segment lengths are

$$
\|A_i-K_i\| = L,
$$

$$
\|K_i-P\| = L,
$$

$$
\|P-T_c\| = L,
$$

$$
\|T_c-T_t\| = L,
$$

$$
\|C-P\| = 0.75L.
$$

## 2. The terrain as a polyline

### 2.1. Definition of the terrain

The terrain is a finite ordered polyline

$$
\Gamma = \bigcup_{j=1}^{N} S_j,
$$

where each segment is

$$
S_j = [V_j, V_{j+1}].
$$

The vertices satisfy

$$
V_j \in \mathbb{R}^2.
$$

Each segment has length

$$
\ell_j = \|V_{j+1} - V_j\|.
$$

We assume

$$
\ell_j > 0
$$

for every segment.

### 2.2. Tangent and normal of one segment

For segment $S_j$, define the unit tangent

$$
t_j = \frac{V_{j+1} - V_j}{\|V_{j+1} - V_j\|}.
$$

The corresponding unit normal is obtained by a rotation of $+\pi/2$:

$$
n_j = (-t_{j,y}, t_{j,x}).
$$

If

$$
t_j = (t_{j,x}, t_{j,y}),
$$

then the slope angle of the segment is

$$
\alpha_j = \mathrm{atan2}(t_{j,y}, t_{j,x}).
$$

### 2.3. Curvilinear abscissa along the terrain

Define

$$
L_0 = 0,
$$

and for $j \ge 1$,

$$
L_j = \sum_{k=1}^{j} \ell_k.
$$

Then the terrain can be parameterized by a curvilinear abscissa

$$
s_\Gamma \in [0, L_N].
$$

The corresponding terrain point is denoted

$$
\gamma(s_\Gamma).
$$

This parameterization is piecewise affine and continuous, but generally not differentiable at vertices. That is not a defect. It is the correct geometry for a segmented terrain.

## 3. The ankle as the reference point of the foot

### 3.1. Geometric foot interval

Let $A$ be the ankle of one foot. Let $f_{\mathrm{foot}}$ be the visible unit direction of the foot.

Then the geometric foot is the segment

$$
\mathcal{F}(A, f_{\mathrm{foot}})
=
\{A + \sigma f_{\mathrm{foot}} : \sigma \in [-L_{\mathrm{heel}}, L_{\mathrm{toe}}]\}.
$$

At the current stage of the project, the heel length is initialized to zero:

$$
L_{\mathrm{heel}} = 0.
$$

Therefore the implemented foot is

$$
\mathcal{F}(A, f_{\mathrm{foot}})
=
\{A + \sigma f_{\mathrm{foot}} : \sigma \in [0, L_{\mathrm{toe}}]\}.
$$

### 3.2. Facing compatibility

A visible foot direction is admissible only if it points in the facing direction.

Formally,

$$
f \, (f_{\mathrm{foot}} \cdot e_X) > 0.
$$

This prevents a visible foot from pointing backward relative to the character facing.

## 4. One-segment stance contact

Suppose the ankle $A$ belongs to one terrain segment $S_i$, and the full foot can lie on that segment.

Then the natural visible foot direction is

$$
f_{\mathrm{foot}} = f\, t_i.
$$

The visible toe point is

$$
T = A + L_{\mathrm{toe}} f_{\mathrm{foot}}.
$$

The corresponding support frame is

$$
t_{\mathrm{sup}} = f\, t_i,
$$

$$
n_{\mathrm{sup}} = (-t_{{\mathrm{sup}},y}, t_{{\mathrm{sup}},x}).
$$

## 5. Two-segment visible foot geometry

### 5.1. Forward terrain path from the ankle

Let the ankle $A$ lie on the terrain. Let the facing direction be $f$.

Define the forward terrain path starting at $A$ as the terrain arc-length parameterization in the facing direction:

$$
\gamma_A(\sigma), \qquad \sigma \ge 0,
$$

with

$$
\gamma_A(0) = A.
$$

### 5.2. Effective toe point on the terrain path

If the forward terrain path exists up to curvilinear distance $L_{\mathrm{toe}}$, define the terrain-based effective toe point by

$$
T_c = \gamma_A(L_{\mathrm{toe}}).
$$

### 5.3. Visible foot direction over two segments

The visible direction of the foot is then defined by the mini-segment from ankle to toe:

$$
f_{\mathrm{foot}} = \frac{T_c - A}{\|T_c - A\|}.
$$

This is preferable to averaging segment angles directly because it comes from actual world points and varies continuously when the toe passes from one segment to the next.

### 5.4. Why this is a weighted mean in practice

Suppose the foot spans two consecutive segments with tangents $t_i$ and $t_{i+1}$.

Let $\lambda_i$ be the terrain arc-length portion of the foot lying on $S_i$, and let $\lambda_{i+1}$ be the portion lying on $S_{i+1}$.

Then

$$
\lambda_i + \lambda_{i+1} = L_{\mathrm{toe}}.
$$

The displacement from ankle to toe is

$$
T_c - A = f\,(\lambda_i t_i + \lambda_{i+1} t_{i+1}).
$$

Therefore

$$
f_{\mathrm{foot}}
=
\frac{f\,(\lambda_i t_i + \lambda_{i+1} t_{i+1})}{\|\lambda_i t_i + \lambda_{i+1} t_{i+1}\|}.
$$

So the visible foot direction is exactly the normalized weighted combination of the two segment tangents, with weights equal to the terrain arc-length portions covered by the foot.

## 6. Admissibility of a two-segment foot

A two-segment visible foot is admissible only if all the following hold.

1. The terrain path from $A$ to $T_c$ crosses exactly one vertex.

2. The terrain angle jump is bounded:

$$
\Delta \alpha = |\alpha_{i+1} - \alpha_i| \le \alpha_{\mathrm{foot}}^{\max}.
$$

3. The visible segment $[A,T_c]$ does not create a forbidden terrain crossing away from the intended support path.

If any of these conditions fails, the foot must revert to a one-segment support model.

## 7. Effective support point and effective support tangent

### 7.1. Why the visible foot is not enough

The visible foot direction is a rendering object.

The CM dynamics and support logic need a point and a tangent representing where the support is effectively applied.

Therefore we introduce an effective support point $Q$ and an effective support tangent $t_{\mathrm{sup}}$.

### 7.2. One-segment support point

If the foot lies on a single segment with tangent $t_i$, then

$$
t_{\mathrm{sup}} = f\, t_i,
$$

and the effective support point is

$$
Q = A + \sigma t_{\mathrm{sup}},
$$

with

$$
\sigma \in [0, L_{\mathrm{toe}}].
$$

### 7.3. Two-segment support weights

Suppose the foot spans two consecutive segments with terrain arc-length portions $\lambda_i$ and $\lambda_{i+1}$.

Define normalized support weights

$$
w_i = \frac{\lambda_i}{\lambda_i + \lambda_{i+1}},
$$

$$
w_{i+1} = \frac{\lambda_{i+1}}{\lambda_i + \lambda_{i+1}}.
$$

Then

$$
w_i + w_{i+1} = 1.
$$

### 7.4. Effective support tangent on two segments

The effective support tangent is

$$
t_{\mathrm{sup}}
=
\frac{f\,(w_i t_i + w_{i+1} t_{i+1})}{\|w_i t_i + w_{i+1} t_{i+1}\|}.
$$

Its normal is

$$
n_{\mathrm{sup}} = (-t_{{\mathrm{sup}},y}, t_{{\mathrm{sup}},x}).
$$

### 7.5. Effective support point on two segments

Let $V_\star$ be the unique vertex crossed by the forward terrain path under the foot.

Define the support midpoint on the first terrain portion by

$$
Q_i = A + \frac{\lambda_i}{2} f\, t_i.
$$

Define the support midpoint on the second terrain portion by

$$
Q_{i+1} = V_\star + \frac{\lambda_{i+1}}{2} f\, t_{i+1}.
$$

Then a first effective support point is

$$
Q = w_i Q_i + w_{i+1} Q_{i+1}.
$$

This gives a simple and geometrically meaningful interpolation of support over two consecutive segments.

## 8. Local support coordinates of the CM

Once $Q$ and $t_{\mathrm{sup}}$ are defined, the CM can be decomposed in the effective support frame:

$$
C = Q + s_Q t_{\mathrm{sup}} + z_Q n_{\mathrm{sup}}.
$$

Thus

$$
s_Q = (C-Q) \cdot t_{\mathrm{sup}},
$$

$$
z_Q = (C-Q) \cdot n_{\mathrm{sup}}.
$$

The quantity $s_Q$ is the tangential progress of the CM relative to the support.

The quantity $z_Q$ is the support-normal height of the CM.

## 9. Exact IK feasibility of one rigid leg

Given an ankle point $A$, a pelvis point $P$, and two rigid segments of length $L$, we ask when there exists a knee point $K$ such that

$$
\|A-K\| = L,
$$

and

$$
\|K-P\| = L.
$$

### 9.1. Necessary and sufficient condition for existence

Define

$$
d = \|P-A\|.
$$

A solution exists if and only if

$$
0 \le d \le 2L.
$$

This is the exact rigid two-link IK condition.

### 9.2. Midpoint construction of the knee

Assume

$$
0 < d < 2L.
$$

Define the midpoint

$$
M = \frac{A+P}{2}.
$$

Define the unit direction from ankle to pelvis

$$
\nu = \frac{P-A}{\|P-A\|}.
$$

Define the perpendicular unit vector

$$
\nu^{\perp} = (-\nu_y, \nu_x).
$$

Then every solution is of the form

$$
K = M \pm h\, \nu^{\perp},
$$

where

$$
h = \sqrt{L^2 - \left(\frac{d}{2}\right)^2 }.
$$

### 9.3. Fully extended case

If

$$
d = 2L,
$$

then

$$
h = 0,
$$

so the two solutions coincide and

$$
K = M.
$$

### 9.4. Degenerate folded case

If

$$
d = 0,
$$

then the ankle and pelvis coincide and the knee may lie anywhere on the circle of radius $L$ centered at $A=P$.

This case is mathematically possible but physically irrelevant for normal locomotion.

## 10. Exact knee flexion angle

Let $\beta$ be the internal knee angle.

By the law of cosines,

$$
d^2 = L^2 + L^2 - 2L^2 \cos \beta.
$$

Therefore

$$
\cos \beta = \frac{2L^2 - d^2}{2L^2}.
$$

Hence

$$
\beta = \arccos\left(\frac{2L^2 - d^2}{2L^2}\right).
$$

In particular:

- if $d = 2L$, then $\beta = \pi$ and the leg is fully extended,
- if $d < 2L$, then $\beta < \pi$ and the leg is flexed.

## 11. Exact IK-feasible region of the CM for one stance ankle

If the torso angle is $\theta$, then

$$
P = C - 0.75L e_\theta.
$$

Therefore, for one stance ankle $A$, the exact IK-feasible region of the CM is

$$
\mathcal{V}_A(\theta)
=
\left\{ C \in \mathbb{R}^2 : \|C - 0.75L e_\theta - A\| \le 2L \right\}.
$$

This is a disk of radius $2L$ centered at

$$
A + 0.75L e_\theta.
$$

This set is a geometric feasibility set. It is not a proof that the CM should be artificially trapped near the center of that set at all times.

## 12. Two-leg simultaneous IK feasibility

If both feet are in contact, with ankles $A_L$ and $A_R$, then simultaneous rigid-leg feasibility requires

$$
\|P-A_L\| \le 2L,
$$

and

$$
\|P-A_R\| \le 2L.
$$

Equivalently,

$$
\|C - 0.75L e_\theta - A_L\| \le 2L,
$$

and

$$
\|C - 0.75L e_\theta - A_R\| \le 2L.
$$

So the exact double-support IK-feasible region is

$$
\mathcal{V}_{LR}(\theta) = \mathcal{V}_{A_L}(\theta) \cap \mathcal{V}_{A_R}(\theta).
$$

Again, this is a geometric fact, not a proof that the CM should always stay near the midpoint between both feet.

## 13. Main conclusions of this document

1. The terrain must be treated as a segmented polyline.
2. The ankle is the reference point of the foot, but it is not the whole support.
3. The mini-segment from ankle to toe gives the correct cheap visual interpolation of foot slope over two segments.
4. A two-segment foot must satisfy explicit admissibility conditions.
5. The effective support point $Q$ and tangent $t_{\mathrm{sup}}$ must be defined separately from the visible foot direction.
6. The exact rigid two-link IK condition is simply $\|P-A\| \le 2L$.
7. The knee reconstruction formula is exact and closed-form.
8. The IK-feasible region is a geometric set, not a dynamic prison for the CM.
