---
layout: post
title: "Wick rotation and Minkowski space"
---

  <!-- MathJax Script -->
  <script type="text/javascript" async
    src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
  </script>


A general form of the Wick's rotation is the "Weyl's unitary trick" or [Unitarian trick][uni] which is related with groups like literally everything else in physics.

In quantum field theory, Wick rotation refers to a transformation from real time to imaginary time. This is done by replacing __t → iτ__, where __τ__ is the new imaginary time. The goal of this transformation is often to simplify the mathematical treatment of quantum systems by turning oscillatory integrals (involving $$ e^{-iEt / ℏ} $$) into exponentially decaying functions (like $$ e^{-Eτ / ℏ} $$), which are easier to handle. So transformation looks like this:

$$
\large e^{-iEt / \hbar} => e^{-E / k_B T}
$$

Wick rotation consideres __t__ in quantum mechanics as an imaginary time, so

$$
\large t = -iτ
$$

Here, we no longer have an oscillatory behavior but an exponentially decaying factor __τ__ . And this is the connection to statistical mechanics by,

$$
\large e^{-E / \hbar τ} => e^{-E / k_B T}
$$












[uni]: https://en.wikipedia.org/wiki/Unitarian_trick