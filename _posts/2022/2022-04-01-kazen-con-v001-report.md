---
layout: post
title: Kazen Con 2022-Q1 总结
tags: [nano-Kazen]
author: Zhong LingXiao
---
[![]({{site.url}}/content/images/2022/kazen-con-v001-report/final_3.4h_4k_10000spp.jpg)]({{site.url}}/content/images/2022/kazen-con-v001-report/final_3.4h_4k_10000spp.jpg)

## 1. Core Feature

1. Monte Carlo unbiased path tracing 
2. multiple importance sample
3. Material ：kazen initial standard surface ( kiss )
   1. diffuse ：Disney diffuse with Retro-Reflective
   2. specular ：GGX-Smith BRDF with VNDF
   3. clearcoat ：GGX-Smith BRDF with VNDF
   4. sheen ：Disney sheen
4. Textures and simple textures ops ( nested blend, ramp color, scale uv )
5. OIIO texture system
6. Normal mapping
7. Camera : Perspective / Thin Lens
8. Light : mesh light, basic environment ( Blinn/Newell Latitude Mapping )

------
## 2. KISS PBR shading model
**KISS** (kazen initial standard surface) , is a linear blend of a metallic BSDF and a dielectric BSDF, see **Figure 1**. Currently we only support opaque dielectric BSDF and it can be used as materials like plastics, wood, or stone.

<div align=center><img src="{{site.url}}/content/images/2022/kazen-con-v001-report/kiss_sm.jpg" width="320" /></div>  
<div class="figcaption">Figure 1: Structure of the KISS PBR Shading Model.</div>


**KISS** blend metallic and dielectric BSDFs based on parameters **metallic**.

<div align=center><img src="{{site.url}}/content/images/2022/kazen-con-v001-report/kiss_algo.jpg" width="320" /></div>  
<div class="figcaption">Figure 2: The KISS BRDF is a blend of metallic and dielectric BRDF models based on a metallic shader parameter.</div>


For sampling the BRDF, I first use Russian-roulette to decide between sampling the diffuse lobe or the specular lobes with the following ratio:

<div align=center>
$$ ratio_{diffuse} = \frac{1 - metallic}{2} $$
</div>


For non-metallic materials, 1/2 the samples are sampled with cosine weighted hemisphere sampling, the 1/2 with specular sampling described below. For metallic materials, all samples are sampled using specular sampling.



The specular samples are divided into sampling the GGX (specular lobe) and GGX (clearcoat lobe) distribution with the following ratio:

<div align=center>
$$ ratio_{GGX} = \frac{1}{1 + clearcoat} $$
</div>


For materials with no clearcoat, samples are only sampled using the GGX distribution, for materials with 100% clearcoat, 1/2 the samples are directed to either of the distributions.
```c++
// diffuse weight
float ratioDiffuse = (1.0f - metallic) / 2;

// specular weight
float ratioGGX = 1.0f / (1.0f + clearcoat);
```


------
These following images shows material parameters support by **KISS**:

### 2.1 **`roughness`**


|             0.0              |              0.5               |
| :--------------------------: | :----------------------------: |
| ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/m0.0_r0.png) | ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/m0.0_r0.5.png) |
|           **0.0**            |            **1.0**             |
| ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/m0.0_r0.png) |  ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/m0.0_r1.png)  |

------

### 2.2 **`metallic`**

|            0.0             |             0.5              |
| :------------------------: | :--------------------------: |
| ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/m1_r0.png) | ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/m1_r0.5.png) |
|          **0.0**           |           **1.0**            |
| ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/m1_r0.png) |  ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/m1_r1.png)  |

------

### 2.3 **`specular`**

|               0.0                |                0.5                 |
| :------------------------------: | :--------------------------------: |
| ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/m0_r0_spec0.png) | ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/m0_r0_spec0.5.png) |
|             **0.0**              |              **1.0**               |
| ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/m0_r0_spec0.png) |  ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/m0_r0_spec1.png)  |

------

### 2.4 **`specularTint`**

