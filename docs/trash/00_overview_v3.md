# 00 — Scientific and mathematical foundations of the `CM-first` model (v3)

This file supersedes `docs/00_overview_v2.md` for mathematical rendering.

From this document onward:

- inline mathematics is written with `$...$`,
- display mathematics is written with `$$ ... $$`,
- `\operatorname{...}` is not used,
- every display block is separated from surrounding text by blank lines.

## 0. Goal of this document

The goal of this document is to define, rigorously and progressively, the mathematical objects manipulated by the future locomotion engine of `BobTricksBeta`.

The central design principle is the following.

- There exists exactly one authoritative center of mass, denoted $C$.
- This $C$ is a virtual object, not a geometric center of mass reconstructed from body-part masses.
- The rest of the body is reconstructed from $C$ and from contact constraints.
- Walking and running must be modeled by two different nominal mechanical protocols.
- Perturbation recovery must emerge from the same architecture, not from an unrelated extra feature.

This document does not yet define the full operational algorithm. It fixes the mathematical foundation on top of which the later code and control laws will be built.

## 1. Global frame and basic conventions

We work in the sagittal plane.

The global world frame is denoted $W = (O, e_X, e_Y)$, with:

$$
e_X = (1,0),
\qquad
e_Y = (0,1).
$$

The $X$ axis is horizontal and the $Y$ axis is vertical upward.

Gravity is

$$
g_W = (0,-g),
\qquad g>0.
$$

We use a virtual unit mass:

$$
m = 1.
$$

Therefore Newton's law becomes numerically

$$
\ddot C = \sum F.
$$

This is a modeling convention: all control terms applied to the virtual center of mass are directly interpreted as accelerations.

## 2. Rigid morphology of BobTricks

### 2.1. Fundamental length unit

The body is parameterized by a unique length unit $L$.

The current project convention is

$$
H = 5L,
$$

where $H$ is the total body height.

In the current public configuration,

$$
H = 1.80\,\mathrm{m},
\qquad
L = 0.36\,\mathrm{m}.
$$

### 2.2. Anatomical nodes

We define the following nodes:

- $A_L, A_R$: left and right ankles,
- $K_L, K_R$: left and right knees,
- $P$: pelvis,
- $T_c$: center of the torso,
- $T_t$: upper torso node,
- $C$: virtual center of mass.

### 2.3. Rigid segment lengths

The segment lengths are fixed and must remain exact in every rendered frame.

For each leg $i \in \{L,R\}$,

$$
\|A_i - K_i\| = L,
$$

$$
\|K_i - P\| = L.
$$

For the torso,

$$
\|P - T_c\| = L,
$$

$$
\|T_c - T_t\| = L.
$$

The pelvis–CM distance is fixed by design:

$$
\|C - P\| = 0.75L.
$$

This means in particular that the maximum pelvis–ankle distance is

$$
\ell_{\max} = 2L.
$$

Indeed, by the triangle inequality,

$$
\|P - A_i\| \le \|P - K_i\| + \|K_i - A_i\| = 2L.
$$

Equality holds if and only if the hip, knee and ankle are aligned and the knee is fully extended.

This is a first important structural consequence.

The leg cannot change its bone length. Any reduction of $\|P-A_i\|$ must come from knee flexion.

## 3. The unique authoritative CM

### 3.1. Definition

The only authoritative dynamic variable is the virtual center of mass

$$
C(t) \in \mathbb{R}^2.
$$

Its velocity and acceleration are

$$
V(t) = \dot C(t),
\qquad
A(t) = \ddot C(t).
$$

There is no second geometric center of mass computed from the rendered body.

The rendered body must be compatible with $C$, but it does not define it.

### 3.2. Why this choice is structurally sound

This choice makes the architecture genuinely `CM-first`.

1. The dynamic core manipulates one principal object.
2. Contacts and footholds are computed with respect to that object.
3. Body reconstruction becomes a constrained inverse problem rather than the source of the dynamics.

