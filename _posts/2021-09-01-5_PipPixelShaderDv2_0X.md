---
layout: post
title: Pip Pixel Shader Dv2.0X – Architecture
date: 2021-09-01 14:56 -0400
description: A redesign of the Pip Pixel Shader Dv1.XX for live audio-reactivity.
permalink: post/05_Dv2_0X
tags: Audio_Visual
math: true
image:
  path: https://res.cloudinary.com/dp5qoqnat/image/upload/TunnelPulseLoopZoomOut_216p_PlasmaPip_dfbkbw.gif
  alt: GIF 1; thumbnail of twisting pixel tunnel
---

*This post is a continuation of a previous post: Pip Pixel Shader Dv1.XX ([Link](https://www.plasmapip.com/post/01_Dv1_XX))*

---

After making an initial “expressive pixel” shader demo in 2020 (Dv1.XX), an optimization re-design was needed to perpare the shader for real-time audio-reactivity. Dv2.XX is that remake, however I added similar amounts of optimization and complexity so there is still room for improvement.

## Functional Goals for Dv2.0X

1. Audio visualization for live music (Examples on my [commission page](https://plasmapip.com/commission){:target="_blank"})
2. “Real Time” shader: >30 FPS performance
    - Consolidation of FX “layers”
    - Parallel FX chains
    - Simplified Shader UI in Max to reduce UI-system lag
3. Multiple Modes for multiple scenes
4. Wide Aspect Ratio (1:1 to 16:9)

{%
  include embed/video.html
  src='https://res.cloudinary.com/dp5qoqnat/video/upload/HyperWaveZoom_SocialExport_PlasmaPip_zptbhw.mp4'
  title='Video 1: Dv2.0X Mode 1 - Hyper Wave Zoom'
  autoplay=false
  loop=true
  muted=true
%}

## Shader Architecture and the FX Chain

![PipShader_Architecture](assets/posts/5_Dv2_0X/PixelShaderV2_FXchainDiagram_ThumbsPatch_PlasmaPip.jpg)
_Figure 1a: Pip Shader Architecture - Simplified High-level Patch and Intermediate FX Layer Pictures_

![PipShader_Architecture](assets/posts/5_Dv2_0X/PixelShaderV2_FXchainDiagram2_PlasmaPip.jpg)
_Figure 1b: Pip Shader Architecture - Intermediate FX Layer Matrix Blocks_

Figure 1 shows the high-level FX structure in the MAX patch.
The image matrix begins as an empty three-channel pixel matrix [$A_0$]. We can consider each image to contain six degrees of information: Each pixel has a red, green, and blue channel (R,G,B), Alpha channel for blending (A), and pixels are spacially located across X and Y coordinates (R,G,B,A,X,Y). These degrees are controlled by functions with audio signals as their input. The “$dB_1$” “$dB_2$” are two filtered gain meters (i.e. when kick drum is recorded, $dB_1 = 1.0$). This audio control is applied through multiple layers of the FX chain, starting at the pre-FX layer [$A_1$].

### FX Modes 1-3:

Mode 1 [$A_2$]: This mode uses some Complex equations to create non-linear deformations, with an effect similar to "[jit.gl.bfg](https://docs.cycling74.com/legacy/max8/refpages/jit.gl.bfg){:target="_blank"}". There is also a visual feedback overlay (time “t” is a variable, loosely
F(t)= [$A_2$] + [$A_2$ +”time offset”] ). This non-linear feedback effect is discussed more in the next post.

Mode 2 [B]: Mode 1 but with a spherical coordinate transformation with audio-reactive control on the angle component.

Mode 3 [C]: Multiple sin/cos and spherical-cartesian mixing to make a block-like pattern.

{%
  include embed/video.html
  src='https://res.cloudinary.com/dp5qoqnat/video/upload/TunnelPulseLoopZoomOut_1080p_PlasmaPip_r7kjki.mp4'
  title='Video 2: Dv2.0X Mode 2 - Tunnel Pulse Loop Zoom-Out'
  autoplay=false
  loop=true
  muted=true
%}

{%
  include embed/video.html
  src='https://res.cloudinary.com/dp5qoqnat/video/upload/Mode3PinkLinePatterns320x180px_DV2-19_PlasmaPip_nqblyi.mp4'
  title='Video 3: Dv2.0X Mode 3 - Pink Line Patterns'
  autoplay=false
  loop=true
  muted=true
%}

### Mode 4: “Matrix coordinate map mixing”

Mode 4 [D]: This mode uses a fun max trick I’ve discovered: Blending displacement maps within jit.gl.pix .

**— Warning: Math rant —**

From a math point of view, each FX chain in Modes 1-3 moving pixels from point A to point B per some equation. To use an example, moving from cartesian coordinates to [Polar coordinates](https://en.wikipedia.org/wiki/Polar_coordinate_system#Converting_between_polar_and_Cartesian_coordinates){:target="_blank"} is


$f(x, y) = <r, θ>$ where $r = ( x^2 + y^2 )^{1/2}$ and $θ = arctan(y/x)$.


This can be generalized as $$ F(x, y) = <f_{1a}(x,y), f_{1b}(x,y)> = <x_f, y_f> $$, where $x_f$ is the new coordinate variable like r or θ. Now with multiple coordinate re-mapping, you can combine the remapping functions together, where $$ M_{4a}(x,y) = i*f_{1a}(x,y) + j*f_{2a}(x,y) + k*f_{3a}(x,y) = x_M $$, making a total function where for each pixel coordinate, $ M_4(x,y) = <M_{4a}(x,y), M_{4b}(x,y)> = <x_M, y_M> $ .
This enables continuous blending between modes since the modes ARE the input displacement equations. There is a lot of directions to take with this technique, I did percent mixing here (i.e. Mode 4 = 20% mode 1, 50% mode 2, 30% mode 3), but one could also **subtract, multiply, or divide** modes together. In practice, this gets visually complex VERY rapidly and needs a tight variable control to prevent high visual noise.

*Note: A more accurate way to represent this concept is through vector spaces and (non)linear maps, the equations above are not mathematically correct and only representative. The current code is built on Max’s ‘block’ structures using jit.gl.pix and not directly linear matrix functions.*

## See Dv2.0X/2.1X in action:

{% include embed/youtube.html id='o7LftWucmNc'%}
_Video 4: Saitone's URL set on MICROMUSIC.ITALY, 2021._

## Issues and Remarks on Dv2.0X

At first, these changes improves audio response and rendering FPS by about ~25% from Dv1.XX. However, I later added additional effects like a kaleidoscope filter and video overlay (more in Dv2.2X), causes some lag issues, and now performs between 20 to 40 FPS… Further structure simplification is required.

*This post has a continuation post:  Pip Pixel Shader Dv2.1X ([Link](https://www.plasmapip.com/post/06_Dv2_1X))*