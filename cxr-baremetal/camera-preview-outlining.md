# Camera Preview Outlining (Bare-Metal)

> Source: <https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/us/index.html?documentId=53bafa27a2904367ae08f73be51f84ef> (official English documentation, fetched 2026-08-24)
>
> **Doc version: v1.0.0**

## Overview

Camera preview outlining displays **object outlines** from the camera feed on the glasses screen in real time: every frame goes through edge detection, high-gradient regions are rendered as green outline strokes, and everything else is rendered black, producing a monochrome "black background + green outlines" picture. The effect matches the single-green display of the glasses — only the green channel is needed.

Typical use cases: document / whiteboard framing aid, outline preview, low-power vision demos.

Implementation route: **CameraX frame capture (Y plane only) → OpenGL ES 2.0 shader 5-pass pipeline → TextureView display**, entirely on the glasses, no phone-side involvement.

## How it works

### Data flow

```
CameraX ImageAnalysis (YUV_420_888, Y plane only)
        │  per-frame callback (single thread)
        ▼
Y plane → GL_LUMINANCE texture (rowStride handled)
        ▼
EGL14 window surface (bound to TextureView's SurfaceTexture)
        ▼
FBO ping-pong 5-pass pipeline (fixed 480×640 output)
        ▼
TextureView display (green outline picture)
```

### The 5-pass pipeline