|               0.0                |                  0.5                   |
| :------------------------------: | :------------------------------------: |
| ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/m0_r0_spec1.png) | ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/m0_r0_spec1_st0.5.png) |
|             **0.0**              |                **1.0**                 |
| ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/m0_r0_spec1.png) |  ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/m0_r0_spec1_st1.png)  |

------

### 2.5 **`clearcoat`**

|             0.0              |              0.5               |
| :--------------------------: | :----------------------------: |
| ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/r0.5_c0.png) | ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/r0.5_c0.5.png) |
|           **0.0**            |            **1.0**             |
| ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/r0.5_c0.png) |  ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/r0.5_c1.png)  |

------
### 2.6 **`clearcoatRoughness`**

|             0.0              |                0.5                 |
| :--------------------------: | :--------------------------------: |
| ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/r0.5_c1.png) | ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/r0.5_c1_cr0.5.png) |
|           **0.0**            |              **1.0**               |
| ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/r0.5_c1.png) |  ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/r0.5_c1_cr1.png)  |

------

### 2.7 **`sheen`**

|            0.0             |             0.5              |
| :------------------------: | :--------------------------: |
| ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/r0_s0.png) | ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/r0_s0.5.png) |
|          **0.0**           |           **1.0**            |
| ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/r0_s0.png) |  ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/r0_s1.png)  |

------

### 2.8 **`sheenTint`**

|            0.0             |               0.5                |
| :------------------------: | :------------------------------: |
| ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/r0_s1.png) | ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/r0_s1_st0.5.png) |
|          **0.0**           |             **1.0**              |
| ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/r0_s1.png) |  ![]({{site.url}}/content/images/2022/kazen-con-v001-report/param/r0_s1_st1.png)  |

------

## 3. Debug Error

[![]({{site.url}}/content/images/2022/kazen-con-v001-report/m0_r0_spec0.5_error.png)]({{site.url}}/content/images/2022/kazen-con-v001-report/m0_r0_spec0.5_error.png)
<div class="figcaption">Figure 3: Firefly error and incorrect color bleeding in shadow | 4096 spp.</div>


This error occurred when we set integrator **`maxDepth`** to a very high value, so the light path will be terminated only by russian roulette. The scenario is the light will bounce between floor and sphere forever, and make the shadow part look aweful. 

Althrought, kill light path by **`maxDepth`** without compensation will cause statistics bias.



<div class='embed-container'>
    <iframe src="/content/images/2022/kazen-con-v001-report/comparisons/beforeafter_maxdepth_error.html" frameborder="0" border="0" scrolling="no"></iframe>
</div>
<div class="figcaption">Figure 4: maxDepth=5 (left) and maxDepth=1024 (right). For a full screen comparison, <a href="/content/images/2022/kazen-con-v001-report/comparisons/beforeafter_maxdepth_error.html">click here.</a></div>

> `maxDepth` parameter is found when I check cycles render setting : Light path, by default it sets `maxBounces` to 4.<br>
> **So the take-away is : Reverse thinking code logic by ground truth (cycles) results, not just debug through code.**

------

## 4. Look-dev

This shows the **cycles** shading graph.

[![]({{site.url}}/content/images/2022/kazen-con-v001-report/cycles_shader_graph.jpg)]({{site.url}}/content/images/2022/kazen-con-v001-report/cycles_shader_graph.jpg)
<div class="figcaption">Figure 5: The shading graph for creating the look.</div>

The corresponding **nano-Kazen** material scene graph:

```xml
<bsdf type="normalmap">
    <texture type="imagetexture">
        <string name="filename" value="textures/MSMC_Brass_Hammered_Normal.jpg"/>
        <float name="scale" value="1.0"/>
        <string name="colorspace" value="linear"/>
    </texture>
    <bsdf type="kazenstandard">
        <texture type="blend" id="baseColor">
            <string name="blendmode" value="mix"/>
            <texture type="constanttexture" id="input1">
                <color name="color" value="0.887923 0.351533 0.002125"/>
            </texture>
            <texture type="constanttexture" id="input2">
                <color name="color" value="0.982251 0.822786 0.62396"/>
            </texture>
            <texture type="imagetexture" id="mask">
                <string name="filename" value="textures/MSMC_Circle_Outline_Grid.jpg"/>
                <float name="scale" value="1.0"/>
                <string name="colorspace" value="linear"/>
            </texture>
        </texture>
        <texture type="constanttexture" id="roughness">
            <color name="color" value="0.166233 0.166233 0.166233"/>
        </texture>
        <texture type="imagetexture" id="metallic">
            <string name="filename" value="textures/MSMC_Circle_Outline_Grid.jpg"/>
            <float name="scale" value="1.0"/>
            <string name="colorspace" value="srgb"/>
        </texture>
        <float name="anisotropy" value="0.0"/>
        <float name="specular" value="0.5"/>
        <float name="specularTint" value="0.0"/>
        <float name="clearcoat" value="0.0"/>
        <float name="clearcoatRoughness" value="0.0"/>
        <float name="sheen" value="0.0"/>
        <float name="sheenTint" value="0.0"/>
    </bsdf>
</bsdf>
```

