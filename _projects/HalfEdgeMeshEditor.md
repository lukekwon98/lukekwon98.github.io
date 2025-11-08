---
layout: page
title: Half-Edge Mesh Editor
description: "Half-Edges, Face Triangulation, Catmull-Clark Subdivision <span style='display:block; margin-top:0.5em;'></span><b>C++, OpenGL, QT</b>"
img: assets/img/HE_7.gif
importance: 1
category: work
related_publications: false
---

In this project, I created and displayed mesh objects using half-edges to represent data read from .obj files. I also included features such as edge splitting, triangulation, and Catmull-Clark subdivison.

```c++
class HalfEdge {
private:
    HalfEdge* next;
    HalfEdge* sym;
    Face* face;
    Vertex* vert;
};
```

Half-edges are a useful way of representing mesh data, since they provide topoligical information such as adjacency and connectivity, in addition to geometric data.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/HE_1.gif" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Traversing through a complicated mesh with half-edges
</div>

Given adjacency information, we can now easily add or modify elements of the mesh.

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/HE_2.gif" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/HE_3.gif" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/HE_4.gif" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Splitting an edge / Triangulating a polygon of any number of edges
</div>

Half-edges also allow us to perform Catmull-Clark subdivision.
This is done by first calculating the centroids of each face, computing the smoothed midpoint of each edge, smoothing the location of the original vertices, and quadrangulating each original face into the number of vertices that the face originally had.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/HE_5.gif" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/HE_6.gif" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The number of vertices, half-edges, and faces are updated accordingly after subdivision.
</div>

{% raw %}

{% endraw %}
