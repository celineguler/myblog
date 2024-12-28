---
layout: post
title: "Wick rotation and Minkowski space"
---

  <!-- MathJax Script -->
  <script type="text/javascript" async
    src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
  </script>


A general form of Wick rotation is the "Weyl's unitary trick" or [Unitarian trick][uni] which is related with groups like literally everything else in physics.

In quantum field theory, Wick rotation refers to a transformation from real time to imaginary time. This is done by replacing __t → iτ__, where __τ__ is the new imaginary time. The goal of this transformation is often to simplify the mathematical treatment of quantum systems by turning oscillatory integrals (involving $$ e^{-iEt / ℏ} $$) into exponentially decaying functions (like $$ e^{-Eτ / ℏ} $$), which are easier to handle apperantly. So transformation looks like this:

$$
\large e^{-iEt / \hbar} => e^{-E / k_B T}
$$

Wick rotation consideres __t__ in quantum mechanics as an imaginary time, so

$$
\large t = -iτ
$$

Here, we no longer have an oscillatory behavior but an exponentially decaying factor __τ__ . And this is the connection to statistical mechanics by,

$$
\large e^{-Eτ / \hbar} => e^{-E / k_B T}
$$

<br>

In order to understand Wick rotation better, we can introduct a little bit of quantum field theory without resorting to complex branches of mathematics, especially groups. A field in physics defined as a physical quantity with components of scalars, vectors and tensors that has a value for each point in space and time. A classical field is a function of all space and time coordinates, like __E__(*__r__*, *t*), __B__(*__r__*, *t*). They have infinite amount of degrees of freedom since they are *continuous*. A field has to specify a value for each of these points, leading to infinitely many independent variables to describe its state. These independent variables correspond to the degrees of freedom.

When a classical field is quantized (e.g., in quantum field theory), the infinite degrees of freedom translate into an infinite number of quantum states or modes, often corresponding to particles or quanta of the field. So there it is, *__quantizing classical fields__* is the central idea of *__quantum field theory__*.

So the question arises: *How to quantize a classical field?*

<br>

#### __QUANTIZING CLASSICAL FIELDS__

In classical field theory, scalar fields *ϕ*(*__r__*, *t*) and vector field *__A__*(*__r__*, *t*) are functions of space and time and motions are described by Euler-Lagrange equations.

Quantizing these fields starts with __Canonical formalism__. For a scalar field *ϕ*(*__r__*, *t*), the conjugate momentum is:

$$
\large \pi (r, t) = {\nabla L} / {\nabla (\nabla_t \phi}
$$







[uni]: https://en.wikipedia.org/wiki/Unitarian_trick