---
layout: page
title: BVH Accelerated Ray Tracer
description: "Bounding Volume Hierarchy, Acceleration Structures <span style='display:block; margin-top:0.5em;'></span><b>C++, GLM, QT</b>"
img: assets/img/BVHCover.png
importance: 10
category: work
---

## Why We Need Acceleration Structures

A naive ray tracer intersects every ray against every primitive in the scene. This is fine for a few spheres, but real meshes routinely contain thousands to millions of triangles — *Awakening* by Michelangelo, for example, has been digitized into hundreds of millions of triangles. Testing each ray against every triangle is hopelessly slow.

The mesh scene used for this assignment contains 5,172 triangles, and a brute force renderer takes anywhere from one to five minutes to finish a single image. After building a BVH, the same image renders in under five seconds.

The trick is to **test groups of elements at once**. If a ray misses a bounding volume that contains many primitives, we can reject all of them in a single test.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BVH1_Awakening.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

## Survey of Acceleration Structures

Before settling on a BVH, it's worth knowing the alternatives.

### Grid Acceleration
Divide space into a regular grid of cubes. Each cube stores pointers to the elements inside it, and rays walk the grid in front-to-back order. Primitives crossing a grid boundary are either duplicated or subdivided.

Pros: very simple to build and traverse.
Cons: choosing a good grid resolution is hard, and if all geometry clusters in one cell the structure provides no benefit.

### Octree Acceleration
A 3D quadtree. The bounding box of the scene is recursively divided into eight equal sub-boxes whenever a cell contains more than *n* elements (up to a maximum depth *d*). Traversal uses a stack — at a leaf, every primitive is checked, and to find the next leaf we walk up to a sibling and back down.

Pros: adapts to the geometry, easily skips empty space, and can be compressed using **Morton codes** (Z-order curves) which interleave the bits of integer coordinates so that points close in 3D end up close in linear memory. Many GPUs use Morton order to store textures for fast bilinear filtering.

Cons: poor performance with heterogeneous geometry, and primitives that overlap node boundaries must be duplicated or stored at non-leaf nodes.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BVH2_MortonCode.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

### k-D Tree
A binary tree that splits along one axis at a time, but the split position is free — you can divide by even surface area, even volume, or equal primitive count. Most commonly used in photon mapping.

Pros: more flexible than an octree, no overlap between nodes, great for static scenes and point sets.
Cons: doesn't handle overlapping geometry well, can have multiple primitives per node, slow to construct, and harder to compress.

### BSP Tree
A "better" k-D tree that splits on arbitrary planes rather than axis-aligned ones. The splitting planes can hug the geometry more tightly, but choosing the best plane is genuinely hard and the implementation is full of numerical headaches. *Doom*, *Quake*, and similar engines (up through Doom 3) built BSP trees from level walls.

## Bounding Volume Hierarchy

Rather than partitioning *space* (like grids, octrees, and k-D trees), a BVH partitions *objects*. Every node owns an axis-aligned bounding box that encloses all the primitives in its subtree. The root box covers the entire scene, and each leaf holds a single primitive.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BVH3_TopDown.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Why axis-aligned boxes? Tighter shapes exist — oriented bounding boxes squeeze geometry more snugly, and spheres have very fast intersection tests — but both are hard to compute. AABBs are the practical sweet spot.

The key tradeoff vs. spatial partitioning: BVH bounding boxes can **overlap**, but every primitive lives in exactly one leaf. k-D trees have non-overlapping nodes but a single primitive can span multiple leaves. Concretely:

- BVH: faster to build, friendly to dynamic scenes, no duplication of primitives, but doesn't explicitly skip empty space.
- k-D tree: tighter spatial bounds, but a moving primitive may need to be re-inserted into many nodes.

## Construction: Top-Down Recursive Build

The build is recursive. At each step:

1. Compute a bounding box that encloses all primitives in the current set.
2. If the set is small enough, make a leaf node and store the primitive.
3. Otherwise, choose a split axis, sort by that axis, partition the primitives, and recurse on each half.

The skeleton lives in `recursiveBVHBuild`, called from `Mesh::buildBVH`. A `Union` helper computes a bounding box that encloses two input boxes — useful when combining child bounds back up the tree. A correct build produces exactly one leaf per triangle, so for the 5,172-triangle Mario mesh the leaf count should match.

### Equal Primitive Count Heuristic
The simplest split: pick the longest axis, sort primitives along it by their centroid, and partition into equal halves.

Pros: very fast and produces a balanced tree.
Cons: ignores primitive size, so the resulting bounding boxes can overlap heavily.

### Surface Area Heuristic (SAH)
A smarter split picks the plane *p* minimizing
```
f(p) = SA_L(p) * prims_L(p) + SA_R(p) * prims_R(p)
```
where `SA(p)` is the surface area of each side and `prims(p)` is the primitive count on each side. The intuition is that the probability a ray hits a child box is roughly proportional to its surface area, and the cost of entering a child is proportional to the primitives inside.

Evaluating every possible split would be expensive, so the **Binned SAH** approximates it: pick a split axis, sort by centroid, drop the primitives into 2^x bins evenly distributed along the axis, and evaluate the SAH formula at each bin boundary L to R. More bins means a tighter split but a slower build.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BVH4_BinnedSAH.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

The optimal tree structure is actually NP-hard, so every real implementation is making approximations:

- Split on midpoint: fastest, but not much better than an octree.
- Split on median: usually pretty good.
- Split on approximate median (binned): much faster construction, almost as good.

## Traversal: Intersecting the BVH

Intersection is also recursive — a depth-first search through the tree.

```
BVHNode::intersect(Ray ray) {
    if ray hits this->boundingBox {
        if (this is a leaf node) {
            intersect this node's primitives with the ray
            return the resulting intersection
        }
        test ray against L and R children, only recurse on those that were hit
        if L and R bboxes do not overlap:
            recurse on the closer child first; only recurse on the farther
            child if the closer one missed
        if L and R do overlap:
            must recurse on both
        return the closer of the two child intersections
    }
    return no intersection
}
```

The "closer child first" optimization matters. If the near child returns a hit, that hit might be closer than anything in the far child — but only if the boxes don't overlap, since overlapping boxes can contain primitives that interleave in depth. When the boxes are disjoint we can short-circuit the far traversal entirely.

`BVHBounds::intersect` returns just a float (the parametric `t` of the AABB hit) rather than a full Intersection object, since we don't need surface data while walking the tree — only the leaves produce real intersections.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BVH5_Traversal.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

## Linear BVH

Once the tree is built, pointer-chasing through nodes scattered across the heap kills cache performance. A **Linear BVH** flattens the tree into an array:

```cpp
class LinearBVHNode {
    BoundingBox bbox;
    int primitiveIndex;
    int rightChildOffset;
};
```

The left child of a node sits directly after it in the array, so no pointer is needed. The right child is reached via an offset stored in the parent. Primitives are kept in a second array, indexed from the leaf nodes. PBRT's compact BVH representation uses exactly this layout.

The result is dramatically better memory coherence during traversal — sibling nodes that get visited together are often loaded together in a single cache line.

## Result

Brute-force ray-mesh intersection on the 5,172-triangle Mario mesh: **1–5 minutes** per frame.
With an equal-primitive-count BVH: **under 5 seconds** for the same image.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BVH6_MarioRender.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

The whole point: a BVH replaces O(n) primitive tests per ray with something close to O(log n), and that's the difference between a ray tracer that's a science experiment and one that can actually render a frame.

Slide images from:
CIS 5610 Acceleration Structures - Adam Mally
