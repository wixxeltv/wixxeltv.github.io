---
layout: single
title: "Architecture & Optimization"
permalink: /pages/architecture.html
toc: true
toc_label: "Table of Contents"
toc_icon: "cogs"
---

My engine relies on a **Deferred Rendering** architecture. Unlike Forward Rendering, which calculates lighting during the geometry pass, Deferred Rendering splits the process into two distinct stages: **Geometry Pass** and **Lighting Pass**.

This approach decouples the scene complexity (number of objects) from the lighting complexity (number of lights), allowing for hundreds of dynamic light sources with minimal performance impact.

## 1. The G-Buffer Strategy

The core of the Deferred pipeline is the **G-Buffer** (Geometry Buffer). Instead of outputting a final color to the screen, the geometry pass writes raw surface data into multiple render targets (MRT).

### Framebuffer Layout

I configured a Framebuffer Object (FBO) with **5 Color Attachments** to store high-precision PBR data.

| Attachment | Format | Channels | Content |
| :--- | :--- | :--- | :--- |
| **0** | `GL_RGBA16F` | RGB | **World Position** (High precision for accurate point shadows) |
| **1** | `GL_RGBA16F` | RGB | **World Normal** (High precision to avoid banding) |
| **2** | `GL_RGBA` | RGB + A | **Albedo** (Color) + Specular Intensity |
| **3** | `GL_RGBA` | RGB + A | **PBR Data** (R=Metallic, G=Roughness, B=AO) |
| **4** | `GL_RGB16F` | RGB | **Emissive** (For glowing objects) |

### Fragment Shader Output

In the shader (`deferred_geometry.frag`), I pack the PBR properties efficiently. Note how I handle both textured and uniform materials directly in the G-Buffer packing.

```glsl
#version 300 es
layout (location = 0) out vec4 gPosition;
layout (location = 1) out vec4 gNormal;
layout (location = 2) out vec4 gAlbedo;
layout (location = 3) out vec4 gPBR;      // R=Metallic, G=Roughness
layout (location = 4) out vec3 gEmissive;

void main() {
    gPosition = vec4(FragPos, 1.0);
    
    if (usePbrMap == 1) {
        // Advanced Normal Mapping calculation
        gNormal = vec4(getNormalFromMap(), 1.0);
        
        // Packing PBR Data
        vec3 pbrData = texture(texture_specular1, TexCoords).rgb;
        if (useArmMap == 1) {
            gPBR.r = pbrData.b; // Metallic
            gPBR.g = pbrData.g; // Roughness
        }
    } else {
        gNormal = vec4(normalize(Normal), 1.0);
        gPBR.r = uMetallic;
        gPBR.g = uRoughness;
    }
}
```

## 2. Hardware Instancing Optimization

One of the main challenges of the scene was rendering the dense **asteroid field** (1000+ moving objects) without bottlenecking the CPU with thousands of draw calls.

To solve this, I implemented **Hardware Instancing**. This technique allows drawing the same mesh thousands of times in a **single API call**, with unique transformations for each instance.

### Vertex Shader Adaptation

In the shader (`deferred_geometry_instanced.vert`), the Model Matrix is no longer a uniform but a **vertex attribute** at location 5. This allows the GPU to fetch the transformation matrix directly from a dedicated VBO for each instance.

```glsl
#version 300 es
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aNormal;
// ...
layout (location = 5) in mat4 aInstanceMatrix; // Per-instance Model Matrix

void main()
{
    // Apply the instance matrix directly to the vertex position
    vec4 worldPos = aInstanceMatrix * vec4(aPos, 1.0);
    FragPos = worldPos.xyz;

    // Calculate Normal Matrix on the fly from the Instance Matrix
    mat3 normalMatrix = transpose(inverse(mat3(aInstanceMatrix)));
    Normal = normalMatrix * aNormal;

    gl_Position = projection * view * worldPos;
}
```

### Performance Result

By switching from `glDrawElements` (standard loop) to `glDrawElementsInstanced`, the CPU overhead for the asteroid field was virtually eliminated, allowing the engine to calculate physics and AI while maintaining stable **FPS**.

[Next: Lighting & Shadows Implementation &rarr;](/pages/lighting.html){: .btn .btn--primary}