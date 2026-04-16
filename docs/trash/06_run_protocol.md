# 06 — Formal running protocol

## 0. Goal of this document

The goal of this document is to define the mathematical protocol of running.

The previous documents fixed:

- rigid body geometry,
- segmented terrain geometry,
- exact leg IK,
- the formal walking protocol,
- the foothold planner,
- the swing generator,
- the exact body reconstruction.

This document now defines:

- the hybrid modes of running,
- the structural difference between walking and running,
- the effective running stance model,
- the flight phase,
- touchdown after flight,
- the role of reactivity during running,
- the continuity of the `CM-first` architecture across gait regimes.

This document does not yet define jumping or landing as separate high-level protocols. It defines the running regime from which jump and land will later emerge naturally.

## 1. Why running cannot be modeled as accelerated walking

Walking and running are not the same phenomenon at different speeds.

In walking, the nominal stance leg behaves approximately like an inverted pendulum. The CM tends to pass above the support and reaches a local height maximum around mid-stance.

In running, the body behaves approximately like a spring-mass system. The effective leg compresses during stance, the CM tends to reach a local height minimum around mid-stance, and a flight phase appears between successive contacts.

Therefore the formal protocol of running must differ from the formal protocol of walking in at least three essential ways.

1. The stance geometry is no longer a nearly fixed pendular radius.
2. A flight phase must exist explicitly.
3. The support transition is no longer a standard double-support walking transfer.

The same virtual CM $C$ remains authoritative, but the contact protocol changes.

## 2. Running mode set

The minimal running mode set is

$$
\mathcal{M}_{\mathrm{run}} = \{RS_L,\; FLIGHT,\; RS_R\}.
$$

The meanings are:

- $RS_L$: left running stance,
- $FLIGHT$: no foot in contact,
- $RS_R$: right running stance.

This is the minimal alternating running automaton.

At this stage, no explicit aerial asymmetry or special submodes are introduced. Those can be added later if needed.

## 3. Authoritative running state

The authoritative dynamic object remains the virtual center of mass

$$
C(t) \in \mathbb{R}^2.
$$

Its derivatives are

$$
V(t) = \dot C(t), \qquad A(t) = \ddot C(t).
$$

The running state must also contain:

- the torso angle $\theta$,
- the active stance ankle when stance exists,
- the swing ankle,
- the next planned foothold,
- the support frame when stance exists,
- the current mode,
- the current reactive intensity.

A minimal abstract running state is therefore

$$
X_{\mathrm{run}} = (C, V, \theta, A_{\mathrm{st}}, A_{\mathrm{sw}}, A_{\mathrm{next}}, Q, t_{\mathrm{sup}}, n_{\mathrm{sup}}, \tau_{\mathrm{mode}}, \rho, \mathcal{M}).
$$

In flight, some of these objects are inactive or only predicted. In stance, they are fully defined.

## 4. Effective running leg length

### 4.1. Exact rigid geometry remains unchanged

The leg bones remain rigid:

$$
\|A_i-K_i\| = L,
$$

$$
\|K_i-P\| = L.
$$

So the exact pelvis-to-ankle distance still satisfies

$$
\|P-A_i\| \le 2L.
$$

No bone stretching is allowed.

### 4.2. Effective leg length in running

Running is modeled through an effective compressive stance behavior.

Define the running stance length by

$$
\ell = \|P-A_{\mathrm{st}}\|.
$$

Unlike nominal walking, where the stance leg should remain close to a nearly extended value, nominal running stance allows $\ell$ to vary significantly during the stance phase.

### 4.3. Running stance reference length

Define a running reference stance length

$$
\ell_r^\star < 2L.
$$

This is not a hard constraint. It is a reference around which the effective spring-like stance dynamics are organized.

The inequality

$$
\ell_r^\star < 2L
$$

expresses the fact that nominal running uses a more compressed effective stance than nominal walking.

## 5. Running stance support frame

When the body is in running stance, one foot is in contact with the ground.

The support frame is defined exactly as in walking:

$$
Q = A_{\mathrm{st}} + \sigma t_{\mathrm{sup}},
$$

with

$$
\sigma \in [0, L_{\mathrm{toe}}].
$$

The support-normal decomposition of the CM is therefore

$$
C = Q + s_Q t_{\mathrm{sup}} + z_Q n_{\mathrm{sup}}.
$$

Likewise, relative to the stance ankle,

$$
C = A_{\mathrm{st}} + s_A t_{\mathrm{sup}} + z_A n_{\mathrm{sup}}.
$$

The relation remains

$$
s_Q = s_A - \sigma,
$$

$$
z_Q = z_A.
$$

So the same local support geometry is retained across walking and running. This is one of the main benefits of the `CM-first` architecture.

## 6. Dynamic equation in running stance

The global CM equation remains

$$
\ddot C = g_W + u.
$$

But the meaning of the virtual control changes.

For running stance, the natural decomposition is

$$
u = u_{\mathrm{spring}} + u_{\mathrm{tan}} + u_{\mathrm{torso}}.
$$

### 6.1. Effective spring-like stance term

Let

