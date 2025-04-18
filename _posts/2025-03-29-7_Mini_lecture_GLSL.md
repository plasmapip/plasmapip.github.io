---
layout: post
title: Mini Lecture - GLSL Spiral Walk-through
date: 2025-03-29 14:56 -0400
description: How does GLSL work anyways?
permalink: post/07_GLSL-mini-lecture
tags: Learning
math: True
image:
  path: assets/posts/7_GLSL-mini-lecture/GLSL-mini-cover_pp.jpg
  alt: thumbnail for a GLSL Shader.
---

# Mini Lecture - GLSL Spiral Walk-through

More detailed documentation is coming soon. For now, enjoy the mini interactive GLSL shader below. Please edit, modify, and explore!

## Resources

Live GLSL compiler:		  [https://shawnlawson.github.io/The_Force/](https://shawnlawson.github.io/The_Force/){:target="_blank"}


GLSL Uniforms:		      [https://thebookofshaders.com/03/](https://thebookofshaders.com/03/){:target="_blank"}

GLSL Functions:		      [https://thebookofshaders.com/glossary/](https://thebookofshaders.com/glossary/){:target="_blank"}


Polar Coordinates:	  	[https://en.wikipedia.org/wiki/Polar_coordinate_system](https://en.wikipedia.org/wiki/Polar_coordinate_system){:target="_blank"}

Coord. Rotation Matrix:	[https://en.wikipedia.org/wiki/Rotation_matrix](https://en.wikipedia.org/wiki/Rotation_matrix){:target="_blank"}

Golden Spiral:		      [https://en.wikipedia.org/wiki/Golden_spiral](https://en.wikipedia.org/wiki/Golden_spiral){:target="_blank"}

## GLSL Example: Interactive Spiral (Shadertoy Implementation)

Interact with the shader by clicking and dragging on the screen.

<iframe width="800" height="450" frameborder="0" src="https://www.shadertoy.com/embed/W3s3z8?gui=true&t=10&paused=false&muted=true" allowfullscreen></iframe>

You can run and modify this code in real time by copying the code below and visiting The Force!

[https://shawnlawson.github.io/The_Force/](https://shawnlawson.github.io/The_Force/){:target="_blank"}

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

// Function: Generaqte Spiral
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
    
    // Normalized pixel coordinates to unit vectors: x,y -> u,v (from 0 to 1)
    vec2 uv = (gl_FragCoord.xy / resolution.xy);
    
    // Generate spiral pattern
    // Create new UV, centered at the screen center, for polar coordinates.
    vec2 square_aspect_ratio = resolution.xy/resolution.x;
    vec2 uv_centered = (2.*uv - 1.) * square_aspect_ratio;
    // Create Spiral
    float gray_spiral = (-50. + spiral_width) + 50. * spiralWave(uv_centered, goldenRatio, zoom_speed, num_spirals);
    // Invert half of the spiral
    float invert_rotate = 0.25*time; //   - bands.x*1.
    float gray_spiral_split = gray_spiral*sin( sin(invert_rotate)*uv_centered.y + cos(invert_rotate)*uv_centered.x);
    // Invert half of the spiral again, but in the opposite direction
    float gray_spiral_rotate = gray_spiral_split*(sin( cos(invert_rotate+0.5)*uv_centered.y + sin(invert_rotate+0.5)*uv_centered.x)*20.);
    // Apply fract
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
	//gl_FragColor = vec4(uv.x, gray_spiral_fract, uv.y, 1.);
	//gl_FragColor = vec4(vec3(gray_spiral), 1.);
	gl_FragColor = frag_rgba_slider;

}

// Shader Outputs //

// uniform vec4     gl_FragColor;       // color of fragment shader [R, G, B, A]
// uniform vec4     backbuffer;         // Stores gl_FragColor from previous frame
```

## Code Walkthrough - Web version (Active Draft):

NOTE: I am actively building up this documentation. Please check back in a month or so for the complete version!

### 1) Blank Screen:
```c++
void main () {
    
	// Output to screen
	gl_FragColor = vec4(0., 0., 0., 0.);
}
```

![1_Blank-Screen](assets/posts/7_GLSL-mini-lecture/1_Blank-Screen.png)

_Figure 1: Blank Screen_

### 2) Colored Coordinates
```c++
// Shader Inputs (on by default) //
// more info: https://thebookofshaders.com/03/

// uniform vec3     gl_FragCoord;       // window coordinates of fragment shader (x,y,z)
// uniform vec2     resolution;         // viewport resolution (in pixels)
// uniform float    time;               // shader playback time (in seconds)
// uniform vec4     mouse;              // mouse X,Y coordinate, mouse click X,Y coordinate

void main () {

    // Normalized pixel coordinates to unit vectors: x,y -> u,v (from 0 to 1)
    vec2 uv = (gl_FragCoord.xy / resolution.xy);
    
	// Output to screen
	gl_FragColor = vec4(uv.x, 0., uv.y, 1.);
}

// Shader Outputs //

// uniform vec4     gl_FragColor;       // color of fragment shader [R, G, B, A]
// uniform vec4     backbuffer;         // Stores gl_FragColor from previous frame
```

![2_Blank-Screen](assets/posts/7_GLSL-mini-lecture/2_Color-Coordinates.png)

_Figure 2: Colored Unit Coordinates (U,V)_

### 3) Center the coordinate system
```c++
void main () {

    // Normalized pixel coordinates to unit vectors: x,y -> u,v (from 0 to 1)
    vec2 uv = (gl_FragCoord.xy / resolution.xy);
    
    // Create new UV, centered at the screen center, for polar coordinates.
    vec2 square_aspect_ratio = resolution.xy/resolution.x;
    vec2 uv_centered = (2.*uv - 1.) * square_aspect_ratio;

    // Output to screen
	gl_FragColor = vec4(uv.x, 0., uv.y, 1.);
}
```

![3_Center-Coordinate-System](assets/posts/7_GLSL-mini-lecture/3_Center-Coordinate-System.png)

_Figure 3: Centered Unit Coordinate System_

### 4) Create fuction to generate a spiral

Explain polar coordinates.

```c++
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

// use the fuction in our main output
void main () {
    // Normalized pixel coordinates to unit vectors: x,y -> u,v (from 0 to 1)
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

### 5) Make spiral aesthetic with the Golden Ratio
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

// use the fuction in our main output
void main () {
    // Normalized pixel coordinates to unit vectors: x,y -> u,v (from 0 to 1)
    // Create new UV, centered at the screen center, for polar coordinates.
    ...
    // Create sin spiral
    float gray_spiral = spiralWave(uv_centered, goldenRatio, zoom_speed, num_spirals);
    
    // Output to screen
	gl_FragColor = vec4(vec3(gray_spiral), 1.);
}
```

<iframe width="800" height="450" frameborder="0" src="https://www.shadertoy.com/embed/t32GWR?gui=true&t=10&paused=false&muted=true" allowfullscreen></iframe>

### 6) Make spiral pattern more interesting

```c++
void main () {
    // Normalized pixel coordinates to unit vectors: x,y -> u,v (from 0 to 1)
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
    
    // Apply fract
    float gray_spiral_fract = fract(gray_spiral_rotate/100.);
    vec4 rbga_spiral_fract = vec4(vec3(gray_spiral_fract),1.0);
    
    // Output to screen
	gl_FragColor = rbga_spiral_fract;
}
```

Over saturate values beyond 0:1 for clipping / threshold effect.

Invert half of the spiral, twice.

Take the fractional value of the oversaturated sin to see contours of values that were above 1 (or 255), using the "fract()" effect. More info on fract() at [The Book of Shaders](https://thebookofshaders.com/glossary/?search=fract){:target="_blank"}

<iframe width="800" height="450" frameborder="0" src="https://www.shadertoy.com/embed/t3j3WR?gui=true&t=10&paused=false&muted=true" allowfullscreen></iframe>

### 7) Video Feedback - Frame Linear Translation

(backbuffer) ...

### 8) Video Feedback - Frame Linear Rotation about center

(rotation matrix) ...

### 9) Add final complexity with fract() effect, and adding Mouse interactivity

(embed)

---

*The files within this web page are licensed under a [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/){:target="_blank"} Attribution-NonCommercial-ShareAlike International license.*