# 06 — Formal running protocol

## 0. Goal of this document

The goal of this document is to define the mathematical protocol of running in a form compatible with the same `CM-first` architecture used for walking.

This document fixes:

- the hybrid mode set of running,
- the authoritative running state,
- the effective running stance geometry,
- the spring-like stance law,
- the flight phase,
- takeoff,
- touchdown after flight,
- the role of reactivity in running,
- the precise sense in which run reuses the same global architecture as walk.

This document does **not** yet formalize:

- walk-to-run or run-to-walk transitions,
- jump as an intentional transition into flight,
- landing as a high-energy capture protocol.

Those belong to a later document.

## 1. Why running is not accelerated walking

Walking and running are not the same phenomenon at different speeds.

In nominal walking:

- the stance leg behaves approximately like an inverted pendulum,
- the center of mass reaches a local height maximum around mid-stance,
- the support transition is organized around a finite transfer phase.

In nominal running:

- the stance leg behaves like an effective compressive spring-mass system,
- the center of mass reaches a lower configuration around mid-stance,
- a flight phase appears between successive stances.

Therefore the running protocol must differ from the walking protocol in at least three essential ways.

1. The stance law cannot be the same.
2. Flight must exist explicitly.
3. The stance transition is no longer a standard walking double-support transfer.

The same authoritative CM $C$ remains the primary dynamic object, but the contact protocol changes.

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

More refined variants may later split stance into subphases, but this is the correct first clean hybrid model.

## 3. Running state

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
- the current stance ankle $A_{\mathrm{st}}$ when stance exists,
- the current swing ankle $A_{\mathrm{sw}}$,
- the next ankle target $A_{\mathrm{next}}$,
- the effective support point $Q$ when stance exists,
- the effective support tangent $t_{\mathrm{sup}}$ when stance exists,
- the effective support normal $n_{\mathrm{sup}}$ when stance exists,
- the rocker scalar $\sigma$ when stance exists,
- the local time in the current mode $\tau_{\mathrm{mode}}$,
- the reactive intensity $\rho$,
- the current mode $\mathcal{M}$.

A minimal abstract running state is therefore

$$
X_{\mathrm{run}}
=
(C, V, \theta, A_{\mathrm{st}}, A_{\mathrm{sw}}, A_{\mathrm{next}}, Q, t_{\mathrm{sup}}, n_{\mathrm{sup}}, \sigma, \tau_{\mathrm{mode}}, \rho, \mathcal{M}).
$$

In flight, the support variables are inactive or only predicted. In stance, they are fully defined.

## 4. Exact rigid geometry remains unchanged

The body morphology remains exactly the same as in walking.

The leg segments are rigid:

$$
\|A_i-K_i\| = L,
$$

$$
\|K_i-P\| = L.
$$

Therefore the pelvis-to-ankle distance still satisfies

$$
\|P-A_i\| \le 2L.
$$

No bone stretching is allowed.

The difference between walking and running is therefore **not** morphological.

It is dynamical and hybrid.

## 5. Effective running stance length

During running stance, define the effective stance length by

$$
\ell = \|P-A_{\mathrm{st}}\|.
$$

Unlike nominal walking, where the stance leg should remain close to a nearly extended value, nominal running allows $\ell$ to vary substantially during the stance phase.

Define a running reference stance length

$$
\ell_r^{\star} < 2L.
$$

This is a reference quantity, not a hard geometric constraint.

It expresses the fact that nominal running stance is more compressed than nominal walking stance.

## 6. Local support frame in running stance

When the body is in running stance, define the effective support point and support frame exactly as in walking:

$$
Q = A_{\mathrm{st}} + \sigma t_{\mathrm{sup}},
$$

with

$$
\sigma \in [0, L_{\mathrm{toe}}].
$$

The CM decomposes relative to the support point as

$$
C = Q + s_Q t_{\mathrm{sup}} + z_Q n_{\mathrm{sup}}.
$$

Likewise, relative to the stance ankle,

$$
C = A_{\mathrm{st}} + s_A t_{\mathrm{sup}} + z_A n_{\mathrm{sup}}.
$$