$$
r = P - A_{\mathrm{st}},
$$

and let

$$
\hat r = \frac{r}{\|r\|}.
$$

Define the running stance compression by

$$
\delta_r = \ell_r^\star - \ell,
$$

where

$$
\ell = \|P-A_{\mathrm{st}}\|.
$$

Only positive compression should generate spring loading. Therefore define

$$
(\delta_r)_+ = \max(\delta_r, 0).
$$

A minimal effective spring-damper running stance term is then

$$
u_{\mathrm{spring}} = k_r (\delta_r)_+ \hat r - c_r \dot \ell \, \hat r.
$$

This formula has the intended interpretation.

- If the effective leg is compressed below its reference length, the stance generates a restoring push.
- If the effective leg is lengthening quickly, the damping term reduces excessive rebound.

This is not a literal physical leg spring. It is a virtual stance law compatible with a rigid two-link leg whose effective length changes through posture and knee flexion.

## 7. Tangential running control

The spring-like radial term is not enough.

The running controller still needs tangential regulation along the support direction.

Define the forward tangential speed by

$$
\dot s_A = \dot C \cdot t_{\mathrm{sup}}.
$$

Then a minimal tangential running control is

$$
u_{\mathrm{tan}} = k_t (v_t^\star - \dot s_A) t_{\mathrm{sup}}.
$$

This term regulates the running speed along the current support tangent.

Later, this term may be extended by more explicit slope or energy-control components, but its formal role is already clear: preserve the target forward dynamics during stance.

## 8. Running stance geometry and mid-stance minimum

### 8.1. Contrast with walking

In walking, the nominal CM path during single support has a local maximum.

In running, the nominal spring-like stance must instead produce a local minimum in the support-normal height of the CM around mid-stance.

This difference must appear formally.

### 8.2. Running stance length minimum

The support-normal stance geometry in running should satisfy the following qualitative property.

As the runner enters stance, the effective leg length decreases.

At some interior stance instant,

$$
\ell = \|P-A_{\mathrm{st}}\|
$$

reaches a local minimum.

Then the leg re-extends before takeoff.

Since the CM is attached to the pelvis by a fixed offset and the pelvis is attached to the stance ankle through the leg, a local minimum of $\ell$ implies a lower support-normal CM height.

Thus the running protocol should naturally produce a lower CM at mid-stance than at touchdown or takeoff.

This is the exact opposite of the walking apex geometry.

### 8.3. Formal criterion for running-like stance

A running stance trajectory should satisfy, at some interior time $t_m$,

$$
\dot \ell(t_m) = 0,
$$

and

$$
\ddot \ell(t_m) > 0.
$$

This expresses that $\ell$ has a strict local minimum at mid-stance.

Equivalently, one may formulate the same condition in support-normal CM height:

$$
\dot z_Q(t_m) = 0,
$$

$$
\ddot z_Q(t_m) > 0.
$$

provided the local support frame remains well defined.

## 9. Takeoff condition

### 9.1. Why takeoff must be explicit

Running differs from walking because the stance phase ends in flight, not in a classical finite double-support transfer.

Therefore takeoff must be modeled explicitly.

### 9.2. Geometric takeoff condition

A minimal geometric takeoff condition is that the support-loading state has transitioned into re-extension and that the support point has advanced sufficiently.

A formal example is:

$$
RS \to FLIGHT
$$

when

$$
\ell \ge \ell_{\mathrm{takeoff}},
$$

$$
\dot \ell > 0,
$$

and

$$
\sigma \ge \sigma_{\mathrm{takeoff}}.
$$

This means:

- the effective leg is re-extending,
- the support has advanced far enough toward the forefoot,
- the stance is ready to release contact.

This is a formal triggering condition, not yet a tuning law.

## 10. Flight phase

### 10.1. Flight dynamics

During flight, no foot is in contact with the ground.

Therefore the CM follows ballistic motion:

$$
\ddot C = g_W.
$$

This is the simplest and correct formal flight equation.

### 10.2. Flight state variables

During flight:

- there is no active support point,
- there is no active support tangent or normal,
- the stance foot of the previous phase becomes the trailing swing foot,
- the other foot becomes the leading swing foot that will prepare touchdown.

The body reconstruction still uses the same rigid rules, but there is no current support frame coming from active contact. Instead, one may use a predicted touchdown frame or a torso-based continuity frame for body reconstruction during aerial motion.

## 11. Touchdown after flight

### 11.1. Need for a touchdown event

The flight phase ends when one foot makes valid contact with the terrain and can enter a new running stance.

This touchdown event is the running analogue of the end of swing in walking, but it directly creates a new single stance rather than a standard double-support transfer.

### 11.2. Touchdown conditions

A touchdown candidate is valid if all the following hold.

1. The ankle reaches the planned touchdown location up to tolerance:

$$
\|A_{\mathrm{sw}}(t_{\mathrm{td}}) - A_{\mathrm{next}}^\star\| \le \varepsilon_A.
$$

2. The future foot geometry is admissible on the terrain.

3. The new stance leg is rigidly reconstructible:

$$
\|P_{\mathrm{td}} - A_{\mathrm{next}}^\star\| \le 2L.
$$

