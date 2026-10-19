# stereogram-embed-assets

Public static assets for Stereogram's Framer HTML embeds, served via jsDelivr.

| File | Use |
| --- | --- |
| `shadergradient-embed.js` | ShaderGradient (React + three.js) bundle. Mounts on every `[data-shader-gradient]` element; options as JSON in the attribute. |

```html
<section data-shader-gradient='{"color1":"#0e001a","color2":"#7c3cb5","color3":"#a742ff"}'>…</section>
<script src="https://cdn.jsdelivr.net/gh/Studio-Stereogram/stereogram-embed-assets@3cbdda5/shadergradient-embed.js" defer></script>
```

Pin to a commit (`@3cbdda5`) so the CDN URL stays stable and cacheable; a changed bundle gets a new commit and a new URL.

Third-party code in the bundle (React, three.js, @shadergradient/react, camera-controls) is MIT licensed; license notices are kept at the end of the file. No font files are stored here.

## Docs

- [docs/shader-gradient.md](docs/shader-gradient.md): when and how to use the shader (presets, layering, 1.2 s fade-in, rules). The stereogram-design skill references this.
- [docs/example.html](docs/example.html): minimal hero + CTA example.
