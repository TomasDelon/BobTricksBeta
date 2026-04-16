# 01 — Exact geometry of terrain, foot support, and rigid leg reconstruction

## 0. Goal of this document

The goal of this document is to define, with mathematical precision, the geometric objects manipulated by the locomotion model.

This document answers the following questions.

- What exactly is the terrain from a mathematical point of view?
- What exactly is a foot if the ankle is the reference point?
- How do we define the visible direction of a foot on one or two terrain segments?
- What exactly is the support point used by balance and CM dynamics?
- Given a pelvis and an ankle, when does a rigid two-segment leg exist?
- If it exists, how do we reconstruct the knee exactly?

This document is purely geometric. It does not define the walking, standing, or running control laws themselves.

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

At the current stage of the project, the heel length is assumed to be small but strictly positive:

$$
0 < L_{\mathrm{heel}} \ll L.
$$

This expresses the intended compromise of the project.

The ankle remains visually very close to the heel, but the support interval is no longer purely one-sided.

### 3.2. Facing compatibility

A visible foot direction is admissible only if it points in the facing direction.

Formally,

$$
f \, (f_{\mathrm{foot}} \cdot e_X) > 0.
$$

This prevents a visible foot from pointing backward relative to the character facing.

## 4. One-segment visible foot geometry

Suppose the ankle $A$ belongs to one terrain segment $S_i$, and the full foot can lie on that segment.

Then the natural visible foot direction is

$$
f_{\mathrm{foot}} = f\, t_i.
$$

The visible toe point is

$$
T_c = A + L_{\mathrm{toe}} f_{\mathrm{foot}}.
$$

The visible heel point is

$$
H_c = A - L_{\mathrm{heel}} f_{\mathrm{foot}}.
$$

## 5. Terrain-following foot path

### 5.1. Need for a support path

The visible foot direction is not sufficient to define support progression.

The locomotion state needs a support point that can move continuously inside the active support interval, including on non-flat terrain and across admissible vertices.

Therefore the support model must be based on a terrain-following contact path under the foot.

### 5.2. Forward and backward terrain paths from the ankle

Let the ankle $A$ lie on the terrain and let the facing direction be $f$.

Define the forward terrain path starting at $A$ by

$$
\gamma_A^{+}(\sigma), \qquad \sigma \in [0, +\infty),
$$

with

$$
\gamma_A^{+}(0) = A,
$$

and increasing in the facing direction.

Define the backward terrain path starting at $A$ by

$$
\gamma_A^{-}(\sigma), \qquad \sigma \in [0, +\infty),
$$

with

$$
\gamma_A^{-}(0) = A,
$$

and increasing in the opposite terrain direction.

### 5.3. Foot contact path

Define the foot contact path by