This is mathematically cleaner and also more extensible to walking, running, flight, landing and recovery.

## 4. Reconstruction of pelvis and torso from the CM

Let $\theta$ be the torso angle measured from the global vertical.

Define the torso direction vector by

$$
e_\theta = (\sin\theta, \cos\theta).
$$

Then the pelvis is reconstructed from the CM by

$$
P = C - 0.75L\, e_\theta.
$$

The torso nodes are then reconstructed by

$$
T_c = P + L\, e_\theta,
$$

$$
T_t = P + 2L\, e_\theta.
$$

This immediately implies

$$
\|C-P\| = 0.75L,
\qquad
\|P-T_c\| = L,
\qquad
\|T_c-T_t\| = L.
$$

So these three segment constraints are satisfied identically once $C$ and $\theta$ are known.

## 5. The ground model

### 5.1. Piecewise-linear terrain

The ground is not modeled as a globally smooth curve.

It is modeled as a polyline:

$$
\Gamma = \bigcup_{j=1}^{N} S_j,
$$

where each $S_j$ is a segment

$$
S_j = [V_j, V_{j+1}].
$$

Each segment carries a constant tangent $t_j$, a constant normal $n_j$, and a constant slope angle $\alpha_j$.

If

$$
t_j = (t_{j,x}, t_{j,y}),
$$

then

$$
\alpha_j = \mathrm{atan2}(t_{j,y}, t_{j,x}).
$$

### 5.2. Vertices are true geometric events

The vertices $V_j$ are not numerical artifacts. They are discrete geometric events:

- slope discontinuities,
- short support patches,
- ridge-like or edge-like transitions,
- step up / step down situations,
- situations where a foot simultaneously covers two neighboring segments.

Therefore the final mathematical model must explicitly distinguish

- motion on one segment,
- motion over a transition between two segments.

## 6. Foot, ankle, and effective support

### 6.1. Ankle-based foot geometry

The current project convention is that the reference point of the foot is the ankle.

Let $A$ be the ankle of the stance foot, and let $f$ be the current visible direction of the foot.

The visible foot segment is

$$
\mathcal{F}(A,f)=\{A+\sigma f:\ \sigma\in[-L_{\text{heel}},L_{\text{toe}}]\}.
$$

The current project choice is

$$
L_{\text{heel}} = 0,
$$

or at least very close to zero, so the visible foot extends essentially forward from the ankle.

### 6.2. Contact-effective support point

The ankle is not the whole support.

For locomotion dynamics we define an effective support point $Q$ inside the foot:

$$
Q = A + \sigma t,
$$

with

$$
\sigma \in [0, L_{\text{toe}}].
$$

Here $t$ is the local ground tangent used by the stance foot.

Interpretation:

- at early stance, $\sigma$ is small,
- during heel rise, $\sigma$ increases,
- near toe-off, $Q$ approaches the front part of the foot.

This distinction between ankle $A$ and support point $Q$ is essential.

The leg constraint is written with respect to $A$, because the leg is anatomically connected to the ankle.

The balance and support-transfer logic is written with respect to $Q$, because the support effectively moves forward within the foot.

## 7. Visible foot over two segments

Suppose the ankle remains on segment $S_i$ but the toe already lies over segment $S_{i+1}$.

A very natural visible foot direction is obtained from the mini-segment joining the ankle to the effective toe point $T_c$:

$$
f_{\mathrm{vis}} = \frac{T_c - A}{\|T_c - A\|}.
$$

This is preferable to averaging angles directly, because:

1. it comes from an actual geometric segment,
2. it avoids artificial angular interpolation,
3. it gives a visually smooth orientation over slope changes.

However, the visible direction of the foot should not automatically be treated as the full truth of support dynamics.

The support point $Q$ and the support tangent used by the CM may still need a separate, slightly more physical construction.

