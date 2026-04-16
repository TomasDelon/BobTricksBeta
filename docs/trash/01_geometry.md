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

This document is purely geometric. It does not yet define the walking or running control laws. It defines the constraints and constructions that those control laws must respect.

Throughout the document, inline mathematics is written with `$...$`, and display mathematics is written with `$$ ... $$`.

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

Here $A_i$ is an ankle, $K_i$ a knee, $P$ the pelvis, $T_c$ the torso center, $T_t$ the upper torso node, and $C$ the unique virtual center of mass.

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

To reason rigorously about a foot spanning several segments, it is useful to define the terrain arc-length.

Let

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

The point of the terrain corresponding to $s_\Gamma$ is the unique point located at distance $s_\Gamma$ from the beginning of the polyline, measured along the terrain itself.

We denote this point by

$$
\gamma(s_\Gamma).
$$

This parameterization is piecewise affine. It is continuous but, in general, not differentiable at vertices.

That is not a defect. It faithfully represents the geometry of BobTricks terrain.

## 3. The ankle as the reference point of the foot

### 3.1. Geometric foot interval

The ankle is the reference point of the foot.

Let $A$ be the ankle of one foot. Let $f_{\mathrm{foot}}$ be the visible unit direction of the foot.

Then the geometric foot is defined as the segment

$$
\mathcal{F}(A, f_{\mathrm{foot}}) = \{A + \sigma f_{\mathrm{foot}} : \sigma \in [-L_{\mathrm{heel}}, L_{\mathrm{toe}}]\}.
$$

At the current stage of the project, the heel length is initialized to zero:

$$
L_{\mathrm{heel}} = 0.
$$

Therefore the implemented foot is essentially

$$
\mathcal{F}(A, f_{\mathrm{foot}}) = \{A + \sigma f_{\mathrm{foot}} : \sigma \in [0, L_{\mathrm{toe}}]\}.
$$

This expresses exactly what was decided in the design discussion: the foot extends forward from the ankle in the current facing direction.

### 3.2. Facing compatibility

A visible foot direction is admissible only if it points in the facing direction.

Formally, this means

$$
f_{\mathrm{foot}} \cdot e_X > 0 \quad \text{if } f = +1,
$$

and

$$
f_{\mathrm{foot}} \cdot e_X < 0 \quad \text{if } f = -1.
$$

Equivalently,

$$
f \,(f_{\mathrm{foot}} \cdot e_X) > 0.
$$

This prevents geometric solutions where the foot would visually point backward relative to the facing direction.

## 4. One-segment stance contact

### 4.1. Exact one-segment support

Suppose the ankle $A$ belongs to one terrain segment $S_i$, and suppose the full foot can lie on that same segment.

Then the natural visible foot direction is

$$
f_{\mathrm{foot}} = f\, t_i.
$$

This expression already contains the facing sign.

If $f=+1$, the foot points along $t_i$.

If $f=-1$, the foot points along $-t_i$.

The visible toe point is then

$$
T = A + L_{\mathrm{toe}} f_{\mathrm{foot}}.
$$

In that case, the whole visible foot lies on a single terrain segment, and the orientation is simply the local ground orientation.

### 4.2. Immediate consequence for the support frame

When the foot is supported by a single segment $S_i$, the support frame is simply

$$
t_{\mathrm{sup}} = f\, t_i,
$$

$$
n_{\mathrm{sup}} = (-t_{{\mathrm{sup}},y}, t_{{\mathrm{sup}},x}).
$$

This frame will later be used to decompose the CM motion and to define local tangential and normal coordinates.

## 5. Two-segment visible foot geometry

### 5.1. Why a separate definition is needed

If the ankle remains on one segment $S_i$ but the toe reaches the following segment $S_{i+1}$, then taking

$$
f_{\mathrm{foot}} = f\, t_i
$$

or

$$
f_{\mathrm{foot}} = f\, t_{i+1}
$$

would create a visible discontinuity.

The foot would visually snap from one angle to the other.

That is not acceptable.

### 5.2. Forward terrain path from the ankle

Let the ankle $A$ lie on the terrain. Let the facing direction be $f$.

Define the forward terrain path starting at $A$ as the arc-length parameterization of the terrain in the facing direction.

We denote it by

$$
\gamma_A(\sigma), \qquad \sigma \ge 0,
$$

with

$$
\gamma_A(0) = A.
$$

The value of $\gamma_A(\sigma)$ is the terrain point located at curvilinear distance $\sigma$ from $A$, measured along the terrain and taken in the facing direction.

### 5.3. Effective toe point on the terrain path

If the forward terrain path exists at least up to curvilinear distance $L_{\mathrm{toe}}$, then the terrain-based effective toe point is defined by