4. The post-touchdown support frame is well defined.

If these conditions hold, the mode transition is

$$
FLIGHT \to RS_{\mathrm{next}}.
$$

## 12. Running foothold planning

The same foothold-planning architecture used for walking should be retained.

The planner still chooses the next stance ankle

$$
A_{\mathrm{next}}^\star = \arg\min_{A \in \mathcal{A}(\rho)} J(A; \rho).
$$

However, the cost terms will later differ quantitatively because:

- running allows longer steps,
- touchdown comes from flight rather than from a standard walking transfer,
- support capture is evaluated relative to a future running stance, not a walking pendular stance.

The architecture remains the same. Only the gait-specific geometry and timing differ.

## 13. Running reactivity

### 13.1. Why reactivity still matters in running

Running is not exempt from perturbation response.

The same reactive intensity variable

$$
\rho \in [0,1]
$$

must be retained across gait regimes.

It should control at least:

- the allowed foothold horizon,
- the allowed step length,
- the swing clearance,
- the stance compression reference,
- the touchdown timing tolerance.

### 13.2. Reactive compression reference

A first formal running reactivity law is

$$
\ell_r^\star(\rho) = \ell_{r,0}^\star - \rho\,\Delta \ell_r^\star.
$$

Thus stronger perturbations allow a more crouched running stance by lowering the effective spring reference length.

### 13.3. Reactive running speed and horizon

The same planning horizon deformation idea can be reused:

$$
\sigma_{\max}^{\mathrm{plan}}(\rho) = \sigma_{\max,0}^{\mathrm{plan}} + \rho\,\Delta \sigma_{\max}^{\mathrm{plan}}.
$$

So running recovery can also emerge from the same architecture by allowing larger and farther steps.

## 14. Continuity with the walking architecture

One of the main goals of this whole formal development is to avoid building separate incompatible controllers for each locomotion regime.

The running protocol shows that this is possible.

The following objects remain unchanged across walk and run:

- the authoritative CM $C$,
- the rigid body geometry,
- the exact leg IK,
- the terrain geometry,
- the ankle-based foot model,
- the foothold planning architecture,
- the swing-generator architecture,
- the reconstruction architecture,
- the reactive intensity variable $\rho$.

Only the following change:

- the hybrid mode set,
- the stance dynamics,
- the existence of flight,
- the takeoff and touchdown events.

This is exactly what a good `CM-first` architecture should do: keep the geometry and state organization stable while changing the gait protocol.

## 15. Formal running protocol summary

The formal running protocol is now defined by the following objects.

### Modes

$$
\mathcal{M}_{\mathrm{run}} = \{RS_L,\; FLIGHT,\; RS_R\}.
$$

### State

$$
X_{\mathrm{run}} = (C, V, \theta, A_{\mathrm{st}}, A_{\mathrm{sw}}, A_{\mathrm{next}}, Q, t_{\mathrm{sup}}, n_{\mathrm{sup}}, \tau_{\mathrm{mode}}, \rho, \mathcal{M}).
$$

### Effective running stance length

$$
\ell = \|P-A_{\mathrm{st}}\|.
$$

### Running stance reference

$$
\ell_r^\star < 2L.
$$

### Running stance control

$$
u = u_{\mathrm{spring}} + u_{\mathrm{tan}} + u_{\mathrm{torso}}.
$$

with

$$
u_{\mathrm{spring}} = k_r (\delta_r)_+ \hat r - c_r \dot \ell \, \hat r,
$$

$$
\delta_r = \ell_r^\star - \ell,
$$

and

$$
u_{\mathrm{tan}} = k_t (v_t^\star - \dot s_A) t_{\mathrm{sup}}.
$$

### Flight dynamics

$$
\ddot C = g_W.
$$

### Takeoff condition

A representative formal trigger is

$$
\ell \ge \ell_{\mathrm{takeoff}}, \qquad \dot \ell > 0, \qquad \sigma \ge \sigma_{\mathrm{takeoff}}.
$$

### Touchdown condition

A representative formal trigger is

$$
\|A_{\mathrm{sw}}(t_{\mathrm{td}}) - A_{\mathrm{next}}^\star\| \le \varepsilon_A,
$$

with valid foot admissibility and exact rigid-leg reachability.

## 16. Main conclusions of this document

1. Running must be a distinct hybrid protocol from walking.
2. The same `CM-first` geometry can be preserved across both gaits.
3. Running stance must be modeled by an effective spring-like compressed leg law.
4. Flight must exist explicitly and obey ballistic dynamics.
5. Takeoff and touchdown are explicit mode transitions, not hidden heuristics.
6. The same foothold-planning and swing-generation architecture can be reused.
7. The same reactive intensity variable $\rho$ can organize recovery in running as well.

## 17. What comes next

The next document should formalize gait transitions and the introduction of jump and landing.

In particular, it should specify:

- walk-to-run transition,
- run-to-walk transition,
- jump as a stance phase whose terminal state intentionally enters flight,
- landing as a high-energy contact capture problem,
- the continuity of reconstruction and foothold logic across all these regimes.