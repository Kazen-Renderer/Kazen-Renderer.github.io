---
layout: post
title: Kazen Con 2022-Q2 总结
tags: [nano-Kazen]
author: Zhong LingXiao
---

{:toc}

[![]({{site.url}}/content/images/2022/kazen-con-v002-report/kc_v02.png)]({{site.url}}/content/images/2022/kazen-con-v002-report/kc_v02.png)

## 1. Core Feature

1. New samplers.
   1. stratified
   2. correlated
   3. pmj02-bn
2. Geometric shadow terminator : remove the shadow-line artifact cause by geometry.
3. Fire fly reduction: increase roughness per bounce.
4. Configurable ray bias for reduce ray intersection computations error (floating point error).
5. Light primary visibility : toggle light visibility.

------
## 2. Sampler Compare
   1. stratified
   2. correlated
   3. pmj02-bn

### 2.1 Stratified Sampling

分层采样 (stratified sampling) 将域划分为离散数量的层，并且在每个层上独立得到一个采样点。相比随机采样，通常会减少采样点的聚簇的现象，所以会有更好的收敛性。

<div class='embed-container'>
    <iframe src="/content/images/2022/kazen-con-v002-report/comparisons/independent_stratified.html" frameborder="0" border="0" scrolling="no"></iframe>
</div>
<div class="figcaption">Figure x: 256spp indenpendent (left) | stratified (right). For a full screen comparison, <a href="/content/images/2022/kazen-con-v002-report/comparisons/independent_stratified.html">click here.</a></div>



------

### 2.2 Correlated multi-jittered sampling


<div class='embed-container'>
    <iframe src="/content/images/2022/kazen-con-v002-report/comparisons/stratified_correlated.html" frameborder="0" border="0" scrolling="no"></iframe>
</div>

<div class="figcaption">Figure 3: 256spp stratified (left) | correlated (right). For a full screen comparison, <a href="/content/images/2022/kazen-con-v002-report/comparisons/stratified_correlated.html">click here.</a></div>



------

### 2.3 Progressive multi-jittered sampling wtih blue nose


<div class='embed-container'>
    <iframe src="/content/images/2022/kazen-con-v002-report/comparisons/stratified_pmj02bn_256spp.html" frameborder="0" border="0" scrolling="no"></iframe>
</div>

<div class="figcaption">Figure 5: 256spp stratified (left) | pmj02 (right). For a full screen comparison, <a href="/content/images/2022/kazen-con-v002-report/comparisons/stratified_pmj02bn_256spp.html">click here.</a></div>





------
## 3. Geometric shadow terminator
[![]({{site.url}}/content/images/2022/kazen-con-v002-report/gst.jpg)]({{site.url}}/content/images/2022/kazen-con-v002-report/gst.jpg)
<div class="figcaption">Figure x: Geometric shadow terminator 问题</div>

这个问题是由于：**着色法线 (shading normal)** 与 **几何法线 (geometric normal)** 不一致导致的。尤其在底模的情况下非常明显。

一般来说，光线追踪过程中，要得到光滑的表面，需要对顶点法线进行插值。计算方法是：进行相交检测获得三角形的重心坐标，然后根据重心坐标进行插值。这个法线叫做**着色法线 (shading normal)**。

根据图x来看，在计算光源遮挡的时候，插值出来的着色法线与 shadowray 的夹角小于 90°，所以开始进行几何体相交检测，发现着色点被遮挡，那么可以认为这个点处于阴影中。通常是整个三角形被更靠近光源的三角形遮挡住，导致了带棱角的阴影瑕疵。

[![]({{site.url}}/content/images/2022/kazen-con-v002-report/QandTShadowLineShadowtest2.png)]({{site.url}}/content/images/2022/kazen-con-v002-report/QandTShadowLineShadowtest2.png)
<div class="figcaption">Figure x: shadowray 检测</div>

