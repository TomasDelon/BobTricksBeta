# 00 — Fundamentos científicos y matemáticos del modelo `CM-first`

## 0. Objeto del documento

$x$

$$
x
$$

Este documento fija las decisiones científicas y matemáticas de más alto nivel del proyecto **BobTricksBeta**. El objetivo no es todavía escribir el protocolo completo de marcha, carrera, salto y aterrizaje, sino fijar con rigor:

1. qué objetos geométricos y dinámicos existen;
2. cuáles son las restricciones rígidas del cuerpo;
3. por qué el sistema debe ser **CM-first**;
4. por qué **caminar** y **correr** no deben compartir el mismo modelo mecánico nominal;
5. qué cantidades se usarán más adelante para estabilidad, colocación del pie y recuperación.

La idea central es la siguiente:

- existe **un único centro de masa virtual** \(C\);
- la pose del cuerpo se **reconstruye a partir de \(C\)**;
- el terreno es una **polilínea segmentada**;
- la marcha debe explotar la mecánica tipo **péndulo invertido** durante el apoyo simple;
- la carrera debe explotar una mecánica tipo **spring-mass / SLIP**;
- la recuperación ante perturbaciones no debe aparecer como un “modo mágico” añadido al final, sino como consecuencia natural de la misma arquitectura.

Las decisiones de este documento se apoyan principalmente en:
- la perspectiva de *dynamic walking* para la marcha humana y la importancia energética de la transición paso a paso [Kuo 2007];
- el concepto de *extrapolated center of mass* (XCoM) para estabilidad y colocación del apoyo [Hof 2008];
- la distinción biomecánica entre caminar (mecánica pendular) y correr (mecánica spring-mass) [Blickhan 1989];
- la biomecánica de la marcha en pendientes y el papel de la transducción pendular y del trabajo mecánico en subida y bajada [Dewolf et al. 2017a, 2018];
- el hecho de que el terreno desigual aumenta el coste mecánico y exige más trabajo proximal, más variabilidad y estrategias más cautelosas [Voloshina et al. 2013];
- la relación entre desequilibrio, momento angular global y estrategia de paso en marcha perturbada [Leestma et al. 2023].

## 1. Convenciones generales

Se trabaja en el plano sagital \( (X,Y) \), con:

\[
X \text{ horizontal,}\qquad Y \text{ vertical hacia arriba.}
\]

La gravedad global es:

\[
g_W = (0,-g),\qquad g>0.
\]

Se adopta masa virtual unitaria:

\[
m=1.
\]

Por tanto, en todas las ecuaciones del CM, fuerza y aceleración tienen la misma dimensión numérica:

\[
\ddot C = \sum F.
\]

Esta convención no significa que el cuerpo humano real tenga masa unitaria. Significa simplemente que el modelo dinámico autoritativo del proyecto se escribe directamente a nivel de aceleraciones del CM virtual.

## 2. Geometría rígida del cuerpo

### 2.1. Unidad fundamental de longitud

Se define una unidad geométrica fundamental \(L\). En el proyecto actual, el personaje nominal tiene altura:

\[
H = 5L.
\]

En la configuración actual del repositorio público, se usa:

\[
H = 1.80\ \text{m},\qquad L = 0.36\ \text{m}.
\]

### 2.2. Nodos rígidos del cuerpo

Se introducen los siguientes nodos:

- \(A_L, A_R\): tobillos izquierdo y derecho;
- \(K_L, K_R\): rodillas izquierda y derecha;
- \(P\): pelvis;
- \(T_c\): centro del torso;
- \(T_t\): parte superior del torso;
- \(C\): centro de masa virtual.

Las distancias rígidas impuestas por diseño son:

\[
\|A_i-K_i\| = L,
\]
\[
\|K_i-P\| = L,
\]
\[
\|P-T_c\| = L,
\]
\[
\|T_c-T_t\| = L,
\]
\[
\|C-P\| = 0.75L.
\]

Aquí \(i\in\{L,R\}\).

Estas igualdades deben ser verdaderas en **todo frame renderizado**. No son preferencias suaves. Son restricciones rígidas del modelo morfológico.

### 2.3. Consecuencia inmediata: longitud máxima de pierna

La longitud máxima pelvis–tobillo de una pierna es:

