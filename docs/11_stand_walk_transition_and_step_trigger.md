# 11 — Stand-walk transition and step trigger

In the `CM-first` procedural locomotion architecture, the transition from standing to walking must not be modeled as an arbitrary timer that suddenly lifts one foot. The scientific literature on gait initiation shows that the human transition from quiet standing to walking is preceded by a structured anticipatory redistribution of support. In particular, center-of-pressure motion typically includes a posterior component that helps generate forward progression of the center of mass, and a lateral component toward the future swing foot that transfers body support toward the future stance foot. The correct canonical translation of this phenomenon into BobTricksBeta is not to copy laboratory force-plate trajectories literally, but to preserve their architectural meaning: a step begins only after bilateral support has been intentionally reorganized so that one foot can be unloaded while the other becomes the dominant support.

The standing-to-walking transition therefore requires an explicit pre-step phase, denoted here by

$$
GI_{L\to R} \qquad \text{or} \qquad GI_{R\to L},
$$

depending on which foot is selected as the initial swing foot. If the left foot is selected to swing first, the right foot becomes the future initial stance foot. If the right foot is selected to swing first, the left foot becomes the future initial stance foot. The gait-initiation phase is still a bilateral-support phase, but it is no longer a quiet-standing phase. Its role is to move the support state from a nominal bilateral standing configuration to a unilateral-support-ready configuration.

The current standing state is bilateral and is parameterized by the left and right support coordinates

$$
\sigma_L, \qquad \sigma_R,
$$

together with the support-allocation variable

$$
\beta \in [0,1].
$$

In the bilateral convention adopted in the previous working document, the effective support point is

$$
Q = (1-\beta)Q_L + \beta Q_R.
$$

For a symmetric quiet stand,

$$
\beta \approx \frac{1}{2}.
$$

The first decision to be taken in gait initiation is therefore the swing-foot choice. Let the future swing foot index be denoted by

$$
i_{\mathrm{sw}} \in \{L,R\},
$$

and let the future stance foot index be denoted by

$$
i_{\mathrm{st}} \in \{L,R\}, \qquad i_{\mathrm{st}} \neq i_{\mathrm{sw}}.
$$

The architecture must support two classes of step-trigger causes. The first is intentional locomotion onset, when the system is in bilateral standing and the locomotion goal demands forward walking. The second is support-loss recovery, when bilateral support is no longer sufficient to maintain a viable standing configuration. The canonical architecture must treat both through the same state transition logic. The difference lies in the cause of the trigger, not in the structure of the ensuing support reorganization.

The standing state remains viable only as long as bilateral support can keep the authoritative center of mass within a reasonable support margin while preserving a bounded support-allocation variable and bounded support-local corrective demands. Let the current bilateral support frame be

$$
(Q, t_{\mathrm{sup}}, n_{\mathrm{sup}}).
$$

Write

$$
C = Q + s_Q t_{\mathrm{sup}} + z_Q n_{\mathrm{sup}}.
$$

Standing becomes non-nominal if the standing support error

$$
E_{\mathrm{stand}}
$$

exceeds its nominal tolerance. A canonical form is

$$
E_{\mathrm{stand}} = w_s |s_Q - s_Q^{\star}| + w_v |\dot s_Q| + w_\beta \left|\beta - \frac{1}{2}\right| + w_\rho \rho.
$$

Standing remains nominal while

$$
E_{\mathrm{stand}} \le E_{\mathrm{stand}}^{\max}.
$$

A step trigger is generated if either the locomotion command requests forward walking or if

$$
E_{\mathrm{stand}} > E_{\mathrm{stand}}^{\max}
$$

for a sufficiently persistent duration. This rule ensures that the system does not wait until one-foot standing has effectively occurred. Instead, it initiates gait once bilateral support has ceased to be a good standing solution.

Once the swing side has been selected, the support-allocation variable must move toward the future stance foot. Suppose the future stance foot is the right foot. Then the correct direction of bilateral support transfer is toward larger values of

$$
\beta.
$$

Suppose instead that the future stance foot is the left foot. Then the support-allocation variable must move toward smaller values. This is the `CM-first` equivalent of the lateral anticipatory support redistribution observed in gait initiation.

To write the transition law without duplicating formulas, define the signed target-support coordinate

