You are a senior creative technologist and WebXR specialist. Build a SINGLE-FILE, production-grade HTML5 web application called "NEURAL MASK PROTOCOL" — a cinematic, mobile-first AR face filter experience that rivals Snapchat/Instagram AR quality while running entirely in a browser with zero build step.

═══════════════════════════════════════════════
CORE TECHNICAL STACK (mandatory, CDN-only)
═══════════════════════════════════════════════
- Three.js r160 (ES modules via importmap, jsdelivr CDN)
- MindAR Face Tracking v1.2.5 (mind-ar/dist/mindar-face-three.prod.js)
- Three.js EffectComposer, UnrealBloomPass, RGBShiftShader, FilmPass for post-processing
- Tailwind CSS via CDN (play version)
- Google Fonts: 'Orbitron' + 'Rajdhani' + 'JetBrains Mono'
- Everything in ONE .html file, no external assets, no npm, no bundler

═══════════════════════════════════════════════
ARCHITECTURAL REQUIREMENTS
═══════════════════════════════════════════════
1. Structure the JavaScript as an ES6 class hierarchy:
   - `AppState` (finite state machine: IDLE → CALIBRATING → TRACKING → CAPTURING → ERROR)
   - `MaskEngine` (manages mask presets, shader materials, face mesh lifecycle)
   - `AREngine` (wraps MindAR, handles permissions, fallback logic)
   - `DemoEngine` (WebGL fallback with OrbitControls + animated head proxy)
   - `UIController` (declarative UI state sync, event delegation)
   - `CaptureEngine` (photo snapshot + MediaRecorder video capture with audio)
   - `NotificationSystem` (toast queue with priority levels)
   - `PerformanceMonitor` (FPS counter, auto-quality downgrade for low-end devices)

2. Use a central `CONFIG` object with deeply nested settings (bloom strength, scan speed, palette definitions, mask presets) so everything is tweakable from one place.

3. Implement proper cleanup: dispose geometries/materials/textures on mode switch, remove event listeners, stop MediaStream tracks. No memory leaks.

═══════════════════════════════════════════════
VISUAL DESIGN LANGUAGE
═══════════════════════════════════════════════
Aesthetic: Blade Runner 2049 meets Ghost in the Shell meets Arc browser. Brutalist-holographic.

