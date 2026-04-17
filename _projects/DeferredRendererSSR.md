---
layout: page
title: Deferred Renderer with SSR
description: "Deferred Rendering, Screen Space Reflections, OpenGL <span style='display:block; margin-top:0.5em;'></span><b>C++, OpenGL, Nsight Graphics</b>"
img: assets/img/Deferred_Pos.png
importance: 1
category: work
---

## Deferred Rendering

## Motivation

Forward rendering shades every fragment the rasterizer produces, even ones that get overwritten later by closer geometry. It also scales poorly with light count: the cost grows as O(objects × lights), because every object has to evaluate every light in its shader.

Deferred rendering decouples *geometry* from *lighting*. The first pass writes surface attributes into textures (the G-Buffer), and the second pass reads those textures and computes lighting once per screen pixel — regardless of how much geometry was behind it. This makes the lighting cost scale with screen resolution instead of scene complexity, and enables techniques that need access to neighboring pixel data (like SSR).

## Geometry Pass & G-Buffer

The first pass renders the scene into multiple render targets simultaneously using MRT (Multiple Render Targets). No shading happens here — each fragment just writes its raw attributes to the appropriate buffer.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Deferred_nsight.png" title="G-Buffer layout" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

```glsl
layout (location = 0) out vec4 gb_WorldSpacePosition;
layout (location = 1) out vec4 gb_Normal;
layout (location = 2) out vec4 gb_Albedo;
layout (location = 3) out vec4 gb_Metal_Rough_Mask;
```

The G-Buffer layout I used stores:

- **World-space position** — the starting point for reflection ray marching
- **World-space normal** — used to compute reflection directions and lighting
- **Albedo** — base surface color
- **Metallic / Roughness / Mask** — packed into a single RGBA texture. The mask channel flags whether a pixel contains geometry at all, so downstream passes can early-out on empty background pixels
- **PBR** — the fully shaded color from the environment map (via split-sum IBL), pre-computed here so the SSR pass can read reflected color directly at ray-hit points

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Deferred_Pos.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Deferred_Normals.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Deferred_Albedo.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Deferred_MRM.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Deferred_PBR.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Deferred_Blurred1.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Because all of these are written in a single pass, the vertex shader only runs once per vertex regardless of how many attributes we need downstream.

## Lighting Pass

The second pass draws a single full-screen quad. The fragment shader runs once per screen pixel, samples the G-Buffer textures at the current fragment's UV, reconstructs the surface properties, and computes lighting.

Because lighting now runs per *visible* pixel instead of per *rasterized* fragment, overdraw stops being a shading problem — it's only a bandwidth problem for the G-Buffer writes. Adding more lights also becomes cheap, since each additional light is just another loop iteration inside the same full-screen pass.

## Screen Space Reflections

With the G-Buffer already holding position, normal, and color for every visible pixel, SSR becomes tractable: we can trace reflection rays through screen space itself, using the depth buffer as a cheap proxy for scene geometry.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Deferred_Reflect.png" title="SSR result" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

For each fragment, I reflect the view direction around the surface normal to get a reflection ray, then march that ray in screen space. At each step, I compare the ray's depth to the G-Buffer depth at that screen position — if the ray has gone behind a surface, we've hit geometry, and the pixel color at the intersection becomes the reflected color.

So basically we start off in world space by sampling the world space coordinates via the position gBuffer

**World Space** -> compute wi by reflecting wo from normal -> compute start and end of ray march in world using max_distance -> convert start and end position to **Pixel Space** (but store view space z coordinate of ray start and end) -> start marching along the pixel space -> for each march, use t to compute perspective correct interpolation to get the z coordinate in view space -> sample and compute **View Space** position of geometry located in the current pixel and compare with current ray's z coordinate -> if the current ray's z coordinate is smaller than the geometry's z coordinate, geome"
try detected, sample color using uv of current ray's pixel location -> store in texture, with blended alpha values depending on position

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Deferred_PixelMarch.png" title="SSR result" class="img-fluid rounded z-depth-1" %}
    </div>
</div>


## Result
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Deferred_finish.png" title="G-Buffer layout" class="img-fluid rounded z-depth-1" %}
    </div>
</div>


References:

UPenn CIS 5610 Deferred Rendering Course Notes - Adam Mally

UPenn CIS 5610 Screen-Space Reflections Course Notes - Adam Mally 