## 8. Nominal standing height

Before defining gait dynamics, we compute the nominal standing CM height.

Let the nominal ankle separation be

$$
\|A_R - A_L\| = d_{\mathrm{pref}} L.
$$

Assume a symmetric standing pose with both legs fully extended and the pelvis centered between both ankles.

Then each leg forms a right triangle with:

- hypotenuse $2L$,
- horizontal leg offset $\dfrac{d_{\mathrm{pref}}L}{2}$,
- pelvis height $h_P^{\mathrm{stand}}$.

By Pythagoras,

$$
(2L)^2 = \left(\frac{d_{\mathrm{pref}}L}{2}\right)^2 + \left(h_P^{\mathrm{stand}}\right)^2.
$$

Therefore,

$$
h_P^{\mathrm{stand}} = \sqrt{(2L)^2 - \left(\frac{d_{\mathrm{pref}}L}{2}\right)^2 }.
$$

Since the CM is located $0.75L$ above the pelvis when $\theta = 0$, the nominal standing CM height is

$$
h_C^{\mathrm{stand}} = h_P^{\mathrm{stand}} + 0.75L.
$$

With the current choice $d_{\mathrm{pref}} = 0.90$,

$$
h_P^{\mathrm{stand}}
= \sqrt{4L^2 - 0.2025L^2}
= \sqrt{3.7975}\,L
\approx 1.9487L,
$$

hence

$$
h_C^{\mathrm{stand}} \approx 2.6987L.
$$

If $L = 0.36\,\mathrm{m}$, then

$$
h_C^{\mathrm{stand}} \approx 0.9715\,\mathrm{m}.
$$

This quantity is a standing reference, not a general law for locomotion.

## 9. Local frame of support and gravity decomposition

Let the stance foot currently use a local tangent $t$ and a local normal $n$.

Then the CM can be decomposed relative to the ankle as

$$
C = A + s_A t + z_A n,
$$

with

$$
s_A = (C-A) \cdot t,
\qquad
z_A = (C-A) \cdot n.
$$

The gravity vector decomposes in this frame as

$$
g_t = g_W \cdot t = -g\sin\alpha,
$$

$$
g_n = g_W \cdot n = -g\cos\alpha,
$$

where $\alpha$ is the local ground slope angle.

This yields immediately:

- downhill in the facing direction: $\alpha < 0 \Rightarrow -g\sin\alpha > 0$,
- flat ground: $\alpha = 0 \Rightarrow -g\sin\alpha = 0$,
- uphill: $\alpha > 0 \Rightarrow -g\sin\alpha < 0$.

So on a slope the tangential component of gravity already explains why the CM naturally tends to accelerate downhill and decelerate uphill.

This is a first-principles reason why a good downhill walking model cannot be obtained from a purely vertical CM correction.

## 10. Why walking needs a pendular nominal model

The dynamic walking literature shows that the stance phase of walking is not well described as a flat vertical-preserving motion.

The body behaves approximately like an inverted pendulum during single support, and the costly part of the gait is concentrated in the step-to-step transition.

Therefore the nominal model for walking must be built around a stance leg that is nearly extended.

Let

$$
\ell_w^\star = 2L - \varepsilon_w L,
\qquad 0 < \varepsilon_w \ll 1.
$$

This is the target effective hip-to-ankle distance during nominal walking.

Then the geometric nominal walking manifold is

$$
\Phi_w(C,\theta,A)
=
\|P-A\|^2 - (\ell_w^\star)^2
= 0.
$$

Using

$$
P = C - 0.75L e_\theta,
$$

this becomes

$$
\Phi_w(C,\theta,A)
=
\|C - 0.75L e_\theta - A\|^2 - (\ell_w^\star)^2
= 0.
$$

## 11. Explicit formula for the nominal CM trajectory in walking

We now derive the explicit local shape of the nominal CM trajectory.