$$
T_c = \gamma_A(L_{\mathrm{toe}}).
$$

This point is not obtained by taking a straight segment in the world. It is obtained by following the terrain itself.

### 5.4. Visible foot direction over two segments

Once $T_c$ is known, the visible direction of the foot is defined by the mini-segment joining ankle and toe:

$$
f_{\mathrm{foot}} = \frac{T_c - A}{\|T_c - A\|}.
$$

This definition has four important properties.

1. It is continuous when the toe passes from one segment to the next.
2. It does not require any arbitrary average of angles.
3. It is defined from actual points in the world.
4. It naturally produces a weighted visual inclination when the foot spans two segments.

### 5.5. Why this is a weighted mean in practice

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
f_{\mathrm{foot}} = \frac{f\,(\lambda_i t_i + \lambda_{i+1} t_{i+1})}{\|\lambda_i t_i + \lambda_{i+1} t_{i+1}\|}.
$$

So the visible foot direction is exactly the normalized weighted combination of the two segment tangents, with weights given by the arc-length of foot support on each segment.

This proves rigorously why the mini-segment construction is the correct cheap way to obtain a weighted visual slope when the foot spans two terrain segments.

## 6. Admissibility of a two-segment foot

### 6.1. Need for admissibility conditions

The previous construction is geometrically natural, but it cannot be accepted in every possible configuration.

For example, if the next segment is a large vertical drop or a sharp step, connecting the ankle and the toe by a visible segment may produce a foot that unrealistically cuts through terrain or bridges a gap that should not be bridged.

Therefore a two-segment foot must satisfy admissibility conditions.

### 6.2. First admissibility condition: consecutive segments only

The visible foot may span at most two consecutive support segments in the first implementation.

So a two-segment foot is admissible only if the terrain path from $A$ to $T_c$ crosses exactly one vertex.

### 6.3. Second admissibility condition: bounded angle jump

Let the angle jump between both segments be

$$
\Delta \alpha = |\alpha_{i+1} - \alpha_i|.
$$

Then the two-segment foot is admissible only if

$$
\Delta \alpha \le \alpha_{\mathrm{foot}}^{\max}.
$$

The constant $\alpha_{\mathrm{foot}}^{\max}$ is a model parameter.

Its role is purely geometric: prevent one visible foot from bridging an unrealistically abrupt terrain angle.

### 6.4. Third admissibility condition: no terrain crossing by the visible segment

Let the visible foot segment be

$$
[A, T_c].
$$

This segment must not cross the terrain away from the intended support path.

A practical admissibility condition is the following.

For every point $X$ on the visible foot segment, the shortest distance from $X$ to the terrain must be achieved on the local support path covered by the foot.

If another terrain part intersects the interior of $[A, T_c]$, the two-segment construction is rejected and the support collapses to a simpler fallback geometry.

This condition is intentionally phrased geometrically. The exact implementation will later depend on the available terrain queries.

### 6.5. Fallback rule

If any admissibility condition fails, then the visible foot must revert to a one-segment support model.

That means:

- use the dominant segment under the ankle,
- set the visible direction to the tangent of that segment with the correct facing sign,
- do not attempt a two-segment interpolation.

This fallback is important because it keeps the geometric model robust even on badly shaped terrain events.

## 7. Effective support point and effective support tangent

### 7.1. Why the visible foot is not enough

The visible foot direction is a rendering and kinematic object.

The CM dynamics and the support logic need something more specific: a point and a tangent representing where the stance foot is effectively supported.

For that reason we introduce an effective support point $Q$ and an effective support tangent $t_{\mathrm{sup}}$.

### 7.2. One-segment support point

If the foot lies on a single segment with tangent $t_i$, then the effective support point is

$$
Q = A + \sigma t_{\mathrm{sup}},
$$

with

$$
t_{\mathrm{sup}} = f\, t_i,
$$

and

$$
\sigma \in [0, L_{\mathrm{toe}}].
$$

The scalar $\sigma$ represents progression of support from ankle toward toe.

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

The effective support tangent is defined by the normalized weighted combination

$$
t_{\mathrm{sup}} = \frac{f\,(w_i t_i + w_{i+1} t_{i+1})}{\|w_i t_i + w_{i+1} t_{i+1}\|}.
$$

Its normal is

$$
n_{\mathrm{sup}} = (-t_{{\mathrm{sup}},y}, t_{{\mathrm{sup}},x}).
$$

This is not exactly the same object as the visible foot direction, although in practice both will often be close.

### 7.5. Effective support point on two segments

Let

$$
Q_i = A + \sigma_i f\, t_i,
$$

and let $Q_{i+1}$ be the corresponding point on the next segment support portion.

