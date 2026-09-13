---
title: "How to replace Font Awesome with compile-time inline SVGs in Vite"
subtitle: "Eliminate heavy icon webfonts, prevent layout shifts, and bundle only the icons you need using unplugin-icons and Iconify."
date: "September 13, 2026"
tags: ["Web performance", "Vite", "Vue", "Frontend", "Optimization"]
---

## Introduction

When starting a new web application, adding an icon library is often one of the first steps. For years, [Font Awesome](https://fontawesome.com/) was my default choice. It was intuitive, easy to get started with, and let me prototype interfaces without worrying about individual SVG files.

While building [SetupRkhis](https://setuprkhis.com/), a price comparison and hardware tracker for PC stores in Morocco, I did the same thing. At first, Font Awesome made adding icons easy. But when I checked the production build for performance, I noticed a problem in the network tab. The app was loading over 354 kB of icon files on the first page load:

- A render-blocking global stylesheet (`all.min.css`) at 110.77 kB (33.18 kB gzip).
- A webfont file (`fa-solid-900.woff2`) at 243.74 kB containing thousands of vector glyphs.

The entire application only needed 38 icons. That means over 98% of the downloaded data was completely unused. On top of that, because the font loaded asynchronously, the page shifted and icons flickered while the browser waited for the `.woff2` file to download.

To fix this, I migrated SetupRkhis from Font Awesome webfonts to inline SVGs using [unplugin-icons](https://github.com/unplugin/unplugin-icons) and [Iconify](https://iconify.design/). By bundling only the icons the app actually used, I removed the webfont file completely, cut the stylesheet size by nearly 70%, and reduced the initial gzipped bundle by 89.3%, from 298.30 kB down to 31.96 kB.

In this guide, I will show you why icon fonts hurt performance and how to migrate your Vue 3 and Vite app to inline SVGs step by step.
