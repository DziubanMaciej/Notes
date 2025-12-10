parentTopic:: [[Computer graphics]]

# Buffers named after letters

## A-buffer
An old method for drawing transparent intersecting objects invented in 1984. It is an extension of z-buffer, which is more complicated and consumes more memory, but work for non-opaque objects. See [DOI:10.1145/964965.808585](https://sci-hub.se/10.1145/964965.808585)(1984) and [geeksforgeeks](https://www.geeksforgeeks.org/a-buffer-method/) for details.

## G-buffer
Geometry buffer. Intermediate buffer in deferred rendering. It can contain data such as normals, colors, emisiveness, motion vectors, etc. Usually there are 2-4 G-buffers in a deferred renderer.

## N-Buffer
Neighbourhood buffer. It is a multi-level depth buffer with the same resolution. All levels have the same resolution. Level 0 is a standard depth buffer. A pixel at level $i$ stores the _maximum_ depth of the pixels in a _neighborhood_ of size $i$ in level 0. Different definitions of neighbourhood can be used. See [DOI:10.1111/j.1467-8659.2005.00864.x](https://sci-hub.se/10.1111/j.1467-8659.2005.00864.x)(2005) for details.

## P-buffer
Pixel buffer. A very old and clunky OpenGL mechanism for offscreen-rendering introduced before FBOs. It didn't suffer a problem that a normal default framebuffer did - occluded pixels might have not been returned by `glReadPixels()`. See [OpenGL wiki](https://www.khronos.org/opengl/wiki/P-buffer) for details.

## V-buffer
Visibility buffer. Similar to deferred rendering, but more resilient to very small triangles. First pass renders an `(objectId, triangleId)` pair to a texture. In second pass the vertex attributes are reconstructed from this data. It's a part of Nanite in UE5. See this [article](http://filmicworlds.com/blog/visibility-buffer-rendering-with-material-graphs/) or a [paper from Intel](https://jcgt.org/published/0002/02/04/paper.pdf).

## W-buffer
A buffer used to store depth with linear precision. It's a legacy feature available in fixed function APIs until DirectX9. Nowadays it's up to applications to transform depth in a way that avoids Z-fighting.

## Z-buffer
A buffer used to store depth. It's typicallyd store as floating points, meaning it offers varying precision - higher for near objects, lower for far away objects.
