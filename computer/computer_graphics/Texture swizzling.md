parentTopic:: [[Computer graphics]]

# Swizzling in shader
Swizzling components of sampled pixels can be performed by the shader code (GLSL/HLSL) itself, i.e. `pixel = pixel.gbar`. However, we may want the shader to not know about how to swizzle the data. Also, this way various HW-specific optimizations may not kick in correctly. For example, sometimes alpha may be compressed separately from RGB components. So if we store pixels as *ARGB* and swizzle in shader, then actually blue component will be treated as alpha and the compression rate may be worse than expected. By using [[#Swizzling in graphics API]], the driver will be aware of the swizzle being done and will be able to optimize for it.





# Swizzling in graphics API
As mentioned before, many graphics APIs expose a way to define sampling swizzle outside the shader. This allows HW-specific optimizations, simplifies shader code and allows using different textures (swizzled differently) with the same shader binary. Support in most prominent APIs:
  - OpenGL - call `glTexParameteriv` with `GL_TEXTURE_SWIZZLE_RGBA` or `GL_TEXTURE_SWIZZLE_R`, `GL_TEXTURE_SWIZZLE_G`, `GL_TEXTURE_SWIZZLE_B` and `GL_TEXTURE_SWIZZLE_A`. Can be set either on a texture or a texture view.
  - D3D11 - not possible
  - D3D12 - set `Shader4ComponentMapping` when creating an SRV
  - Vulkan - create a texture view and pass `VkComponentMapping`.
  - Metal - set `swizzle` in `MTLTextureDescriptor`.
  
  
# Swizzle order
Unlike for rendering (writing), swizzle order for sampling (reading) may sometimes be confusing. That is because we have to think about it the other way round. Consider a texture with pixels stored in memory in *ARGB* order. We sample it in a shader and expect to get a `vec4` stored as *RGBA*. To achieve this we have to swizzle the components:
- *R* component has to get its value from *G* component of the texture (because red is on 2nd position of *ARGB*)
- *G* component has to get its value from *B* component of the texture (because green is on 3rd position of *ARGB*)
- *B* component has to get its value from *A* component of the texture (because blue is on 4th position of *ARGB*)
- *A* component has to get its value from *R* component of the texture (because alpha is on 1st position of *ARGB*)

![[gbar_swizzle.png]]

So the actual swizzling pattern is *GBAR* which is an inversed swizzle of *ARGB*. Although using *ARGB* swizzle feels more natural at a first glance, if we did that we would end up with a vector with components swizzled as *BARG*. That is why we have to actually *inverse* the pattern.

Of course some patterns can't be inversed, i.e. if they don't contain all components, like *RRBA*. This is unlikely to be useful, but if our texture data was stored as *RRBA*, it would mean red component is duplicated, blue and alpha are swapped and there is no green. Hence, we would have no way to get the green component. We could say that nonexistant components would get a value of zero and in case of duplicated components we just pick the first one. With these assumptions an inverse swizzle for *RRBA* would be *R0AB*. This is, however, not an inverse operation in its strictest sense, because if we were to inverse *R0AB* again we would get *R0AB*, not RRBA, meaning $inv(inv(x)) = x$ would not be satisfied.

From the 4 most common swizzling patterns, 3 are involutory - if we apply them two times in a row they cancel out. The definition of involution is that a function (i.e. a swizzling pattern) is its own inverse. Common swizzles:

| Pattern | Inverse | Traits               | Notes                                                                                              |
| ------- | ------- | -------------------- | -------------------------------------------------------------------------------------------------- |
| *RGBA*  | *RGBA*  | involutory, identity |                                                                                                    |
| *BGRA*  | *BGRA*  | involutory           | Swaps *G* with *B*                                                                                 |
| *ARGB*  | *GBAR*  |                      | Cancels out after 4 applications - the sequence is *RGBA* -> *ARGB* - > *BARG* -> *GBAR* -> *RGBA* |
| *ABGR*  | *ABGR*  | involutory           | Swaps *G* with *B* and *A* with *R*                                                                |

