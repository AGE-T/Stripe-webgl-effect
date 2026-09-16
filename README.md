<img width="2278" height="1224" alt="image" src="https://github.com/user-attachments/assets/8d1195c4-604c-49fd-b313-e51664dee8ae" />
<img width="2278" height="1225" alt="image" src="https://github.com/user-attachments/assets/5e45eb5c-76a6-4e41-b3b5-17cb99496e39" />

# Stripe WebGL Effect Research

Standalone WebGL2 recreation study of the Stripe style animated ribbon hero.

## Goal

Recreate the visual techniques behind the Stripe ribbon in an original, self contained implementation.

The project is focused on:

* folded ribbon geometry
* procedural twist
* simplex noise displacement
* shader based fibre lines
* palette driven colour
* WebGL2 rendering
* 3D ribbon interaction
* depth based focal effects

## Key Findings

### Geometry

The original ribbon starts from a dense 400 × 400 plane with approximately:

```text
128 subdivisions X
256 subdivisions Y
```

The plane is folded in geometry space before the shader runs.

Important consequence:

The ribbon is not simply a curved strip. Its base geometry already contains a strong spatial fold and the original version also turns the outer section back toward itself.

A separate half fold test was created by removing the final X reversal.

### Vertex Shader

The main deformation pipeline is:

```text
plane position
    ↓
noise displacement
    ↓
twist X
    ↓
twist Y
    ↓
twist Z
    ↓
model view transform
    ↓
projection
```

Main parameters:

```text
u_time
u_speed

u_twistFrequencyX
u_twistFrequencyY
u_twistFrequencyZ

u_twistPowerX
u_twistPowerY
u_twistPowerZ

u_displaceFrequencyX
u_displaceFrequencyZ
u_displaceAmount
```

The shader uses a custom simplex noise implementation with xxHash based hashing.

### Fragment Shader

The visible ribbon colour is palette based.

The shader also contains:

* contrast
* saturation
* hue shift
* procedural surface noise
* derivative based fine line rendering

The fibre effect is shader based rather than separate geometry.

Important line parameters:

```text
u_lineAmount
u_lineThickness
u_lineDerivativePower
u_maxWidth
```

### Camera

The original Stripe renderer uses an orthographic camera.

Observed setup:

```text
camera z ≈ 5000
camera x ≈ 100
```

The standalone implementation keeps this model for the ribbon.

## WebGL2 Lessons

The original shader came from a Three.js pipeline. Three.js normally handles several things automatically.

Raw WebGL2 requires explicit:

```glsl
#version 300 es
in vec3 position
in vec2 uv
```

and WebGL2 texture syntax.

Important state rule:

Every render pass must explicitly set its required WebGL state.

In particular:

```text
active texture unit
texture binding
shader program
uniforms
framebuffer
viewport
depth state
blend state
```

A major bug occurred when the focal pass left the wrong texture bound to `TEXTURE0`. Rebinding the palette before every ribbon render fixed the resulting disappearing ribbon.

## Depth Of Field

Simple bottom screen blur was tested but is not true depth of field.

The desired pipeline is:

```text
Ribbon colour
      +
Ribbon depth
      ↓
Depth aware DOF / Bokeh
      ↓
Final image
```

The official Three.js reference is:

https://threejs.org/examples/webgl_postprocessing_dof2.html

Depth debugging was eventually made to work with a direct depth shader.

This proved that the ribbon geometry can produce usable depth data.

The final DOF integration should remain isolated from the primary ribbon renderer so that disabling or breaking the postprocess can never hide the ribbon.

## Current UI

Current controls include:

### Animation

Speed

### Twist

Twist X
Twist Y
Twist Z

### Displacement

Amount
Frequency X
Frequency Z

### 3D Tilt

Tilt X
Tilt Y
Tilt Z

Mouse:

```text
drag horizontal → Y rotation
drag vertical → X rotation
wheel → Z rotation
```

### Scale

Scale X
Scale Y
Scale Z

These are mesh scale values, not camera zoom.

### Focal / Depth Of Field

Focus depth
Blur amount
Depth debug
Enabled

The useful focus range was normalised so the UI maps the practical internal range to 0 to 100%.

Blur amount was also normalised so the useful visual range occupies the slider instead of exposing extreme values.

### Presentation Mode

Hide all UI hides the panel and status text.

A small red indicator remains in the top right and restores the interface when clicked.

## Stable Development Rule

Do not modify the working ribbon renderer and the DOF system simultaneously.

Always use:

```text
working ribbon
    ↓
one isolated change
    ↓
test
    ↓
keep working version
```

The primary ribbon renderer must always remain independently renderable.

## Important Reference

The original Stripe source investigation identified these relevant modules:

```text
56878  vertex shader
39798  light fragment shader
98230  alternate fragment shader
26850  postprocess shader
4014   main renderer
82401  folded geometry and material
```

These modules are the main source reference for the reverse engineering work.

## Repository

Current repository:

https://github.com/AGE-T/Stripe-webgl-effect

Current experimental versions include multiple standalone HTML stages.

## Security

Never store GitHub tokens, personal access tokens, passwords, API keys or other credentials in this repository.