- Color system: deep obsidian blacks (#05070d), cyan primaries (#00f0ff), electric magenta accents (#ff00aa), with semantic variants.
- Typography: Orbitron for headings (tracked wide), Rajdhani for body, JetBrains Mono for telemetry/numbers.
- Surfaces: glassmorphism with animated gradient borders (use conic-gradient + CSS @property for border rotation).
- Micro-animations: every interactive element has hover/press/focus states with spring easing.
- Subtle CRT/VHS overlay: ultra-faint scanlines + chromatic aberration on UI panels (opacity < 0.08).
- Corner brackets `⌐ ¬ └ ┘` framing active panels for HUD feel.
- Animated reticle/targeting cursor when hovering interactive elements.
- Respect `prefers-reduced-motion`: disable scanlines, pulses, and rotations.

═══════════════════════════════════════════════
MASK PRESETS (build at least 6, not just color swaps)
═══════════════════════════════════════════════
Each preset is a distinct material + geometry treatment, not merely a hue change:

1. **CIPHER** — Obsidian armor base + cyan wireframe overlay + additive glow (the classic look).
2. **LIQUID CHROME** — MeshPhysicalMaterial with metalness 1.0, transmission 0.4, iridescence 1.0, clearcoat — no wireframe.
3. **FRACTURED** — Shader material with animated Voronoi cracks emitting glowing plasma through the fissures.
4. **PARTICLE STORM** — Thousands of GPU-instanced particles attached to face mesh vertices, drifting with face motion.
5. **HOLOGRAPHIC FOIL** — Iridescent rainbow shader reacting to face normal + camera direction (fresnel-based).
6. **NEURAL WEB** — Dense wireframe with animated dashed-line "data packets" traveling along edges, periodic pulse waves radiating from face center.

Each preset defines: name, subtitle, primary color, secondary color, mask material(s), optional particle system, optional custom shader, HUD theme color, and a 1-sentence lore tag shown on switch.

Implement a radial/arc selector (NOT a simple cycle button) at the bottom of the screen — swipeable on mobile, scroll-wheel on desktop — with preset thumbnails (generate procedurally with CSS or canvas icons, not images).

═══════════════════════════════════════════════
FEATURE SET (all must work)
═══════════════════════════════════════════════
- **Camera permission flow**: graceful permission prompt UI explaining why, with retry option.
- **Front/rear camera toggle** with facingMode switching.
- **Photo capture**: composites video frame + 3D layer to canvas, downloads as PNG with embedded EXIF timestamp.
- **Video recording**: uses MediaRecorder on a composite canvas capturing at 30fps, max 30s, downloads as WebM. Show recording indicator + elapsed time.
- **Share API**: if `navigator.share` exists, offer native share sheet for captured media.
- **Face-not-detected state**: when no face tracked for >2s, show elegant "SEARCHING…" HUD with pulsing target reticle centered on video.
- **Multi-face support** where MindAR permits (detect and warn user).
- **Performance telemetry HUD** (toggleable with a settings gear): FPS, render ms, tracker confidence, device pixel ratio, viewport dims.
- **Auto-quality**: if FPS < 24 for 3 consecutive seconds, reduce pixel ratio, disable bloom, simplify particles. Notify user subtly.
- **Intensity slider**: controls bloom strength + wireframe opacity + particle count (maps to single 0–100 "INTENSITY" value).
- **Mirror toggle**: flip horizontal for selfie-mode UX.
- **Fullscreen API** with exit button.
- **Haptic feedback** on mobile (`navigator.vibrate`) for mode switch + capture.
- **Keyboard shortcuts** (desktop): Space = capture photo, R = record, ←/→ = cycle mask, F = fullscreen, M = mirror.
- **Demo mode fallback**: if camera fails OR user declines, spin up a rotating stylized head proxy (parametric geometry — low-poly sculpted head via modified IcosahedronGeometry + noise) demonstrating all masks identically.
- **Proper loading sequence**: boot screen with staged messages ("INITIALIZING NEURAL MESH…", "LOADING TOPOLOGY…", "CALIBRATING SENSORS…") — NOT fake progress; tie to actual load events.

═══════════════════════════════════════════════
THE VIEWPORT / RESPONSIVENESS MANDATE (critical)
═══════════════════════════════════════════════
You MUST solve, at the CSS + JS level:
- iOS Safari 100vh bug (use dvh/svh with fallback + visualViewport API listener).
- Mobile notch/safe-area-inset padding for UI elements.
- The MindAR video canvas stretching / black-bar issue caused by Tailwind Preflight — scope preflight away from AR containers OR override with `!important` rules targeting `#ar-container video, #ar-container canvas` with object-fit cover and unconstrained min/max dimensions.
- Orientation change: re-layout on `orientationchange` AND `resize` AND `visualViewport.resize`, debounced at 150ms.
- Landscape mode: lock to portrait where supported, OR adapt UI gracefully.
- Handle zoom/pinch prevention on the AR canvas without breaking pinch-to-zoom on settings panels.
- DO NOT use a setInterval polling hack to detect black bars — solve the root cause with proper CSS + resize handling.

═══════════════════════════════════════════════
CODE QUALITY STANDARDS
═══════════════════════════════════════════════
- Every class and method has a JSDoc comment.
- Magic numbers extracted to CONFIG constants.
- No global variables except the single app instance.
- Try/catch around every async boundary with user-facing error messages (never show raw stack traces).
- Console logs gated behind `CONFIG.debug` flag.
- Use `AbortController` for cancellable async ops (permission requests, MediaRecorder).
- ARIA labels on every interactive element, proper focus management, keyboard navigable.
- Semantic HTML: <header>, <main>, <section>, <button type="button">, <dialog> for settings.
- Comment sections with banner-style dividers so future devs can navigate a long file.

═══════════════════════════════════════════════
POLISH DETAILS THAT SEPARATE GOOD FROM GREAT
═══════════════════════════════════════════════
- On first successful face lock, trigger a brief "SYSTEM ONLINE" overlay flash with bloom ramp-up (0 → target over 800ms cubic-bezier).
- Wireframe opacity pulses at a rate synced to a simulated heartbeat (72bpm) not a random sine.
- Mask switch has a 400ms "dissolve" — old mask fades + displaces outward while new one rebuilds from particles.
- Lighting rig in AR scene: key light + cyan rim light + magenta fill — rotates subtly based on device orientation (DeviceOrientation API with permission prompt on iOS).
- Telemetry HUD numbers use monospace font with leading zeros (e.g., "FPS: 060 / DPR: 3.00").
- Notification toasts stack vertically when multiple fire in sequence, each with independent lifetime.
- Idle state (60s no interaction): dim UI to 40% opacity, restore on any input.

═══════════════════════════════════════════════
DELIVERABLE
═══════════════════════════════════════════════
Return ONE complete, copy-paste-ready .html file. It must run correctly by simply opening it in a modern mobile browser (iOS Safari 16+, Chrome Android) served over HTTPS or localhost. No build step. No external files. No placeholder comments like "// add logic here" — every function fully implemented.

Before writing the code, briefly outline (max 10 bullets) the architecture and confirm you understand the constraints. Then deliver the full file.

Prioritize, in order: (1) it actually works on mobile Safari without visual bugs, (2) the aesthetic feels premium and cohesive, (3) code is maintainable and well-structured, (4) feature completeness.