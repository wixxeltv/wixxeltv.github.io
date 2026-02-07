---
layout: single
title: "Hybrid Rendering & Effects"
permalink: /pages/hybrid-rendering.html
toc: true
toc_label: "Table of Contents"
toc_icon: "magic"
---

While Deferred Rendering is powerful for lighting, it natively struggles with transparent objects and specific stencil effects. To solve this, I implemented a **Hybrid Pipeline** that combines the strengths of both Deferred and Forward rendering.

## 1. The Hybrid Workflow (Depth Blitting)

The core mechanism involves copying the geometry's depth information from the **G-Buffer** to the **Forward Framebuffer** (HDR FBO). This allows Forward-rendered objects, such as glass or special effects, to be correctly depth-tested against the existing opaque scene.

### Implementation

In `RenderForwardPass`, I use `glBlitFramebuffer` to transfer the depth data before drawing transparent objects:

```cpp 
// 1. Copy Depth Buffer from G-Buffer to HDR FBO
glBindFramebuffer(GL_READ_FRAMEBUFFER, gBufferFBO_);
glBindFramebuffer(GL_DRAW_FRAMEBUFFER, hdrFBO_);
glBlitFramebuffer(0, 0, width, height, 0, 0, width, height, 
                  GL_DEPTH_BUFFER_BIT, GL_NEAREST);

// 2. Switch to HDR FBO for Forward Rendering
glBindFramebuffer(GL_FRAMEBUFFER, hdrFBO_);
glEnable(GL_DEPTH_TEST);
```

---

## 2. Transparency & Blending

With the depth buffer populated, I can render transparent materials using standard alpha blending. I validated this by rendering a glass panel using a constant alpha value to simulate thickness and tint.

```cpp
// Enable Blending for transparency
glEnable(GL_BLEND);
glBlendColor(0.0f, 0.0f, 0.0f, 0.3f);
glBlendFunc(GL_CONSTANT_ALPHA, GL_ONE_MINUS_CONSTANT_ALPHA);

// Disable Depth Mask to prevent transparent objects from occlusion
glDepthMask(GL_FALSE);
glassModel_.Draw(forwardPipeline_);
glDepthMask(GL_TRUE);

glDisable(GL_BLEND);
```
<figure>
    <img src="/assets/images/blending.png" alt="Lighting Result">
    <figcaption>Transparent cube</figcaption>
</figure>
---

## 3. Stencil Object Outline

I utilized the **Stencil Buffer** to create a highlight effect around the main character. This is achieved in two distinct passes within the Forward pipeline:

1.  **First Pass:** Draw the object normally while writing a value of `1` to the stencil buffer.
2.  **Second Pass:** Draw a slightly scaled-up version of the object (1.02x). Using `glStencilFunc(GL_NOTEQUAL, 1, 0xFF)`, the outline is only rendered where the original object is not present.

To further refine the outline, I used **Front-Face Culling** (`glCullFace(GL_FRONT)`) during the second pass to render only the back-faces of the scaled model, creating a silhouette.

### Implementation Constraints
While efficient, this "Vertex Extrusion via Scaling" technique has known limitations regarding complex geometry:
* **Geometry Artifacts:** Scaling from the center (`glm::scale`) works well for convex shapes (cubes, spheres) but causes artifacts on complex meshes like the Knight (e.g., gaps or uneven thickness in concave areas).
* **Perspective Thickness:** Since the scaling is uniform in World Space, the outline appears thicker when the object is close to the camera and thinner when far away. A more advanced approach would involve extruding vertices along their normals in the Vertex Shader to maintain constant screen-space thickness.
<figure>
    <img src="/assets/images/stencil.png" alt="Lighting Result">
    <figcaption>Stencil demonstration</figcaption>
</figure>
---

## 4. Post-Processing: Bloom & HDR

The final stage of the pipeline handles visual enhancements. I extract the bright parts of the image (Luminance) into a separate buffer and apply a Gaussian Blur to create the **Bloom** effect.

### Ping-Pong Blurring
To optimize the Gaussian blur, the engine uses two framebuffers (**Ping-Pong**) to blur the image horizontally and then vertically over 10 iterations. This separates a 2D convolution into two 1D passes, significantly improving performance.

### Tone Mapping & Gamma Correction
Finally, the High Dynamic Range (HDR) color is mapped to the monitor's range using exposure-based Tone Mapping. This ensures that bright areas (like light sources) retain detail instead of simply clipping to white.

```cpp
// Exposure-based Tone Mapping
vec3 mapped = vec3(1.0) - exp(-hdrColor * exposure);

// Gamma Correction (2.2)
mapped = pow(mapped, vec3(1.0 / 2.2));
```

[Back to Architecture &rarr;](/pages/architecture.html){: .btn .btn--light-outline}