Then a first effective support point is

$$
Q = w_i Q_i + w_{i+1} Q_{i+1}.
$$

This construction is simple, cheap, and already sufficient for a first mechanically meaningful support interpolation over two segments.

## 8. Local support coordinates of the CM

Once $Q$ and $t_{\mathrm{sup}}$ are defined, the CM can be decomposed in the effective support frame.

Let

$$
n_{\mathrm{sup}} = (-t_{{\mathrm{sup}},y}, t_{{\mathrm{sup}},x}).
$$

Then

$$
C = Q + s_Q t_{\mathrm{sup}} + z_Q n_{\mathrm{sup}}.
$$

with

$$
s_Q = (C-Q) \cdot t_{\mathrm{sup}},
$$

$$
z_Q = (C-Q) \cdot n_{\mathrm{sup}}.
$$

The quantity $s_Q$ is the tangential progress of the CM relative to the effective support.

The quantity $z_Q$ is the support-normal height of the CM.

Later, walking and running dynamics will be written in terms of these local support coordinates.

## 9. Exact IK feasibility of one rigid leg

We now solve the inverse problem of one rigid leg exactly.

Given:

- an ankle point $A$,
- a pelvis point $P$,
- two rigid segments of length $L$,

when does there exist a knee point $K$ such that

$$
\|A-K\| = L,
$$

and

$$
\|K-P\| = L?
$$

### 9.1. Necessary and sufficient condition for existence

Define the pelvis–ankle distance

$$
d = \|P-A\|.
$$

A solution exists if and only if

$$
0 \le d \le 2L.
$$

**Proof.**

The point $K$ must belong simultaneously to:

- the circle of center $A$ and radius $L$,
- the circle of center $P$ and radius $L$.

Two circles of equal radius $L$ intersect if and only if the distance between their centers is at most $2L$.

So existence is equivalent to

$$
d \le 2L.
$$

The lower bound $d \ge 0$ is automatic because $d$ is a norm.

This proves the claim.

### 9.2. Midpoint construction of the knee

Assume now that

$$
0 < d < 2L.
$$

Define the midpoint of segment $[A,P]$:

$$
M = \frac{A+P}{2}.
$$

Define the unit vector from ankle to pelvis:

$$
u = \frac{P-A}{\|P-A\|}.
$$

Define a perpendicular unit vector

$$
\nu^{\perp} = (-u_y, u_x).
$$

Then every solution $K$ must be of the form

$$
K = M \pm h\, \nu^{\perp},
$$

where

$$
h = \sqrt{L^2 - \left(\frac{d}{2}\right)^2 }.
$$

**Proof.**

The intersection points of two equal circles lie on the perpendicular bisector of the segment joining the centers. Since both circles have radius $L$, the line of possible solutions is the perpendicular bisector of $[A,P]$, which passes through $M$ and has direction $\nu^{\perp}$.

Now the triangle formed by $A$, $M$, and $K$ is right-angled, with

$$
\|A-M\| = \frac{d}{2},
$$

and

$$
\|A-K\| = L.
$$

Therefore, by Pythagoras,

$$
L^2 = \left(\frac{d}{2}\right)^2 + h^2.
$$

So

$$
h = \sqrt{L^2 - \left(\frac{d}{2}\right)^2 }.
$$

This proves the formula.

### 9.3. Fully extended case

If

$$
d = 2L,
$$

then

$$
h = 0.
$$

So the two solutions coincide, and the knee lies exactly on the midpoint of the segment $[A,P]$:

$$
K = M.
$$

This is the fully extended leg configuration.

### 9.4. Degenerate folded case

If

$$
d = 0,
$$

then the ankle and the pelvis coincide. In that degenerate case the knee can lie anywhere on the circle of radius $L$ centered at $A=P$.

This case is mathematically possible but physically irrelevant for normal locomotion. The implementation should treat it as a singular fallback case.

## 10. Choosing the correct knee branch

The midpoint formula yields two possible knees:

$$
K_+ = M + h\,\nu^{\perp},
$$

$$
K_- = M - h\,\nu^{\perp}.
$$

Only one of them corresponds to the visual side of the knee that we want.

In sagittal 2D locomotion, the simplest rule is to choose the branch that places the knee on the anatomically expected side relative to the facing direction.

A practical criterion is to choose the sign so that the knee points forward in the current gait convention.

This sign choice is a modeling convention, not a geometric theorem. The important point is that once the convention is fixed, the reconstruction is exact and deterministic.

## 11. Exact knee flexion angle

The geometry also gives the knee angle explicitly.

Let $\beta$ be the internal knee angle, that is, the angle between the femur segment and the shank segment.

By the law of cosines applied to the triangle with sides $L$, $L$, and $d$,

