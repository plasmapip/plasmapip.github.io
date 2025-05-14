---
layout: post
title: Mini Lecture - A GLSL Tutorial in Practice
date: 2025-03-29 14:56 -0400
description: How does GLSL work, and how is it used?
permalink: post/07_GLSL-mini-lecture
tags: Learning
math: True
image:
  path: assets/posts/7_GLSL-mini-lecture/GLSL-mini-cover_pp.jpg
  alt: thumbnail for a GLSL Shader.
---

## Before we begin: Why learn GLSL?

There is an extensive list of creative coding and audio/visual software that one can learn to create art (Visualist/DJ lime68k has listed 57 on their [vjing tools and resources list](https://l68k.com/things/vjing){:target="_blank"} ). 

**DO NOT try to learn all of them.** The time you spend learning skills is wasted if you do not balance that time with practicing and applying those skills.

Here are the top outcomes I think you can achieve by learning GLSL:

1. Create simple light-weight custom visual OpenGL generators for browser, parallel processing, or low-performance devices such as a Raspberry Pi.
2. Edit simple custom visual generators in real-time in a higher-level visual software such as TouchDesigner, Max/MSP, or Hydra.
3. Create and use effects that do not exist in higher-level visual software, with the power of weird math.

## Why this GLSL tutorial?

Better tutorials exist. Below are GLSL tutorials that helped me learn and inspired this tutorial. They cover the fundamentals deeply. This tutorial, however, is an applied example of how those fundamentals can come together to create a visual composition, step by step.

1. **[The Book of Shaders](https://thebookofshaders.com/00/){:target="_blank"} by Patricio Gonzalez Vivo & Jen Lowe.**\
If this is your first time seeing GLSL code, sections 00 to 03 will help answer: What is 'GLSL' and 'Fragment Shaders' and what is the code syntax. Sections 04-on have thoughtful visual explanations of common functions used to draw in GLSL.

2. **Inigo Quilez's [video](https://iquilezles.org/live/){:target="_blank"} and [written](https://iquilezles.org/articles/){:target="_blank"} computer graphics tutorials.**\
Inigo has many wonderful resources on 2D/3D GLSL rendering and elegant math code. A few of my favorites include: [Learn to Paint with Mathematics](https://www.youtube.com/watch?v=0ifChJ0nJfM){:target="_blank"}, [Coding the main bulb of the Mandelbrot set](https://iquilezles.org/articles/mset1bulb/){:target="_blank"}, and [coding wavelet image compression](https://iquilezles.org/articles/wavelet/){:target="_blank"} (aka, part of .JPG compression)

3. **'Shadertoy – unofficial' tutorial [blog posts](https://shadertoyunofficial.wordpress.com/){:target="_blank"}.**\
Provides guidance to the quirks of Shadertoy as a compiler, as well as [tips and tricks](https://shadertoyunofficial.wordpress.com/2016/07/21/usual-tricks-in-shadertoyglsl/){:target="_blank"}.


## Goals for this GLSL code:

Before writing any code, I have found it helpful to write out what patterns or effects you are trying to create. This also helps define an end point.

1. Create a hypnotizing spiral pattern.
2. Use frame buffer feedback delay.
3. Make the pattern colorful and nice to look at (eye candy).
4. Code runs in real time (< 17 ms, or > 60 FPS).

## The GLSL Example: Interactive Spiral

The end result is shown below, running in Shadertoy. However, I recommend using "[The Force](https://shawnlawson.github.io/The_Force/){:target="_blank"}" GLSL compiler by Shawn Lawson for learning and experimenting, as it live-compiles and has some small quality-of-life functions. I encourage you to follow along and copy the codes into [The Force](https://shawnlawson.github.io/The_Force/){:target="_blank"} so you can experiment along the way.

I have include some mouse interactivity in this shader. Click and drag on the screen!

<iframe width="800" height="450" frameborder="0" src="https://www.shadertoy.com/embed/W3s3z8?gui=true&t=10&paused=false&muted=true" allowfullscreen></iframe>

## Step-by-step Walkthrough (Active Draft):

NOTE: I am actively building up this documentation. Please check back in early June 2025 for the complete version!

### 1) Create a blank screen.

We first must create a canvas to 'draw' on. In GLSL, the canvas is a fragment shader which we specify using the global variable, or 'uniform' called `gl_FragColor` (A list of common default GLSL uniform variables is documented in the [Book of Shaders - Chapter 03](https://thebookofshaders.com/03/){:target="_blank"}). Each pixel on the shader can have a red, green, blue, and alpha value. Setting them all to zero produces a black screen.

```c++
void main () {
    
	// Output to screen
	gl_FragColor = vec4(0., 0., 0., 0.);
}

// Default Shader Outputs //
// uniform vec4     gl_FragColor;       // color of fragment shader [R, G, B, A]
```

![1_Blank-Screen](assets/posts/7_GLSL-mini-lecture/1_Blank-Screen.png)

_Figure 1: Blank Screen_

### 2) Color the basis coordinates.

We often color shader pixels based on their coordinate location. We access the pixel coordinates through the input uniform `gl_FragCoord`, with the `[0.,0.]` origin at the bottom left corner, and the top right corner having being the `[width,height]` resolution of the window. It is common to normalize the coordinate system values from 0. to 1. for any resolution using a new set of "basis" vectors, (U,V). This is done by dividing the coordinate data by the uniform variable `resolution`. I've colored the blue and red channels by the X (U) and Y (V) axes below.

```c++
// Default Shader Inputs //
// more info: https://thebookofshaders.com/03/

// uniform vec3     gl_FragCoord;       // window coordinates of fragment shader (x,y,z)
// uniform vec2     resolution;         // viewport resolution (in pixels)

void main () {

    // Normalized pixel coordinates to basis vectors: x,y -> u,v (from 0. to 1.)
    vec2 uv = (gl_FragCoord.xy / resolution.xy);
    
	// Output to screen
	gl_FragColor = vec4(uv.x, 0., uv.y, 1.);
}

// Default Shader Outputs //
// uniform vec4     gl_FragColor;       // color of fragment shader [R, G, B, A]
```

![2_Blank-Screen](assets/posts/7_GLSL-mini-lecture/2_Color-Coordinates.png)

_Figure 2: Colored Basis Coordinates (U,V)_

### 3) Center the coordinate system.

It will be easier to draw a spiral starting from the origin. We can move our origin by adding to or multiplying our coordinate basis vectors. Also, the basis coordinate system above is not square since we have a 16:9 aspect ratio. We can make our two basis vectors the same unit length in space by multiplying correction factor using the aspect ratio. I've annotated the output in post for clarity.

```c++
void main () {

    // Normalized pixel coordinates to basis vectors: x,y -> u,v (from 0. to 1.)
    vec2 uv = (gl_FragCoord.xy / resolution.xy);
    
    // Create new UV, centered at the screen center, for polar coordinates.
    vec2 square_aspect_ratio = resolution.xy/resolution.x; // Correction factor
    vec2 uv_centered = (2.*uv - 1.) * square_aspect_ratio;

    // Output to screen
	gl_FragColor = vec4(uv.x, 0., uv.y, 1.);
}
```

![3_Center-Coordinate-System](assets/posts/7_GLSL-mini-lecture/3_Center-Coordinate-System.png)

_Figure 3: Centered Basis Coordinate System_

### 4) Function to generate a spiral.

We can draw a diagonal line in an U,V cartesian coordinate system by coloring pixels where U=V. Similarly, we can draw a spiral line in an R,$\theta$ polar coordinate system by coloring pixels where r=theta ([Wiki link](https://en.wikipedia.org/wiki/Polar_coordinate_system){:target="_blank"} for those not familiar with polar coordinates).

I've created a function called `spiralWave()` to draw the spiral, with lines drawn as the positive portion of a `sin()`. After converting the U,V coordinates to R,$\theta$, we can draw a set of shrinking circles in `circles_zoom` over time (_try it in The Force_). Adding the $\theta$ coordinate shifts the circles into spirals. The output of the function (0. to 1. values across the coordinate system) are then assigned to the R,G,B channels of the fragment shader output.

```c++
// Default Shader Inputs //
// more info: https://thebookofshaders.com/03/

// uniform float    time;               // shader playback time (in seconds)

// Function: Generate Spiral
float spiralWave(vec2 uv_centered, float curve_ratio, float rate, float num_spirals) {
    
    // Calculate polar coordinates of centered UV space (r, theta)
    float r = length(uv_centered);
    float theta = atan(uv_centered.x,uv_centered.y);
    
    // Generate circle, and add theta to offset into spirals
    float circles = r; 
    float circles_zoom = sin(r*1. + time); // sin(r*10 + time); 
    float spiral_zoom  = sin(r*10. + theta*0.05 + time); // sin(r*10. + theta + time)
    float n_spirals    = sin(r*10. + theta * num_spirals + time);
    
    return n_spirals;
}

// use the function in our main output
void main () {
    // Normalized pixel coordinates to basis vectors: x,y -> u,v (from 0. to 1.)
    vec2 uv = (gl_FragCoord.xy / resolution.xy);
    // Create new UV, centered at the screen center, for polar coordinates.
    vec2 square_aspect_ratio = resolution.xy/resolution.x;
    vec2 uv_centered = (2.*uv - 1.) * square_aspect_ratio;

    // Create sin spiral
    float gray_spiral = spiralWave(uv_centered, goldenRatio, zoom_speed, num_spirals);
    
    // Output to screen
	gl_FragColor = vec4(vec3(gray_spiral), 1.);
}
```

<iframe width="800" height="450" frameborder="0" src="https://www.shadertoy.com/embed/W323Dz?gui=true&t=10&paused=false&muted=true" allowfullscreen></iframe>

### 5) Aesthetic Golden Ratio spiral.

To make the spiral pattern more '_hypnotizing_', I modified the spiral code above to draw curve paths similar to that of the golden ratio curve. Some of the mathematical logic to why `log(r)` shows up is explained on [Wiki](https://en.wikipedia.org/wiki/Golden_spiral#Mathematics){:target="_blank"}.

```c++
// Function: Generate Golden Spiral
float spiralWave(vec2 uv_centered, float curve_ratio, float rate, float num_spirals) {
    
    // Calculate polar coordinates of centered UV space (r, theta)
    float r = length(uv_centered);
    float theta = atan(uv_centered.x,uv_centered.y);
    
    // Modifying curve rate to be a golden ratio
    float golden_ratio_spiral = log(r)/curve_ratio + theta;
    float one_golden_ratio_spiral = sin( golden_ratio_spiral + time);
    float n_golden_ratio_spiral = sin( golden_ratio_spiral * num_spirals + time);
    
    return n_golden_ratio_spiral;
}

// use the function in our main output
void main () {
    // Normalized pixel coordinates to basis vectors: x,y -> u,v (from 0. to 1.)
    // Create new UV, centered at the screen center, for polar coordinates.
    ...
    // Create sin spiral
    float gray_spiral = spiralWave(uv_centered, goldenRatio, zoom_speed, num_spirals);
    
    // Output to screen
	gl_FragColor = vec4(vec3(gray_spiral), 1.);
}
```

<iframe width="800" height="450" frameborder="0" src="https://www.shadertoy.com/embed/t32GWR?gui=true&t=10&paused=false&muted=true" allowfullscreen></iframe>

### 6) Add spiral pattern eye candy.

Let's use three steps to add some eye candy. 

1) Oversaturate the `sin()`-generated line to get sharp edges. We can only see pixel intensities from 0. to 1., so scaling the sin to -50. to 50 will clip all values above 1. .

2) Invert half of the spiral output, changing the $\theta$ slicing angle with time.

3) Use the `fract()` function to take the fractional value of the oversaturated `sin()*50.` to see contours of values that were above 1. . `fract()` is explained further in [The Book of Shaders](https://thebookofshaders.com/glossary/?search=fract){:target="_blank"}.

```c++
// Function: Generate Golden Spiral

void main () {
    // Normalized pixel coordinates to basis vectors: x,y -> u,v (from 0. to 1.)
    // Create new UV, centered at the screen center, for polar coordinates.
    ...
    // Create sin spiral
    float gray_spiral = spiralWave(uv_centered, goldenRatio, zoom_speed, num_spirals);
    // Over saturate values beyond 0:1 for clipping / threshold effect.
    float gray_spiral_clipped = (-50. + spiral_width) + 50. * gray_spiral;

    // Invert half of the spiral
    float invert_rotate = 0.25*time;
    float gray_spiral_split = gray_spiral*sin( sin(invert_rotate)*uv_centered.y + cos(invert_rotate)*uv_centered.x);
    
    // Invert half of the spiral again, but in the opposite direction
    float gray_spiral_rotate = gray_spiral_split*(sin( cos(invert_rotate+0.5)*uv_centered.y + sin(invert_rotate+0.5)*uv_centered.x)*20.);
    
    // Apply fractional value function, fract()
    float gray_spiral_fract = fract(gray_spiral_rotate/100.);
    vec4 rbga_spiral_fract = vec4(vec3(gray_spiral_fract),1.0);
    
    // Output to screen
	gl_FragColor = rbga_spiral_fract;
}
```

<iframe width="800" height="450" frameborder="0" src="https://www.shadertoy.com/embed/t3j3WR?gui=true&t=10&paused=false&muted=true" allowfullscreen></iframe>

### 7) Video feedback - linear translation.

Feedback delay is achieved by saving the previous rendered frame and mixing it with the current frame. Fortunately, this is easily accessible in The Force by a pre-defined function `texture(backbuffer, uv)`, where the `backbuffer` is mapped to a U,V coordinate system. In Shadertoy and Max/MSP, however, one often has to get the previous frame using a second buffer shader. See the [Shadertoy implementation code](https://www.shadertoy.com/view/3XsSW8){:target="_blank"} for an example.

To start simple, the coordinate system of the previous frame is diagonally offset by creating a new transformed U,V.

For clarity, I've only applied the feedback on the right side of the shader, `uv.x >= 0.5`, using an if statement.\
**This split view is a helpful trick for understanding and debugging any visual code.**

```c++
// Function: Generate Golden Spiral

void main () {
    // Normalized pixel coordinates to basis vectors: x,y -> u,v (from 0. to 1.)
    // Create new UV, centered at the screen center, for polar coordinates.
    // Create sin spiral
    // Over saturate values beyond 0:1 for clipping / threshold effect.
    // Invert half of the spiral
    // Invert half of the spiral again, but in the opposite direction
    // Apply fract(), get 'vec4 rbga_spiral_fract'.

    // Apply linear displacement to the 'uv' coordinate system
    vec2 uv_transform = uv + vec2(-0.01, 0.0025);
    // Get the previously rendered frame into the current buffer.
    // In Shadertoy, iChannel0 = backbuffer
    vec4 buffer_frame = texture(backbuffer, uv_transform);
    
    // average the previous frame with current frame
    float mix_amount = 0.1;
    vec4 rbga_feedback = mix(buffer_frame, rbga_spiral_fract, mix_amount);
    
    // Split the rendering screen per effect layer
    vec4 rgba_mode_layer; // empty vec4
    if(uv.x < 0.5)
        rgba_mode_layer = rbga_spiral_fract;
    else if(uv.x >= 0.5)
        rgba_mode_layer = rbga_feedback;
    
    // Output to screen
	gl_FragColor = rgba_mode_layer;
}
```

<iframe width="800" height="450" frameborder="0" src="https://www.shadertoy.com/embed/3XsSW8?gui=true&t=10&paused=false&muted=true" allowfullscreen></iframe>

### 8) Video feedback - centered rotation.

To make things more '_hypnotizing_', I wanted the feedback to also rotate. Thanks to linear algebra, our basis vectors can be rotated to any angle by applying a transformation matrix, defined below as `mat2 rotate(float angle)` ([3Blue1Brown has a wonderful video](https://youtu.be/kYB8IZa5AuE?list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab&t=212){:target="_blank"} explaining basis vector transformations).

There is one problem though: the transformation matrix assumes the basis vectors are at the default bottom left origin, not the center of the screen. While this can be corrected within the matrix, one simple solution is to temporarily move our centered fragment shader back to the corner origin, apply the rotation transformation, and re-center the shader. (_There may be a more elegant solution to this, let me know if so!_)

```c++
// Function: Generate Golden Spiral

// Function: Rotation Matrix. https://en.wikipedia.org/wiki/Rotation_matrix
mat2 rotate(float angle) {

    return mat2(
        cos(angle), -sin(angle),
        sin(angle), cos(angle)
    );
}

void main () {
    // Normalized pixel coordinates to basis vectors: x,y -> u,v (from 0. to 1.)
    // Create new UV, centered at the screen center, for polar coordinates.
    // Create sin spiral
    // Over saturate values beyond 0:1 for clipping / threshold effect.
    // Invert half of the spiral
    // Invert half of the spiral again, but in the opposite direction
    // Apply fract(), get 'vec4 rbga_spiral_fract'.

    // Rotate the 'uv' coordinate system about the center of the screen.
    vec2 uv_transform = uv - vec2(0.5);
    uv_transform = uv_transform*rotate(0.05);
    uv_transform = uv_transform + vec2(0.5);
    // Get the previously rendered frame into the current buffer.
    // In Shadertoy, iChannel0 = backbuffer
    vec4 buffer_frame = texture(backbuffer, uv_transform);
    
    // average the previous frame with current frame
    float mix_amount = 0.1;
    vec4 rbga_feedback = mix(buffer_frame, rbga_spiral_fract, mix_amount);
    
    // Split the rendering screen per effect layer
    // Output to screen
	gl_FragColor = rgba_mode_layer;
}
```

<iframe width="800" height="450" frameborder="0" src="https://www.shadertoy.com/embed/3XjXRV?gui=true&t=10&paused=false&muted=true" allowfullscreen></iframe>

### 9) Delay eye-candy and fun functions.

At this stage of shader development, it's ok to experiment by trial and error of different function combinations to see what produces interesting results. To add another level of eye candy, I added 2 more steps:

1. Slightly shift the color of every delayed frame buffer.

2. Experiment with combinations of functions (`max()` and `fract()` in my case) to find pretty patterns.

```c++
// Function: Generate Golden Spiral
// Function: Rotation Matrix. https://en.wikipedia.org/wiki/Rotation_matrix

void main () {
    // Normalized pixel coordinates to vectors: x,y -> u,v (from 0. to 1.)
    // Create new UV, centered at the screen center, for polar coordinates.
    // Create sin spiral
    // Over saturate values beyond 0:1 for clipping / threshold effect.
    // Invert half of the spiral
    // Invert half of the spiral again, but in the opposite direction
    // Apply fract(), get 'vec4 rbga_spiral_fract'.
    // Rotate the 'uv' coordinate system about the center of the screen.
 
    // Get the previously rendered frame into the current buffer.
    // In Shadertoy, iChannel0 = backbuffer
    vec4 buffer_frame = texture(backbuffer, uv_transform); 
    // add slight color shift per buffer
    buffer_frame = (buffer_frame - vec4(0.0, 0.01, 0.05, 1.0)) * decay;
    
    // average the previous frame with current frame
    float mix_amount = 0.1;
    vec4 rbga_feedback = mix(buffer_frame, rbga_spiral_fract, mix_amount);

    // fun fract() effect
    vec4 rbga_feedback_fract = fract(max(fract(rbga_feedback*1.02), 1.-rbga_spiral_fract)*1.1);

    // Output to screen
	gl_FragColor = rbga_feedback_fract;
}
```

<iframe width="800" height="450" frameborder="0" src="https://www.shadertoy.com/embed/W32XRV?gui=true&t=10&paused=false&muted=true" allowfullscreen></iframe>

### 10) Full code with mouse interactivity.

With that, the shader is complete. As a bonus step, we can add mouse interactivity by adding `float mouse_pos_x = mouse.x/resolution.x;`, and adjusting a parameter such as the rotation speed, frame delay mixing intensity, or the position of which layers get displayed.

```c++
// Shader Inputs (on by default) //
// more info: https://thebookofshaders.com/03/

// uniform vec3     gl_FragCoord;       // window coordinates of fragment shader (x,y,z)
// uniform vec2     resolution;         // viewport resolution (in pixels)
// uniform float    time;               // shader playback time (in seconds)
// uniform vec4     mouse;              // mouse X,Y coordinate, mouse click X,Y coordinate

// Defined Variables
float goldenRatio = 0.618;
float spiral_width = 20.;
float zoom_speed = 2.;
float num_spirals = 3.;
float decay = .99;

// Function: Generate Spiral
float spiralWave(vec2 uv_centered, float curve_ratio, float rate, float num_spirals) {
    
    // Calculate polar coordinates of centered UV space (r, theta). https://en.wikipedia.org/wiki/Polar_coordinate_system
    float r = length(uv_centered);
    float theta = atan(uv_centered.x,uv_centered.y);
    
    // Generate circle, and add theta to offset into spirals
    float circles = r; 
    float circles_zoom = sin(r*1. + time); // sin(r*10 + time); 
    float spiral_zoom  = sin(r*10. + theta*0.05 + time); // sin(r*10. + theta + time)
    float n_spirals    = sin(r*10. + theta * num_spirals + time);
    
    // Modifying curve rate to be a golden ratio. https://en.wikipedia.org/wiki/Golden_spiral
    float golden_ratio_spiral = log(r)/curve_ratio + theta;
    float one_golden_ratio_spiral = sin( golden_ratio_spiral + time);
    float n_golden_ratio_spiral = sin( golden_ratio_spiral * num_spirals + time);
    
    return n_golden_ratio_spiral;
}

// Function: Rotation Matrix. https://en.wikipedia.org/wiki/Rotation_matrix
mat2 rotate(float angle) {

    return mat2(
        cos(angle), -sin(angle),
        sin(angle), cos(angle)
    );
}

void main () {

    // Interactive inputs
    float mouse_pos_x = mouse.x/resolution.x;
    float mouse_pos_y = mouse.y/resolution.y;
    
    // Normalized pixel coordinates to basis vectors: x,y -> u,v (from 0. to 1.)
    vec2 uv = (gl_FragCoord.xy / resolution.xy);
    
    // Generate spiral pattern
    // Create new UV, centered at the screen center, for polar coordinates.
    vec2 square_aspect_ratio = resolution.xy/resolution.x;
    vec2 uv_centered = (2.*uv - 1.) * square_aspect_ratio;

    // Create sin spiral
    float gray_spiral = spiralWave(uv_centered, goldenRatio, zoom_speed, num_spirals);
    // Over saturate values beyond 0:1 for clipping / threshold effect.
    float gray_spiral_clipped = (-50. + spiral_width) + 50. * gray_spiral;
    // Invert half of the spiral
    float invert_rotate = 0.25*time;
    float gray_spiral_split = gray_spiral*sin( sin(invert_rotate)*uv_centered.y + cos(invert_rotate)*uv_centered.x);
    // Invert half of the spiral again, but in the opposite direction
    float gray_spiral_rotate = gray_spiral_split*(sin( cos(invert_rotate+0.5)*uv_centered.y + sin(invert_rotate+0.5)*uv_centered.x)*20.);
    // Apply fractional value function, fract()
    float gray_spiral_fract = fract(gray_spiral_rotate/100.);
    vec4 rbga_spiral_fract = vec4(vec3(gray_spiral_fract),1.0);
    
    // FEEDBACK!
    
    // Linear displacement
    //vec2 uv_transform = uv + vec2(0.01, 0.000);
    // Rotate coordinate system, about the center of the screen.
    vec2 uv_transform = uv - vec2(0.5);       // 2
    uv_transform = uv_transform*rotate(0.05*mouse_pos_y);   // 1 // uv_transform*rotate(0.05*mouse_pos_y)
    uv_transform = uv_transform + vec2(0.5);  // 3
    
    vec4 buffer_frame = (texture2D(backbuffer, uv_transform) - vec4(0.0, 0.01, 0.05, 1.0)) * decay; // adding slight color shift per buffer
    
    // average the previous frame with current frame
    vec4 rbga_feedback = mix(buffer_frame, rbga_spiral_fract, 0.01);
    
    // fun fract
    vec4 rbga_feedback_fract = fract(max(fract(rbga_feedback*1.02), 1.-rbga_spiral_fract)*1.1);
    
    
    // Interactive mode switcher
    float slider = mouse_pos_x*4.;
    
    vec4 frag_rgba_slider; // empty vec4
    
    if(uv.x < 0.25*slider)
        frag_rgba_slider = vec4(vec3(gray_spiral_rotate),1.0);
    else if(uv.x < 0.5*slider)
        frag_rgba_slider = rbga_spiral_fract;
    else if(uv.x < 0.75*slider)
        frag_rgba_slider = buffer_frame;
    else
        frag_rgba_slider = rbga_feedback_fract;
    
	// Output to screen
	gl_FragColor = frag_rgba_slider;

}

// Shader Outputs //

// uniform vec4     gl_FragColor;       // color of fragment shader [R, G, B, A]
// uniform vec4     backbuffer;         // Stores gl_FragColor from previous frame
```

## How are GLSL shaders used in practice?

In addition to being fun to look at, what purpose can this spiral shader serve for a performing artist? Here is how I used part of this very shader in my mainstage visuals at MAGFest Super 2025 with [AmateurLSDJ](https://amateurlsdj.bandcamp.com/music){:target="_blank"} (Tyrese).

I wanted to create a unique visual theme for every song in Tyrese's set. One song was called [Golden Ratio](https://amateurlsdj.bandcamp.com/track/golden-ratio-2){:target="_blank"}, and I had the golden ratio spiral stuck in my head. A pre-rendered video of a spiral asset was not an option as all my live visuals are audio-reactive and I like tweaking pattern details on stage. Once a GLSL shader is made, it can run within Max/MSP using a `jit.gl.slab @file Golden-Ratio.jxs` block and accept external 'param' variables. Note that pure GLSL or shadertoy code requires some minor translation to run Max and other software, see [Federico Foderaro's video](https://www.youtube.com/watch?v=WpcsiuqrjwQ){:target="_blank"} for guidance.

In addition to live rendering, GLSL codes are so efficient that you can run multiple of the same shader in parallel on a laptop GPU. So, I added a 2D physics engine in my Max/MSP patch, where every 'ball' was a mini spiral shader. For simplicity, each 'ball' had the same shader input and slight transformations on the 'balls' themselves. While I only fully controlled two GLSL shaders in parallel, one could in theory have a unique shader for every 'ball'. The bottleneck then becomes managing variables in your higher-level visual software, not the compute time.

With some `jit.gl.pix` effects layered on top (which is just node-code GLSL), this is how it turned out!
(skip to 14:20)

{% include embed/youtube.html id='x2Vx9wDsfDc'%}

## Concluding remarks

I learned a lot more about GLSL by writing and explaining my code in this tutorial. **I am no GLSL expert**, so there may be errors in some of my explanations. If you have more experience and spot an opportunity to improve this page, please let me know and I will revise!

-_pip :P_

---

*The files within this web page are licensed under a [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/){:target="_blank"} Attribution-NonCommercial-ShareAlike International license.*