Thus

$$
s_Q = s_A - \sigma,
$$

$$
z_Q = z_A.
$$

This is an important architectural continuity point: the same local support geometry is retained across walking and running.

## 7. Running stance dynamics

The global CM equation remains

$$
\ddot C = g_W + u.
$$

For running stance, the virtual control is decomposed as

$$
u = u_{\mathrm{spring}} + u_{\mathrm{tan}} + u_{\mathrm{aux}}.
$$

The term $u_{\mathrm{aux}}$ groups supplementary regularization terms that are not part of the core running stance law.

### 7.1. Effective spring-like stance term

Let

$$
r = P - A_{\mathrm{st}},
$$

and

$$
\hat r = \frac{r}{\|r\|}.
$$

Define the running stance compression by

$$
\delta_r = \ell_r^{\star} - \ell,
$$

where

$$
\ell = \|P-A_{\mathrm{st}}\|.
$$

Only positive compression should generate spring loading. Therefore define

$$
(\delta_r)_+ = \max(\delta_r, 0).
$$

A minimal effective spring-damper stance law is

$$
u_{\mathrm{spring}} = k_r (\delta_r)_+ \hat r - c_r \dot \ell \, \hat r.
$$

This means:

- if the effective leg is compressed below its reference length, the stance generates a restoring push,
- if the effective leg is lengthening too quickly, the damping term reduces excessive rebound.

This is not a literal physical leg spring. It is a virtual stance law compatible with a rigid two-link leg whose effective length changes through posture and knee flexion.

### 7.2. Tangential running control

Define the tangential stance velocity

$$
\dot s_A = \dot C \cdot t_{\mathrm{sup}}.
$$

Then a minimal tangential running control is

$$
u_{\mathrm{tan}} = k_t (v_t^{\star} - \dot s_A)t_{\mathrm{sup}}.
$$

This regulates running speed along the current support tangent.

## 8. Mid-stance compression criterion

A nominal running stance should have a compressed interior phase.

A formal running-like stance criterion is that there exists some interior time $t_m$ such that

$$
\dot \ell(t_m) = 0,
$$

and

$$
\ddot \ell(t_m) > 0.
$$

Thus $\ell$ has a strict local minimum during stance.

Equivalently, when the support frame remains well defined, the support-normal CM height should satisfy

$$
\dot z_Q(t_m) = 0,
$$

and

$$
\ddot z_Q(t_m) > 0.
$$

This expresses formally that running has a lowered mid-stance configuration, in contrast with the walking apex.

## 9. Rocker progression in running stance

The support-foot rocker scalar $\sigma$ remains meaningful in running stance because forefoot progression still matters for takeoff.

A first consistent choice is to reuse the same formal rocker structure as in walking.

Define a running-stance phase scalar

$$
\eta_{\mathrm{run}} \in [0,1].
$$

Let $\gamma_{\sigma}^{\mathrm{run}}$ be a smooth monotone map on $[0,1]$ such that

$$
\gamma_{\sigma}^{\mathrm{run}}(0)=0,
\qquad
\gamma_{\sigma}^{\mathrm{run}}(1)=1.
$$

Then define

$$
\sigma = L_{\mathrm{toe}}\,\gamma_{\sigma}^{\mathrm{run}}(\eta_{\mathrm{run}}).
$$

The exact choice of $\eta_{\mathrm{run}}$ may later be tied to stance compression and fore-aft support progression.

The important point is that $\sigma$ remains an explicit state variable in running too.

## 10. Takeoff condition

Running differs from walking because stance ends in flight, not in a standard walking transfer phase.

Therefore takeoff must be an explicit mode transition.

A representative formal trigger is

$$
RS \to FLIGHT
$$

when all the following hold:

$$
\ell \ge \ell_{\mathrm{takeoff}},
$$

$$
\dot \ell > 0,
$$

$$
\sigma \ge \sigma_{\mathrm{takeoff}}.
$$

This means:

- the effective leg is re-extending,
- support has progressed sufficiently toward the forefoot,
- stance is ready to release contact.

## 11. Flight phase

