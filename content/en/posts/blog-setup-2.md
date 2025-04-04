---
title: "Setting up this Blog! - Part 2"
description: "Part 2 of my blog setup series—customizing icons, replacing stock images, and tweaking theme visuals in Hugo with the Niello theme."
date: 2025-04-02T00:33:55-06:00
draft: false
tags: ["hugo", "blog setup", "static site", "niello theme", "customization", "web dev"]
category: "dev notes"
seo_keywords: ["Hugo blog customization", "Niello theme Hugo", "how to replace favicon in Hugo", "custom logo static site", "adjusting outline in Hugo theme", "static site setup tutorial"]
---

Welcome to part 2 of the blog setup series. In this one, I'm replacing some stock assets and making a few small visual tweaks to the navigation bar.

---

## 🖼️ Replacing Stock Icons

### ✅ Custom Logo

Copy your custom logo to the `/static/images` directory.

### 🌐 Favicon

Drop your custom favicon at the top level of the `/static` directory. Hugo will serve it from the root when the site builds.

---

## 🧪 Tweak the Nav Logo Outline

This one’s just a minor nitpick—I wanted to reduce the thick white outline around the logo in the nav bar. If you're using the **Niello** theme, head to:

```bash
themes/Niello/layout/partials/nav.html
```

Find this snippet:

```html
<img src="/image/logo.webp?v={{ now.Unix }}" alt="Logo"
    class="w-16 h-16 my-5 p-1 bg-gray-100 rounded-full object-cover cursor-pointer hover:scale-110" />
```

And change the padding slightly:

```html
<img src="/image/logo.webp?v={{ now.Unix }}" alt="Logo"
    class="w-16 h-16 my-5 p-.8 bg-gray-100 rounded-full object-cover cursor-pointer hover:scale-110" />
```

*Tiny tweak, but it made a visual difference I liked.*

---

## 📂 Other Stock Images

In the `/static/images` folder, you'll find a few more stock assets you can replace:

- `common-404.webp`
- `common-writing-empty.webp`

Once I come up with better versions of these, I’ll drop them into the same directory and they’ll overwrite the theme defaults automatically.

---
