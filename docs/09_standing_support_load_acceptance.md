In the `CM-first` procedural architecture, standing cannot be treated as walking with zero speed. The correct object to stabilize is still the authoritative center of mass $C$, but now under a bilateral support regime in which both feet remain in contact with the terrain and no nominal unipodal standing is allowed. The standing protocol must therefore be explicit. Its role is to maintain a viable support configuration while the terrain slope may vary under one foot or under both feet, and to trigger a step before bilateral support becomes non-viable. This makes standing the natural entry state for walking and the natural return state after walking slowdown.

The visible foot model must be adjusted slightly. The ankle remains the visual rear reference of the foot, so the heel length must stay small, but it should no longer be zero. The correct compromise is to introduce a very small positive heel length

$$
L_{\mathrm{heel}} = \varepsilon_h L,
$$

with

$$
0 < \varepsilon_h \ll 1.
$$

A practical initial choice is

$$
\varepsilon_h = 0.08.
$$

This value is not claimed to be anatomically exact. Its role is to give the support model a finite rear margin while preserving the current visual interpretation in which the ankle appears almost at the heel. The active support interval of one foot therefore becomes

$$
[-L_{\mathrm{heel}}, L_{\mathrm{toe}}].
$$

The key modeling choice is that the effective support point must not be constructed by a purely geometric average in world coordinates. For each active foot $i$, the support point must be defined from a local contact path parameterization

$$
\chi_i(\sigma), \qquad \sigma \in [-L_{\mathrm{heel}}, L_{\mathrm{toe}}],
$$

such that

$$
\chi_i(0) = A_i.
$$

When the foot lies on one terrain segment, $\chi_i$ is simply the corresponding line segment along the local terrain tangent. When the foot spans two admissible terrain segments, $\chi_i$ is the terrain-following path under the foot, with one branch behind the ankle of length $L_{\mathrm{heel}}$ and one branch forward of length $L_{\mathrm{toe}}$. The effective support coordinate of foot $i$ is then a scalar

$$
\sigma_i \in [-L_{\mathrm{heel}}, L_{\mathrm{toe}}],
$$

and the support point is defined by

$$
Q_i = \chi_i(\sigma_i).
$$

The corresponding support tangent is the path tangent at the active support coordinate,

$$
t_i = \frac{\partial_{\sigma} \chi_i(\sigma_i)}{\|\partial_{\sigma} \chi_i(\sigma_i)\|},
$$

and the support normal is

$$
n_i = (-t_{i,y}, t_{i,x}).
$$

This is the correct `CM-first` interpretation of support progression. The scalar $\sigma_i$ is primary, and $Q_i$ is derived from it. This makes support progression continuous over hills and across admissible terrain vertices, because support evolves along the actual foot-terrain contact path instead of being reconstructed afterward by a world-space average.

In all bilateral-support phases, including quiet standing and walking double support, the allocation of support between the left and right feet is described by a single scalar

$$
\beta \in [0,1].
$$

The correct interpretation is now broader than before. The value

$$
\beta = 0
$$

means full support dominance of the left or old support foot, and

$$
\beta = 1
$$

means full support dominance of the right or new support foot. Intermediate values represent partial bilateral support. This variable should not be restricted to a short transition-only meaning. It is the canonical support-allocation variable whenever both feet are simultaneously active. In a standing state, $\beta$ remains in the interior of the interval and expresses how the effective support is distributed between both feet. In a load-acceptance state after touchdown, $\beta$ evolves from old-foot dominance toward new-foot dominance.

Let the active bilateral support points be $Q_L$ and $Q_R$, with corresponding tangents $t_L$ and $t_R$. The effective bilateral support point is

$$
Q = (1-\beta) Q_L + \beta Q_R.
$$

The effective bilateral support tangent is

$$
t_{\mathrm{sup}} = \frac{(1-\beta) t_L + \beta t_R}{\|(1-\beta) t_L + \beta t_R\|},
$$

provided the denominator is nonzero. The associated support normal is

$$
n_{\mathrm{sup}} = (-t_{{\mathrm{sup}},y}, t_{{\mathrm{sup}},x}).
$$