\[
\ell_{\max} = \|P-K_i\|+\|K_i-A_i\| = 2L.
\]

Por desigualdad triangular,

\[
\|P-A_i\| \le 2L.
\]

La igualdad se alcanza cuando la rodilla está totalmente extendida y pelvis, rodilla y tobillo son colineales.

Esto es importante porque prohíbe una simplificación peligrosa: **la pierna no puede cambiar de longitud ósea**. La única manera de reducir \(\|P-A_i\|\) es flexionando la rodilla.

## 3. Principio CM-first

### 3.1. Objeto dinámico primario

El único objeto dinámico autoritativo es el centro de masa virtual:

\[
C(t)\in\mathbb R^2.
\]

Con velocidad y aceleración:

\[
V(t)=\dot C(t),\qquad A(t)=\ddot C(t).
\]

No existe en el modelo otro “CM geométrico” derivado de una distribución de masas segmentarias. El cuerpo reconstruido debe ser compatible con \(C\), pero no define a \(C\).

### 3.2. Reconstrucción de la pelvis y del torso a partir del CM

Sea \(\theta\) el ángulo del torso medido desde la vertical global. Definimos el versor axial del torso:

\[
e_\theta = (\sin\theta,\cos\theta).
\]

Entonces la pelvis se reconstruye como:

\[
P = C - 0.75L\, e_\theta.
\]

Y los demás nodos del torso son:

\[
T_c = P + L\, e_\theta,
\]
\[
T_t = P + 2L\, e_\theta.
\]

Esta reconstrucción tiene dos ventajas fundamentales.

La primera es geométrica: garantiza automáticamente

\[
\|C-P\|=0.75L,\qquad \|P-T_c\|=L,\qquad \|T_c-T_t\|=L.
\]

La segunda es arquitectónica: el sistema queda verdaderamente **CM-first**. El torso y la pelvis no “mandan” sobre el CM. Se derivan de él.

## 4. Modelo del terreno

### 4.1. Terreno como polilínea

El terreno no se modela como una función suave única, sino como una polilínea:

\[
\Gamma = \bigcup_{j=1}^{N} S_j,
\qquad
S_j = [V_j,V_{j+1}].
\]

Cada segmento \(S_j\) tiene:

- tangente unitaria \(t_j\),
- normal unitaria \(n_j\),
- ángulo \(\alpha_j\) respecto a la horizontal.

Si

\[
t_j=(t_{j,x},t_{j,y}),
\]

entonces

\[
\alpha_j = \operatorname{atan2}(t_{j,y},t_{j,x}).
\]

### 4.2. Vértices como eventos geométricos

Los vértices \(V_j\) no son un detalle numérico. Son eventos geométricos reales del problema.

Hay que distinguir explícitamente:

- tramos lisos largos;
- segmentos cortos;
- cambios bruscos de ángulo;
- bordes de escalón;
- situaciones donde el pie cubre simultáneamente dos segmentos.

Esta decisión es esencial porque BobTricks no vive en un mundo de rampas analíticas infinitamente suaves. Vive en un mundo de **segmentos conectados**.

## 5. El pie, el tobillo y el soporte efectivo

### 5.1. El tobillo no es el soporte completo

El tobillo \(A\) es una articulación. No es toda la base de apoyo.

Como diseño geométrico del pie, se define el intervalo:

\[
\mathcal F(A,f)=\{A+\sigma f:\ \sigma\in[-L_{\text{heel}},L_{\text{toe}}]\}.
\]

Por decisión actual:

\[
L_{\text{heel}}=0,
\]

de modo que el pie sale prácticamente del tobillo hacia delante.

Cuando el pie está apoyado, su dirección visible \(f\) debe alinearse con la tangente local del suelo o, si cubre dos segmentos, con una dirección efectiva derivada del mini-segmento tobillo–punta.

### 5.2. Punto de soporte efectivo

Para la dinámica del CM conviene distinguir entre tobillo \(A\) y soporte efectivo \(Q\):

\[
Q = A + \sigma t,
\qquad
\sigma\in[0,L_{\text{toe}}].
\]

La variable \(\sigma\) representa el desplazamiento del soporte efectivo desde el tobillo hacia el antepié.

Interpretación:

