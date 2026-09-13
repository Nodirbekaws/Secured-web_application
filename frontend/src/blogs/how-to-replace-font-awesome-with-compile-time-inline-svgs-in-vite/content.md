---
title: "How to replace Font Awesome with compile-time inline SVGs in Vite"
subtitle: "Eliminate heavy icon fonts, prevent layout shifts, and bundle only the icons you need using unplugin-icons and Iconify."
date: "September 13, 2026"
tags: ["Web performance", "Vite", "Vue", "Frontend", "Optimization"]
---

## Introduction

When starting a new web application, adding an icon library is often one of the first steps. For years, [Font Awesome](https://fontawesome.com/) was my default choice. It was intuitive, easy to get started with, and let me prototype interfaces without worrying about individual SVG files.

While building [SetupRkhis](https://setuprkhis.com/), a price comparison and hardware tracker for PC stores in Morocco, I did the same thing. At first, Font Awesome made adding icons easy. But when I checked the production build for performance, I noticed a problem in the network tab. The app was loading over 354 kB of icon files on the first page load:

- A render-blocking global stylesheet (`all.min.css`) at 110.77 kB (33.18 kB gzip).
- A webfont file (`fa-solid-900.woff2`) at 243.74 kB containing thousands of vector glyphs.

The entire application only needed 38 icons. That means over 98% of the downloaded data was completely unused. On top of that, because the font loaded asynchronously, the page shifted and icons flickered while the browser waited for the `.woff2` file to download.

To fix this, I migrated SetupRkhis from Font Awesome icon fonts to inline SVGs using [unplugin-icons](https://github.com/unplugin/unplugin-icons) and [Iconify](https://iconify.design/). By bundling only the icons the app actually used, I removed the webfont file completely, cut the stylesheet size by nearly 70%, and reduced the initial gzipped bundle by 89.3%, from 298.30 kB down to 31.96 kB.

In this guide, I will show you why icon fonts hurt performance and how to migrate your Vue 3 and Vite app to inline SVGs step by step.

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

::: video ./font_icon_flicker_demo.mp4 "The page renders with missing icons before the font file loads and abruptly shifts the layout."
:::