[![]({{site.url}}/content/images/2022/kazen-con-v001-report/look1.png)]({{site.url}}/content/images/2022/kazen-con-v001-report/look1.png)
<div class="figcaption">Figure 6: 1920x1080 | 4096 spp | 11.7 min.</div>

------

## 5. Final result

[![]({{site.url}}/content/images/2022/kazen-con-v001-report/final_3.4h_4k_10000spp.jpg)]({{site.url}}/content/images/2022/kazen-con-v001-report/final_3.4h_4k_10000spp.jpg)
<div class="figcaption">Figure 7: 3840x2160 | 10000 spp | 3.4 h.</div>




Compare with Blender render result.
<div class='embed-container'>
    <iframe src="/content/images/2022/kazen-con-v001-report/comparisons/compare_nano_and_cycles.html" frameborder="0" border="0" scrolling="no"></iframe>
</div>
<div class="figcaption">Figure 8: nano-Kazen: 3840x2160 | 10000 spp | 3.4 h (left) and cycles: 1920x1080 | 2500 spp | 4.47 min (right). For a full screen comparison, <a href="/content/images/2022/kazen-con-v001-report/comparisons/compare_nano_and_cycles.html">click here.</a></div>


------

## Reference

[1] [Physically Based Shading at Disney](https://media.disneyanimation.com/uploads/production/publication_asset/48/asset/s2012_pbs_disney_brdf_notes_v3.pdf) **Brent Burley**. 2012

[2] [Extending the Disney BRDF to a BSDF with Integrated Subsurface Scattering](https://blog.selfshadow.com/publications/s2015-shading-course/burley/s2015_pbs_disney_bsdf_notes.pdf) **Brent Burley**. 2015

[3] [Simon Kallweit's project report](http://simon-kallweit.me/rendercompo2015/report/) **Simon Kallweit**. 2015

[4] [Microfacet Models for Refraction through Rough Surfaces](https://www.cs.cornell.edu/~srm/publications/EGSR07-btdf.pdf) **Bruce Walter**. 2007

[5] [Understanding the Masking-Shadowing Function in Microfacet-Based BRDFs](https://jcgt.org/published/0003/02/03/paper.pdf) **Eric Heitz**. 2014

[6] [Sampling the GGX Distribution of Visible Normals](https://jcgt.org/published/0007/04/01/paper.pdf) **Eric Heitz**. 2018

[7] [ROBUST MONTE CARLO METHODS FOR LIGHT TRANSPORT SIMULATION](https://graphics.stanford.edu/papers/veach_thesis/thesis.pdf) **Eric Veach**. 1997

[8] [Physically Based Rendering: from theory to implementation](https://www.pbrt.org/) **Matt Pharr, Wenzel Jakob and Greg Humphreys**. 2016

[9] [Mitsuba 2: Physically Based Renderer](https://www.mitsuba-renderer.org/) **Wenzel Jakob**

[10] [Embree 3: High Performance Ray Tracing](https://www.embree.org/) **Intel**

[11] [OpenImageIO: a library for reading and writing images, and a bunch of related classes, utilities, and applications](https://sites.google.com/site/openimageio/home) **Larry Gritz**

[12] [Greyscalegorilla: modern surface material collection](https://greyscalegorilla.com/product/modern-surface-material-collection/) **Greyscalegorilla**. 2021