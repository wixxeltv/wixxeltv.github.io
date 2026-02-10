---
layout: splash
permalink: /
title: "Custom OpenGL Engine"
author_profile: true
header:
  overlay_color: "#1a1a1a"
  overlay_filter: "0.6"
  overlay_image: /assets/images/final_scene_showcase.gif
  caption: "C++ & OpenGL ES 3.0 Custom Engine"
  actions:
    - label: "Read Architecture"
      url: "/pages/architecture.html"
      btn_class: "btn--primary"
    - label: "View Source Code"
      url: "https://github.com/SAE-Geneve/compgraphsample-wixxeltv"
      btn_class: "btn--light-outline"
excerpt: >
  Technical showcase of a 3D Rendering Engine built from scratch.
  Featuring Deferred Rendering, PBR, IBL, and Hybrid Transparency.
---

# Welcome to my Technical Portfolio
{: .text-center}

My goal was to build a **3D Engine** from scratch using **C++**, moving away from standard forward rendering to implement a **Deferred Rendering** architecture.
{: .text-center .max-width-800 .margin-auto}

<div style="display: flex; flex-wrap: wrap; gap: 20px; justify-content: center; margin-bottom: 3em; margin-top: 3em;">

  <div style="flex: 1; min-width: 250px; background: #252a34; padding: 20px; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.3);">
    <h3 style="color: #61dafb;"><i class="fas fa-cogs"></i> Architecture</h3>
    <p>Full <strong>Deferred Shading</strong> implementation using a G-Buffer. Decouples geometry from lighting to handle a lots of light sources efficiently.</p>
    <a href="/pages/architecture.html" style="font-size: 0.9em; text-decoration: underline;">See G-Buffer Analysis &rarr;</a>
  </div>

  <div style="flex: 1; min-width: 250px; background: #252a34; padding: 20px; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.3);">
    <h3 style="color: #f1c40f;"><i class="fas fa-lightbulb"></i> PBR & IBL</h3>
    <p>Realistic <strong>Physically Based Rendering</strong> (Metallic/Roughness) using Cook-Torrance BRDF. Integrated <strong>Image Based Lighting</strong> for accurate environmental reflections.</p>
    <a href="/pages/lighting.html" style="font-size: 0.9em; text-decoration: underline;">See Lighting & Shadows &rarr;</a>
  </div>

  <div style="flex: 1; min-width: 250px; background: #252a34; padding: 20px; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.3);">
    <h3 style="color: #2ecc71;"><i class="fas fa-tachometer-alt"></i> Performance</h3>
    <p>Intensive use of <strong>Hardware Instancing</strong> to render massive asteroid fields (1000+ objects) in a single CPU Draw Call, maintaining high performance.</p>
  </div>

</div>

## The Hybrid Challenge
{: .text-center}

Deferred Rendering has a major flaw: transparency. I solved this by creating a **Hybrid Pipeline**. 
The engine copies the Depth Buffer (Blit) after the opaque pass, allowing transparent objects (glass) and special effects (Stencil Outlines) to be drawn on top with correct depth testing.
{: .notice--info}

<p style="text-align: center;">
  <a href="/pages/hybrid-rendering.html" class="btn btn--info">Read about Hybrid Rendering</a>
</p>

## Technical Gallery

<div style="display: flex; justify-content: center; gap: 20px; margin-top: 20px; flex-wrap: wrap;">
  
  <div style="flex: 1 1 45%; min-width: 300px; text-align: center;">
    <img src="/assets/images/gbuffer_debug.gif" alt="G-Buffer Debug View" style="border-radius: 8px; box-shadow: 0 4px 15px rgba(0,0,0,0.5); width: 100%; object-fit: cover;">
    <p style="color: #888; font-size: 0.9em;"><em>Debug Views</em></p>
  </div>

  <div style="flex: 1 1 45%; min-width: 300px; text-align: center;">
    <img src="/assets/images/pbr_comparison.gif" alt="Bloom + IBL" style="border-radius: 8px; box-shadow: 0 4px 15px rgba(0,0,0,0.5); width: 100%; object-fit: cover;">
    <p style="color: #888; font-size: 0.9em;"><em>Bloom + IBL</em></p>
  </div>

  <div style="flex: 1 1 45%; min-width: 300px; text-align: center;">
    <img src="/assets/images/bloom.gif" alt="Changing Bloom" style="border-radius: 8px; box-shadow: 0 4px 15px rgba(0,0,0,0.5); width: 100%; object-fit: cover;">
    <p style="color: #888; font-size: 0.9em;"><em>Changing bloom</em></p>
  </div>

  <div style="flex: 1 1 45%; min-width: 300px; text-align: center;">
    <img src="/assets/images/freezed_frustum.gif" alt="Freezed Frustum" style="border-radius: 8px; box-shadow: 0 4px 15px rgba(0,0,0,0.5); width: 100%; object-fit: cover;">
    <p style="color: #888; font-size: 0.9em;"><em>Freezed Frustum</em></p>
  </div>

  <div style="flex-basis: 100%; text-align: center; margin-top: 20px;">
    <img src="/assets/images/final_scene_showcase.gif" alt="Final Result" style="border-radius: 8px; box-shadow: 0 4px 15px rgba(0,0,0,0.5); width: 85%; max-width: 1000px;">
    <p style="color: #888; font-size: 0.9em;"><em>Final Result</em></p>
  </div>

</div>
---

### Tech Stack

* **Language:** C++
* **Graphics API:** OpenGL ES 3.0
* **Maths:** GLM (OpenGL Mathematics)
* **Windowing & Events:** SDL3
* **Assets:** Assimp (3D Models), Stb_image (Textures)