这次实现的是 [Johannes Hanika](https://jo.dreggn.org/home/2021_terminator.pdf) 提出的一种方法。简单来说：计算一个着色点上方的位置，作为 shadowray 的 origin 位置。其实这也很好理解：我们希望得到一个光滑的效果，也可以认为其实是对几何体构造了一个虚拟的光滑表面，这个表面顶点是沿着几何法线位移了一些，类似于曲面细分。效果如图x。

[![]({{site.url}}/content/images/2022/kazen-con-v002-report/virtualsurface.jpg)]({{site.url}}/content/images/2022/kazen-con-v002-report/virtualsurface.jpg)
<div class="figcaption">Figure x: xxxx</div>

添加 geometric shadow terminator 的对比图
<div class='embed-container' style="padding-bottom:100.0%">
    <iframe src="/content/images/2022/kazen-con-v002-report/comparisons/geom_shadow_terminator_compare.html" frameborder="0" border="0" scrolling="no"></iframe>
</div>

<div class="figcaption">Figure 3: 256spp stratified (left) | correlated (right). For a full screen comparison, <a href="/content/images/2022/kazen-con-v002-report/comparisons/geom_shadow_terminator_compare.html">click here.</a></div>

------
## 4. Firefly reduction

Firefly 产生的原因可以看 Matt Pharr 在《Ray Tracing Gems》的第 17 章的 [Ignoring the Inconvenient When
Tracing Rays](https://link.springer.com/content/pdf/10.1007/978-1-4842-4427-2_17.pdf)，介绍的比较全面。

简单来说：根据距离平方反比定律，一个小光源想要照亮场景的话就需要很大的能量值，比如说 radiance = 500 这样。从某个像素发出的路径，在非常小的概率下击中光源，导致 throughput 过大。一个像素的值范围是 [0, 1]，如果要消除这个很大的值，就需要后面非常多的采样才能平滑掉。

这次实现了 [Yuriy O'Donnell](https://twitter.com/YuriyODonnell/status/1199253959086612480) 提出的方法：每一次弹射光线与场景相交，就累加一个粗糙度偏移值 (roughness bias)，然后将这个值累加给当前材质的粗糙度上。这样，随着光线深度的增加，材质累加的粗糙度会越来越大。

```c++
// https://twitter.com/YuriyODonnell/status/1199253959086612480
float oldRoughness = payload.roughness;
payload.roughness = min(1.0, payload.roughness + roughnessBias);
roughnessBias += oldRoughness *  0.2; 
```

作者原来的实现是: `roughnessBias += oldRoughness *  0.75;` 但是 0.75 这个乘数过大，导致金属和地面交接处的焦散消失了。所以我们将这个系数暴露出来:  `accumulatedRoughnessCoff`，使得我们可以在外面手动调节这个参数。下面的例子是 `accumulatedRoughnessCoff` 为 0.2 的效果。

<div class='embed-container' style="padding-bottom:100.0%">
    <iframe src="/content/images/2022/kazen-con-v002-report/comparisons/firefly_compare.html" frameborder="0" border="0" scrolling="no"></iframe>
</div>
<div class="figcaption">Figure x:  Naive (left) | Roughness accumulated (right). <br> accumulatedRoughnessCoff 为 0.2 的对比效果 <a href="/content/images/2022/kazen-con-v002-report/comparisons/firefly_compare.html">click here.</a></div>

> 这里可以看到，accumulated a roughness 后的 Firefly 明显减少，但会使金属材质变得更亮了，**效果错误**。
> 因为在金属之间的弹射 SS 路径，累加粗糙度似乎不正确，或许这种方法只适合 SDS 路径？<br>
> <br>
> 后续需要阅读 [Path Space Regularization for Holistic and Robust Light Transport](https://cg.ivd.kit.edu/publications/p2013/PSR_Kaplanyan_2013/PSR_Kaplanyan_2013.pdf) 来寻找答案，文中提到了一种更加通用的 Vertex Connection and Merging 方法。



------
## 5. Ray epsilon

由于在路径追踪过程中的 [数值精度问题](https://en.wikipedia.org/wiki/Floating-point_arithmetic#Accuracy_problems)，会出现类似 "self shadowing" 的渲染瑕疵。

这是由于反射射线与被反射的表面相交。为了避免这个问题，反射光并不是从相交点开始，而是从离表面一定距离的地方开始，我们用 $$\epsilon$$ `rayEpsilon` 表示这个距离，单位是米。例如，如果射线的 $$\epsilon$$ 为 0.1，那么偏移量为 0.1 米 (=10厘米)。

一般来说，我们希望让射线的 $$\epsilon$$ 变得非常小。根据经验，它大约是摄像机 <===> 物体距离的 1 / 100000 ~ 1 / 1000。在大多数“现实场景”的尺度范围内 ( 对象尺寸为 0.1 米 ~ 100 米 )，默认值 0.001 应该就可以了。

<div class='embed-container' style="padding-bottom:100.0%">
    <iframe src="/content/images/2022/kazen-con-v002-report/comparisons/rayEpsilon_compare.html" frameborder="0" border="0" scrolling="no"></iframe>
</div>
<div class="figcaption">Figure x:  rayEpsilon=0.00001 (left) | rayEpsilon=0.001 (right). <br> 左侧比右侧暗，这是因为自阴影导致反射光线被截断了。另外，也能看到背景物体上明显的线条瑕疵。增加 rayEpsilon 就可以去除这类渲染瑕疵 (self-shadowing, dark lines, etc.) <a href="/content/images/2022/kazen-con-v002-report/comparisons/rayEpsilon_compare.html">click here.</a></div>


在 integrator 中的 shadowRay 和 bsdf 生成新的 ray 都需要重新设置一下 rayEpsilon。例如：

```c++
/* Apply ray epsilon to shadow ray. */
lRec.shadowRay.mint = m_rayEpsilon;
lRec.shadowRay.maxt -= m_rayEpsilon;

/* Apply ray epsilon to trace ray. */
ray = Ray3f(its.p, its.toWorld(bRec.wo));
ray.mint = m_rayEpsilon;
```


------

## 6. Final result
[![]({{site.url}}/content/images/2022/kazen-con-v002-report/Stormtrooper_16384spp_1.1h.png)]({{site.url}}/content/images/2022/kazen-con-v002-report/Stormtrooper_16384spp_1.1h.png)
<div class="figcaption">Figure x: 1920x1080 | 16384 spp | 1.1 h. <a href="https://blendswap.com/blend/13953"> Stormtrooper (by ScottGraham) License: CC BY 3.0 </a></div>

------

## Reference

[1] [Stochastic Sampling in Computer Graphics](http://www.cs.cmu.edu/afs/cs/academic/class/15462-s15/www/lec_slides/p51-cook.pdf) **ROBERT L. COOK**. 1986

[2] [Correlated Multi-Jittered Sampling](https://graphics.pixar.com/library/MultiJitteredSampling/paper.pdf) **Andrew Kensler**. 2013

[3] [Andrew Kensler's permute()](https://andrew-helmer.github.io/permute/) **Andrew Helmer**. 2021

[4] [Progressive Multi-Jittered Sample Sequences](https://graphics.pixar.com/library/ProgressiveMultiJitteredSampling/paper.pdf) **Per Christensen, Andrew Kensler and Charlie Kilpatrick**. 2015

[5] [Physically Based Rendering v4: from theory to implementation](https://www.pbrt.org/) **Matt Pharr, Wenzel Jakob and Greg Humphreys**. 2023

[6] [Hacking the Shadow Terminator](https://jo.dreggn.org/home/2021_terminator.pdf) **Johannes Hanika**. 2021

[7] [Accumulate a roughness bias based on roughness of hitting surface](https://twitter.com/YuriyODonnell/status/1199253959086612480) **Yuriy O'Donnell**. 2019

[8] [Ray Tracing Denoisinge](https://alain.xyz/blog/ray-tracing-denoising#statistical-analysis) **Alain Galvan**. 2020

