---
layout: page
title: Scene Graph
description: "Creating a simple scene with scene graphs <span style='display:block; margin-top:0.5em;'></span><b>C++, QT</b>"
img: assets/img/SG2.gif
# redirect: https://unsplash.com
importance: 11
category: work
---

In this project, I created a simple scene using a scene graph.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/SG1.gif" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    You can see the scene graph's heirarchy in the tree widget box to the right. Note that when you click on a node, you can check the type of node it is by looking at which spinboxes are active.
</div>

A scene graph is a tree of nodes that hold transformation data, geometry data, and pointers to other nodes. As you traverse through a scene graph from the root, each transformation matrix gets applied post-order. Once you reach a node that has geometry data, the geometry is drawn with all of the previous matrix transformations applied. The GUI was made using QT Creator.

Scene graphs are the basis of various scene data structures in games, mesh editor applciations, and animated films.
