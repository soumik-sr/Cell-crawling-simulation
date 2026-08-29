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

1.  **Passive relaxation / surface-tension-like physics:** the cell
    boundary tends to become smooth.
2.  **Active treadmilling:** activity near the leading edge produces
    directed motion.

Conceptually:

``` text
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

\[ (x,y,t). \]

We interpret it approximately as

\[ \]

The transition is smooth:

``` text
inside              interface              outside

φ ≈ 1                  0 < φ < 1             φ ≈ 0

███████████████████▓▓▓▓▓▓▓▓▓░░░░░░░░░░░░░░
```

This is a **diffuse-interface phase-field description**.

The approximate cell boundary is the contour

\[ . \]

### Why use a phase field?

A crawling cell has a boundary that continuously moves and deforms.
Tracking that boundary explicitly is difficult. With a phase field, the
boundary is simply the region where `φ` changes rapidly.

------------------------------------------------------------------------

# 3. Initial Cell

The prototype starts with a smooth circular droplet:

\[ \]

where

\[ r=, \]

`R` is the initial radius and `ε` controls the interface width.

## Why `tanh`?

For points deep inside the cell,

\[ rR \]

so

\[ () \]

and therefore

\[ . \]

At the boundary,

\[ r=R \]

and

\[ (0)=0, \]

so

\[ . \]

Outside,

\[ rR \]

so

\[ () \]

and

\[ . \]

Therefore,

\[ \]

------------------------------------------------------------------------

# 4. Free Energy

The phase field is given a free energy:

\[ \]

There are two main contributions.

## 4.1 Gradient / interface energy

\[ \]

Inside and outside the cell, `φ` is almost constant, so its gradient is
small. Near the boundary, `φ` changes rapidly, so the gradient is large.

Thus this term assigns an energetic cost to the interface.

Physically:

\[ \]

This is the phase-field analogue of surface-tension-like behaviour.

## 4.2 Double-well potential

\[ \]

It has minima at

\[ V(0)=0,V(1)=0. \]

Thus the preferred states are

\[ \]

These correspond to outside and inside the cell.

So the two terms have complementary jobs:

\[ \]

\[ \]

Together they produce a stable diffuse droplet.

------------------------------------------------------------------------

# 5. Chemical Potential

To determine how the phase field relaxes, we take the functional
derivative of the free energy:

\[ \]

For the free energy above,

\[ \]

This is the **chemical-potential-like quantity** used in the prototype.

## Derivation of the double-well contribution

Starting with

\[ V()=<sup>2(1-)</sup>2, \]

differentiate:

\[ = \[<sup>2(1-)</sup>2\]. \]

Using the product rule,

\[ = (1-)<sup>2-</sup>2(1-). \]

Factor:

# \[

(1-)(1-2). \]

Hence,

\[ \]

which gives the second part of `μ`.

------------------------------------------------------------------------

# 6. Why Does the Laplacian Appear?

The gradient part of the free energy is

\[ F_g= \|\|^2,dA. \]

When taking the functional derivative, integration by parts gives

\[ \]

Therefore,

\[ \]

The important physical interpretation is:

\[ \]

It naturally appears because we are penalizing spatial gradients of the
phase field.

------------------------------------------------------------------------

# 7. Numerical Laplacian

A computer works on a grid rather than a continuous function.

For the 2D Laplacian,

\[ ^2 =  + . \]

We approximate it using neighbouring grid points:

\[ \]

This compares the current point with its four nearest neighbours.

For example:

``` text
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

\[ . \]

We use the central difference:

\[ \]

This is a second-order finite-difference approximation.

------------------------------------------------------------------------

# 9. Active Treadmilling

A passive phase-field droplet would relax toward an equilibrium shape. A
crawling cell is active.

In Version 0, we do **not yet solve for the actin polarization field**.
Instead, we prescribe an active velocity

\[ u_x(x). \]

The velocity is intentionally stronger toward the `+x` leading edge.