Write the pelvis–CM offset in the support frame:

$$
d_t = 0.75L (e_\theta \cdot t),
\qquad
d_n = 0.75L (e_\theta \cdot n).
$$

Since

$$
C-A = s_A t + z_A n,
$$

we obtain

$$
P-A = (s_A - d_t)t + (z_A - d_n)n.
$$

Therefore,

$$
\|P-A\|^2 = (s_A-d_t)^2 + (z_A-d_n)^2.
$$

Plugging this into $\Phi_w = 0$ gives

$$
(s_A-d_t)^2 + (z_A-d_n)^2 = (\ell_w^\star)^2.
$$

Solving for the physically relevant upper branch,

$$
z_A^\star(s_A,\theta)
=
d_n + \sqrt{(\ell_w^\star)^2 - (s_A-d_t)^2 }.
$$

This is the correct nominal CM height law for walking.

It is not guessed heuristically. It is a direct consequence of:

1. rigid geometry,
2. a single virtual CM,
3. a nearly extended stance leg.

## 12. Existence of the walking apex

We now prove that the nominal trajectory has a local height maximum.

Differentiate with respect to $s_A$:

$$
\frac{d z_A^\star}{d s_A}
=
-\frac{s_A-d_t}{\sqrt{(\ell_w^\star)^2-(s_A-d_t)^2}}.
$$

So

$$
\frac{d z_A^\star}{d s_A} = 0
\iff
s_A = d_t.
$$

Differentiate once more:

$$
\frac{d^2 z_A^\star}{d s_A^2}
=
-\frac{(\ell_w^\star)^2}{\left((\ell_w^\star)^2 - (s_A-d_t)^2\right)^{3/2}}.
$$

Whenever the square root is defined, the denominator is positive, so

$$
\frac{d^2 z_A^\star}{d s_A^2} < 0.
$$

Hence $s_A = d_t$ is a strict local maximum.

This proves that the apex of the CM in nominal walking emerges naturally from the rigid geometry.

It does not need to be artificially animated.

## 13. Role of the support point $Q$

The previous derivation is written with respect to the ankle $A$, because the leg is attached there.

However, the support logic is better written with respect to $Q$:

$$
Q = A + \sigma t.
$$

Define the CM tangential coordinate relative to the support point by

$$
s_Q = (C-Q)\cdot t = s_A - \sigma.
$$

Because $\sigma$ progresses forward inside the foot during stance, the support-based phase variable and the ankle-based geometric phase variable are not identical.

This is not a bug. It is exactly what we want:

- the leg geometry is controlled through $A$,
- the support logic is controlled through $Q$.

This separation is one of the keys to obtaining realistic heel rise and toe-off without overcomplicating the model.

## 14. Dynamic equation of the CM

The global CM equation is

$$
\ddot C = g_W + u,
$$

where $u$ is a virtual control acceleration.

For walking, it is natural to decompose it into

$$
u = u_{\mathrm{leg}} + u_{\mathrm{tan}} + u_{\mathrm{trans}}.
$$

### 14.1. Geometric stance term

Let

$$
r = P-A,
\qquad
\hat r = \frac{r}{\|r\|}.
$$

A first nominal geometric correction is

$$
u_{\mathrm{leg}}
=
-k_\Phi \Phi_w \hat r - c_\Phi \dot \Phi_w \hat r.
$$

Its role is to keep the walking state near the nominal manifold.

This term is not meant to trap the system inside a hard box.

It only defines the nominal regime.

### 14.2. Tangential speed and dynamic stability term

Define the local tangential velocity

$$
\dot s_A = \dot C \cdot t.
$$

Define a nominal local pendular frequency by

$$
\omega_0 = \sqrt{\frac{g\cos\alpha}{\bar z}},
$$

where $\bar z > 0$ is a reference normal height.

Then define a local XCoM-like variable