$$
\chi_A(\sigma)
=
\left\{
\begin{array}{ll}
\gamma_A^{-}(-\sigma) & \text{if } \sigma \in [-L_{\mathrm{heel}}, 0], \\
\gamma_A^{+}(\sigma) & \text{if } \sigma \in [0, L_{\mathrm{toe}}].
\end{array}
\right.
$$

Thus

$$
\chi_A(0) = A.
$$

This path is continuous and piecewise affine. It is the primary geometric object for support progression.

## 6. Visible foot direction on one or two segments

### 6.1. Effective toe and heel points on the terrain path

If the foot contact path exists over the full support interval, define the terrain-based effective toe point by

$$
T_c = \chi_A(L_{\mathrm{toe}}),
$$

and the terrain-based effective heel point by

$$
H_c = \chi_A(-L_{\mathrm{heel}}).
$$

### 6.2. Visible foot direction from the path endpoints

The visible direction of the foot is then defined by the mini-segment joining the effective heel and toe points:

$$
f_{\mathrm{foot}} = \frac{T_c - H_c}{\|T_c - H_c\|}.
$$

If the heel length is very small, this remains visually close to the ankle-to-toe direction while still respecting the two-sided support interval.

### 6.3. One-segment and two-segment interpretation

If the full active foot interval lies on one terrain segment, then the previous definition reduces to the terrain-aligned foot direction.

If the foot spans two admissible consecutive segments, then the same formula yields the normalized direction of the actual terrain-following chord between the effective heel and toe points.

This is preferable to averaging segment angles directly because it is built from actual world points and remains geometrically meaningful.

## 7. Admissibility of a foot support path

A full foot support path is admissible only if all the following hold.

1. The path exists on the full interval

$$
[-L_{\mathrm{heel}}, L_{\mathrm{toe}}].
$$

2. The active foot interval spans at most two admissible consecutive terrain segments.

3. If two segments are spanned, the corresponding terrain angle jump is bounded:

$$
|\alpha_{i+1} - \alpha_i| \le \alpha_{\mathrm{foot}}^{\max}.
$$

4. The visible chord between effective heel and toe does not create a forbidden terrain crossing away from the intended contact path.

5. The resulting visible foot direction remains facing-compatible.

If any of these conditions fails, the corresponding stance or touchdown geometry is not admissible.

## 8. Effective support point and effective support tangent

### 8.1. Primary support coordinate

The support state of one foot is not defined by world-space averaging.

It is defined by a support coordinate

$$
\sigma \in [-L_{\mathrm{heel}}, L_{\mathrm{toe}}].
$$

The effective support point is then

$$
Q = \chi_A(\sigma).
$$

Thus the support coordinate is primary and the support point is derived from it.

### 8.2. Effective support tangent

Whenever the path tangent exists at the active support coordinate, define

$$
t_{\mathrm{sup}} = \frac{\partial_{\sigma} \chi_A(\sigma)}{\|\partial_{\sigma} \chi_A(\sigma)\|}.
$$

Its normal is

$$
n_{\mathrm{sup}} = (-t_{{\mathrm{sup}},y}, t_{{\mathrm{sup}},x}).
$$

On one terrain segment, this reduces to the terrain tangent of that segment in the active support direction.

On a two-segment foot support, the support tangent may change when $\sigma$ crosses the vertex location along the support path.

### 8.3. Why this replaces weighted support averaging

The previous weighted-average construction of a two-segment support point is no longer canonical.

The correct support-centered geometry is obtained from the support path itself.

This is what allows support progression to remain meaningful in standing, walking, load acceptance, and later running.

## 9. Local support coordinates of the CM

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

Relative to the ankle,

$$
C = A + s_A t_{\mathrm{sup}} + z_A n_{\mathrm{sup}}.
$$

Thus

$$
s_A = (C-A) \cdot t_{\mathrm{sup}},
$$

$$
z_A = (C-A) \cdot n_{\mathrm{sup}}.
$$

Since

$$
Q = \chi_A(\sigma),
$$

there is no longer a globally valid linear identity of the form

$$
Q = A + \sigma t_{\mathrm{sup}}
$$

on arbitrary two-segment contact geometry.

The ankle-local and support-local coordinates must therefore be kept distinct, with the relation mediated by the contact path itself.

## 10. Exact IK feasibility of one rigid leg

Given an ankle point $A$, a pelvis point $P$, and two rigid segments of length $L$, we ask when there exists a knee point $K$ such that

$$
\|A-K\| = L,
$$

and

$$
\|K-P\| = L.
$$

### 10.1. Necessary and sufficient condition for existence

Define

$$
d = \|P-A\|.
$$

A solution exists if and only if

$$
0 \le d \le 2L.
$$

This is the exact rigid two-link IK condition.

### 10.2. Midpoint construction of the knee

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

### 10.3. Fully extended case

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

### 10.4. Degenerate folded case

If

$$
d = 0,
$$

then the ankle and pelvis coincide and the knee may lie anywhere on the circle of radius $L$ centered at $A=P$.

This case is mathematically possible but physically irrelevant for normal locomotion.

## 11. Exact knee flexion angle

Let $\beta_{\mathrm{knee}}$ be the internal knee angle.

By the law of cosines,

$$
d^2 = L^2 + L^2 - 2L^2 \cos \beta_{\mathrm{knee}}.
$$

Therefore

$$
\cos \beta_{\mathrm{knee}} = \frac{2L^2 - d^2}{2L^2}.
$$

Hence

$$
\beta_{\mathrm{knee}} = \arccos\left(\frac{2L^2 - d^2}{2L^2}\right).
$$

In particular:

- if $d = 2L$, then $\beta_{\mathrm{knee}} = \pi$ and the leg is fully extended,
- if $d < 2L$, then $\beta_{\mathrm{knee}} < \pi$ and the leg is flexed.

## 12. Exact IK-feasible region of the CM for one stance ankle

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

## 13. Two-leg simultaneous IK feasibility

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

## 14. Main conclusions of this document

1. The terrain must be treated as a segmented polyline.
2. The ankle is the reference point of the foot, but the support interval is two-sided with a small positive heel length.
3. The visible foot direction should be derived from effective heel and toe points on the terrain-following contact path.
4. A foot support geometry must satisfy explicit admissibility conditions on the terrain path.
5. The effective support point $Q$ and tangent $t_{\mathrm{sup}}$ must be defined from the support path and the support coordinate, not by old weighted averaging rules.
6. The exact rigid two-link IK condition is simply $\|P-A\| \le 2L$.
7. The knee reconstruction formula is exact and closed-form.
8. The IK-feasible region is a geometric set, not a dynamic prison for the CM.
