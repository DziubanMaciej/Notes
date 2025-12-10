parentTopic:: [[Computer graphics]]
links:: https://www.cse.msu.edu/~cse872/tutorial5.html

# Accumulation buffer
Accumulation buffer. Old feature exposed in OpenGL, now deprecated. It is an internal buffer with the same dimensions as default framebuffer, but it can have different format. There is no direct access to the accumulation buffer or its pixels. Application is able to perform 6 operations:
- `glClearAccum(float, float, float, float)` + `glClear(GL_ACCUM_BUFFER_BIT)` - clear A-buffer.
- `glAccum(GL_ACCUM, float)` - take read-framebuffer, multiply it by `float` and add to A-buffer.
- `glAccum(GL_LOAD, float)` - take read-framebuffer, multiply it by `float` and store in A-buffer.
- `glAccum(GL_ADD, float)` - add `float` to A-buffer.
- `glAccum(GL_MULT, float)` - multiply A-buffer by `float`.
- `glAccum(GL_RETURN, float)` - take A-buffer, multiply it by `float` and store in draw-framebuffer.

Accumulation buffer has undefined behaviour when it comes to values outside 0..1 range.

It can be used for antialiasing - render the image multiple times, jittering it in screen coordinates by half a pixel. After each draw call `glAccum(GL_ACCUM, 1/numberOfRenders)` to get averaged image.

It can be used for depth of field.