$$
\zeta =
\left\{
\begin{array}{ll}
\beta & \text{if } i_{\mathrm{st}} = R, \\
1-\beta & \text{if } i_{\mathrm{st}} = L.
\end{array}
\right.
$$

Thus in both cases larger values of $\zeta$ correspond to greater support dominance of the future stance foot.

The gait-initiation phase must now generate two simultaneous effects. First, support allocation must shift toward the future stance foot. Second, the support coordinate under the future swing foot must evolve toward unloading, while the support coordinate under the future stance foot must evolve toward a support configuration that can safely accept unilateral stance.

The canonical support-allocation law during gait initiation is therefore

$$
\dot \zeta = k_{\mathrm{GI}}^{\zeta} \max\{0, \zeta_{\mathrm{raw}} - \zeta\},
$$

where $\zeta_{\mathrm{raw}}$ is a state-based target lying in $[0,1]$. The target must be built from two components: an intentional gait-onset progression and a support-necessity progression. A canonical mixed form is

$$
\zeta_{\mathrm{raw}} = \mathrm{clamp}\left((1-w_{\mathrm{GI}})\eta_t^{\mathrm{GI}} + w_{\mathrm{GI}}\eta_s^{\mathrm{GI}}, 0, 1\right),
$$

where

$$
\eta_t^{\mathrm{GI}} = \mathrm{clamp}\left(\frac{t-t_{\mathrm{GI},0}}{T_{\mathrm{GI}}}, 0, 1\right)
$$

is a temporal onset phase and

$$
\eta_s^{\mathrm{GI}}
$$

is a state-based support-necessity coordinate that increases when the authoritative CM moves toward the future stance foot or when the current standing support error requires unloading of the future swing foot. The key point is that gait initiation must not be purely time-driven. The bilateral support redistribution must remain coupled to the actual support state.

The support coordinates of the two feet must also be guided during gait initiation. Let the future swing foot support coordinate be denoted by

$$
\sigma_{\mathrm{sw}},
$$

and the future stance foot support coordinate by

$$
\sigma_{\mathrm{st}}.
$$

The future swing foot must move toward an unloading-ready support configuration. Because the human APA includes a redistribution of support that unloads the stepping leg before heel-off, the `CM-first` analogue is to move the swing-foot support coordinate toward a region where the swing foot no longer carries dominant support. The simplest canonical rule is to drive the swing-foot support coordinate toward a foot-specific unloading target

$$
\sigma_{\mathrm{sw}}^{\mathrm{unload}}.
$$

At the same time the future stance foot support coordinate should move toward a unilateral-stance preparation target

$$
\sigma_{\mathrm{st}}^{\mathrm{prep}}.
$$

The corresponding filtered laws are

$$
\dot \sigma_{\mathrm{sw}} = \frac{\sigma_{\mathrm{sw}}^{\mathrm{raw}} - \sigma_{\mathrm{sw}}}{\tau_{\sigma}^{\mathrm{GI,sw}}},
$$

$$
\dot \sigma_{\mathrm{st}} = \frac{\sigma_{\mathrm{st}}^{\mathrm{raw}} - \sigma_{\mathrm{st}}}{\tau_{\sigma}^{\mathrm{GI,st}}},
$$

with bounded raw targets inside the support intervals of the corresponding feet. The future swing foot should not be forced to keep a support coordinate incompatible with unloading. The future stance foot should not remain in a support configuration that is good only for quiet bilateral standing.

A critical point of the transition is the actual step-release event. The future swing foot must not leave the ground merely because gait initiation has started. It may leave the ground only after a unilateral-support-ready condition has been satisfied. This condition must include at least the following requirements. The support-allocation variable must have become sufficiently favorable to the future stance foot. The future stance foot must have a valid support path and support point. The pelvis-to-future-stance-ankle geometry must be compatible with exact rigid-leg reconstruction. The reactive intensity must remain compatible with a controlled step onset. Formally, define the unilateral-support-readiness predicate

$$
\mathcal{R}_{\mathrm{US}}(X, i_{\mathrm{st}}).
$$

The release event

$$
GI \to SS
$$

is allowed only if

$$
\mathcal{R}_{\mathrm{US}}(X, i_{\mathrm{st}}) = \mathrm{true}.
$$

This is the correct place where gait initiation ends and single support begins.

The previous construction also clarifies how standing and walking are linked in the opposite direction. Walking must not terminate directly by freezing the support state at an arbitrary instant. The correct end of walking is a support-capture process that returns the system to a bilateral-support solution. In practice, walking enters a terminal bilateral-support phase in which support-allocation and support coordinates converge toward a quiet-standing-compatible configuration. The standing state may be re-entered only when the walking speed target has become sufficiently small, the bilateral support state is viable, and the standing support error has returned below its standing threshold. In other words, standing is not simply the absence of stepping. It is a recovered bilateral support regime.

This leads to the correct procedural interpretation. The transition from standing to walking is a support-reorganization process. It begins when standing is either voluntarily left or ceases to be a good solution. It unfolds through a bilateral phase in which support is shifted toward the future stance foot and the future swing foot is unloaded. It ends only when a unilateral-support-ready configuration has been reached. Conversely, the transition from walking to standing is a support-capture process that restores a viable bilateral support state. With this formulation, the same variables $\sigma$, $\beta$, and $\rho$ remain active across stand, gait initiation, walking, and later running, which is exactly the kind of architectural continuity required by a `CM-first` procedural locomotion system.