| Pass | Shader | Purpose |
| --- | --- | --- |
| 1 SAMPLE | `SAMPLE_FRAGMENT` | Rotation correction (270°) + nearest-neighbor downsampling to output resolution (as in the sample's fullscreen display) |
| 2 BLUR | `BLUR_FRAGMENT` | 3×3 Gaussian blur; suppresses noise and reduces false edges |
| 3 SOBEL | `SOBEL_FRAGMENT` | Sobel operator: gradient magnitude (edge strength) |
| 4 COLOR | `COLOR_FRAGMENT` | Dual-threshold mapping: weak edges fade to green, strong edges use the outline color, everything else is black |
| 5 FLIP | `FLIP_FRAGMENT` | Vertical flip, output to screen |

Consecutive passes alternate between two FBOs (ping-pong) as intermediate buffers, so no texture is read and written at the same time.

## Key implementation details

### 1. Camera frame input

- Use `ImageAnalysis` with `OUTPUT_IMAGE_FORMAT_YUV_420_888`; take only plane 0 (Y).
- Backpressure strategy `STRATEGY_KEEP_ONLY_LATEST` with a single-thread analyzer; backlogged frames are dropped.
- Resolution is set via `ResolutionSelector` (default 640×480, switchable at runtime).

### 2. Y-plane texture upload

- Upload as a single-channel `GL_LUMINANCE` texture with `GL_UNPACK_ALIGNMENT = 1`.
- When `rowStride` differs from the width, pack row by row to remove end-of-row padding.

### 3. Edge detection (Sobel)

`BLUR` first applies a 3×3 Gaussian blur to the grayscale image (center weight 0.25, orthogonal neighbors 0.125, diagonals 0.0625), then each pixel takes a 3×3 neighborhood to compute the Sobel gradient magnitude:

```glsl
// SOBEL_FRAGMENT (excerpt)
float tl = texture2D(uTexture, vTexCoord + vec2(-uTexelSize.x, -uTexelSize.y)).r * 255.0;
// … tc / tr / ml / mr / bl / bc / br likewise …
float gx = -tl + tr - 2.0 * ml + 2.0 * mr - bl + br;
float gy = -tl - 2.0 * tc - tr + bl + 2.0 * bc + br;
float mag = sqrt(gx * gx + gy * gy);
gl_FragColor = vec4(mag / 1440.0, 0.0, 0.0, 1.0);
```

The magnitude is normalized by dividing by 1440 (for 255-level grayscale, the max Sobel magnitude is about 4×255×√2 ≈ 1442).

### 4. Dual-threshold outline coloring

```glsl
// COLOR_FRAGMENT (excerpt)
float mag = texture2D(uMagTexture, vTexCoord).r * 1440.0;
if (mag < uLowThreshold) {
    gl_FragColor = vec4(0.0);        // below low threshold: black
    return;
}
if (mag >= uHighThreshold) {
    gl_FragColor = vec4(               // at/above high threshold: solid outline color
        uContourColor.r, uContourColor.g, uContourColor.b, uContourColor.a
    );
    return;
}
float t = (mag - uLowThreshold) / (uHighThreshold - uLowThreshold);
float green = (uWeakGreenMin + t * (255.0 - uWeakGreenMin)) / 255.0;
float alpha = (uWeakAlphaMin + t * (255.0 - uWeakAlphaMin)) / 255.0;
gl_FragColor = vec4(0.0, green, 0.0, alpha);   // in between: green and alpha ramp
```

### 5. Display and cleanup

- Set the TextureView SurfaceTexture default buffer size to 480×640.
- On preview stop, clear the surface to pure black (black emits no light on the glasses screen) to avoid ghosting.
- Call `makeNotCurrent()` after rendering each frame to release the EGL context and avoid conflicts with other threads.

## Parameters and tuning

| Parameter | Default | Notes |
| --- | --- | --- |
| Output resolution | 480×640 | Same as the screen |
| Rotation | 270° | Actual rotation of the Rokid Glasses camera |
| Low threshold `uLowThreshold` | 48 | Gradients below this are treated as noise |
| High threshold `uHighThreshold` | 96 | Gradients at/above this are strong edges |
| Outline color `uContourColor` | (0, 1, 0, 1) | Green |
| Weak-edge start values | alpha 64 / green 64 | Mid-range gradients ramp by strength |

Lowering the low threshold detects more weak edges (with more noise); a larger blur kernel (e.g. 5×5) further reduces noise at the cost of thicker edges.

## Viewport adaptation

The sample is a **fullscreen portrait** implementation: the render output is fixed at 480×640 (the screen resolution), and the camera frame is rotated by a fixed 270° and mapped to the whole output with nearest-neighbor sampling. This works as-is for:

- Fullscreen immersive portrait apps (viewport identical to the glasses screen)

But the render output does **not** have to equal the screen resolution. The app viewport is determined by the window and the View; common differences:

| Scenario | Viewport example |
| --- | --- |
| Preview occupying part of the screen (horizontal strip, UI controls / status area) | 480×320, 480×480, 300×400 |
| Non-fullscreen window | Any size |

Using the sample's fixed output in these cases leads to: **non-uniform stretching** (output aspect ratio ≠ View aspect ratio) and unnecessary render cost (still rendering 480×640 when the View is smaller).

Adaptation principles (pipeline structure unchanged):

1. **Follow the View size**: use the `width` / `height` from the `onSurfaceTextureAvailable` callback (or the actual Surface size); derive the FBO textures, `glViewport` and `surface.setDefaultBufferSize()` from it, keeping 480×640 only as the fullscreen default.
2. **Aspect ratio handling**: in the SAMPLE pass, apply `scale = min(srcW/outW, srcH/outH)` for center-crop (drop excess image), or letterbox with black bars (black emits no light on the glasses screen — free of cost), instead of the plain nearest-neighbor mapping that stretches.
3. **Pick the camera resolution by aspect ratio**: use the `ResolutionSelector` aspect-ratio strategy (or filter the supported sizes manually) to prefer a size matching the output aspect ratio, avoiding wasted pixels and bandwidth.

With these principles, the FBO ping-pong pipeline, shaders and gesture logic stay unchanged — only the output size and coordinate mapping parameters differ.

## Performance and notes

- The glasses are a low-power Android Go device: keep the output resolution fixed and matching the target viewport (see [Viewport adaptation](#viewport-adaptation)), process the Y plane only, use a single thread with `KEEP_ONLY_LATEST`, and avoid full-frame YUV conversion.
- Edge detection alone needs no color information; skipping the UV planes saves roughly 2/3 of the texture bandwidth.
- The rotation angle is a fixed device value (270° verified on real hardware); the camera-supported resolution list varies by firmware — verify on real hardware.
- To overlay outlines on the original picture, output the grayscale image in the SAMPLE pass as well and alpha-blend it with the outline result; the pipeline structure stays the same.

## Sample project

A complete runnable implementation is in the **CameraPreviewOutliningSample** sample project (package `com.rokid.cameragpushader`):

- Zip: `https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR_Bare/CameraPreviewOutliningSample.zip`

| File | Contents |
| --- | --- |
| `gl/CameraEdgeView.kt` | TextureView + EGL + CameraX binding, Y-plane upload and frame loop |
| `gl/GpuPipeline.kt` | FBO ping-pong 5-pass pipeline and all parameters |
| `gl/GpuShaders.kt` | All GLSL shader sources |
| `gl/ShaderProgram.kt` | Shader compile/link wrapper |
| `MainActivity.kt` | Gesture wiring: two-finger tap switches resolution, single-finger swipe forward opens the preview, single-finger swipe back closes it |

Gesture handling is described in the [Keys, Wear Detection, and Fold Events](./key-broadcasts.md) chapter.
