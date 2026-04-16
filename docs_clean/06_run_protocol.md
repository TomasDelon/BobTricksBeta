# 06 — Formal running protocol

## 0. Goal of this document

The goal of this document is to define the mathematical protocol of running in a form compatible with the same support-centered `CM-first` architecture used by the current clean line.

This document fixes:

- the hybrid mode set of running,
- the authoritative running state,
- the local support geometry used during running stance,
- the spring-like stance law,
- the role of the support coordinate $\sigma$ during running stance,
- takeoff,
- flight,
- touchdown after flight,
- the role of reactivity in running,
- the precise sense in which run reuses the same architectural backbone as standing and walking.

This document does **not** yet formalize:

- walk-to-run or run-to-walk transitions,
- jump as an intentional transition into flight,
- landing as a high-energy capture protocol,
- multi-contact running variants.

Those belong to later documents.

## 1. Why running is not accelerated walking

Walking and running are not the same phenomenon at different speeds.

In nominal walking:

- the stance leg behaves approximately like an inverted pendulum,
- the center of mass reaches a local height maximum around mid-stance,
- support transition is organized around explicit bilateral transfer.

In nominal running:

- the stance leg behaves like an effective compressive spring-mass system,
- the center of mass reaches a lower configuration around mid-stance,
- a flight phase appears between successive stances.

Therefore the running protocol must differ from the walking protocol in at least three essential ways.

1. The stance law cannot be the same.
2. Flight must exist explicitly.
3. Running stance ends in takeoff rather than in the standard walking load-acceptance sequence.

What remains unchanged is the global architecture:

- one authoritative virtual center of mass,
- the same rigid morphology,
- the same support-centered foot geometry,
- the same prediction, planning, swing, and reconstruction backbone.

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

More refined later variants may split running stance into subphases, but the present file uses the minimal clean mode set.

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
- the next planned ankle target $A_{\mathrm{next}}$,
- the current stance support coordinate $\sigma$ when stance exists,
- the current stance support point $Q$ when stance exists,
- the current stance support tangent $t_{\mathrm{sup}}$ when stance exists,
- the current stance support normal $n_{\mathrm{sup}}$ when stance exists,
- the local time in the current mode $\tau_{\mathrm{mode}}$,
- the reactive intensity $\rho$,
- the current mode $\mathcal{M}$.

A minimal abstract running state is therefore

$$
X_{\mathrm{run}} = (C, V, \theta, A_{\mathrm{st}}, A_{\mathrm{sw}}, A_{\mathrm{next}}, \sigma, Q, t_{\mathrm{sup}}, n_{\mathrm{sup}}, \tau_{\mathrm{mode}}, \rho, \mathcal{M}).
$$

In flight, support variables are inactive or only predicted. In stance, they are fully defined.

## 4. Exact rigid geometry remains unchanged

The body morphology remains exactly the same as in standing and walking.

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

The difference between walking and running is therefore not morphological. It is dynamical and hybrid.

## 5. Effective running stance length

During running stance, define the effective stance length by

$$
\ell = \|P-A_{\mathrm{st}}\|.
$$

Unlike nominal walking, where the stance leg should remain close to a nearly extended value, nominal running allows $\ell$ to vary substantially during stance.

Define a running reference stance length by

$$
\ell_r^{\star} < 2L.
$$

This is a reference quantity, not a hard geometric constraint.

It expresses the fact that nominal running stance is more compressed than nominal walking stance.

## 6. Local support frame in running stance

### 6.1. Support-path-based running stance geometry

Running stance must use the same support-centered geometry as the current clean line.

Let the active running foot induce a contact path

$$
\chi_{\mathrm{st}}(\sigma), \qquad \sigma \in [-L_{\mathrm{heel}}, L_{\mathrm{toe}}],
$$

with

$$
\chi_{\mathrm{st}}(0) = A_{\mathrm{st}}.
$$

Then the effective running support point is

$$
Q = \chi_{\mathrm{st}}(\sigma).
$$

The stance support tangent is

$$
t_{\mathrm{sup}} = \frac{\partial_{\sigma} \chi_{\mathrm{st}}(\sigma)}{\|\partial_{\sigma} \chi_{\mathrm{st}}(\sigma)\|},
$$

and the stance support normal is

$$
n_{\mathrm{sup}} = (-t_{{\mathrm{sup}},y}, t_{{\mathrm{sup}},x}).
$$

Thus running does **not** revert to the old global linear rule

$$
Q = A_{\mathrm{st}} + \sigma t_{\mathrm{sup}}.
$$

The effective support point remains path-based.

### 6.2. Support-local and ankle-local coordinates

The CM decomposes in the running support frame as

$$
C = Q + s_Q t_{\mathrm{sup}} + z_Q n_{\mathrm{sup}}.
$$

Relative to the stance ankle,

$$
C = A_{\mathrm{st}} + s_A t_{\mathrm{sup}} + z_A n_{\mathrm{sup}}.
$$

Likewise the pelvis can be decomposed as

$$
P = A_{\mathrm{st}} + p_A t_{\mathrm{sup}} + q_A n_{\mathrm{sup}}.
$$

The quantity

$$
p_A = (P - A_{\mathrm{st}}) \cdot t_{\mathrm{sup}}
$$

is the natural stance-progression coordinate for the running support law.