- al inicio del apoyo, \(\sigma\) es pequeño;
- durante heel-rise y toe-off, \(\sigma\) aumenta;
- en un modelo más rico, \(\sigma\) representa un “rocker” efectivo.

Este punto \(Q\) será útil para estabilidad y transición de soporte. Pero la restricción rígida de la pierna seguirá escrita respecto al tobillo \(A\), porque la pierna anatómica se conecta al tobillo, no al centro de presión.

## 6. Altura nominal de pie

Antes de estudiar marcha o carrera, conviene calcular la altura nominal del CM en postura de pie cuasi-simétrica.

Sea \(d_{\text{pref}}\) la separación nominal entre tobillos expresada en unidades de \(L\):

\[
\|A_R-A_L\| = d_{\text{pref}}L.
\]

Si la pelvis está centrada horizontalmente entre ambos pies y ambas piernas están totalmente extendidas, la altura de la pelvis es:

\[
h_P^{\text{stand}}
=
\sqrt{(2L)^2 - \left(\frac{d_{\text{pref}}L}{2}\right)^2 }.
\]

**Demostración.**

Cada pierna forma un triángulo rectángulo cuya hipotenusa vale \(2L\) y cuyo cateto horizontal vale \(d_{\text{pref}}L/2\). Por Pitágoras:

\[
(2L)^2 = \left(\frac{d_{\text{pref}}L}{2}\right)^2 + \left(h_P^{\text{stand}}\right)^2.
\]

Luego:

\[
h_P^{\text{stand}}
=
\sqrt{(2L)^2 - \left(\frac{d_{\text{pref}}L}{2}\right)^2 }.
\]

Como el CM está a \(0.75L\) por encima de la pelvis cuando \(\theta=0\),

\[
h_C^{\text{stand}}
=
h_P^{\text{stand}} + 0.75L.
\]

En la configuración pública actual del proyecto, \(d_{\text{pref}}=0.90\). Entonces:

\[
h_P^{\text{stand}}
=
\sqrt{4L^2 - 0.2025L^2}
=
\sqrt{3.7975}\,L
\approx 1.9487L.
\]

Y por tanto:

\[
h_C^{\text{stand}}
\approx 1.9487L + 0.75L
= 2.6987L.
\]

Con \(L=0.36\text{ m}\),

\[
h_C^{\text{stand}} \approx 0.9715\text{ m}.
\]

Esta cantidad es una **referencia de postura de pie**, no una ley general de la locomoción.

## 7. Frame local del apoyo y descomposición de la gravedad

Sea el pie de apoyo sobre un segmento \(S_j\), con tangente \(t\) y normal \(n\). Si el pie cubre dos segmentos, estos vectores podrán sustituirse más adelante por una versión efectiva.

Descomponemos el CM respecto al tobillo \(A\) en el frame local del apoyo:

\[
C = A + s_A\, t + z_A\, n.
\]

Aquí:

\[
s_A = (C-A)\cdot t,
\qquad
z_A = (C-A)\cdot n.
\]

La gravedad global se proyecta como:

\[
g_t = g_W\cdot t = -g\sin\alpha,
\]
\[
g_n = g_W\cdot n = -g\cos\alpha.
\]

**Consecuencia inmediata.**

- si \(\alpha<0\) (descenso en la dirección de avance), entonces \(-g\sin\alpha>0\), así que la gravedad empuja hacia delante;
- si \(\alpha=0\), la componente tangencial es nula;
- si \(\alpha>0\) (subida), entonces \(-g\sin\alpha<0\), así que la gravedad frena el avance.

Este cálculo elemental ya explica por qué una caminata realista en bajada **no** puede obtenerse con un simple “path vertical” del CM: hay una contribución tangencial física de la gravedad que debe entrar explícitamente en el modelo.

## 8. Modelo nominal de caminar

### 8.1. Razón biomecánica

La literatura moderna sobre marcha humana no apoya la idea antigua de “aplanar” la trayectoria del CM. Al contrario: el apoyo simple aprovecha una mecánica pendular, y el coste energético principal aparece de manera inevitable en la transición paso a paso entre una pierna y la otra [Kuo 2007].

Por tanto, el modelo nominal de **walk** no debe ser:
- ni una simple animación de piernas;
- ni un tracking vertical del CM;
- ni una cadena de poses.

Debe ser una dinámica del CM compatible con una pierna de apoyo casi extendida.