$$
\xi = s_Q + \lambda \frac{\dot s_A}{\omega_0},
\qquad 0 < \lambda \le 1.
$$

This follows the logic of Hof's extrapolated center of mass, but it will be used as a signal, not as the sole law of locomotion.

A nominal tangential control term is then

$$
u_{\mathrm{tan}}
=
k_v(v_t^\star - \dot s_A)t
+
k_\xi(\xi^\star - \xi)t.
$$

Here:

- $v_t^\star$ is the desired tangential speed,
- $\xi^\star$ is a support-compatible capture reference.

## 15. Double support and step-to-step transition

Dynamic walking theory shows that step-to-step transition is an energetically essential part of walking.

Therefore double support cannot be reduced to a visual interpolation.

Let:

- $A_b$ be the back ankle,
- $A_f$ be the front ankle,
- $\beta \in [0,1]$ be the support transfer parameter.

Define two soft constraints:

$$
\Phi_b = \|P-A_b\|^2 - (\ell_b^\star)^2,
$$

$$
\Phi_f = \|P-A_f\|^2 - (\ell_f^\star)^2.
$$

Then a double-support geometric control can be written as

$$
u_{\mathrm{leg}}^{DS}
=
-(1-\beta)(k_b\Phi_b + c_b\dot\Phi_b)\hat r_b
-\beta(k_f\Phi_f + c_f\dot\Phi_f)\hat r_f.
$$

This is already enough to state an important scientific principle.

Double support must be its own dynamic subproblem, especially on slopes and at terrain transitions.

## 16. Recovery must not come from a hard feasibility prison

A central architectural decision is the following.

The nominal walking manifold must not be interpreted as a hard prison that the CM can never leave.

If a strong perturbation strikes the body, the correct behavior is not:

- to clamp the CM inside a nominal support box,
- nor to fake the recovery visually.

The correct behavior is:

- the CM leaves the nominal regime,
- the controller enters a recovery regime,
- the body lowers the CM,
- the allowed step length grows,
- one or several recovery steps may occur.

This is not an optional extra feature. It must be a natural consequence of the architecture.

We therefore define a reactive intensity variable

$$
\rho \in [0,1].
$$

A generic form is

$$
\rho
=
\mathrm{clamp}
\left(
\max\left\{
\frac{d(\xi,\mathcal S)-m_\xi}{\Delta_\xi},
\frac{\|P-A_{\mathrm{stance}}\| - \ell_w^\star}{\Delta_\ell},
\frac{|\dot s_A - v_t^\star|}{\Delta_v}
\right\},
0,1
\right).
$$

Then the stance-leg target length may become

$$
\ell_w^\star(\rho)
=
2L - \varepsilon_w L - \rho\,\Delta \ell_w L,
$$

which means that stronger perturbations naturally allow more knee flexion and therefore a lower CM.

Likewise, the admissible step length can increase with $\rho$:

$$
\ell_{\max}^{\mathrm{step}}(\rho)
=
\ell_{\max,0}^{\mathrm{step}} + \rho\,\Delta \ell_{\max}^{\mathrm{step}}.
$$

So lowering the CM and taking larger recovery steps emerge from the same mathematical structure.

## 17. Why running must use another nominal protocol

Walking and running are not the same phenomenon at different speeds.

The biomechanical distinction is classical:

- walking is approximately pendular,
- running is approximately spring-mass / SLIP-like.

Therefore the same $C$ can remain authoritative, but the contact protocol must change.

A minimal running mode set is

$$
\mathcal M_{\mathrm{run}} = \{RS_L,\; FLIGHT,\; RS_R\}.
$$

During running stance, define

$$
\ell = \|P-A\|.
$$

Then a minimal virtual spring-damper law is

$$
u_{\mathrm{run}}
=
k_r(\ell_0 - \ell)_+\,\hat r
-
c_r \dot \ell\,\hat r
+
k_t(v_t^\star - \dot s_A)t,
$$

