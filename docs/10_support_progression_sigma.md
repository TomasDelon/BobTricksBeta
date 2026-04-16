# 10 — Exact support progression law for the rocker scalar

In the `CM-first` locomotion architecture, the rocker scalar must not be treated as a decorative interpolation parameter. It is the primary support coordinate inside the active foot. The correct mathematical role of the scalar is to parameterize the progression of effective support along the actual foot-terrain contact path. The effective support point is then derived from that scalar and not the other way around.

Let one active foot have ankle point $A$, heel length $L_{\mathrm{heel}} > 0$, toe length $L_{\mathrm{toe}} > 0$, and contact path

$$
\chi(\sigma), \qquad \sigma \in [-L_{\mathrm{heel}}, L_{\mathrm{toe}}],
$$

with

$$
\chi(0) = A.
$$

The support point is defined by

$$
Q = \chi(\sigma),
$$

and the local support tangent is

$$
t_{\mathrm{sup}} = \frac{\partial_{\sigma} \chi(\sigma)}{\|\partial_{\sigma} \chi(\sigma)\|}.
$$

The support normal is

$$
n_{\mathrm{sup}} = (-t_{{\mathrm{sup}},y}, t_{{\mathrm{sup}},x}).
$$

The scalar

$$
\sigma \in [-L_{\mathrm{heel}}, L_{\mathrm{toe}}]
$$

is therefore the canonical support coordinate. When the foot lies on one terrain segment, $\chi$ is the straight terrain-aligned foot path. When the foot spans two admissible terrain segments, $\chi$ is the piecewise terrain-following path under the full foot. This formulation preserves continuity of support progression over hills and admissible vertices.

The support progression law must be state-based. A purely time-based law is not acceptable as the canonical law because it cannot correctly adapt to changes of speed, slope, local support geometry, or perturbation intensity. The canonical support progression law must instead be built from a phase variable that depends on the locomotion state and, when relevant, on the active gait protocol.

The natural local quantity for support progression must be referenced to the stance foot, not to the support point itself, because using $s_Q$ directly would introduce a circular dependence through $Q = \chi(\sigma)$. The correct primary tangential coordinate is therefore an ankle-local quantity. Let

$$
s_A = (C-A) \cdot t_{\mathrm{sup}}.
$$

Likewise define the pelvis-local tangential coordinate by

$$
p_A = (P-A) \cdot t_{\mathrm{sup}}.
$$

Since support progression is mechanically more closely associated with how the pelvis-body system advances over the stance foot than with the raw location of the support point itself, the canonical variable for rocker progression should be based on $p_A$, not on $s_Q$.

Define two protocol-dependent pelvis-reference thresholds

$$
p_{\mathrm{heel}} < p_{\mathrm{toe}}.
$$

These thresholds delimit the effective anterior progression interval of the pelvis over the active foot. The normalized support-progression phase is then

$$
\eta_{\sigma} = \mathrm{clamp}\left(\frac{p_A - p_{\mathrm{heel}}}{p_{\mathrm{toe}} - p_{\mathrm{heel}}}, 0, 1\right).
$$

The rocker scalar is defined from this phase by a monotone support map

$$
\sigma = \Gamma_{\sigma}(\eta_{\sigma}).
$$

The canonical support map must satisfy

$$
\Gamma_{\sigma}(0) = -L_{\mathrm{heel}},
$$

$$
\Gamma_{\sigma}(1) = L_{\mathrm{toe}},
$$

and it must be smooth and monotone nondecreasing. A convenient normalized representation is

$$
\Gamma_{\sigma}(\eta) = -L_{\mathrm{heel}} + (L_{\mathrm{heel}} + L_{\mathrm{toe}}) \gamma_{\sigma}(\eta),
$$

where

$$
\gamma_{\sigma} : [0,1] \to [0,1]
$$

is smooth, monotone, and satisfies

$$
\gamma_{\sigma}(0) = 0,
$$

$$
\gamma_{\sigma}(1) = 1.
$$

This formulation has the correct geometric meaning. When the pelvis remains behind the effective anterior progression interval, support remains close to the heel-side region of the active foot. As the pelvis advances over the active foot, the support point progresses continuously toward the forefoot. The support coordinate remains bounded by construction inside the active foot interval.

The differential form of the support law is obtained by differentiating the previous expression. Whenever the support phase is differentiable,

$$
\dot \sigma = \Gamma_{\sigma}'(\eta_{\sigma}) \dot \eta_{\sigma},
$$

with

$$
\dot \eta_{\sigma} = \frac{\dot p_A}{p_{\mathrm{toe}} - p_{\mathrm{heel}}}
$$

whenever $\eta_{\sigma}$ lies strictly inside $(0,1)$. Here

$$
\dot p_A = \dot P \cdot t_{\mathrm{sup}} + (P-A) \cdot \dot t_{\mathrm{sup}}.
$$

In early implementations, the term involving $\dot t_{\mathrm{sup}}$ may be neglected when the support tangent evolves slowly over one support phase. The canonical model, however, should keep the dependence conceptually explicit because the support tangent changes on non-flat terrain.

The previous law is still incomplete unless the thresholds $p_{\mathrm{heel}}$ and $p_{\mathrm{toe}}$ are fixed. The correct architectural choice is that they must be protocol-dependent and possibly slope-dependent, but not freely changing at every frame. They are support-law parameters, not uncontrolled observables.

For bilateral standing, support progression under one foot should remain modest, because standing is not a full heel-to-toe rollover protocol. The appropriate choice is therefore to use a narrow admissible support band around an internal preferred standing coordinate,

$$
\sigma_{\mathrm{stand}}^{\star}.
$$

For standing, the canonical law should not use the full rocker map from heel to toe. Instead define a raw standing support target