Because the support point is defined by

$$
Q = \chi_{\mathrm{st}}(\sigma),
$$

there is no globally valid linear identity of the old form

$$
s_Q = s_A - \sigma
$$

on arbitrary support geometry. The ankle-local and support-local coordinates must remain distinct.

## 7. Running stance dynamics

The global CM equation remains

$$
\ddot C = g_W + u.
$$

For running stance, the virtual control is decomposed as

$$
u = u_{\mathrm{spring}} + u_{\mathrm{tan}} + u_{\mathrm{aux}}.
$$

The term $u_{\mathrm{aux}}$ groups supplementary regularization terms not belonging to the core running stance law.

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

This is not a literal leg spring. It is a virtual stance law compatible with rigid morphology.

### 7.2. Tangential running control

Define the tangential stance velocity by

$$
\dot s_A = \dot C \cdot t_{\mathrm{sup}}.
$$

Then a minimal tangential running control is

$$
u_{\mathrm{tan}} = k_t (v_t^{\star} - \dot s_A)t_{\mathrm{sup}}.
$$

This regulates forward progression along the active support tangent.

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

Equivalently, when the support frame remains well defined, the support-normal CM height may satisfy

$$
\dot z_Q(t_m) = 0,
$$

and

$$
\ddot z_Q(t_m) > 0.
$$

This expresses formally that running has a lowered mid-stance configuration, in contrast with the walking apex.

## 9. Support progression during running stance

The support coordinate $\sigma$ remains meaningful in running stance because forefoot progression still matters for takeoff.

However, running must not simply reuse the old walking linear support rule.

The canonical running support law should be state-based.

Define a running support phase scalar

$$
\eta_{\sigma}^{\mathrm{run}} \in [0,1]
$$

from a running-appropriate state quantity such as pelvis progression, stance compression-release progression, or a mixed running-support state.

Let

$$
\Gamma_{\sigma}^{\mathrm{run}} : [0,1] \to [-L_{\mathrm{heel}}, L_{\mathrm{toe}}]
$$

be a smooth monotone nondecreasing map.

Then the canonical running support law is

$$
\sigma = \Gamma_{\sigma}^{\mathrm{run}}\bigl(\eta_{\sigma}^{\mathrm{run}}\bigr).
$$

The running map should generally be more forefoot-biased than the walking one.

## 10. Takeoff condition

Running differs from walking because stance ends in flight, not in a standard walking load-acceptance transfer.

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

The precise threshold values may later be refined, but the architectural meaning is fixed here.

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
- the last takeoff support frame may still be stored for continuity,
- the predicted touchdown support frame may be used by reconstruction and planning.

This is one reason why the reconstruction document uses a reconstruction-forward tangent separate from the active support tangent.

## 12. Touchdown after flight

The flight phase ends when one foot makes a valid contact that can initiate a new running stance.

A touchdown is valid only if all the following hold.

1. The swing ankle reaches the planned touchdown location up to tolerance:

$$
\|A_{\mathrm{sw}}(t_{\mathrm{td}}) - A_{\mathrm{next}}\| \le \varepsilon_A.
$$

2. The future foot geometry is admissible on the terrain over the full interval

$$
[-L_{\mathrm{heel}}, L_{\mathrm{toe}}].
$$

3. The future contact path is well defined.

4. The new stance leg is rigidly reconstructible:

$$
\|P_{\mathrm{td}} - A_{\mathrm{next}}\| \le 2L.
$$

5. The new support frame is well defined.

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
- touchdown timing or viability margins.

A first formal reactive compression law is

$$
\ell_r^{\star}(\rho) = \ell_{r,0}^{\star} - \rho\,\Delta \ell_r^{\star}.
$$

Thus strong perturbations allow a more crouched running stance.

Likewise, the planning horizon may grow as

$$
\ell_{\max}^{\mathrm{plan}}(\rho) = \ell_{\max,0}^{\mathrm{plan}} + \rho\,\Delta \ell_{\max}^{\mathrm{plan}}.
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
\|P_{\mathrm{td}}-A_{\mathrm{next}}\| \le 2L.
$$

Thus running may change stance dynamics, but it never changes morphology.

## 15. Continuity with the same `CM-first` architecture

The following objects remain shared across standing, walking, and running:

- the authoritative CM $C$,
- the rigid body geometry,
- the exact leg IK,
- the terrain geometry,
- the ankle-based foot model,
- the support-path-based foot geometry,
- the swing-generation architecture,
- the body reconstruction architecture,
- the reactive intensity variable $\rho$.

What changes is:

- the hybrid mode set,
- the stance law,
- the role of bilateral support,
- the presence or absence of flight,
- the event structure of takeoff and touchdown.

This is exactly the intended benefit of the `CM-first` design: stable global geometry, changing gait protocol.

## 16. Main conclusions of this document

1. Running must be a distinct hybrid protocol from walking.

2. The same support-centered `CM-first` architecture is preserved across both gaits.

3. Running stance must use the same path-based support geometry as the rest of the clean line.

4. Running stance must be modeled by an effective spring-like compressed-leg law.

5. Flight must exist explicitly and obey ballistic dynamics.

6. Takeoff and touchdown are explicit mode transitions.

7. The same reactive intensity variable $\rho$ can organize running recovery.

8. Exact rigid-leg feasibility remains non-negotiable.