where

$$
(x)_+ = \max(x,0).
$$

This does not mean that the bones stretch. It means that the effective leg length compresses through posture and knee flexion.

This difference has a key geometric consequence:

- in walking, the CM tends to reach a local maximum near mid-stance,
- in running, the CM tends to reach a local minimum near mid-stance.

That is why `Run` cannot be treated as a simple accelerated `Walk`.

## 18. Summary of the architecture implied by the mathematics

The scientific structure fixed by this document is

$$
\text{TerrainModel}
\rightarrow
\text{ContactModel}
\rightarrow
\text{CMController}
\rightarrow
\text{FootstepPlanner}
\rightarrow
\text{SwingGenerator}
\rightarrow
\text{BodyReconstruction}
\rightarrow
\text{Render}.
$$

This has the following interpretation.

1. The terrain defines local geometry.
2. The contact model defines stance / swing / support transfer.
3. The CM controller evolves the authoritative dynamic state.
4. The footstep planner selects the next ankle target.
5. The swing generator creates a collision-free swing trajectory.
6. Body reconstruction imposes rigid distances exactly.
7. Rendering only draws the result.

## 19. Main conclusions of this document

1. The project must remain strictly `CM-first`.
2. The ground must be modeled as a segmented polyline with explicit transition events.
3. The ankle and the support point must be distinguished.
4. Walking must be governed by a nearly extended stance-leg geometry.
5. The nominal walking CM trajectory is

$$
z_A^\star(s_A,\theta)
=
d_n + \sqrt{(\ell_w^\star)^2 - (s_A-d_t)^2 }.
$$

6. The walking apex is not a visual heuristic. It is a direct consequence of rigid geometry.
7. Recovery must arise from the same architecture by increasing reactivity, lowering the CM and allowing longer steps.
8. Running must use a spring-mass-like effective protocol and cannot be a simple speed-up of walking.

## 20. Primary bibliography

### Kuo 2007

Arthur D. Kuo, *The six determinants of gait and the inverted pendulum analogy: A dynamic walking perspective*, Human Movement Science, 26(4):617–656, 2007.

DOI: `10.1016/j.humov.2007.04.003`

### Hof 2008

A. L. Hof, *The 'extrapolated center of mass' concept suggests a simple control of balance in walking*, Human Movement Science, 27(1):112–125, 2008.

DOI: `10.1016/j.humov.2007.08.003`

### Blickhan 1989

R. Blickhan, *The spring-mass model for running and hopping*, Journal of Biomechanics, 22(11-12):1217–1227, 1989.

DOI: `10.1016/0021-9290(89)90224-8`

### Dewolf et al. 2017

A. H. Dewolf, Y. Ivanenko, F. Lacquaniti, P. A. Willems, *Pendular energy transduction within the step during human walking on slopes at different speeds*, PLOS ONE, 12(10):e0186963, 2017.

DOI: `10.1371/journal.pone.0186963`

### Dewolf et al. 2018

A. H. Dewolf, Y. Ivanenko, K. E. Zelik, F. Lacquaniti, P. A. Willems, *Kinematic patterns while walking on a slope at different speeds*, Journal of Applied Physiology, 125(2):642–653, 2018.

DOI: `10.1152/japplphysiol.01020.2017`

### Voloshina et al. 2013

A. S. Voloshina, A. D. Kuo, M. A. Daley, D. P. Ferris, *Biomechanics and energetics of walking on uneven terrain*, Journal of Experimental Biology, 216(21):3963–3970, 2013.

DOI: `10.1242/jeb.081711`

### Leestma et al. 2023

J. K. Leestma, P. R. Golyski, C. R. Smith, G. S. Sawicki, A. J. Young, *Linking whole-body angular momentum and step placement during perturbed human walking*, Journal of Experimental Biology, 226(6):jeb244760, 2023.

DOI: `10.1242/jeb.244760`
