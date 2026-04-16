1. **Bootstrap**: importmap for Three.js r160 ES modules, MindAR as UMD script tag, Tailwind Play CDN, Google Fonts preloaded; viewport meta uses `viewport-fit=cover` with `interactive-widget=resizes-content` for iOS.

2. **Viewport strategy**: CSS custom property `--vvh` updated by `visualViewport` listener (debounced 150ms); `100dvh` fallback with `100svh` safety; `#ar-container video, #ar-container canvas` get `object-fit:cover !important` + unconstrained `min/max` to defeat Tailwind Preflight's width:100% rule that causes MindAR black-bar stretching.

3. **Class hierarchy**: `CONFIG` → `AppState` (FSM) → `NotificationSystem` → `PerformanceMonitor` → `MaskEngine` (6 distinct preset factories returning `{meshes[], update(t), dispose()}`) → `AREngine` / `DemoEngine` (pluggable renderers sharing same MaskEngine) → `CaptureEngine` → `UIController` → `App` (orchestrator).

4. **Mask engineering**: each preset is a factory that clones MindAR's face geometry and applies unique material — custom ShaderMaterial for Fractured (voronoi + plasma) and Holographic (fresnel iridescent), InstancedMesh for Particle Storm, LineSegments with dashed GPU-animated shader for Neural Web, MeshPhysicalMaterial for Liquid Chrome, MeshStandardMaterial+wireframe overlay for Cipher.

5. **Post-FX pipeline**: EffectComposer with RenderPass → UnrealBloom → RGBShift → FilmPass; composer shares the MindAR renderer; animation loop swapped from `renderer.render` to `composer.render` inside MindAR start callback.

6. **Capture**: MediaRecorder runs on an OffscreenCanvas compositor that `drawImage`s the MindAR renderer canvas at 30fps via `captureStream()`; photo path writes PNG with synthetic EXIF chunk (iTXt) for timestamp; Share API used when available.

7. **Fallback/Demo**: `DemoEngine` creates noise-displaced IcosahedronGeometry head (phi lattice), OrbitControls, same MaskEngine bolted on; triggers automatically on permission denial, NotAllowedError, or NotReadable.

8. **Auto-quality**: `PerformanceMonitor` samples 60-frame EMA FPS; if <24 for 3s, emits `quality:degrade` → disable bloom pass, halve particles, clamp `setPixelRatio(1)`, notify user.

9. **Input layer**: single `PointerEvent` delegation root, keyboard shortcuts via `keydown` on window, `DeviceOrientation` with iOS 13+ permission gate, `navigator.vibrate` haptic wrapper, idle timer `setTimeout(60s)` resets on any input.

10. **Lifecycle/cleanup**: every engine exposes `dispose()` — releases geometries/materials/textures via WeakRef registry, stops MediaStream tracks, removes `visualViewport`/`orientationchange` listeners, aborts in-flight `AbortController`s. State machine guards forbid double-start.

