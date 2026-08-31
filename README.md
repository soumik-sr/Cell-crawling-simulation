# Cell Crawling Simulation — Version 0

A minimal quasi-2D phase-field prototype for studying cell crawling
using active treadmilling.

## Overview

This project is **Version 0** of a physics-based cell-crawling
simulation inspired by the research paper:

> Tjhung, E., Marenduzzo, D. & Cates, M. E.  
> **Minimal Physical Model Captures Shapes of Crawling Cells.**  
> Nature Communications 6, 5420 (2015). DOI: `10.1038/ncomms6420`

The paper develops a much more complete 3D active-fluid model. This
project deliberately starts with a small quasi-2D prototype so that the
physics and numerical methods can be understood before adding
polarization, active stress, and full hydrodynamics.

The current model contains:

- a diffuse phase field `φ(x,y,t)` representing the cell;
- a smooth cell interface;
- a free-energy model containing a double-well potential and gradient
  energy;
- a prescribed active velocity representing treadmilling;
- finite-difference spatial derivatives;
- explicit Euler time integration.

**Important:** Version 0 is **not a reproduction of the full paper**. It
is the first computational milestone toward that model.

------------------------------------------------------------------------

# 1. Physical Picture

We represent the cell as a soft droplet sitting on a substrate.

At this stage, we focus on two competing effects:

1. **Passive relaxation / surface-tension-like physics:** the cell
   boundary tends to become smooth.
2. **Active treadmilling:** activity near the leading edge produces
   directed motion.

Conceptually:

```text
                 active treadmilling
                         ↓
              ┌──────────────────┐
              │       CELL       │ → motion
              │                  │
              └──────────────────┘
                         ↑
                    relaxation
```

The goal is to see how a cell-like object can move and deform when
active transport is added to a phase-field model.

------------------------------------------------------------------------

# 2. The Phase Field `φ`

Instead of explicitly tracking the cell boundary, we define a scalar
field

$$
\phi(x,y,t).
$$

We interpret it approximately as

$$
\phi \approx 1 \quad \text{inside the cell},
\qquad
\phi \approx 0 \quad \text{outside the cell}.
$$

The transition is smooth:

```text
inside              interface              outside

φ ≈ 1                  0 < φ < 1             φ ≈ 0

███████████████████▓▓▓▓▓▓▓▓▓░░░░░░░░░░░░░░
```

This is a **diffuse-interface phase-field description**.

The approximate cell boundary is the contour

$$
\phi(x,y,t)=\frac12.
$$

### Why use a phase field?

A crawling cell has a boundary that continuously moves and deforms.
Tracking that boundary explicitly is difficult. With a phase field, the
boundary is simply the region where `φ` changes rapidly.

------------------------------------------------------------------------

# 3. Initial Cell

The prototype starts with a smooth circular droplet:

$$
\phi(r)=\frac12\left[
1-\tanh\left(\frac{r-R}{\varepsilon}\right)
\right].
$$

where

$$
r=\sqrt{x^2+y^2}.
$$

`R` is the initial radius and `ε` controls the interface width.

## Why `tanh`?

For points deep inside the cell,

$$
r<R,
$$

so

$$
\frac{r-R}{\varepsilon}\ll0
\qquad\Rightarrow\qquad
\tanh\left(\frac{r-R}{\varepsilon}\right)\approx-1,
$$

and therefore

$$
\phi\approx1.
$$

At the boundary,

$$
r=R,
$$

and

$$
\tanh(0)=0,
$$

so

$$
\phi(R)=\frac12.
$$

Outside,

$$
r>R,
$$

so

$$
\frac{r-R}{\varepsilon}\gg0
\qquad\Rightarrow\qquad
\tanh\left(\frac{r-R}{\varepsilon}\right)\approx1,
$$

and

$$
\phi\approx0.
$$

Therefore, the field smoothly interpolates between the two phases:

$$
\phi(r)\longrightarrow
\begin{cases}
1,& r\ll R,\\
0,& r\gg R.
\end{cases}
$$

------------------------------------------------------------------------