### 8.2. Longitud efectiva de pierna en marcha

En apoyo simple de marcha, se desea que la pierna de apoyo esté casi extendida:

\[
\ell_w^\star = 2L - \varepsilon_w L,
\qquad 0<\varepsilon_w\ll 1.
\]

Típicamente \(\varepsilon_w\) será pequeño. Su único papel es permitir que la rodilla no esté matemáticamente bloqueada todo el tiempo.

La longitud pelvis–tobillo es:

\[
\ell = \|P-A\|.
\]

La variedad geométrica nominal de marcha es entonces:

\[
\Phi_w(C,\theta,A)
=
\|P-A\|^2 - (\ell_w^\star)^2
=
0.
\]

Como

\[
P = C - 0.75L\, e_\theta,
\]

esta ecuación también puede escribirse como

\[
\Phi_w(C,\theta,A)
=
\|C - 0.75L\, e_\theta - A\|^2 - (\ell_w^\star)^2
=
0.
\]

### 8.3. Forma explícita de la trayectoria nominal del CM

Escribamos el offset pelvis–CM en el frame local:

\[
d_t = 0.75L\,(e_\theta\cdot t),
\qquad
d_n = 0.75L\,(e_\theta\cdot n).
\]

Como

\[
C-A = s_A t + z_A n,
\]

se obtiene

\[
P-A = (s_A-d_t)t + (z_A-d_n)n.
\]

Por tanto,

\[
\|P-A\|^2 = (s_A-d_t)^2 + (z_A-d_n)^2.
\]

La ecuación \(\Phi_w=0\) se convierte en

\[
(s_A-d_t)^2 + (z_A-d_n)^2 = (\ell_w^\star)^2.
\]

Despejando \(z_A\), la rama físicamente relevante es

\[
z_A^\star(s_A,\theta)
=
d_n + \sqrt{(\ell_w^\star)^2 - (s_A-d_t)^2 }.
\]

Esta es la ley geométrica nominal correcta de la altura del CM en marcha, porque:

1. sale de las longitudes rígidas reales del modelo;
2. incorpora explícitamente la inclinación del torso;
3. no inventa una oscilación vertical artificial.

### 8.4. Demostración de la existencia del ápice del CM

Derivando respecto a \(s_A\),

\[
\frac{d z_A^\star}{d s_A}
=
-\frac{s_A-d_t}{\sqrt{(\ell_w^\star)^2-(s_A-d_t)^2}}.
\]

Por tanto,

\[
\frac{d z_A^\star}{d s_A}=0
\iff
s_A=d_t.
\]

La segunda derivada es

\[
\frac{d^2 z_A^\star}{d s_A^2}
=
-\frac{(\ell_w^\star)^2}{\left((\ell_w^\star)^2-(s_A-d_t)^2\right)^{3/2}} < 0
\]

siempre que el radical sea positivo.

Luego \(s_A=d_t\) es un máximo local estricto de \(z_A^\star\).

**Conclusión.**

Durante apoyo simple, el CM alcanza un ápice natural sin necesidad de imponer ninguna onda vertical heurística. Ese ápice emerge de la pura geometría rígida de pelvis, torso y pierna de apoyo.

### 8.5. Relación con el soporte efectivo del pie

La ecuación anterior se escribe respecto al tobillo \(A\), porque la pierna se ancla ahí.

Pero la estabilidad y la transición de apoyo se entienden mejor respecto al soporte efectivo \(Q\). Si

\[
Q = A+\sigma t,
\]

entonces la coordenada tangencial del CM respecto al soporte efectivo es

\[
s_Q = (C-Q)\cdot t = s_A - \sigma.
\]

Como \(\sigma\) avanza durante el apoyo, el hecho de que el ápice geométrico aparezca en \(s_A=d_t\) no contradice el hecho de que el soporte efectivo pueda quedar “debajo” del CM más adelante dentro del pie.

Esto permite tener simultáneamente:
- una pierna anatómica correcta;
- un apoyo efectivo que progresa de tobillo a antepié;
- una transición más realista de soporte.

## 9. Dinámica del CM en marcha

La ecuación básica es

\[
\ddot C = g_W + u.
\]

El control \(u\) no debe entenderse como “fuerzas musculares literales”. Debe entenderse como una aceleración virtual de control que organiza el movimiento del CM.