$$
d^2 = L^2 + L^2 - 2L^2 \cos \beta.
$$

Therefore

$$
\cos \beta = \frac{2L^2 - d^2}{2L^2}.
$$

So

$$
\beta = \arccos\left(\frac{2L^2 - d^2}{2L^2}\right).
$$

This formula is useful because it quantifies leg flexion exactly.

In particular:

- if $d = 2L$, then $\cos \beta = -1$, hence $\beta = \pi$ and the leg is fully extended;
- if $d < 2L$, then $\beta < \pi$ and the leg is flexed.

This gives an exact mathematical way to say whether the current gait solution bends the leg too much.

## 12. Exact IK-feasible region of the CM for one stance ankle

The CM is not itself attached to the ankle, but the pelvis is.

If the torso angle is $\theta$, then

$$
P = C - 0.75L e_\theta.
$$

Therefore, for one stance ankle $A$, the exact IK-feasible region of the CM is

$$
\mathcal{V}_A(\theta) = \left\{ C \in \mathbb{R}^2 : \|C - 0.75L e_\theta - A\| \le 2L \right\}.
$$

This set is a disk of radius $2L$ centered at

$$
A + 0.75L e_\theta.
$$

This region is an exact geometric fact.

It is crucial to distinguish two statements.

First statement:

$\mathcal{V}_A(\theta)$ is the exact region where rigid-leg IK is possible for that ankle and torso angle.

Second statement:

The locomotion controller should always keep the CM strictly trapped inside a small nominal subset of that region.

The first statement is true.

The second statement is false and should not be adopted as a design principle, because large perturbations must be able to push the system away from nominal gait and trigger recovery behavior.

So $\mathcal{V}_A(\theta)$ is a geometric feasibility set, not a dynamic prison.

## 13. Two-leg simultaneous IK feasibility

Suppose both feet are in contact, with ankles $A_L$ and $A_R$.

Then simultaneous rigid-leg feasibility requires

$$
\|P-A_L\| \le 2L,
$$

and

$$
\|P-A_R\| \le 2L.
$$

Equivalently, in terms of the CM,

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

Again, this intersection is a geometric fact. It tells us when the body can be reconstructed with both feet attached.

It must not be confused with a mandatory support box for the dynamic evolution of the CM under perturbation.

## 14. Summary of the geometric objects defined here

We have now defined the exact meaning of the core geometric objects.

### Terrain

$$
\Gamma = \bigcup_{j=1}^{N} S_j
$$

with segment tangents $t_j$, normals $n_j$, lengths $\ell_j$, and vertices $V_j$.

### Foot

$$
\mathcal{F}(A, f_{\mathrm{foot}}) = \{A + \sigma f_{\mathrm{foot}} : \sigma \in [0, L_{\mathrm{toe}}]\}
$$

in the current implementation.

### Visible toe point on terrain

$$
T_c = \gamma_A(L_{\mathrm{toe}})
$$

when this is admissible.

### Visible foot direction

$$
f_{\mathrm{foot}} = \frac{T_c - A}{\|T_c - A\|}.
$$

### Effective support tangent

$$
t_{\mathrm{sup}} = \frac{f\,(w_i t_i + w_{i+1} t_{i+1})}{\|w_i t_i + w_{i+1} t_{i+1}\|}
$$

in the two-segment case.

### Effective support point

$$
Q = w_i Q_i + w_{i+1} Q_{i+1}
$$

in the two-segment case.

### Exact rigid-leg IK feasibility

$$
\|P-A\| \le 2L.
$$

### Exact knee reconstruction

$$
K = M \pm h\,\nu^{\perp},
$$

with

$$
M = \frac{A+P}{2}, \qquad h = \sqrt{L^2 - \left(\frac{d}{2}\right)^2 }, \qquad d = \|P-A\|.
$$

## 15. Main conclusions of this document

1. The terrain must be treated as a true polyline, not as a globally smooth curve.
2. The ankle is the foot reference point, but it is not the whole support.
3. The mini-segment from ankle to toe point gives the correct cheap visual interpolation of foot slope over two segments.
4. A two-segment foot must satisfy explicit admissibility conditions.
5. The effective support point $Q$ and tangent $t_{\mathrm{sup}}$ must be defined separately from the visible foot orientation.
6. The exact rigid two-link IK condition is simply $\|P-A\| \le 2L$.
7. The knee reconstruction formula is exact and closed-form.
8. The IK-feasible region is a geometric set, not a dynamic prison for the CM.

## 16. What comes next

This document fixes the geometry.

The next document should use these objects to define the exact local coordinates of the CM, the nominal walking manifold, the support-transfer model, and the trigger conditions for step switching.