# 4. Free Energy

The phase field is given a free energy:

$$
F[\phi]
=
\int_\Omega
\left[
\frac{\kappa}{2}|\nabla\phi|^2
+
V(\phi)
\right]\,dA.
$$

There are two main contributions.

## 4.1 Gradient / interface energy

$$
F_g
=
\int_\Omega
\frac{\kappa}{2}|\nabla\phi|^2\,dA.
$$

Inside and outside the cell, `φ` is almost constant, so its gradient is
small. Near the boundary, `φ` changes rapidly, so the gradient is large.

Thus this term assigns an energetic cost to the interface.

Physically,

$$
|\nabla\phi|\text{ large}
\quad\Rightarrow\quad
\text{large interface energy}.
$$

This is the phase-field analogue of surface-tension-like behaviour.

## 4.2 Double-well potential

$$
V(\phi)=\phi^2(1-\phi)^2.
$$

It has minima at

$$
V(0)=0,
\qquad
V(1)=0.
$$

Thus the preferred states are

$$
\phi=0
\qquad\text{and}\qquad
\phi=1.
$$

These correspond to outside and inside the cell.

So the two terms have complementary jobs:

$$
F_g
\quad\text{penalizes sharp interfaces},
$$

$$
V(\phi)
\quad\text{favours the two bulk phases }0\text{ and }1.
$$

Together they produce a stable diffuse droplet.

------------------------------------------------------------------------

# 5. Chemical Potential

To determine how the phase field relaxes, we take the functional
derivative of the free energy:

$$
\mu=\frac{\delta F}{\delta\phi}.
$$

For the free energy above,

$$
\mu
=
-\kappa\nabla^2\phi
+
V'(\phi).
$$

This is the **chemical-potential-like quantity** used in the prototype.

## Derivation of the double-well contribution

Starting with

$$
V(\phi)=\phi^2(1-\phi)^2,
$$

differentiate:

$$
\frac{dV}{d\phi}
=
\frac{d}{d\phi}
\left[
\phi^2(1-\phi)^2
\right].
$$

Using the product rule,

$$
V'(\phi)
=
2\phi(1-\phi)^2
-
2\phi^2(1-\phi).
$$

Factor:

$$
V'(\phi)
=
2\phi(1-\phi)(1-2\phi).
$$

Hence,

$$
\boxed{
\mu
=
-\kappa\nabla^2\phi
+
2\phi(1-\phi)(1-2\phi)
}
$$

which gives the second part of `μ`.

------------------------------------------------------------------------

# 6. Why Does the Laplacian Appear?

The gradient part of the free energy is

$$
F_g
=
\int_\Omega
\frac{\kappa}{2}|\nabla\phi|^2\,dA.
$$

When taking the functional derivative, integration by parts gives

$$
\delta F_g
=
-\int_\Omega
\kappa(\nabla^2\phi)\,\delta\phi\,dA
$$

up to boundary terms.

Therefore,

$$
\frac{\delta F_g}{\delta\phi}
=
-\kappa\nabla^2\phi.
$$

The important physical interpretation is:

$$
\nabla^2\phi
\quad\text{measures local curvature / spatial variation of }\phi.
$$

It naturally appears because we are penalizing spatial gradients of the
phase field.

------------------------------------------------------------------------

# 7. Numerical Laplacian

A computer works on a grid rather than a continuous function.

For the 2D Laplacian,

$$
\nabla^2\phi
=
\frac{\partial^2\phi}{\partial x^2}
+
\frac{\partial^2\phi}{\partial y^2}.
$$

We approximate it using neighbouring grid points:

$$
\nabla^2\phi_{i,j}
\approx
\frac{
\phi_{i+1,j}
+\phi_{i-1,j}
+\phi_{i,j+1}
+\phi_{i,j-1}
-4\phi_{i,j}
}{
\Delta x^2
}.
$$

This compares the current point with its four nearest neighbours.

For example:

```text
        neighbour
            |
neighbour - centre - neighbour
            |
        neighbour
```