Propongo descomponerla como

\[
u = u_{\text{leg}} + u_{\text{tan}} + u_{\text{trans}}.
\]

### 9.1. Término geométrico de pierna

Sea

\[
r = P-A,
\qquad
\hat r = \frac{r}{\|r\|}.
\]

Entonces un término simple que mantiene al sistema cerca de \(\Phi_w=0\) es

\[
u_{\text{leg}}
=
-k_\Phi \Phi_w \hat r
-c_\Phi \dot\Phi_w \hat r.
\]

Este término **no** debe interpretarse como una cárcel geométrica absoluta. Su papel es mantener una marcha nominal eficiente mientras el sistema no esté fuertemente perturbado.

### 9.2. Término tangencial de velocidad y estabilidad

En el frame local del apoyo, definimos

\[
\dot s_A = \dot C\cdot t.
\]

Y una frecuencia local nominal:

\[
\omega_0 = \sqrt{\frac{g\cos\alpha}{\bar z}},
\]

donde \(\bar z\) es una altura normal de referencia positiva.

Entonces se define una versión local del XCoM:

\[
\xi = s_Q + \lambda \frac{\dot s_A}{\omega_0},
\qquad
0<\lambda\le 1.
\]

La idea procede de Hof: en un modelo pendular simple, los cambios de velocidad del CM pueden compensarse en gran parte desplazando el apoyo de acuerdo con \(\Delta v/\omega_0\) [Hof 2008].

El control tangencial puede escribirse como

\[
u_{\text{tan}}
=
k_v(v_t^\star-\dot s_A)\,t
+
k_\xi(\xi^\star-\xi)\,t.
\]

Donde:
- \(v_t^\star\) es la velocidad tangencial objetivo;
- \(\xi^\star\) es una consigna de capturabilidad / margen dentro del soporte futuro.

## 10. La doble fase y la transición paso a paso

La literatura de dynamic walking insiste en un punto esencial: la transición entre una pierna y la otra no es un detalle, sino una fuente inevitable de trabajo mecánico [Kuo 2007].

Por tanto, la doble fase no debe modelarse como una simple interpolación visual. Debe existir como subproblema dinámico propio.

Sean:
- \(A_b\): tobillo de la pierna trasera;
- \(A_f\): tobillo de la pierna delantera;
- \(\beta\in[0,1]\): transferencia de soporte.

Definimos dos restricciones blandas:

\[
\Phi_b = \|P-A_b\|^2 - \ell_b^{\star 2},
\]
\[
\Phi_f = \|P-A_f\|^2 - \ell_f^{\star 2}.
\]

Y un control geométrico de doble apoyo:

\[
u_{\text{leg}}^{DS}
=
-(1-\beta)(k_b\Phi_b+c_b\dot\Phi_b)\hat r_b
-
\beta(k_f\Phi_f+c_f\dot\Phi_f)\hat r_f.
\]

Esta formulación tiene tres ventajas:

1. permite una transferencia progresiva de soporte;
2. evita una conmutación demasiado brusca a 60 Hz;
3. deja espacio a que la duración efectiva de doble fase cambie con terreno, velocidad y perturbaciones.

En pendiente, esto es especialmente importante, porque la distribución temporal del trabajo mecánico cambia entre subida y bajada [Dewolf et al. 2017a, 2018].

## 11. Perturbaciones y recuperación: no usar una “caja dura”

Aquí se fija una decisión arquitectónica importante.

No se debe obligar al CM a permanecer siempre dentro de una zona nominal rígida si eso impide respuestas naturales ante perturbaciones.

Se definen tres regímenes:

- \(\mathcal N\): régimen nominal de marcha;
- \(\mathcal R\): régimen recuperable;
- \(\mathcal F\): régimen de fallo / caída / transición de emergencia.

El sistema debe preferir \(\mathcal N\), pero si una perturbación lo expulsa de ahí, debe poder usar automáticamente:
- una bajada del CM;
- pasos más grandes;
- cambios de timing;
- varios pasos de recuperación.

Esto no es una “feature aparte”. Es una consecuencia obligatoria de una buena arquitectura CM-first.

Una manera simple de parametrizar la intensidad reactiva es:

