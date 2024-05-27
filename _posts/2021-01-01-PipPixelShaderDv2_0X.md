---
layout: post
title: Pip Pixel Shader Dv2.0X – Architecture
date: 2021-01-01 14:56 -0400
description: After making an initial “expressive pixel” shader demo in 2020 (Dv1.XX), a re-design was needed to resolve multiple performance issues to make the shader ready for live audio-reactivity. Dv2.XX is the remake that aimed to simplify and optimize the patch.
tags: Audio_Visual
math: true
image:
  path: assets\posts\Dv2_0X\TunnelPulseLoopZoomOut_216p_PlasmaPipPortfolio.gif
  alt: Gif thumbnail of twisting pixel tunnel
---

*This post is a continuation of a previous post: Expressive Pixel Shader Dv1.XX (! link !)*

---

After making an initial “expressive pixel” shader demo in 2020 (Dv1.XX), a re-design was needed to resolve multiple performance issues to make the shader ready for live audio-reactivity. Dv2.XX is the remake that aimed to simplify and optimize the patch.
TL:DR; It got optimized, but I added more complexity and there’s still room for improvement!

## Functional Goals for Dv2.0X

1. Audio visualization for live music (More examples on my commission page)
2. “Real Time” shader: Improve A-V delay from Dv1.XX
    - Less individual visual FX “layers” –> more complex FX
    - Parallel FX chains
    - Simplified UI to reduce UI-system lag coupling in MAX
3. Multiple Modes for multiple scenes
4. W I D E Aspect Ratio (1:1 to 16:9)

{%
  include embed/video.html
  src='assets\posts\Dv2_0X\HyperWaveZoom_SocialExport_PlasmaPipPotfolio.mp4'
  types='mp4|mov'
  title=''
  autoplay=true
  loop=true
  muted=true
%}

## Shader Architecture and the FX Chain

![PipShader_Architecture](assets\posts\Dv2_0X\PixelShaderV2_FXchainDiagram_Thumbs_PlasmaPipPortfolio.jpg)
_Pip Shader Architecture_

![PipShader_Architecture](assets\posts\Dv2_0X\PixelShaderV2_FXchainDiagram2_PlasmaPipPortfolio.jpg)
_Pip Shader Architecture_

Figure 1 is the high-level FX structure happening in the MAX patch (use the slider above!).
From the top [A0]; It’s an empty pixel matrix. I think of each frame as a 6D data set with R,G,B and Alpha per pixel, that is located at a point in <X,Y>. Each visual effect can change one of these 6 variables, and likewise can be controlled by inputs. Here, my “dB1” “dB2” are two filtered gain meters (i.e. when kick drum is recorded, dB1 = 1.0). There is a pre-FX layer [A1] populates color data to the matrix, controlled partially by audio input.

### Modes 1-3:

Mode 1 [A2]: This mode uses some Complex equations to create detailed patterns. The effect is similar to jit.gl.bfg. There is also a visual feedback overlay (time “t” is a variable, loosely
F(t)= [A2] + [A2 +”time offset”] ), which gives this shader it’s signifying look in my opinion. I might elaborate on this in a future post.
Mode 2 [B]: Mode 2 but spherical coordinate with flavor on the angle.
Mode 3 [C]: Multiple sin/cos and spherical-cartesian mixing to make a block-like pattern.

{%
  include embed/video.html
  src='assets\posts\Dv2_0X\TunnelPulseLoopZoomOut_1080p_PlasmaPipPortfolio.mp4'
  types='mp4|mov'
  title=''
  autoplay=true
  loop=true
  muted=true
%}

{%
  include embed/video.html
  src='assets\posts\Dv2_0X\Mode3PinkLinePatterns320x180px_DV2,19_PlasmaPipPortfolio.mp4'
  types='mp4|mov'
  title=''
  autoplay=true
  loop=true
  muted=true
%}

### Mode 4: “Matrix coordinate map mixing”

Mode 4 [D]: This mode uses my favorite max trick I’ve discovered so far:

**— Warning: Math rant —**

From a math point of view, each FX chain in Modes 1-3 moving pixels from point A to point B per some equation. To use an example, moving from cartesian coordinates to Polar coordinates is

$$
f(x, y) = <r, θ> where r = ( x2 + y2 ) 1/2 and θ = arctan(y/x).
$$

This can be generalized as $$ F(x, y) = <f1a(x,y), f1b(x,y)> = <xf, yf> $$, where xf is the new coordinate variable like r or θ. Now with multiple coordinate re-mapping, you can combine the remapping functions together, where $$ M4a(x,y) = i*f1a(x,y) + j*f2a(x,y) + k*f3a(x,y) = xM $$, making a total matrix function where for each pixel coordinate, $ M4(x,y) = <M4a(x,y), M4b(x,y)> = <xM, yM> $ .
In this case, it creates the effect of blending between modes since the modes ARE the input equations. There is a lot of directions to take with this technique, I did percent mixing here (i.e. Mode 4 = 20% mode 1, 50% mode 2, 30% mode 3), but one could also **subtract, multiply, or divide** modes together. In practice, this gets crazy VERY rapidly and needs a tight variable control to prevent high visual noise.

*Note: A more accurate way to represent this concept is through vector spaces and (non)linear maps, the equations above are not mathematically correct and only representative. The current code is built on MAX’s ‘block’ structure. Converting jit.gl.pix directly to matrix functions would allow each mode to be linear maps, or potentially state spaces with feedback control? This should give a performance boost, but is a bit beyond my coding and math ability at the moment.*
(! Links !)

## See Dv2.0X/2.1X in action:

{% include embed/youtube.html id='o7LftWucmNc'%}
_Video “Reverb” in MAX – Dude837 / Sam_

## Issues and Remarks on Dv2.0X

At first, these changes improves audio response and rendering FPS by about ~25% from Dv1.XX. However, I later added additional effects like extra hue/color control, a kaleidoscope filter, and video overlay (more in Dv2.2X). These extra features in addition to screen recording causes some lag issues, and now performs between 20-30 FPS…

Further structure simplification is required to progress this shader, as mentioned above.

*This post has a continuation post:  Expressive Pixel Shader Dv2.1X (! Link !)*