Thus a single support frame still exists in bilateral phases, but it is now derived from both feet and from the current support-allocation state. This preserves architectural continuity with the rest of the `CM-first` model, where the CM is always decomposed in an effective support frame.

The standing mode is therefore a bilateral-support mode, denoted conceptually by

$$
STAND_{LR}.
$$

Its authoritative state contains at least

$$
(C, \dot C, \theta, A_L, A_R, \sigma_L, \sigma_R, \beta, Q_L, Q_R, t_L, t_R, \rho).
$$

The authoritative CM is decomposed in the bilateral support frame as

$$
C = Q + s_Q t_{\mathrm{sup}} + z_Q n_{\mathrm{sup}}.
$$

The natural standing objective is not to freeze all motion, but to stabilize the support-local coordinates of the CM near a nominal standing configuration

$$
s_Q^{\star}, \qquad z_Q^{\star}.
$$

A minimal canonical standing control law is therefore of the form

$$
\ddot C = g_W + u_{\mathrm{stand}},
$$

with

$$
u_{\mathrm{stand}} = k_s (s_Q^{\star} - s_Q) t_{\mathrm{sup}} - c_s \dot s_Q t_{\mathrm{sup}} + k_z (z_Q^{\star} - z_Q) n_{\mathrm{sup}} - c_z \dot z_Q n_{\mathrm{sup}} + u_{\theta}.
$$

The term $u_{\theta}$ denotes the additional contribution induced by the torso law and by the exact reconstruction constraints. This expression should not be read as a final actuator-level law. Its meaning is that standing is governed by support-local regulation of the CM relative to the bilateral support frame.

The support-allocation scalar $\beta$ must be state-based in standing. Define the bilateral support axis by

$$
e_{LR} = \frac{Q_R - Q_L}{\|Q_R - Q_L\|},
$$

assuming the two support points are distinct. Define the pairwise support coordinate of the CM by

$$
u = (C - Q_L) \cdot e_{LR}.
$$

Let

$$
D = \|Q_R - Q_L\|.
$$

Then the raw bilateral support-allocation target is

$$
\beta_{\mathrm{raw}} = \mathrm{clamp}\left(\frac{\nu}{D}, 0, 1\right).
$$

This quantity expresses where the authoritative CM lies relative to the segment joining the two active support points. To avoid jitter, the propagated support-allocation variable is filtered by

$$
\dot \beta = \frac{\beta_{\mathrm{raw}} - \beta}{\tau_{\beta}},
$$

with

$$
\tau_{\beta} > 0.
$$

In quiet bilateral standing on regular terrain, the nominal value should remain near

$$
\beta \approx \frac{1}{2},
$$

but the model should allow moderate deviations induced by slope adaptation or by perturbations. The bilateral standing state becomes non-nominal when $\beta$ is driven too close to the boundaries $0$ or $1$. Because nominal unipodal standing is forbidden, the correct response to persistent boundary approach is not to continue into one-foot standing, but to trigger a step.

The trigger from standing to stepping must therefore be viability-based. A step must be triggered when bilateral support ceases to be a good support solution. This happens if at least one of the following occurs: the bilateral CM margin becomes too small, the filtered support-allocation variable approaches one boundary too persistently, the required support-local correction grows too large, or the reactive intensity rises above a standing threshold. This gives the correct qualitative rule: standing should remain bilateral for as long as bilateral support is viable, and should transition to walking before the architecture is forced into an unintended one-foot stand.

The evolution of the foot support coordinates $\sigma_L$ and $\sigma_R$ must also be state-based. During standing they should not remain frozen by default, because slope adaptation and subtle load redistribution may shift the effective support under each foot. However, they should evolve more slowly than in walking stance. A canonical choice is to define, for each foot $i$, a raw support-progress target

$$
\sigma_{i,\mathrm{raw}} = \Sigma_i(X),
$$

where $\Sigma_i$ is a support-state functional depending on the local CM-support geometry and on the current contact configuration, and then filter it through

$$
\dot \sigma_i = \frac{\sigma_{i,\mathrm{raw}} - \sigma_i}{\tau_{\sigma}^{\mathrm{stand}}}.
$$