If the centre differs strongly from its neighbours, the Laplacian is
large. This is why it is useful for describing smoothing and curvature.

------------------------------------------------------------------------

# 8. Numerical First Derivative

The active transport term needs

$$
\frac{\partial\phi}{\partial x}.
$$

We use the central difference:

$$
\left.
\frac{\partial\phi}{\partial x}
\right|_{i,j}
\approx
\frac{
\phi_{i+1,j}-\phi_{i-1,j}
}{
2\Delta x
}.
$$

This is a second-order finite-difference approximation.

------------------------------------------------------------------------

# 9. Active Treadmilling

A passive phase-field droplet would relax toward an equilibrium shape.
A crawling cell is active.

In Version 0, we do **not yet solve for the actin polarization field**.
Instead, we prescribe an active velocity

$$
u_x(x).
$$

The velocity is intentionally stronger toward the `+x` leading edge.

Conceptually:

```text
rear                                  front

|---------------------------------------->
 weak activity                    strong activity
```

A smooth leading-edge-weighted prototype profile can be written as

$$
u_x(x)
=
w_0
\frac{
1+\tanh\left(\frac{x-x_c}{\ell_u}\right)
}{2},
$$

where `w0` sets the activity scale, `xc` controls where the activity
turns on, and `ℓu` controls the transition width.

The constants in this expression are prototype choices. The important
physical idea is the spatially varying activity, not the particular
numbers.

------------------------------------------------------------------------

# 10. Why Does `u ∂φ/∂x` Move the Cell?

The basic advection equation for a field transported with velocity `u`
is

$$
\frac{\partial\phi}{\partial t}
+
\mathbf u\cdot\nabla\phi
=
0.
$$

For motion only in the `x` direction,

$$
\mathbf u=(u_x,0),
$$

so

$$
\mathbf u\cdot\nabla\phi
=
u_x\frac{\partial\phi}{\partial x}.
$$

Thus

$$
\frac{\partial\phi}{\partial t}
+
u_x\frac{\partial\phi}{\partial x}
=
0
$$

describes translation of the phase-field pattern.

The term

$$
-u_x\frac{\partial\phi}{\partial x}
$$

is therefore the active transport/advection contribution to the
evolution equation.

------------------------------------------------------------------------

# 11. Complete Version 0 Equation

We now combine the two effects.

### Active transport

$$
-u_x\frac{\partial\phi}{\partial x}
$$

### Relaxation

$$
-\lambda\mu
$$

Therefore,

$$
\boxed{
\frac{\partial\phi}{\partial t}
=
-u_x\frac{\partial\phi}{\partial x}
-\lambda\mu
}
$$

and substituting the chemical potential,

$$
\boxed{
\frac{\partial\phi}{\partial t}
=
-u_x\frac{\partial\phi}{\partial x}
+
\lambda\kappa\nabla^2\phi
-
2\lambda\phi(1-\phi)(1-2\phi)
}
$$

This is the central equation of Version 0.

### Physical interpretation

$$
\underbrace{-u_x\frac{\partial\phi}{\partial x}}_{\text{active motion}}
\qquad+\qquad
\underbrace{-\lambda\mu}_{\text{passive relaxation}}.
$$

The first term moves/deforms the cell because of prescribed activity.
The second term tries to return the phase field toward a lower-energy
configuration.

------------------------------------------------------------------------

# 12. Why the Minus Sign Before `μ`?

We have

$$
\mu=\frac{\delta F}{\delta\phi}.
$$

The phase field should relax toward lower free energy, so the evolution
should move in the direction

$$
-\frac{\delta F}{\delta\phi}
=
-\mu.
$$

This is analogous to ordinary mechanics:

$$
\mathbf F=-\nabla U.
$$

Therefore,

$$
-\lambda\mu
$$

is the relaxation term.

The parameter `λ` controls how quickly this relaxation occurs.

------------------------------------------------------------------------

# 13. Explicit Euler Time Integration

The continuous equation can be written as

$$
\frac{\partial\phi}{\partial t}=R(\phi).
$$