Conceptually:

``` text
rear                                  front

|---------------------------------------->
 weak activity                    strong activity
```

The notebook uses

\[ u_x(x) = w_0 ) \]. \]

The constants in this expression are prototype choices. The important
physical idea is the spatially varying activity, not the particular
numbers.

------------------------------------------------------------------------

# 10. Why Does `u ∂φ/∂x` Move the Cell?

The basic advection equation for a field transported with velocity `u`
is

\[ \]

For motion only in the `x` direction,

\[ u=(u_x,0), \]

so

\[ u = u_x. \]

Thus

\[ \]

describes translation of the phase-field pattern.

The term

\[ \]

is therefore the active transport/advection contribution.

------------------------------------------------------------------------

# 11. Complete Version 0 Equation

We now combine the two effects.

### Active transport

\[ -u_x \]

### Relaxation

\[ - \]

Therefore,

\[ \]

and substituting the chemical potential,

\[ \]

This is the central equation of Version 0.

### Physical interpretation

\[ \]

The first term moves/deforms the cell because of prescribed activity.
The second term tries to return the phase field toward a lower-energy
configuration.

------------------------------------------------------------------------

# 12. Why the Minus Sign Before `μ`?

We have

\[ =. \]

The phase field should relax toward lower free energy, so the evolution
should move in the direction

\[ -. \]

This is analogous to ordinary mechanics:

\[ F\_{}=-. \]

Therefore,

\[ \]

is the relaxation term.

The parameter `λ` controls how quickly this relaxation occurs.

------------------------------------------------------------------------

# 13. Explicit Euler Time Integration

The continuous equation can be written as

\[ =R(). \]

Approximate the time derivative by

\[ . \]

Therefore,

\[ \]

This is the **explicit Euler method**.

The simulation therefore performs:

``` text
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

1.  Calculate the active velocity `u_x`.
2.  Calculate the spatial derivative `∂φ/∂x`.
3.  Calculate the Laplacian `∇²φ`.
4.  Calculate the chemical potential `μ`.
5.  Calculate the active advection term.
6.  Update `φ` using explicit Euler.
7.  Keep `φ` within the numerical range `[0,1]`.
8.  Save snapshots when required.

This gives the basic computational loop of the prototype.

------------------------------------------------------------------------

# 15. Current Simulation Parameters

The notebook currently uses:

``` text
N = 160
L = 80
dx = L / N
dt = 0.005
steps = 3000
```

Phase-field parameters:

``` text
radius = 18
interface_width = 2
relaxation = 0.8
```

Activity:

``` text
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

\[ \]

to represent the coarse-grained orientation of actin filaments.

Version 0 does not dynamically solve for `P`.

## 17.2 Active stress / contractility

The paper introduces the active stress

\[ \]

This represents active force generation by the polar active material.
For actomyosin, the relevant activity is contractile.

Version 0 does not include active stress.

## 17.3 Hydrodynamics

The paper solves the fluid velocity field using incompressibility,

\[ \]

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

\[ \]

directly.

### Full model

The paper uses an active contribution based on

\[ \]

where `P` is the actin polarization and `w0` is related to the
treadmilling/self-propulsion speed.

So the development is:

``` text
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

\[ \]

\[ \]

\[ \]

\[ \]

and

\[ \]

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

\[ \]

### Initial droplet

\[ \]

### Free energy

\[ \]

### Chemical potential

\[ \]

### Active transport

\[ \]

### Complete evolution equation

\[ \]

### Central difference

\[ \]

### Laplacian

\[ \]

### Explicit Euler

\[ \]

------------------------------------------------------------------------

# 22. Final Takeaway

The central idea of Version 0 is:

\[ \]

↓

\[ \]

↓

\[ \]

↓

\[ \]

This gives a simple physics-based crawling cell and provides the
foundation for adding the paper’s polarization field, active
contractility, and hydrodynamics in later versions.
