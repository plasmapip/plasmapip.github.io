---
layout: post
title: Pip Pixel Shader Dv2.1X – Displacement Feedback
date: 2022-01-01 14:56 -0400
description: This post focuses on non-linear video feedback.
permalink: post/06_Dv2_1X
tags: Audio_Visual
math: True
image:
  path: https://lh3.googleusercontent.com/drive-viewer/AKGpihYMdJuXZCQSUWlwMnvXsDhGy0nH09NztGrpOBODeraiSrW_2Ep3OekDvJNnNNuUQ3vPD2Hyg2og7EheOe4P6rF1jc3eIcWZRg=w1920-h911
  alt: GIF 1; thumbnail of pixel displacement feedback
---

# Pip Pixel Shader Dv2.1X – Displacement Feedback

*This post is a continuation of a previous post: Pip Pixel Shader Dv2.0X (! Link !)*

---

This post focuses on the **video feedback effect** that is prevalent in this shader. Below are the inspiration for the effect, my twisted implementation, and some fun example loops.

{%
  include embed/video.html
  src='https://res.cloudinary.com/djgbvvcmn/video/upload/v1720126264/TwitterExport22Q3_WIDE_PlasmaPip_kkccwe.mp4'
  types='mp4'
  title='Video 1: Twitter Promo 22Q3'
  autoplay=true
  loop=true
  muted=true
%}

## Video Feedback – Inspiration

[Dude837 / Sam](https://www.youtube.com/c/dude837){:target="_blank"} has a “delicious Max Tutorial” on video “reverb”, where they explain a patch using an alpha-overlay and displacement shader . It’s worth a watch and a subscribe if you deal with MAX. After some testing, I had to incorporate this displacement feedback into this pixel shader in some form.

{% include embed/youtube.html id='a10QZtvn-ro'%}
_Video i-EXTERNAL ([Ref](https://www.youtube.com/watch?v=a10QZtvn-ro){:target="_blank"}): “Reverb” in MAX – Dude837 / Sam_

## Video Feedback – Implementation

Building on this framework, I replaced the td.rota.jxs transform shader (which has parameters like zoom, position, rotation) with a jit.gl.pix. This allows the use of vector math with imaginary numbers to create a new set of non-linear transformations to control the video feedback (fun fact, MAX has imaginary vector functions built in!) The exact vector “math” was arbitrary, mostly done by “guess-and-check”. For others interested in achieving similar effects, the behavior reminds me of jit.gl.bfg, which might be a more direct route for similar transformations.

![PipShader_Architecture](https://lh3.googleusercontent.com/u/0/drive-viewer/AKGpihakZ6RiBKv9ghRh83zbRQdd_RA9joxttinN196dD2XeOLu89ZLLjN4IG5rVJPxgV2jjxbpu8XuXLGWtcDrT0Rey5cPLUsmg5w=w1920-h911-rw-v1)
_Figure 1: Pip Shader Architecture_

For context in the shader architecture, feedback is implemented between image matrices $A_1$ and $A_2$.
(*See the previous post (!!! link !!!) for more context*)

{%
  include embed/video.html
  src='https://res.cloudinary.com/djgbvvcmn/video/upload/v1720126295/22Q3_Quatern16-FluidNoise2_Water_PlasmaPip_li4usi.mp4'
  types='mp4'
  title='Video 2: Fluid Noise ii'
  autoplay=false
  loop=true
  muted=true
%}

## Fluidic Feedback Behavior

This is the fun section. I generally added some pixel noise before the feedback to better follow the direction of the feedback. The result: Much of the following videos have a fluid-like behavior, to the point that it’s eerie considering the lack of any fluid flow modeling.

{%
  include embed/video.html
  src='https://res.cloudinary.com/djgbvvcmn/video/upload/v1720126300/22Q3_Quatern16-FluidNoise1_Water_PlasmaPip_zfcxy7.mp4'
  types='mp4'
  title='Video 3: Fluid Noise i'
  autoplay=false
  loop=true
  muted=true
%}

{%
  include embed/video.html
  src='https://res.cloudinary.com/djgbvvcmn/video/upload/v1720126288/22Q3_Quatern16-FluidNoise3_Water_PlasmaPip_hf93pe.mp4'
  types='mp4'
  title='Video 4: Fluid Noise iii'
  autoplay=false
  loop=true
  muted=true
%}

**Fluidic Feedback – Loose Math Ramplings:**
The feedback effect transforms the incoming image with an arbitrary complex equation (using imaginary numbers), pastes the transformed image on the canvas, and then repeats, transforming the image further along the complex equation. This is a bit reminiscent of a vector field in the sense that every X,Y point has a direction (or “slope”), except it is from a complex equation.

For those who don't know imaginary math concepts, Wiki has some [helpful examples](https://en.wikipedia.org/wiki/Complex_analysis){:target="_blank"} for visualizing complex equations.

![ComplexPlane](https://upload.wikimedia.org/wikipedia/commons/e/e9/Complex-plot.png){: w="500"}
*Figure i-EXTERNAL ([Ref](https://en.wikipedia.org/wiki/Complex_analysis){:target="_blank"}): Color graph of the function f(x) = (x2 − 1)(x − 2 − i)2/x2 + 2 + 2i.*

Many complex equations have local minimums (where points converge), and local maximums (where points diverge). As the image gets transformed according to the complex equation, it makes sense the image itself will start to converge towards the local minimums. However, the nature of displacement feedback means the image is constantly "perturbed" out of local minimums.

While not a mathematically robustly description, the effect is as if the image pixels carry a "momentum" of sorts, and can sling-shot around local minimums. As a grounded analogy, it's like a classic coin vortex game, except the coin is a ball and there is wind blowing on the ball, and there are _many_ balls.

![CoinFunnelGame](https://cdn.shopify.com/s/files/1/0506/3177/products/coin-orbitor-funnel_f6e7d776-72a5-457e-a061-0a9510249320.jpeg?v=1589498176){: w="400"}
*Figure ii-EXTERNAL ([Ref](https://www.reddit.com/r/nostalgia/comments/151ht51/those_coin_spinning_funnel_things/){:target="_blank"}): Coin vortex game*

![FluidFlow](https://www.grc.nasa.gov/WWW/K-12/airplane/Images/mix.gif)
*GIF i-EXTERNAL ([Ref](https://www1.grc.nasa.gov/beginners-guide-to-aeronautics/drag-of-a-sphere/){:target="_blank"}): Computational Fluid Dynamics simulation of a sphere under unsteady occilating flow*

{%
  include embed/video.html
  src='https://res.cloudinary.com/djgbvvcmn/video/upload/v1720126294/22Q3_DataSmearingQuatern_NotPixPerf_Water_PlasmaPip_gir0nu.mp4'
  types='mp4'
  title='Video 5: Data Smearing Quatern'
  autoplay=false
  loop=true
  muted=true
%}

{%
  include embed/video.html
  src='https://res.cloudinary.com/djgbvvcmn/video/upload/v1720126300/22Q3_GritFluidWave2Sphere1_Water_PlasmaPip_qi6unl.mp4'
  types='mp4'
  title='Video 6: Gritty Fluid Wave-to-Sphere'
  autoplay=false
  loop=true
  muted=true
%}

## See Dv2.1X in action:

{% include embed/youtube.html id='MmKRq9VL_3E'%}
_Video 7: 2021 Collab: A: M68K / V: PlasamPip_