Approximate the time derivative by

$$
\frac{\partial\phi}{\partial t}
\approx
\frac{\phi^{n+1}-\phi^n}{\Delta t}.
$$

Therefore,

$$
\phi^{n+1}
=
\phi^n
+
\Delta t\,R(\phi^n).
$$

This is the **explicit Euler method**.

The simulation therefore performs:

```text
current φ
   ↓
calculate ∂φ/∂x
   ↓
calculate ∇²φ
   ↓
calculate chemical potential μ
   ↓
calculate active advection
   ↓
update φ
   ↓
repeat
```

------------------------------------------------------------------------

# 14. Numerical Algorithm

At every time step:

1. Calculate the active velocity `u_x`.
2. Calculate the spatial derivative `∂φ/∂x`.
3. Calculate the Laplacian `∇²φ`.
4. Calculate the chemical potential `μ`.
5. Calculate the active advection term.
6. Update `φ` using explicit Euler.
7. Keep `φ` within the numerical range `[0,1]`.
8. Save snapshots when required.

This gives the basic computational loop of the prototype.

------------------------------------------------------------------------

# 15. Current Simulation Parameters

The notebook currently uses:

```text
N = 160
L = 80
dx = L / N
dt = 0.005
steps = 3000
```

Phase-field parameters:

```text
radius = 18
interface_width = 2
relaxation = 0.8
```

Activity:

```text
w0 = 2.0
```

These are **simulation units**, not directly calibrated biological
units.

------------------------------------------------------------------------

# 16. What Version 0 Captures

Version 0 demonstrates:

- a cell represented by a scalar phase field;
- a diffuse cell boundary;
- double-well phase separation;
- surface-tension-like interface relaxation;
- calculation of a chemical potential;
- finite-difference spatial derivatives;
- active transport;
- stronger activity near the leading edge;
- translation and deformation;
- explicit numerical time evolution.

------------------------------------------------------------------------

# 17. What Version 0 Does Not Capture

The research paper contains several additional physical ingredients that
are intentionally left for later versions.

## 17.1 Polarization field

The paper introduces

$$
\mathbf P(\mathbf r,t)
$$

to represent the coarse-grained orientation of actin filaments.

Version 0 does not dynamically solve for `P`.

## 17.2 Active stress / contractility

The paper introduces the active stress

$$
\boldsymbol{\sigma}^{\mathrm{active}}
=
-\zeta\,\mathbf P\mathbf P.
$$

This represents active force generation by the polar active material.
For actomyosin, the relevant activity is contractile.

Version 0 does not include active stress.

## 17.3 Hydrodynamics

The paper solves the fluid velocity field using incompressibility,

$$
\nabla\cdot\mathbf v=0,
$$

together with a Navier–Stokes equation containing active and passive
stresses.

Version 0 does not solve for fluid velocity.

## 17.4 Polarization dynamics

The paper evolves `P` and couples it to fluid deformation.

Version 0 replaces this entire mechanism with a prescribed velocity.

## 17.5 3D geometry and wall physics

The full model is three-dimensional and includes substrate boundary
conditions, slip, and polarization anchoring.

Version 0 is quasi-2D and does not include these effects.

------------------------------------------------------------------------

# 18. Connection to the Research Paper

The most important connection is the treatment of active velocity.

### Version 0

We prescribe

$$
u_x(x)
$$

directly.

### Full model

The paper uses an active contribution based on

$$
w_0\mathbf P,
$$

where `P` is the actin polarization and `w0` is related to the
treadmilling/self-propulsion speed.

So the development is:

```text
VERSION 0
────────────────────────

φ + prescribed active velocity
            ↓
       basic crawling


VERSION 1
────────────────────────

φ + polarization P
            ↓
       w0 P activity


VERSION 2
────────────────────────

φ + P + active stress
            ↓
         contractility


VERSION 3
────────────────────────

φ + P + stresses + fluid velocity
            ↓
      hydrodynamic feedback


FULL MODEL
────────────────────────

3D active polar fluid
+ substrate
+ anchoring
+ treadmilling
+ contractility
+ hydrodynamics
```