\[
\rho
=
\operatorname{clamp}
\left(
\max
\left\{
\frac{d(\xi,\mathcal S)-m_\xi}{\Delta_\xi},
\frac{\|P-A_{stance}\|-\ell_w^\star}{\Delta_\ell},
\frac{|\dot s_A-v_t^\star|}{\Delta_v}
\right\},
0,1
\right).
\]

Aquí:
- \(\mathcal S\) representa el soporte efectivo actual o combinado;
- \(d(\xi,\mathcal S)\) es una distancia de margen/captura;
- \(\rho=0\) significa marcha nominal;
- \(\rho\approx 1\) significa marcha fuertemente reactiva.

Entonces, por ejemplo, puede definirse:

\[
\ell_w^\star(\rho)
=
2L - \varepsilon_wL - \rho\,\Delta\ell_wL,
\]

de manera que una perturbación grande reduzca la longitud efectiva objetivo de la pierna de apoyo, es decir, baje el CM por flexión.

Y el alcance máximo de paso podría crecer con \(\rho\):

\[
\ell_{\max}^{step}(\rho)
=
\ell_{\max,0}^{step} + \rho\,\Delta \ell_{\max}^{step}.
\]

Esta filosofía es coherente con la evidencia de marcha perturbada: el tamaño del desequilibrio y el momento angular global condicionan la estrategia de paso y de recuperación [Leestma et al. 2023].

## 12. Por qué correr necesita otro protocolo

### 12.1. Diferencia biomecánica esencial

Caminar y correr no son el mismo fenómeno con distinta velocidad.

En términos simplificados:

- **walking**: apoyo simple con mecánica tipo péndulo invertido;
- **running**: apoyo con compresión y restitución tipo spring-mass, más fase de vuelo.

Esta distinción clásica está establecida desde hace décadas [Blickhan 1989].

### 12.2. Consecuencia para BobTricks

El mismo CM virtual \(C\) puede seguir siendo el objeto dinámico primario.

Pero el protocolo de contacto debe cambiar.

En carrera, los modos básicos son:

\[
\mathcal M_{\text{run}}=\{RS_L,\ FLIGHT,\ RS_R\}.
\]

Durante apoyo de carrera, la longitud efectiva de pierna:

\[
\ell = \|P-A\|
\]

ya no debe permanecer cerca de \(2L\). Debe comprimirse y reexpandirse.

Un modelo virtual mínimo es:

\[
u_{\text{run}}
=
k_r(\ell_0-\ell)_+\,\hat r
-
c_r\dot\ell\,\hat r
+
k_t(v_t^\star-\dot s_A)\,t.
\]

Aquí:
- \(\ell_0\) es una longitud efectiva objetivo de resorte virtual;
- \((x)_+ = \max(x,0)\);
- el resorte no representa una “pierna que cambia de hueso”, sino una compresión efectiva conseguida por flexión de rodilla y postura del cuerpo.

La diferencia con la marcha es profunda:

- en **walk**, el CM tiene un máximo de altura cerca de mid-stance;
- en **run**, el CM tiene un mínimo de altura cerca de mid-stance.

Por eso es un error intentar que `Run` nazca simplemente acelerando el protocolo de `Walk`.

## 13. Pie visible sobre dos segmentos

Cuando el tobillo permanece sobre un segmento \(S_i\) pero la punta del pie ya entra en \(S_{i+1}\), una solución simple y eficaz es definir la orientación visible del pie a partir del mini-segmento tobillo–punta.

Sea:
- \(A\): tobillo;
- \(T_c\): punto delantero efectivo del pie;
- \(f_{\text{vis}}\): dirección visible del pie.

Se define:

\[
f_{\text{vis}} = \frac{T_c-A}{\|T_c-A\|}.
\]

Ventajas:

1. no promedia ángulos de manera artificial;
2. la inclinación visible sale de una recta real en el mundo;
3. el pie cambia de orientación de forma continua al cubrir dos segmentos.

Pero conviene **no** identificar automáticamente esta orientación visible con la totalidad de la dinámica del soporte. Para la dinámica del CM seguirá siendo útil una noción separada de soporte efectivo \(Q\), potencialmente combinada entre ambos segmentos.

## 14. Síntesis arquitectónica

El pipeline correcto del proyecto debe ser:

\[
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
\]

### 14.1. `TerrainModel`

