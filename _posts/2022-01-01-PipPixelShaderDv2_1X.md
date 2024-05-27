---
layout: post
title: Pip Pixel Shader Dv2.1X – Displacement Feedback
date: 2022-01-01 14:56 -0400
description: This post focuses on the video feedback effect that is prevalent in this shader. Below are the inspiration for the effect, my twisted implementation, and some fun example loops.
tags: Audio_Visual
image:
  path: assets/posts/Dv2_1X/22Q3_GritFluidWave2Sphere1_Water_PlasmaPipPortfolio_LowQual.gif
  alt: Gif thumbnail of pixel displacement feedback
---

# Pip Pixel Shader Dv2.1X – Displacement Feedback

*This post is a continuation of a previous post: Pip Pixel Shader Dv2.0X (! Link !)*

---

This post focuses on the **video feedback effect** that is prevalent in this shader. Below are the inspiration for the effect, my twisted implementation, and some fun example loops.

{%
  include embed/video.html
  src='assets/posts/Dv2_1X/TwitterExport22Q3_WIDE_PlasmaPipPortfolio.mp4'
  types='mp4|mov'
  title=''
  autoplay=true
  loop=true
  muted=true
%}

## Video Feedback – Inspiration

Dude837 / Sam (!link!) has a “delicious Max Tutorial” on video “reverb”, where they explain a patch using an alpha-overlay and transform shader . It’s worth a watch (and a subscribe if you deal with Max). I played with it for quite some time and knew I had to incorporate this “reverb” effect into this pixel shader in some form.

{% include embed/youtube.html id='a10QZtvn-ro'%}
_Video “Reverb” in MAX – Dude837 / Sam_

## Video Feedback – Implementation

Building on this framework, I replaced the td.rota.jxs transform shader (which has parameters like zoom, position, rotation) with a jit.gl.pix. After much experimentation, I landed on using vector math with imaginary numbers to create a new set of non-linear transformations to control the video feedback (fun fact, Max has imaginary vector functions built in!) The exact vector “math” was arbitrary, mostly done by “guess-and-check”. The behavior reminds me of jit.gl.bfg, so that might be a more direct route for similar transformations.

For context in the shader architecture, feedback is implemented between image matrices $A_1$ and $A_2$.
(*See the previous post (! link !) for more context*)

![PipShader_Architecture](assets/posts/Dv2_1X/ExpressivePixelShaderV21X_FXChain-Feedback_PlasmaPipPortfolio.jpg)
_Pip Shader Architecture_

{%
  include embed/video.html
  src='assets/posts/Dv2_1X/22Q3_Quatern16-FluidNoise2_Water_PlasmaPipPortfolio.mp4'
  types='mp4|mov'
  title='Fluid Noise 2'
  autoplay=false
  loop=true
  muted=true
%}

## Fluidic Feedback Behavior

This is the fun section. I generally added some pixel noise before the feedback to better follow the direction of the feedback. The result: Much of the following videos have a fluid-like behavior, to the point that it’s eerie considering the lack of any fluid flow modeling. What’s going on?

{%
  include embed/video.html
  src='assets/posts/Dv2_1X/22Q3_Quatern16-FluidNoise1_Water_PlasmaPipPortfolio.mp4'
  types='mp4|mov'
  title='Fluid Noise 1'
  autoplay=false
  loop=true
  muted=true
%}

{%
  include embed/video.html
  src='assets/posts/Dv2_1X/22Q3_Quatern16-FluidNoise3_Water_PlasmaPipPortfolio.mp4'
  types='mp4|mov'
  title='Fluid Noise 3'
  autoplay=false
  loop=true
  muted=true
%}

**Fluidic Feedback – My theory [loose math rant warning]:**
The feedback effect transforms the incoming image with an arbitrary complex equation (equations with imaginary numbers), pastes the transformed image on the canvas, and then repeats, transforming the image further along the complex equation. This is a bit reminiscent of a vector field in the sense that every X,Y point has a direction (or “slope”), except it is from a complex equation.

So, what do complex equations look like? Wiki has some helpful examples (! link !)

![ComplexPlane](https://upload.wikimedia.org/wikipedia/commons/e/e9/Complex-plot.png)
_Color graph of the function f(x) = (x2 − 1)(x − 2 − i)2/x2 + 2 + 2i._

Many complex equations have local minimums (lowest point on a 3d plot surface), and points where the solution is infinity. As the image gets transformed according to the complex equation, it makes sense the image will start to converge towards the local minimums. However, the feedback effect displaces and repeats the whole image and transformation process. This is not a mathematically robustly description, but it is like sliding the image above these local minimums but the “momentum(?)” causes the pixels to sling-shot around. The best physical analogy I can think is a coin vortex game, except the coin is a ball and there is wind blowing on the ball, and there are many balls.

![FluidFlow](https://www.bellenviro.co.uk/images/filemanager/uploads/Product%20Images%204/karman%20vortex.gif)
_Fluid Flow_

{%
  include embed/video.html
  src='assets/posts/Dv2_1X/22Q3_DataSmearingQuatern_NotPixPerf_Water_PlasmaPipPortfolio.mp4'
  types='mp4|mov'
  title='Demo video'
  autoplay=false
  loop=true
  muted=true
%}

{%
  include embed/video.html
  src='assets/posts/Dv2_1X/22Q3_GritFluidWave2Sphere1_Water_PlasmaPipPortfolio.mp4'
  types='mp4|mov'
  title='Demo video'
  autoplay=false
  loop=true
  muted=true
%}

## See Dv2.1X in action:

{% include embed/youtube.html id='MmKRq9VL_3E'%}
_2021 Collab: A: M68K / V: PlasamPip_