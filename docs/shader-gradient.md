# Shader gradient (3D ShaderGradient)

## Purpose

A slow, grainy, three-dimensional colour field for **hero backgrounds and closing CTA panels**. It is an ambient layer in the same family as the liquid gradient: it sits behind content, never carries information, and never reduces text contrast. Use it for at most one or two branded moments per page (typically the hero and the final CTA), never behind every section, and never on text, the logo, icons or UI controls.

When the target is Framer and its native `liquid-gradient` shader is available, prefer that (see `liquid-gradient.md`). Use this shader for HTML pages, Framer HTML embeds and other web surfaces where the 3D look is wanted. Loading from the CDN also keeps Framer embeds small enough to publish.

## Source

The shader is loaded **only from GitHub through jsDelivr**. Do not copy the bundle into a project, a skill or an embed, and do not self-host it.

- Repository: https://github.com/Studio-Stereogram/stereogram-embed-assets (public, must stay public)
- Script URL, pinned to a commit so it stays stable and cacheable:
  `https://cdn.jsdelivr.net/gh/Studio-Stereogram/stereogram-embed-assets@3cbdda5/shadergradient-embed.js`
- Contents: a self-contained bundle of React 18, three.js and `@shadergradient/react`, all MIT, with license notices at the end of the file. It is about 1.1 MB raw, or about 230–290 KB compressed.

**Updating:** commit the new bundle to the repository, then replace the commit hash in the URL everywhere it is used. Never point to `@main` or `@latest`: the CDN can serve a stale copy for days, and a broken push would reach every page at once.

## Usage

Mark the host element with `data-shader-gradient`. The script mounts a canvas into every such element on `DOMContentLoaded`. Options are JSON in the attribute; anything omitted falls back to the defaults below.

```html
<!-- as early as possible, so the download starts before the page body -->
<link rel="preconnect" href="https://cdn.jsdelivr.net">
<link rel="preload" as="script" href="https://cdn.jsdelivr.net/gh/Studio-Stereogram/stereogram-embed-assets@3cbdda5/shadergradient-embed.js">

<header class="shader-panel hero" data-shader-gradient>
  <div class="wrap">…content…</div>
</header>

<section class="shader-panel cta" data-shader-gradient='{"color1":"#0e001a","color2":"#7c3cb5","color3":"#a742ff"}'>
  …content…
</section>

<script src="https://cdn.jsdelivr.net/gh/Studio-Stereogram/stereogram-embed-assets@3cbdda5/shadergradient-embed.js" defer></script>
```

You can also call it from script: `ShaderGradientEmbed.mount(element, options)`, `ShaderGradientEmbed.init()`, and read `ShaderGradientEmbed.defaults`.

## Presets

Colours come from the Stereogram shader palettes (`assets/shader-presets.json`); only `color1–3` change between presets.

| Preset | Use | color1 | color2 | color3 |
|---|---|---|---|---|
| Blue (default) | Hero | `#00001A` | `#2962FF` | `#2E3B8F` |
| Violet | Closing CTA | `#0E001A` | `#7C3CB5` | `#A742FF` |

The other palettes (green, amber, teal, red, rose, neutral) map the same way: `color1` = palette[0], `color2` = palette[1], `color3` = palette[2]. Do not show two different palettes in the same viewport.

Default options (the Stereogram look; change colours only unless there is a reason):

```json
{
  "type": "plane", "animate": "on", "uSpeed": 0.4, "uStrength": 4, "uDensity": 1.3,
  "uFrequency": 5.5, "uAmplitude": 1, "positionX": -1.4, "positionY": 0, "positionZ": 0,
  "rotationX": 0, "rotationY": 10, "rotationZ": 50, "cAzimuthAngle": 180, "cPolarAngle": 90,
  "cDistance": 3.6, "cameraZoom": 1, "lightType": "3d", "brightness": 1.2, "reflection": 0.1,
  "grain": "on", "color1": "#00001A", "color2": "#2962FF", "color3": "#2E3B8F"
}
```

