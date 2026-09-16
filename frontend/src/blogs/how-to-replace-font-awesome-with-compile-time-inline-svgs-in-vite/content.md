---
title: "How to replace Font Awesome with compile-time inline SVGs in Vite"
subtitle: "Eliminate heavy icon fonts, prevent layout shifts, and bundle only the icons you need using unplugin-icons and Iconify."
date: "September 16, 2026"
tags: ["Web performance", "Vite", "Vue", "Frontend", "Optimization"]
---

## Introduction

When starting a new web application, adding an icon library is one of the first steps. For years, [Font Awesome](https://fontawesome.com/) was my default choice. It was intuitive, easy to get started with, and let me prototype interfaces without worrying about managing SVG files.

While building [SetupRkhis](https://setuprkhis.com/), a price comparison and deal tracker for tech products in Morocco, I did the same thing. At first, Font Awesome made adding icons easy. But when I checked the production build for performance, I noticed a problem in the network tab. The app was loading over 354 kB of icon files on the first page load:

- A global stylesheet (`all.min.css`) at 110.77 kB (33.18 kB gzip) that blocked the initial page render.
- A webfont file (`fa-solid-900.woff2`) at 243.74 kB containing thousands of icons.

The entire application only needed 38 icons. That means over 98% of the downloaded data was completely unused. On top of that, because the font loaded asynchronously, the page shifted and icons flickered while the browser waited for the `.woff2` file to download.

To fix this, I migrated SetupRkhis from Font Awesome icon fonts to inline SVGs using [unplugin-icons](https://github.com/unplugin/unplugin-icons) and [Iconify](https://iconify.design/). By bundling only the icons the app actually used, I removed the webfont file completely, cut the stylesheet size by nearly 70%, and reduced the initial gzipped bundle by 89.3%, from 298.30 kB down to 31.96 kB.

In this guide, I will show you why icon fonts hurt performance and how to migrate your Vue 3 and Vite app to inline SVGs step by step.

::: info Note
The metrics shared in this guide were recorded while migrating SetupRkhis from Font Awesome to inline SVGs and serve as an indication of what is possible. While exact bundle sizes will vary depending on your app and how many icons you use, the overall performance gains and data savings will be very similar.
:::

## Why icon fonts slow down your site

Icon fonts work exactly like text fonts. Instead of letters like "A" or "B", the font file assigns an icon to a specific character code. While this made icons easy to style with CSS in the past, it creates three clear performance problems for modern web apps.

### Downloading thousands of unused icons

The main issue with an icon font is that it is an all-or-nothing package.

When you use Font Awesome, the font file includes thousands of icons. The browser cannot download only the cart ::fa{fa-solid fa-cart-shopping}:: or search ::fa{fa-solid fa-magnifying-glass}:: icon from inside that file. It must fetch the entire `.woff2` file, which in this case was around 244 kB, even if your page only uses a handful of icons.

### Render-blocking stylesheets

Before the browser can request the font file, it has to load the Font Awesome stylesheet (`all.min.css`).

Because stylesheets block rendering by default, the browser waits until the CSS file finishes downloading and parsing before displaying anything on screen. In SetupRkhis, this meant the browser had to process 110 kB of CSS rules on the first load, most of which defined icons the site never used.

### Layout shifts and flickering

Once the CSS finishes loading, the browser discovers it needs the `.woff2` font file.

Because fonts load over the network, there is always a short delay before the file arrives. During that wait, either the icon space stays completely blank, or the browser displays an empty box or broken character symbol.

When the font finally loads, the icons suddenly pop into place. This jump pushes nearby text and buttons around, creating layout shifts that make the interface feel unstable, as shown below:

::: video ./1_font_icon_flicker_demo.mp4 "The page renders with missing icons before the font file loads and abruptly shifts the layout."
:::

## How inline SVGs fix the problem

Inline SVGs fix all three problems. Instead of relying on character codes inside a separate font file, the browser renders the vector paths directly as part of the component template.

Because the SVG code lives inside the JavaScript chunk, the icon is ready the moment the component mounts. There are no font files to download, no blank spaces, and no layout shifts.

### The old problems with SVGs

In the past, managing SVGs in a large project was tedious. You had to copy and paste raw `<svg>` markup directly into dozens of separate Vue components, or manually download individual files and run them through optimization tools like SVGO. It was repetitive, cluttered the codebase, and made simple updates feel like a chore.

Because of this friction, many developers, including me, simply stuck with icon fonts. Even though icon fonts hurt performance and lacked flexibility, they were convenient enough to justify avoiding the hassle of manual SVG management.

### Modern inline SVGs with unplugin-icons and Iconify

Together, [unplugin-icons](https://github.com/unplugin/unplugin-icons) and [Iconify](https://iconify.design/) remove that friction entirely by turning SVGs into on-demand, compile-time components.

Iconify packages open-source icon collections into standard npm datasets. Instead of downloading files manually, you install the exact icon set you need (such as `@iconify-json/fa7-solid`).

Then, `unplugin-icons` lets you import any icon from those datasets directly as a standard Vue component:

```vue
<script setup>
import IconCart from "~icons/fa7-solid/cart-shopping";
</script>

<template>
  <button type="button">
    <IconCart />
    <span>Cart</span>
  </button>
</template>
```

At build time, the plugin replaces `<IconCart/>` with the actual inline SVG markup. You write clean component syntax, but the browser receives pure HTML without extra runtime overhead:

```html
<button type="button">
  <svg data-v-f7246898="" viewBox="0 0 640 640" width="1em" height="1em" aria-hidden="true">
    <path
      fill="currentColor"
      d="M24 48C10.7 48 0 58.7 0 72s10.7 24 24 24h45.3c3.9 0 7.2 2.8 7.9 6.6l52.1 286.3c6.2 34.2 36 59.1 70.8 59.1H456c13.3 0 24-10.7 24-24s-10.7-24-24-24H200.1c-11.6 0-21.5-8.3-23.6-19.7l-5.1-28.3H475c30.8 0 57.2-21.9 62.9-52.2l31-165.9c3.7-19.7-11.4-37.9-31.5-37.9H124.7l-.4-2c-4.8-26.6-28-46-55.1-46zm184 528c26.5 0 48-21.5 48-48s-21.5-48-48-48s-48 21.5-48 48s21.5 48 48 48"
    ></path>
  </svg>
  <span>Cart</span>
</button>
```

### How it works at build time

When Vite builds your app, `unplugin-icons` intercepts any import starting with `~icons/`. It reads the SVG data directly from the local Iconify package and converts it into a Vue component.

This approach gives you two advantages:

- **Zero bundle bloat:** If your app uses 38 icons, Vite compiles and bundles only those 38 icons. The thousands of other icons in the package stay inside `node_modules` and never reach your production bundle.
- **Seamless styling:** The compiled SVG uses `1em` dimensions and `currentColor`, meaning it automatically scales with your font size and inherits your text color. Your existing CSS classes continue to work without adjustments.

## How to set up unplugin-icons in Vite

Migrating an existing project takes only a few minutes. You remove the old package, register the Vite plugin, delete the global CSS import, and replace your icon tags with components.

### Install the dependencies

First, remove `@fortawesome/fontawesome-free` and install `unplugin-icons` along with the Iconify datasets you need:

```bash
pnpm remove @fortawesome/fontawesome-free
pnpm add -D unplugin-icons @iconify-json/fa7-solid @iconify-json/fa7-regular @iconify-json/fa7-brands
```

::: info
If you use `npm` or `yarn`, replace `pnpm add -D` with `npm install -D` or `yarn add -D`.
:::

:::: tip
You are not limited to Font Awesome. Iconify supports dozens of open-source packs, including Lucide, Material Design, and Tabler Icons. You can browse available collections in the [Iconify Icon Sets directory](https://icon-sets.iconify.design/).

::: image ./2_iconify_sets.png "Iconify icon sets directory showing popular collections"
The Iconify directory lets you search and preview thousands of open-source icon sets.
:::
::::

### Configure the Vite plugin

Next, add `unplugin-icons` to your `vite.config.ts` (or `vite.config.js`). Set the compiler option to `vue3`:

```ts
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import Icons from "unplugin-icons/vite";

export default defineConfig({
  plugins: [
    vue(),
    Icons({
      compiler: "vue3",
      scale: 1,
    }),
  ],
});
```

Setting `scale: 1` gives each generated `<svg>` a width and height of `1em`, allowing the icon to adapt naturally to your font size.

If you use TypeScript, add the type declarations to `env.d.ts` so your editor recognizes the virtual `~icons/` imports:

```ts
/// <reference types="vite/client" />
/// <reference types="unplugin-icons/types/vue" />
```

### Remove the global stylesheet

Open your entry file (`main.ts` or `main.js`) and remove the Font Awesome CSS import:

```ts
// Delete this line:
import "@fortawesome/fontawesome-free/css/all.min.css";
```

Removing this line completely eliminates that 110 kB render-blocking stylesheet.

### Replace icons in your components

Now you can replace your old `<i class="fa-solid fa-*"></i>` tags with explicit icon components:

```vue
<script setup lang="ts">
import IconCart from "~icons/fa7-solid/cart-shopping";
import IconHeart from "~icons/fa7-solid/heart";
</script>

<template>
  <button type="button">
    <IconCart />
    <span>Cart</span>
  </button>

  <button type="button">
    <IconHeart />
    <span>Save</span>
  </button>
</template>
```

## Measured performance impact

To measure the real impact of this migration, I compared the production build of SetupRkhis before and after replacing Font Awesome.

| Asset layer                       | Before (Font Awesome) | After (unplugin-icons) | Difference  | Change     |
| --------------------------------- | --------------------- | ---------------------- | ----------- | ---------- |
| Binary webfonts (.woff2)          | 243.74                | 0.00                   | -243.74     | -100.0%    |
| Production stylesheet (CSS, gzip) | 33.18                 | 6.58                   | -26.60      | -80.2%     |
| Entry JavaScript (gzip)           | 17.17                 | 21.07                  | +3.90       | +22.7%     |
| **Total transfer size (gzip)**    | **298.30**            | **31.96**              | **-266.34** | **-89.3%** |

::: info Note
All file sizes are measured in kilobytes (kB).
:::

Removing the `.woff2` file completely eliminated font requests during the initial page load, while deleting `all.min.css` saved 26.6 kB of compressed CSS and removed a render-blocking bottleneck. The only drawback was a slightly larger JavaScript bundle. Inlining the SVG paths for 38 icons added 3.9 kB of compressed code to the entry file.

Trading a 3.9 kB increase in JavaScript for a 266 kB reduction in compressed assets is an easy decision. It speeds up initial page rendering and prevents icon-related layout shifts entirely.

## Conclusion

Icon fonts were once a convenient way to handle scalable icons, but they carry too much dead weight for modern web applications. Downloading thousands of unused icons just to render a handful of icons slows down the initial page load and creates layout instability.

Compiling inline SVGs with `unplugin-icons` eliminates that friction. You keep the design convenience of large icon catalogs like Font Awesome, but ship only the exact vector paths your application uses.

If your application still relies on a global icon font, audit your network tab. Replacing it with compile-time inline SVGs is one of the fastest ways to eliminate layout shifts and cut unnecessary kilobytes from your production build.
