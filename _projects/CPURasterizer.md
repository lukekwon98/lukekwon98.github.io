---
layout: page
title: CPU Rasterizer
description: "Rasterization, Z-Buffering, Anti-Aliasing, Bresenham's Algorithm <span style='display:block; margin-top:0.5em;'></span><b>C++, GLM, QT</b>"
img: assets/img/RatserizerAAToon.png
importance: 3
category: work
---

## Vertex Processing
Create a camera class, and create a view & projection matrix from that camera
Apply view & projection matrices to mesh to convert it into screen space.

## Rasterization
Loop through each triangle of the mesh and test pixel row intersections.
Create a bounding box around each triangle to optimize row intersection tests.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Raster1.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>


## Barycentric Interpolation
Use Barycentric Interpolation to determine attributes of each fragment.
Make sure to use persepctive-correct Baryentric interpolation in 3D, since
the distance between vertices is no longer linear after perspective divide.
We will not only be interpolating color values but also UV coordinates and normals for texturing and lighting.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Raster2_barycentric.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

## Fragment Processing

Before we set a fragment as a pixel of a final render texture (in this case it was a QImage), we must conduct a z-buffer test, and only replace the pixel value if the fragment's z value is smaller than the current one. In this implementation we used a 1D array for faster access.

## Results
Results have been shaded using the Lambertian reflection model

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Raster3.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Toon shading implemented by clamping the lambert value to constant values depending on their range.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Raster4_Toon.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

## Supersampling Anti Aliasing
Each pixel is subdivided into a grid of subpixels. You render at that higher resolution, then average the subpixel colors to get the final pixel color. Triangle edges that would normally look jagged now blend smoothly. This image was smoothed using 2x2 supersampling.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Raster3.png" title="No AA" class="img-fluid rounded z-depth-1" %}
    </div>
</div>


<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Raster5_AA.png" title="4x4 AA" class="img-fluid rounded z-depth-1" %}
    </div>
</div>


Triangle images from: CIS 5600 Rasterization - Adam Mally