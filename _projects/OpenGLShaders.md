---
layout: page
title: OpenGL Shaders
description: "Surface Shaders, Post-Process Shaders, Worley Noise, Perlin Noise <span style='display:block; margin-top:0.5em;'></span><b>C++, GLSL, QT</b>"
img: assets/img/GL_2.gif
importance: 2
category: work
giscus_comments: false
---

For this project, I wrote multiple shaders in GLSL and constructed a polar spherical camera model to be able to navigate around the scene.

### Polar Spherical Camera

I implemented functions that rotate the camera's eye position around the up vector and the local right vector, as well as functions that allow panning and zooming.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/GL1.gif" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

### Surface Shaders

#### Blinn-Phong

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/GL1.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

#### Matcap

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/GL2.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/GL5.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/GL7.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Red Wax, Plastic, Gold
</div>

#### Vertex Deformation Shader

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/GL2.gif" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The model normalizes all of the vertex positions to create a sphere, and interpolates from its normal form over time. Then, it uses each fragment's UV coordinate to calculate its distance from the center, and gives that offest to a sine function to generate the rainbow wave.
</div>

### Post-Process Shaders

In order to create post-process shader effects, you first have to divert the output of the inital render to a texture within the frame buffer.

#### Sobel Filter

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/GL3.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

#### Gaussian Blur

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/GL4.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

#### Custom Worley Noise Shader

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/GL5.gif" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

#### Custom Perlin Noise Shader

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/GL3.gif" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
