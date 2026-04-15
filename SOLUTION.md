# Introduction
Proton beams can be used to damage the DNA of cancerous cells, rendering them
incapable of replicating. The challenge that this project aims to tackle is delivering
a proton beam dose in a way that maximizes damage to a tumor while minimizing damage
to surrounding healthy tissue. The solution given in this project involves
collimating the proton beam with a computed pin-and-bar geometry, then simulating
the radiation transport and upscaling with a diffusion model...

## Cancer and Proton Therapy
Proton therapy is a way to destroy tumors with a beam of radiation while mitigating
the radiation delivered to the rest of the body. The idea is that we want to
shape the radiation dose to the volume of the target tumor and minimize the
radiation dose immediately outside the tumor. Proton therapy is a radiation treatment
technique that allows us to minimize radiation dose to healthy tissue simply because
protons are “bigger." Compared to photons, protons interact with surrounding
matter, particularly electrons bound to atoms, in a way where they deposit
more of their energy to that surrounding matter.

### Derivation of the Bragg Peak Idea
Imagine a proton traveling between the atoms in the body, where it can interact
with the electrons bounded to those atoms. Using basic classical physics, we can
derive a relationship between the proton's energy loss (from “tugging" the electrons
as it passes by) and its velocity to show that the slower the proton travels, the
more energy it deposits to the surrounding electrons. The diagram below shows
the scenario and illustrates the relevant values to the derivation:

$$
\begin{gather*}
		\lbrack \image \rbrack
\end{gather*}
$$

We can take the interaction to occur from $- \infty$ to $\infty$ in distance (along
the direction of motion of the proton) and time. In this case, the component of
the Coulomb force parallel to the proton's direction of motion would cancel by
symmetry, so we only ever consider the perpendicular component. Then, the
Coulomb force as a function of time $t$ can be written as

$$
\begin{gather*}
		F_{\perp}= \frac{ke^{2}}{r^{2}}\cos \theta = \frac{ke^{2}}{{ \left\lparen \sqrt{b^{2} + v^{2} t^{2}} \right\rparen }^{2}}
		{ \left\lparen \frac{b}{\sqrt{b^{2}+ v^{2}t^{2}}} \right\rparen }= kbe^{2}{ \left\lparen b^{2} + v^{2} t^{2} \right\rparen }
		^{- \frac{3}{2}}
\end{gather*}
$$

From this time-varying force, we can find the impulse on the electron over the
whole interaction:

$$
\begin{gather*}
		\Delta p_{\perp}= \int_{- \infty}^{\infty}F_{\perp}\lparen t \rparen dt = kbe
		^{2}\int_{- \infty}^{\infty}{ \left\lparen b^{2} + v^{2} t^{2} \right\rparen }
		^{- \frac{3}{2}}dt ,{ \left\lbrack t \to \frac{b}{v} \tan \theta \right\rbrack }
		, \\[0.5ex] = kbe^{2}\int_{- \frac{\pi}{2}}^{\frac{\pi}{2}}{ \left\lparen b^{2} + v^{2} { \left\lparen \frac{b}{v} \tan \theta \right\rparen } ^{2} \right\rparen }
		^{- \frac{3}{2}}\frac{b}{v}\sec^{2}\theta d \theta \\[0.5ex] = \frac{ke^{2}}{bv}
		\int_{- \frac{\pi}{2}}^{\frac{\pi}{2}}{ \left\lparen 1 + \tan^{2} \theta \right\rparen }
		^{- \frac{3}{2}}\sec^{2}\theta d \theta \\[0.5ex] = \frac{ke^{2}}{bv}\int_{-
		\frac{\pi}{2}}^{\frac{\pi}{2}}{ \left\lparen \sec^{2} \theta \right\rparen }^{-
		\frac{3}{2}}\sec^{2}\theta d \theta = \frac{ke^{2}}{bv}\int_{- \frac{\pi}{2}}
		^{\frac{\pi}{2}}\cos \theta d \theta \\[0.5ex] = \frac{ke^{2}}{bv}{ \left\lbrack \sin { \left\lparen \frac{\pi}{2} \right\rparen } - \sin { \left\lparen - \frac{\pi}{2} \right\rparen } \right\rbrack }
		= \frac{2ke^{2}}{bv}
\end{gather*}
$$

An electron's energy is $E = \frac{p^{2}}{2m_{e}}$, where $m_{e}$ is its mass.
Therefore, the change in energy of the electron is

$$
\begin{gather*}
		\Delta E = \frac{ \lparen \Delta p_{\perp}\rparen^{2}}{2m_{e}}= \frac{2k^{2}e^{4}}{b^{2}v^{2}m_{e}}
\end{gather*}
$$

Take that value to be one “energy transfer." Now consider a small path length $d
x$. If you have the electron density of the material $n_{e}$, then the number of
electrons in a ring around the proton in that small path length $dx$ is $n_{e}\cdot
2 \pi b \cdot db \cdot dx$. Then, the number of “energy transfers" in a ring
of size $b_{\max}- b_{\min}$ around the proton is

$$
\begin{gather*}
		\int_{b_{\min}}^{b_{\max}}\Delta E{ \left\lparen b \right\rparen }n_{e}\cdot
		2 \pi b \cdot db \cdot dx ,
\end{gather*}
$$

which would also equal the total amount of energy lost by the proton in that
path length $dx$. The energy lost by the proton over the path length $dx$ can then
be written as

$$
\begin{gather*}
		- dE = \int_{b_{\min}}^{b_{\max}}\Delta E{ \left\lparen b \right\rparen }\cdot
		n_{e}\cdot 2 \pi b \cdot db \cdot dx , \\[0.5ex] - \frac{dE}{dx}= \int_{b_{\min}}
		^{b_{\max}}{ \left\lparen \frac{2k^{2}e^{4}}{b^{2}v^{2}m} \right\rparen }\cdot
		n_{e}\cdot 2 \pi b \cdot db \\[0.5ex] - \frac{dE}{dx}= \frac{4 \pi k^{2}e^{4}n_{e}}{v^{2}m_{e}}
		\ln{ \left\lparen \frac{b_{\max}}{b_{\min}} \right\rparen },
\end{gather*}
$$

and now we see the result we've been aiming for:

$$
\begin{gather*}
		- \frac{dE}{dx} & \propto \frac{1}{v^{2}}
\end{gather*}
$$

### Why Proton Therapy?
Looking closely at the result of the derivation above, we see a critical
relationship: $- \frac{dE}{dx}\propto \frac{1}{v^{2}}$. This inverse square
dependence on velocity is the fundamental reason proton therapy works. As the proton
travels through tissue and deposits energy to surrounding electrons, it loses
kinetic energy and its velocity $v$ decreases. As $v$ gets smaller, the rate
of energy loss $- \frac{dE}{dx}$ rapidly increases.\\ This creates a runaway
effect: the proton deposits a relatively low, constant amount of energy as it
enters the body at high speeds, but right before it stops completely, it dumps
the vast majority of its energy in a massive, localized spike. This localized
deposition of energy is known as the Bragg peak. By tuning the initial energy of
the proton beam, we can dictate exactly where the proton stops, ensuring that the
Bragg peak lands perfectly inside the tumor volume while sparing the healthy tissue
situated behind it.
