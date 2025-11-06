---
layout: page
title: 2D Ball Collision Simulation
description: Conservation of Momentum, Convservation of Kinetic Energy, Vector Projections
img: assets/img/BC_2.gif
importance: 12
category: work
---

The purpose of this project is to demonstrate how to determine the change of velocity of balls colliding in a 2D space. We will be implementing the law of Conservation of Momentum and law of Conservation of Kinetic Energy. We will also assume that the space where the objects are moving is elastic (no loss of energy, no friction, etc).

Since we are assuming that our objects are in an elastic world, the total
momentum of two colliding objects is preserved after the collision.
Therefore, if we set m1, m2 as the mass of each object and v1, v2 as their
velocity, in a one-dimensional space the following equation should be true.
v1’, v2’ are the velocity of the objects after the collision.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BC1.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Kinetic Energy should also be preserved, so:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BC2.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Combining the two equations gives us the following velocities after collision:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BC3.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

However, these apply only when the two objects are moving in a one-dimensional space. For two-dimensional space, we must implement normal unit vectors and tangent unit vectors.

Let’s visualize two objects of different mass colliding in 2D space.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BC4.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Now let's take a closer look at the moment of impact.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BC5.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Assume the large circle is C₁ and the small circle is C₂.
Their mass: m₁, m₂
Velocity vectors: v₁, v₂
Center coordinates: x₁, x₂

The unit normal vector passes through both centers: n̂ = (x₂ − x₁) / len(x₂ − x₁)

The unit tangent vector is orthogonal to n̂:
t̂ = [−(y-component of n̂), (x-component of n̂)]

We project v₁ and v₂ to these unit vectors:
v₁n, v₁t, v₂n, v₂t

Projection formula:
p = a(aᵀb / aᵀa),
but since a is a unit vector, we just use the dot product a•b.
v₁n + v₁t = v₁, v₂n + v₂t = v₂

The tangential components (v₁t, v₂t) remain unchanged after collision; only the normal components change, using 1D collision formulas:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BC6.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Then:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BC7.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Final velocities:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BC8.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Below is the collide function that takes in two Ball instances as parameters,
and calculates their next velocity based on their current velocity and mass.
It is the same process mentioned above, but written in code form. In this simulation, the mass of each ball has been set to its area.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BC9.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

According to the Youtube channel ‘Reducible’, these calculations can be simplified as follows.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BC10.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    “Building Collision Simulations: An Introduction to Computer Graphics” YouTube, uploaded by Reducible, 20 Jan
2021, https://www.youtube.com/watch?v=eED4bSkYCB8
</div>

Simplified Function:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BC11.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

References:

- “Building Collision Simulations: An Introduction to Computer Graphics” YouTube, uploaded by
  Reducible, 20 Jan. 2021, https://www.youtube.com/watch?v=eED4bSkYCB8
- Berchek, Chad. “2-Dimensional Elastic Collisions without Trigonometry”, Vobarian Software,
  3 Aug. 2009, https://www.vobarian.com/collisions/2dcollisions2.pdf

{% raw %}

{% endraw %}