$$
\sigma_{\mathrm{raw}}^{\mathrm{stand}} = \Sigma_{\mathrm{stand}}(X),
$$

where $\Sigma_{\mathrm{stand}}$ is a bilateral-support state functional depending on the current standing support state, on the local slope, and on the bilateral load-allocation variable $\beta$. The propagated standing support law is then filtered as

$$
\dot \sigma = \frac{\sigma_{\mathrm{raw}}^{\mathrm{stand}} - \sigma}{\tau_{\sigma}^{\mathrm{stand}}},
$$

with saturation to the interval

$$
[-L_{\mathrm{heel}}, L_{\mathrm{toe}}].
$$

This makes standing support adaptive without forcing a full stance rollover.

For walking single support, by contrast, the full support progression from rear support toward forefoot is a canonical part of the gait. Therefore the pelvis-based support phase should be active. The thresholds should be chosen so that

$$
\eta_{\sigma} \approx 0
$$

shortly after stance acquisition and

$$
\eta_{\sigma} \approx 1
$$

near late stance or toe-off preparation. The simplest canonical choice is to define protocol parameters

$$
p_{\mathrm{heel}}^{\mathrm{walk}}, \qquad p_{\mathrm{toe}}^{\mathrm{walk}},
$$

with

$$
p_{\mathrm{heel}}^{\mathrm{walk}} < 0 < p_{\mathrm{toe}}^{\mathrm{walk}},
$$

and then set

$$
\eta_{\sigma}^{\mathrm{walk}} = \mathrm{clamp}\left(\frac{p_A - p_{\mathrm{heel}}^{\mathrm{walk}}}{p_{\mathrm{toe}}^{\mathrm{walk}} - p_{\mathrm{heel}}^{\mathrm{walk}}}, 0, 1\right).
$$

The walking rocker scalar is then

$$
\sigma^{\mathrm{walk}} = \Gamma_{\sigma}^{\mathrm{walk}}(\eta_{\sigma}^{\mathrm{walk}}).
$$

This law has the right qualitative behavior: early support remains rear-biased, mid-stance support lies in the interior of the foot, and late support moves toward the forefoot. The exact shape of $\Gamma_{\sigma}^{\mathrm{walk}}$ can later be tuned to obtain a more human-like pacing of support progression. The canonical requirement is only that it be monotone, bounded, and sufficiently smooth.

For running stance, the support progression law should not simply reuse the walking thresholds. Running stance is shorter, more compressed, and more forefoot-biased. The architecture should therefore define a distinct running support phase

$$
\eta_{\sigma}^{\mathrm{run}}
$$

and a distinct running support map

$$
\sigma^{\mathrm{run}} = \Gamma_{\sigma}^{\mathrm{run}}(\eta_{\sigma}^{\mathrm{run}}).
$$

The running map should be more anteriorly biased than the walking one. The standing map should be much narrower than both. This is the correct way to keep one common support variable while allowing different locomotion regimes.

Slope adaptation must enter through the support path and through the pelvis progression thresholds, not by replacing the support law with an unrelated slope-specific law. Let the local support slope angle be

$$
\alpha_{\mathrm{sup}} = \mathrm{atan2}(t_{{\mathrm{sup}},y}, t_{{\mathrm{sup}},x}).
$$

A slope-aware support progression law may use weak slope dependence in the pelvis thresholds:

$$
p_{\mathrm{heel}} = p_{\mathrm{heel},0} + \Delta p_{\mathrm{heel}}(\alpha_{\mathrm{sup}}),
$$

$$
p_{\mathrm{toe}} = p_{\mathrm{toe},0} + \Delta p_{\mathrm{toe}}(\alpha_{\mathrm{sup}}).
$$

The point of this dependence is not to invent a different gait for every slope. It is to allow the phase interval of support progression to shift moderately when the support surface becomes more uphill or downhill. This is consistent with the fact that center-of-pressure progression, plantar loading, and the relation between the body COM and support change systematically with slope and speed.

The reactive intensity

$$
\rho \in [0,1]
$$

must not directly override the hard support bounds. It may deform the support law only through admissible parameter changes. The canonical reactive deformation acts by modifying the protocol thresholds or the timing of support progression, for example through

$$
p_{\mathrm{heel}}(\rho) = p_{\mathrm{heel},0} + \Delta p_{\mathrm{heel}}(\rho),
$$

$$
p_{\mathrm{toe}}(\rho) = p_{\mathrm{toe},0} + \Delta p_{\mathrm{toe}}(\rho),
$$

or through a change of support-map shape

$$
\Gamma_{\sigma}(\eta; \rho).
$$

However, even under strong reactivity,

$$
-L_{\mathrm{heel}} \le \sigma \le L_{\mathrm{toe}}
$$

remains a hard non-negotiable bound.

A support state becomes non-viable if one of the following occurs. The contact path under the foot ceases to be admissible. The support coordinate required by the current state would leave the support interval. The support tangent becomes ill-defined. The support progression demanded by the state becomes inconsistent with the current gait protocol. If a non-viable support state persists, the correct response is not to continue forcing support progression. The correct response is to trigger a new event such as stepping, support transfer, or another mode change.

This yields the correct support-centered interpretation of the rocker scalar. The variable $\sigma$ is neither a cosmetic interpolation parameter nor a direct measurement of the physical center of pressure. It is the canonical support coordinate of the `CM-first` locomotion model. Its progression is defined along the actual foot-terrain contact path. Its law is state-based through ankle-local pelvis progression. It is protocol-dependent across standing, walking, and running. It adapts moderately to slope and reactivity. It remains bounded by the true foot support interval. Once this law is fixed, the support point $Q$, the support tangent, the support-normal frame, and the support-local CM coordinates all become well-defined consequences of a single coherent support variable.