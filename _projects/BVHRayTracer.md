---
layout: page
title: BVH Accelerated Ray Tracer
description: "Bounding Volume Hierarchy, Acceleration Structures <span style='display:block; margin-top:0.5em;'></span><b>C++, GLM, QT</b>"
img: assets/img/BVHo.png
importance: 10
category: work
---

## Why We Need Acceleration Structures

A naive ray tracer intersects every ray against every primitive in the scene. This is fine for a few spheres, but real meshes routinely contain thousands to millions of triangles — *Awakening* by Michelangelo, for example, has been digitized into hundreds of millions of triangles. Testing each ray against every triangle is hopelessly slow.

The mesh scene used for this assignment contains 5,172 triangles, and a brute force renderer takes anywhere from one to five minutes to finish a single image. After building a BVH, the same image renders in under five seconds.

The trick is to **test groups of elements at once**. If a ray misses a bounding volume that contains many primitives, we can reject all of them in a single test.


## Bounding Volume Hierarchy

Rather than partitioning *space* (like grids, octrees, and k-D trees), a BVH partitions *objects*. Every node owns an axis-aligned bounding box that encloses all the primitives in its subtree. The root box covers the entire scene, and each leaf holds a single primitive.

Why axis-aligned boxes? Tighter shapes exist — oriented bounding boxes squeeze geometry more snugly, and spheres have very fast intersection tests — but both are hard to compute. AABBs are the practical sweet spot.

The key tradeoff vs. spatial partitioning: BVH bounding boxes can **overlap**, but every primitive lives in exactly one leaf. k-D trees have non-overlapping nodes but a single primitive can span multiple leaves. Concretely:

- BVH: faster to build, friendly to dynamic scenes, no duplication of primitives, but doesn't explicitly skip empty space.
- k-D tree: tighter spatial bounds, but a moving primitive may need to be re-inserted into many nodes.

## Construction: Top-Down Recursive Build

The build is recursive. At each step:

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BVH1.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BVH2.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BVH3.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

1. Compute a bounding box that encloses all primitives in the current set.
2. If the set is small enough, make a leaf node and store the primitive.
3. Otherwise, choose a split axis, sort by that axis, partition the primitives, and recurse on each half.

The skeleton lives in `recursiveBVHBuild`, called from `Mesh::buildBVH`. A `Union` helper computes a bounding box that encloses two input boxes — useful when combining child bounds back up the tree. A correct build produces exactly one leaf per triangle, so for the 5,172-triangle Mario mesh the leaf count should match.

### Equal Primitive Count Heuristic
The simplest split: pick the longest axis, sort primitives along it by their centroid, and partition into equal halves.

Pros: very fast and produces a balanced tree.
Cons: ignores primitive size, so the resulting bounding boxes can overlap heavily.

The optimal tree structure is actually NP-hard, so every real implementation is making approximations:

- Split on midpoint: fastest, but not much better than an octree.
- Split on median: usually pretty good.
- Split on approximate median (binned): much faster construction, almost as good.

## Traversal: Intersecting the BVH

Intersection is also recursive — a depth-first search through the tree.

1. Check if ray hits bounding box
2. If the bounding box is a leaf node: Intersect the node's primitives with the ray and return intersection
3. If it is not, test ray against L and R chidlren, and only recurse on those that are hit
4. If L and R bboxes do not overlap, recurse on the loser child first, and only recurse on the farther child if the closer one missed
5. If L and R overlap, must recurse on both, and return the closer of the two child intersections
6. If not, return no intersection

The "closer child first" optimization matters. If the near child returns a hit, that hit might be closer than anything in the far child — but only if the boxes don't overlap, since overlapping boxes can contain primitives that interleave in depth. When the boxes are disjoint we can short-circuit the far traversal entirely.

`BVHBounds::intersect` returns just a float (the parametric `t` of the AABB hit) rather than a full Intersection object, since we don't need surface data while walking the tree — only the leaves produce real intersections.

## Result

Brute-force ray-mesh intersection on the 5,172-triangle Mario mesh: **147.99 seconds** per frame.
With an equal-primitive-count BVH: **2.27 seconds** for the same image.

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BVHx.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BVHo.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

The whole point: a BVH replaces O(n) primitive tests per ray with something close to O(log n), and that's the difference between a ray tracer that's a science experiment and one that can actually render a frame.

Slide images from:
CIS 5610 Acceleration Structures - Adam Mally

https://pbr-book.org/3ed-2018/Primitives_and_Intersection_Acceleration/Bounding_Volume_Hierarchies
