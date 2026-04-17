---
layout: page
title: Half-Edge Mesh Editor
description: "Half-Edges, Face Triangulation, Catmull-Clark Subdivision <span style='display:block; margin-top:0.5em;'></span><b>C++, OpenGL, QT</b>"
img: assets/img/HE_7.gif
importance: 5
category: work
related_publications: false
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/HE_9.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Naive Connectivity - each element has a pointer to all adjacent elements
</div>

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

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/HE_8.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Half-edges are a useful way of representing mesh data, since they provide topoligical information such as adjacency and connectivity, in addition to geometric data. Say we want to traverse the edges of a face, in a VBO like data structure, this is not possible. A half edge data structures allows us connectivity.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/HE_1.gif" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Traversing through a complicated mesh with half-edges
</div>

A regular mesh stores faces as lists of vertex indices. A half-edge mesh stores connectivity explicitly. Each edge is split into two half-edges, one per direction. 

Each half-edge stores:

- Which vertex it points to
- Its sym (the opposite half-edge on the same geometric edge)
- Its next (the next half-edge going around the same face)
- Which face it belongs to


## Debugging features
User-selected vertices/edges/faces are highlighted. This was implemented by a separate class that inherits from drawable. GL_DEPTH_TEST is disabled when drawing these parts so the user can clearly see which component they have selected, and they can traverse through the mesh via the following.

- N: NEXT half-edge of the currently selected half-edge
- M: SYM half-edge of the currently selected half-edge
- F: FACE of the currently selected half-edge
- V: VERTEX of the currently selected half-edge
- H: HALF-EDGE of the currently selected vertex
- Shift + H: HALF-EDGE of the currently selected face

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

## Qt Widgets updated accordingly after Catmull-Clark Subdivision

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


Half-Edge Illustration source: CIS 5600 Mesh Data Structures - Adam Mally

{% raw %}

{% endraw %}
