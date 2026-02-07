---
layout: single
title: "PBR Lighting & Shadows"
permalink: /pages/lighting.html
toc: true
toc_label: "Table of Contents"
toc_icon: "lightbulb"
---

The lighting pass is where the engine shines. By leveraging the **G-Buffer** data populated in the previous pass, I implement a fully **Physically Based Rendering (PBR)** pipeline using the **Metallic-Roughness** workflow.

## 1. The PBR Mathematics

I implemented the standard **Cook-Torrance BRDF** (Bidirectional Reflectance Distribution Function). This model allows materials to interact realistically with light based on their physical properties.

In the lighting shader, the specular term is calculated using three core functions:
1.  **Normal Distribution (NDF):** Trowbridge-Reitz GGX.
2.  **Geometry (G):** Smith's Schlick-GGX.
3.  **Fresnel (F):** Schlick's approximation.

### GLSL Implementation

Here is my implementation of the core PBR functions:

```cpp
// Trowbridge-Reitz GGX
float DistributionGGX(vec3 N, vec3 H, float roughness) {
    float a = roughness * roughness;
    float a2 = a * a;
    float NdotH = max(dot(N, H), 0.0);
    float NdotH2 = NdotH * NdotH;
    float denom = (NdotH2 * (a2 - 1.0) + 1.0);
    return a2 / max(PI * denom * denom, 0.0000001);
}

// Fresnel Schlick
vec3 FresnelSchlick(float cosTheta, vec3 F0) {
    return F0 + (1.0 - F0) * pow(clamp(1.0 - cosTheta, 0.0, 1.0), 5.0);
}
```

The lighting loop then combines the Diffuse (Lambertian) and Specular (Cook-Torrance) components while respecting energy conservation.

---

## 2. Image Based Lighting (IBL)

Direct lighting is not enough for realism. To simulate ambient light and reflections, I implemented **Image Based Lighting**. 

Instead of a constant ambient color, the engine samples a pre-computed **Irradiance Map** (for diffuse) and a **Prefiltered Environment Map** (for specular reflections) based on the surface roughness.

### Shader Integration

In the deferred shader, I sample the environment maps using the reflection vector `R` and the roughness level to fetch the correct mipmap level.

```glsl
// 1. Diffuse Ambient (Irradiance)
vec3 irradiance = texture(irradianceMap, N).rgb;
vec3 diffuse = irradiance * Albedo;

// 2. Specular Ambient (Prefiltered Env Map)
vec3 prefilteredColor = textureLod(prefilterMap, R, Roughness * 4.0).rgb;
vec2 brdf = texture(brdfLUT, vec2(max(dot(N, V), 0.0), Roughness)).rg;
vec3 specular = prefilteredColor * (F * brdf.x + brdf.y);

// Final Ambient with AO & SSAO
vec3 ambient = (kD * diffuse + specular) * CombinedAO;
```

---

## 3. Dynamic Shadows

The engine supports two types of shadows, both calculated in the fragment shader.

### Directional Shadows
Standard shadow mapping for the main directional light (Sun). It uses an orthographic projection to capture depth data from the light's perspective.

### Omnidirectional Shadows (Point Lights)
For point lights, I use **Shadow Cubemaps**. The depth is stored in a cube texture, and the shader calculates the distance between the fragment and the light source across 6 faces.

I implemented **PCF (Percentage-Closer Filtering)** to soften the shadow edges and reduce aliasing artifacts.

```cpp
float PointShadowCalculation(vec3 fragPos, vec3 lightPos) {
    vec3 fragToLight = fragPos - lightPos;
    float currentDepth = length(fragToLight);
    
    float shadow = 0.0;
    float bias = 0.15;
    int samples = 20;
    
    for(int i = 0; i < samples; ++i) {
        float closestDepth = texture(pointShadowMap, fragToLight + sampleOffsetDirections[i] * diskRadius).r;
        closestDepth *= pointShadowFarPlane;
        if(currentDepth - bias > closestDepth)
            shadow += 1.0;
    }
    return shadow / float(samples);
}
```

<figure>
    <img src="/assets/images/pbr_comparison.png" alt="Lighting Result">
    <figcaption>Final result showing PBR materials, IBL reflections, and PCF Soft Shadows.</figcaption>
</figure>

[Next: Hybrid Rendering & Transparency &rarr;](/pages/hybrid-rendering.html){: .btn .btn--primary}