Keep `lightType: "3d"`. The `"env"` light type downloads an HDR environment map from a third-party host at runtime, which breaks in sandboxes and adds a network dependency.

## Layering

The script prepends its own canvas wrapper to the host at `z-index: 0`. Put any readability overlay above it and the content above both:

```css
.shader-panel { position: relative; isolation: isolate; overflow: hidden; background: #00001a; }
.shader-panel::after { content: ""; position: absolute; inset: 0; z-index: 1; pointer-events: none;
  background: linear-gradient(180deg, rgba(26,26,28,0) 30%, rgba(26,26,28,.55) 70%, #1a1a1c 100%); }
.shader-panel > .wrap { position: relative; z-index: 2; }
```

- Hero: the shader always spans the full viewport width, while the content keeps the page wrapper (max 1280 px, gutters 16 / 24 / 48 px on phone / tablet / desktop).
- CTA: the shader fills the rounded 4 px panel inside the content wrapper; use a horizontal veil (`linear-gradient(90deg, rgba(0,0,26,.72), rgba(0,0,26,.2))`) for text contrast.
- Always set the solid base colour (`#00001a` hero, `#0e001a` CTA). It is what visitors see before the shader loads and if WebGL fails.

## Entrance

Show only the solid base colour while loading, with no placeholder gradient. Once the first frames are drawn, fade the canvas in **softly over 1.2 s**: slow enough not to flash, quick enough not to feel late.

```css
[data-shader-gradient] canvas { opacity: 0; transition: opacity 1.2s cubic-bezier(.25,.1,.25,1); }
[data-shader-gradient].sg-on canvas { opacity: 1; }
@media (prefers-reduced-motion: reduce) { [data-shader-gradient] canvas { transition: none; } }
```

```html
<script>
/* add .sg-on once a canvas has drawn a few frames; the canvas mounts and unmounts as it scrolls in and out of view */
(function(){
  function watch(host){
    if(host.__sgw) return; host.__sgw=1; var t=0;
    function check(){
      if(!host.querySelector("canvas")){ clearTimeout(t); t=0; host.classList.remove("sg-on"); return; }
      if(host.classList.contains("sg-on")||t) return;
      t=setTimeout(function(){ requestAnimationFrame(function(){ requestAnimationFrame(function(){ t=0; if(host.querySelector("canvas")) host.classList.add("sg-on"); }); }); },120);
    }
    new MutationObserver(check).observe(host,{childList:true,subtree:true}); check();
  }
  function init(){ [].forEach.call(document.querySelectorAll("[data-shader-gradient]"),watch); }
  document.addEventListener("DOMContentLoaded",init); setTimeout(init,0); addEventListener("load",init);
})();
</script>
```

## Behaviour to rely on

- **Lazy mounting:** each host renders only while it is in the viewport; the canvas is removed when it scrolls out and re-created when it returns. Two shaders on one page never run at the same time.
- **Reduced motion:** with `prefers-reduced-motion: reduce` the script sets `animate: "off"` and shows a still frame.
- **Pixel density** is fixed at 1, which keeps it light on retina screens; the grain hides the lower resolution.
- **Static host position:** if the host is `position: static`, the script switches it to `relative`.

## Don'ts

- Do not inline, copy or self-host the bundle; always reference the pinned jsDelivr URL.
- Do not add a CSS "placeholder" gradient that imitates the shader; use the solid base colour.
- Do not put the shader behind body copy sections, cards, the navigation or the logo.
- Do not use gradient text or gradient labels anywhere to echo the shader. Chromatic gradients stay in the logo, the shader and tiny status details.
- Do not change the motion parameters to make it faster or more turbulent; it must stay slow and ambient.
- Do not ship it to email, PDF or slides; use a still frame there.