Proporciona:
- segmentos activos;
- tangentes y normales;
- vértices;
- consultas de altura e intersección.

### 14.2. `ContactModel`

Mantiene:
- pie de apoyo;
- pie libre;
- soporte efectivo \(Q\);
- transferencia \(\beta\) en doble apoyo.

### 14.3. `CMController`

Integra:

\[
\ddot C = g_W + u.
\]

Y decide el protocolo dinámico según el modo:
- `Walk`,
- `Run`,
- `Flight`,
- `Land`,
- `Recover`.

### 14.4. `FootstepPlanner`

Elige el siguiente tobillo sobre la polilínea, usando:
- capturabilidad tipo XCoM;
- límites de alcance;
- geometría de segmentos;
- coste de transición.

### 14.5. `SwingGenerator`

Construye la trayectoria del pie libre sin colisión con el terreno.

### 14.6. `BodyReconstruction`

Reconstruye pelvis, torso y piernas, imponiendo exactamente las distancias rígidas.

### 14.7. `Render`

Solo dibuja. No debe inventar física ni corregir inconsistencias geométricas.

## 15. Conclusiones del documento

1. El proyecto debe permanecer estrictamente **CM-first**.
2. El terreno debe modelarse como una **polilínea segmentada**, no como una curva global suave.
3. La morfología rígida del cuerpo impone una restricción geométrica fuerte sobre la relación entre CM, pelvis y tobillo.
4. La marcha nominal debe basarse en una pierna de apoyo casi extendida y en una mecánica tipo péndulo invertido.
5. El ápice del CM en marcha emerge naturalmente de la geometría rígida:

\[
z_A^\star(s_A,\theta)
=
d_n + \sqrt{(\ell_w^\star)^2 - (s_A-d_t)^2 }.
\]

6. La estabilidad y la colocación del pie pueden apoyarse en XCoM, pero XCoM no debe convertirse en ley única del sistema.
7. La recuperación ante perturbaciones debe emerger del mismo modelo, permitiendo bajar el CM y ampliar el paso cuando aumenta la intensidad reactiva \(\rho\).
8. La carrera debe tener un protocolo diferente, de tipo spring-mass / SLIP efectivo, y no ser un simple “walk acelerado”.

## 16. Bibliografía primaria mínima

### [Kuo 2007]
Arthur D. Kuo, *The six determinants of gait and the inverted pendulum analogy: A dynamic walking perspective*, Human Movement Science, 26(4):617–656, 2007.  
DOI: `10.1016/j.humov.2007.04.003`

### [Hof 2008]
A. L. Hof, *The ‘extrapolated center of mass’ concept suggests a simple control of balance in walking*, Human Movement Science, 27(1):112–125, 2008.  
DOI: `10.1016/j.humov.2007.08.003`

### [Blickhan 1989]
R. Blickhan, *The spring-mass model for running and hopping*, Journal of Biomechanics, 22(11-12):1217–1227, 1989.  
DOI: `10.1016/0021-9290(89)90224-8`

### [Dewolf et al. 2017a]
A. H. Dewolf, Y. Ivanenko, F. Lacquaniti, P. A. Willems, *Pendular energy transduction within the step during human walking on slopes at different speeds*, PLOS ONE, 12(10):e0186963, 2017.  
DOI: `10.1371/journal.pone.0186963`

### [Dewolf et al. 2018]
A. H. Dewolf, Y. Ivanenko, K. E. Zelik, F. Lacquaniti, P. A. Willems, *Kinematic patterns while walking on a slope at different speeds*, Journal of Applied Physiology, 125(2):642–653, 2018.  
DOI: `10.1152/japplphysiol.01020.2017`

### [Voloshina et al. 2013]
A. S. Voloshina, A. D. Kuo, M. A. Daley, D. P. Ferris, *Biomechanics and energetics of walking on uneven terrain*, Journal of Experimental Biology, 216(21):3963–3970, 2013.  
DOI: `10.1242/jeb.081711`

### [Leestma et al. 2023]
J. K. Leestma, P. R. Golyski, C. R. Smith, G. S. Sawicki, A. J. Young, *Linking whole-body angular momentum and step placement during perturbed human walking*, Journal of Experimental Biology, 226(6):jeb244760, 2023.  
DOI: `10.1242/jeb.244760`