### 11.1. Flight dynamics

During flight, no foot is in contact with the terrain.

Therefore the simplest formal flight equation is ballistic:

$$
\ddot C = g_W.
$$

### 11.2. Support variables in flight

During flight:

- there is no active support point,
- there is no active support tangent or normal,
- the previous takeoff support may still be stored for continuity,
- the predicted touchdown support may be used by reconstruction and planning.

This is why the clean reconstruction document introduced a reconstruction-forward tangent separate from the active support tangent.

## 12. Touchdown after flight

The flight phase ends when one foot makes a valid contact that can initiate a new running stance.

A touchdown is valid only if all the following hold.

1. The swing ankle reaches the planned touchdown location up to tolerance:

$$
\|A_{\mathrm{sw}}(t_{\mathrm{td}}) - A_{\mathrm{next}}^{\star}\| \le \varepsilon_A.
$$

2. The future foot geometry is admissible on the terrain.

3. The new stance leg is rigidly reconstructible:

$$
\|P_{\mathrm{td}} - A_{\mathrm{next}}^{\star}\| \le 2L.
$$

4. The new support frame is well defined.

If these conditions hold, then

$$
FLIGHT \to RS_{\mathrm{next}}.
$$

## 13. Reactivity in running

The same reactive intensity variable remains active:

$$
\rho \in [0,1].
$$

It should continuously deform at least:

- the allowed foothold horizon,
- the preferred step length,
- the swing clearance,
- the effective running stance reference,
- touchdown tolerance or timing margins.

A first formal reactive compression law is

$$
\ell_r^{\star}(\rho) = \ell_{r,0}^{\star} - \rho\,\Delta \ell_r^{\star}.
$$

Thus strong perturbations allow a more crouched running stance.

Likewise, the planning horizon may grow as

$$
\sigma_{\max}^{\mathrm{plan}}(\rho)
=
\sigma_{\max,0}^{\mathrm{plan}} + \rho\,\Delta \sigma_{\max}^{\mathrm{plan}}.
$$

So recovery in running can emerge from the same architecture by allowing farther and larger steps.

## 14. Exact rigid feasibility remains non-negotiable

Even in running, rigid-leg reconstruction remains exact.

During stance the active stance leg must satisfy

$$
\|P-A_{\mathrm{st}}\| \le 2L.
$$

At touchdown the new stance leg must satisfy

$$
\|P_{\mathrm{td}}-A_{\mathrm{next}}^{\star}\| \le 2L.
$$

Thus running may change stance dynamics, but it never changes morphology.

## 15. Continuity with the same `CM-first` architecture

The following objects remain unchanged across walking and running:

- the authoritative CM $C$,
- the rigid body geometry,
- the exact leg IK,
- the terrain geometry,
- the ankle-based foot model,
- the support-point geometry,
- the swing-generation architecture,
- the body reconstruction architecture,
- the reactive intensity variable $\rho$.

What changes is:

- the hybrid mode set,
- the stance law,
- the presence of flight,
- the event structure of takeoff and touchdown.

This is exactly the intended benefit of the `CM-first` design: stable global geometry, changing gait protocol.

## 16. Main conclusions of this document

1. Running must be a distinct hybrid protocol from walking.
2. The same `CM-first` architecture can be preserved across both gaits.
3. Running stance must be modeled by an effective spring-like compressed-leg law.
4. Flight must exist explicitly and obey ballistic dynamics.
5. Takeoff and touchdown are explicit mode transitions.
6. The same geometric support objects remain meaningful in running stance.
7. The same reactive intensity variable $\rho$ can organize running recovery as well.
8. Exact rigid-leg feasibility remains non-negotiable.

## 17. Why this file can enter `docs_clean/`

This document is stable enough for the clean line because:

- its notation is coherent with the current clean documents,
- it no longer relies on undefined symbols like an unexplained torso-force term,
- it makes explicit what is fixed and what is deferred,
- it preserves the same architectural backbone as the other clean documents.

What is still intentionally deferred is not an inconsistency. It is the future work on:

- gait transitions,
- jump,
- landing,
- higher-energy touchdown capture.
