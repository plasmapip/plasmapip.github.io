---
layout: post
title: Pip Pixel Shader Dv1.XX
date: 2020-01-01 14:56 -0400
description: The beginning of the audio-reactive pixel chaos.
permalink: post/01_Dv1_XX
tags: Audio_Visual
math: true
image:
  path: https://res.cloudinary.com/dp5qoqnat/image/upload/PipPixShaderV1_8Goo216_PlasmaPip_10MB_uohstz.gif
  alt: GIF 1; A pool of corrupted pixels.
---

In 2020 I slipped into the "[jit.gen](https://docs.cycling74.com/legacy/max8/refpages/jit.gen){:target="_blank"}" and "[jit.gl.pix](https://docs.cycling74.com/legacy/max7/refpages/jit.gl.pix){:target="_blank"}" side of Max/MSP for making low-resolution patterns. By stacking ~10 different jit.gl.pix effects and remapping and blending inputs, I got some of the loops here. I didn’t know at the time that this was a “shader”, what a slippery slope indeed.

{%
  include embed/video.html
  src='https://res.cloudinary.com/dp5qoqnat/video/upload/PipPixShaderV1_9StainedGlass_PlasmaPip_zp3rxh.mp4'
  title='Video 1: Loop #9 - Stained Glass'
  autoplay=false
  loop=true
  muted=True
%}

These clips started with a blank three-channel image matrix and use math to change the RGB color values of each pixel. No external images or video inputs. 

The two below include audio from my early attempts at LSDJ:
{%
  include embed/video.html
  src='https://res.cloudinary.com/dp5qoqnat/video/upload/PipPixShaderV1_8GooLSDJaudioLOOP_PlasmaPip_kijgff.mp4'
  title='Video 2: Loop #8 - Goo (LSDJ audio)'
  autoplay=false
  loop=true
  muted=false
%}

{%
  include embed/video.html
  src='https://res.cloudinary.com/dp5qoqnat/video/upload/PipPixShaderV1_11SunRayLSDJaudioLOOP_PlasmaPip_rdgoob.mp4'
  title='Video 2: Loop #11 - Sun Rays (LSDJ audio)'
  autoplay=false
  loop=true
  muted=false
%}

Max/MSP enables audio-based control of multiple variables, and I started trying real-time image generation based on audio frequencies. In December 2020, I ran my first visualist demo on Twitch to see how this reactive shader might behave in a music performance.
Link to the live version Twitch VOD: [Here](https://twitch.tv/videos/851073696){:target="_blank"} (Shader version: Dv1.10)

{% include embed/youtube.html id='1bYrbBa8uT4'%}
Audio: DonutShoes 🍩 – All She Can Have – [https://donutshoes.bandcamp.com/](https://donutshoes.bandcamp.com/){:target="_blank"}

While this version made very complex and deep patterns, there was a lot of performance issues when adding audio-reactivity. Dv2.XX was the remake that aimed to simplify and optimize the patch.

---

*This post has a continuation post:  Pip Pixel Shader Dv2.0X ([Link](https://www.plasmapip.com/post/05_Dv2_0X))*

---

*The files within this web page are licensed under a [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/){:target="_blank"} Attribution-NonCommercial-ShareAlike International license.*