This staged approach makes it possible to validate the physics and
numerics one ingredient at a time.

------------------------------------------------------------------------

# 19. Physical Meaning of the Full Model

The complete paper can be viewed as a competition between:

$$
\text{passive interface relaxation},
$$

$$
\text{active treadmilling / self-propulsion},
$$

$$
\text{active contractility},
$$

and

$$
\text{hydrodynamic flow}.
$$

Different balances between these effects lead to different cell shapes.

The paper reports morphologies resembling lamellipodia, pseudopods,
phagocytic cups, and stationary “fried-egg” states.

------------------------------------------------------------------------

# 20. Roadmap

## Version 0 — Current

- [x] 2D phase field
- [x] Diffuse interface
- [x] Double-well free energy
- [x] Gradient/interface energy
- [x] Chemical potential
- [x] Finite-difference derivatives
- [x] Prescribed treadmilling velocity
- [x] Explicit Euler integration

## Version 1 — Polarization

- [ ] Introduce `P(x,y,t)`
- [ ] Represent local actin orientation
- [ ] Replace prescribed direction with `w0 P`
- [ ] Evolve polarization dynamically

## Version 2 — Active Contractility

- [ ] Add active stress
- [ ] Introduce contractility parameter `ζ`
- [ ] Couple polarization to active force generation

## Version 3 — Hydrodynamics

- [ ] Introduce fluid velocity `v`
- [ ] Solve incompressible flow
- [ ] Couple velocity to polarization
- [ ] Couple active stress to fluid flow

## Version 4 — Full 3D Model

- [ ] Extend to 3D
- [ ] Add substrate boundary conditions
- [ ] Add polarization anchoring
- [ ] Add spatially varying treadmilling
- [ ] Study morphology transitions

------------------------------------------------------------------------

# 21. Core Equations at a Glance

### Phase field

$$
\phi(x,y,t)
$$

### Initial droplet

$$
\phi(r)
=
\frac12
\left[
1-\tanh\left(\frac{r-R}{\varepsilon}\right)
\right],
\qquad
r=\sqrt{x^2+y^2}
$$

### Free energy

$$
F[\phi]
=
\int_\Omega
\left[
\frac{\kappa}{2}|\nabla\phi|^2
+
\phi^2(1-\phi)^2
\right]\,dA
$$

### Chemical potential

$$
\mu
=
-\kappa\nabla^2\phi
+
2\phi(1-\phi)(1-2\phi)
$$

### Active transport

$$
-u_x\frac{\partial\phi}{\partial x}
$$

### Complete evolution equation

$$
\frac{\partial\phi}{\partial t}
=
-u_x\frac{\partial\phi}{\partial x}
-\lambda\mu
$$

### Central difference

$$
\frac{\partial\phi}{\partial x}
\approx
\frac{\phi_{i+1,j}-\phi_{i-1,j}}{2\Delta x}
$$

### Laplacian

$$
\nabla^2\phi_{i,j}
\approx
\frac{
\phi_{i+1,j}
+\phi_{i-1,j}
+\phi_{i,j+1}
+\phi_{i,j-1}
-4\phi_{i,j}
}{
\Delta x^2
}
$$

### Explicit Euler

$$
\phi^{n+1}
=
\phi^n
+
\Delta t
\left[
-u_x\frac{\partial\phi}{\partial x}
-\lambda\mu
\right]^n
$$

------------------------------------------------------------------------

# 22. Final Takeaway

The central idea of Version 0 is:

$$
\text{phase field }\phi
\;\longrightarrow\;
\text{free energy }F
\;\longrightarrow\;
\text{chemical potential }\mu
$$

↓

$$
\text{active advection}
\;\longrightarrow\;
\text{motion/deformation}
\;\longrightarrow\;
\text{relaxation}
\;\longrightarrow\;
\text{repeat}
$$

This gives a simple physics-based crawling cell and provides the
foundation for adding the paper’s polarization field, active
contractility, and hydrodynamics in later versions.