The exact form of $\Sigma_i$ should favor modest support adaptation inside the foot while preserving the hard bound

$$
-L_{\mathrm{heel}} \le \sigma_i \le L_{\mathrm{toe}}.
$$

This means that standing on a changing slope is not handled by freezing the feet and rotating the body visually. The support state itself adapts, and the body is reconstructed from that support-aware locomotion state.

The touchdown of a swing foot must not immediately replace the old support by the new one. The correct sequence is contact acquisition, then load acceptance, then support dominance transfer. Suppose the old dominant support foot is indexed by $b$ and the new contacting foot by $f$. At the touchdown instant, the system enters a bilateral-support acceptance phase with

$$
\beta^{+} = \beta_{\mathrm{init}}^{\mathrm{acc}},
$$

where

$$
0 \le \beta_{\mathrm{init}}^{\mathrm{acc}} \ll 1
$$

if the old foot is still dominant. The new foot already has a valid support geometry

$$
Q_f, \qquad t_f, \qquad n_f,
$$

but it does not yet carry full support responsibility.

The support-allocation target during load acceptance should be mixed time-state based. Define a normalized temporal acceptance phase

$$
\eta_t = \mathrm{clamp}\left(\frac{t - t_{\mathrm{td}}}{T_{\mathrm{acc}}}, 0, 1\right),
$$

and a normalized state-based acceptance coordinate

$$
\eta_s = \mathrm{clamp}\left(\frac{(C - Q_b) \cdot e_{bf}}{(Q_f - Q_b) \cdot e_{bf}}, 0, 1\right),
$$

where

$$
e_{bf} = \frac{Q_f - Q_b}{\|Q_f - Q_b\|}.
$$

Then the raw load-acceptance target is

$$
\beta_{\mathrm{raw}}^{\mathrm{acc}} = \mathrm{clamp}\left((1-w_{\beta}) \eta_t + w_{\beta} \eta_s, 0, 1\right),
$$

with

$$
0 < w_{\beta} < 1.
$$

The propagated support-allocation variable is again filtered through

$$
\dot \beta = k_{\beta} \max\{0, \beta_{\mathrm{raw}}^{\mathrm{acc}} - \beta\}.
$$

This law makes the transfer monotone and prevents premature support dominance inversion. The new support cannot become dominant merely because contact was detected, and the old support cannot remain dominant forever after the CM has already advanced into the new support region.

A touchdown is valid only if all the following hold. The swing foot reaches a geometrically admissible contact pose on the terrain. The new foot support path exists with a valid small heel and forward toe interval. The full foot is admissible over the terrain segment or over two admissible consecutive segments. The future support objects $Q_f, t_f, n_f$ are defined. The rigid morphology remains viable during the acceptance phase, which means that both pelvis-to-ankle distances remain within their exact bounds while support is still bilateral. If these conditions fail, touchdown cannot be considered a valid support-acquisition event.

This makes the correct logic of the chain explicit. The planner does not simply place an ankle. It proposes a candidate support acquisition. The swing generator does not simply end on that ankle. It realizes a collision-free foot transfer toward a touchdown pose. Touchdown does not simply activate a new support. It starts a bilateral support-acceptance phase. Only after load acceptance has progressed sufficiently does the new foot become dominant support.

For the present project, the primary terrain target should remain piecewise-linear hills and slopes with variable segment lengths and variable angles below ninety degrees. This support model is already rich enough to capture natural uphill and downhill walking. Stairs need not be a design constraint at this stage. If they later emerge from the same contact-path and load-acceptance logic with only minor additional conditions, they may be accepted, but they should not dictate the core architecture now.

This working formulation gives the correct direction for the procedural locomotion core. Standing is explicit and bilateral. Each foot has a finite but visually small rear margin. Effective support progresses along the actual contact path under the foot. Bilateral support is governed by a single support-allocation scalar $\beta$ that remains meaningful both in quiet standing and in walking double support. Touchdown and support dominance are separated by a finite acceptance phase. The same support-centered state architecture is preserved, so later running, crouching, and jumping can be introduced by changing the contact protocol and stance law rather than rebuilding the entire model.