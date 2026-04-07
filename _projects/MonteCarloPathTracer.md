---
layout: page
title: Monte Carlo Path Tracer
description: "Path Tracing, BRDFs, PDFs <span style='display:block; margin-top:0.5em;'></span><b>C++, GLM, QT</b>"
img: assets/img/PathTracerCover.png
importance: 3
category: work
---

## Physically Based Rendering - Path Tracer

Takes in multiple bounces unlike the ray tracer which snaps the intersection point to a light source.

The entire algorithm is based on estimating the "Light Transport Equation".
We discretely estimate this value via Monte Carlo since it is not possible to compute the integral of the equation on a computer.
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/PathTracer2_LTEImage.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/PathTracer1_LTE.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Each term has the following meaning:

Lo: Radiance from a point p and direction wo

Le: Emitted radiance from a point (nonzero only if p is a light source)

f: BRDF of surface at P

L: Radiance from direction wi, at point p

V: visibility between p'and p

absdot: Lambert's law

For some discrete subset of S, choose n ws to sample and average their reflected light energies

## Naive Integrator

The first integration method simply bounces rays randomly within a cosine-weighted hemisphere, and blindly hopes that it will eventually reach a light source.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/PathTracer3_Naive.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

The result is physically correct but very noisy, and takes a long time to converge.

## Direct Lighting with Light Source Importance Sampling
Rather than hoping a random ray hits a light, this integrator samples a point directly on a light source and computes the illumination from that direction.
We cast a shadow-feeler ray to check if the path to the light is occluded.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Pathtracer4_Direct.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

The result converges much faster than the naive method, as each pixel is snapped to a light source, which always contributes light. However, notice how there is no indirect lighting supported (no red/green light from the sides of the walls).


## Direct MIS(Multiple Importance Sampling)
Direct light sampling sturggles with glossy surfaces, and BSDF sampling struggles with small light sources. 

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/PathTracer6_BSDFvsDirect.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Direct(small light) vs glossy surfaces: you pick a random point on the light, but a glossy surface only reflects light in a narrow cone. Most of the random points yous ample on the light will be outside of the cone, so the BSDF evalutes to nearly zero for all of them.

BSDF(big lobe) vs small light sources: You sample a direction based on the surface's BSDF, for a glossy surface, that is all directions within a reflection cone. If the light source is tiny, the chance that the sampled direction actually hits the light is very low. You keep missing the light

However, simply taking the results of the estimator and averaging them is not enough because variance is additive. Say estimator A has a variance spike of 10,000 at some sample, and estimator B is well-behaved with variance of 1 at that same sample.

If you average them: (E_a + E_b) / 2
The variance becomes (10,000 + 1) / 4 = 2,500. You divided by 4, but 2,500 is still terrible. The spike from estimator A is smaller but it's still there. You can't dilute away a massive spike by mixing it with a good estimate.

MIS combines both strategies using the Power Hueristic, which weights each sample by the square of its pdf divided by the sum of square PDFs from both strategies.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/PathTracer8_BalanceHeuristic.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/PathTracer9_BalanceResult.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

The problem is when pf(x) is tiny when the brdf value is large as this will produce a spike. This sum is small only if pf(x) is small AND pg(x) is small, both strategies think this sample is unlikely. This is rare.
The sum is large if pf(x) is large OR pg(x) is large — either strategy thinks this sample is reasonable. That's common.

So MIS is essentially saying: a sample is safe as long as at least one strategy thinks it's a good sample. You only get a spike if both strategies agree it's a bad one, which almost never happens because the two strategies are designed to cover each other's weaknesses.

This automatically favors whichever stragtegy is more effective for a given surface-light configuration.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/PathTracer7_DirectMIS2.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

## Fragment Processing

Before we set a fragment as a pixel of a final render texture (in this case it was a QImage), we must conduct a z-buffer test, and only replace the pixel value if the fragment's z value is smaller than the current one. In this implementation we used a 1D array for faster access.

## Global Illumination
The final integrator computes MIS direct lighting at every bounce, while separately using BSDF sampling to determine the ray's next bounce direction for indirect illumination.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LiFullDrawing1.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LiFullDrawing2.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

To avoid double counting Le, Le is only added for rays coming directly from the camera or from specular surfaces (MIS already accounts for direct lighting at diffuse and glossy intersections).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/PathTracer10_Global.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

The result converges much faster.

## Materials

Diffuse(Lambertian): f = albedo /pi , sampled with cosine-weighted hemishpere sampling

Specular Reflection: wi is wo reflected about the surface normal. The PDF is a dirac delta, so in the actual implementation we pre-cancel it out to 1.

Specular Transmission: wi is computed via Snell's law. The function checks for total internal reflection and tracks whether the ray is entering or exiting the object. (Less dense > denser medium : slows down and bends closer to the normal, Denser > less dense medium : speeds up and bends away from the normal)

ni*sinthetaI = nt * sinthetaT
Fresnel: probability that light reflects at a given angle (transparent when you look straight at it, but reflective at grazing angles).

Dielectric(Glass): Uses the Fresnel reflectance coefficient to probabilistically choose reflection and tansmission at each intersection.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/PathTracer12_Ice.png" title="No AA" class="img-fluid rounded z-depth-1" %}
    </div>
</div>


<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/PathTracer13_ChessBoard.png" title="4x4 AA" class="img-fluid rounded z-depth-1" %}
    </div>
</div>


Slide images from: 
CIS 5600 MIS - Adam Mally
CIS 5600 Path Tracing Basics - Adam Mally