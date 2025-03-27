+++
title = 'Setting up this Blog! - Part 1'
date = 2025-03-24T09:32:11+08:00
categories = ["Development", "DevOps"]
tags = ["hugo", "linux", "ci/cd", "programming", "devops", "pipeline", "automation", "blog", "markdown"]
keywords = ["SEO", "Keywords", "Here"]
description = "SEO Description Here"
draft = false
+++

## First Post! 🎉 Automating Blog Deployment with Hugo

Hey everyone—welcome to my brand-new blog! I'm super excited to kick things off by sharing how I set up and automated the deployment process of this very website you're visiting right now. After some research, I decided on **Hugo**, a fantastic static site generator that's incredibly quick, straightforward to use, and ideal for blogs and simpler websites. Let's dive in!

---

### Why Hugo? 🤔

Before we get technical, you might wonder: Why did I pick Hugo among all the options out there? Well, a few things caught my eye:

- 🚀 **Blazing Fast**: Hugo generates static sites almost instantly, even if your site grows to hundreds or thousands of posts. No more waiting around for slow builds!
  
- 🎯 **Easy to Use**: Even if you’re not deeply familiar with web development, Hugo is beginner-friendly. Content is written in Markdown, and setting everything up is a breeze.

- 🎨 **Highly Customizable**: Hugo has tons of beautiful themes, plugins, and community support—letting you easily personalize your blog's look and feel without any headaches.

---

### Step 1: Installing Hugo 💻

I won’t repeat every installation step here, as Hugo’s documentation already covers that wonderfully. To get set up quickly, check out their official instructions:

👉 [Official Hugo Installation Guide](https://gohugo.io/installation/)

For reference, here’s my current Hugo version:

```sh
hugo v0.145.0-666444f0a52132f9fec9f71cf25b441cc6a4f355 linux/amd64 BuildDate=2025-02-26T15:41:25Z VendorInfo=gohugoio
```

Always check you're using the same (or newer) version to follow along smoothly.

---

### Step 2: Setting Up Your Hugo Project 📂

Once Hugo's installed, let's start creating our actual blog project. Here’s exactly how I did mine:

```sh
hugo new site quickstart
cd quickstart
git init
git submodule add https://github.com/guangmean/Niello.git themes/Niello
echo "theme = 'Niello'" >> hugo.toml
hugo server
```

Let's quickly break down what's happening here:

- **New Hugo site**: `quickstart` is my chosen project name, feel free to pick whatever suits your project best.
- **Git Initialization**: I use Git for version control, helping me track changes easily and safely roll back if needed.
- **Theme Setup**: I went with the minimalist [Niello theme](https://github.com/guangmean/Niello)—it's sleek, clean, and keeps readers focused on content.
- **Configuration**: Adding the theme's name to `hugo.toml` tells Hugo which theme to use.
- **Local Preview**: Running `hugo server` gives you instant live preview updates as you write, edit, and tweak your site—extremely helpful!

---

### configure the theme

save the following to the hugo.toml file adjusting as needed.
```
baseURL = "https://www.angularcorp.com/" # Must end with splash
defaultContentLanguage = "en"
defaultContentLanguageInSubdir = true

staticDir = ["themes/Niello/static", "static"]

theme = "Niello"

[languages]
	[languages.en]
		title = "{CodeTrace} - Discover Issues, Share Solutions."
		languageCode = "en-us"
		LanguageName = "🇺🇸EN"
		contentDir = "content/en"
		weight = 1
		[[languages.en.menus.main]]
			name = 'Home'
			pageRef = '/'
			weight = 1
		[[languages.en.menus.main]]
			name = 'Categories'
			pageRef = '/categories'
			weight = 2
		[[languages.en.menus.main]]
			name = 'Tags'
			pageRef = '/tags'
			weight = 3
		[[languages.en.menus.main]]
			name = 'Contact'
			pageRef = '/contact'
			weight = 4
	[languages.zh]
		title = "{码途轨迹} - 发现问题，分享解决."
        	languageCode = "zh-cn"
        	LanguageName = "🇨🇳CN"
        	contentDir = "content/zh"
        	weight = 2
		[[languages.zh.menus.main]]
			name = '首页'
			pageRef = '/'
			weight = 1
		[[languages.zh.menus.main]]
			name = '文章分类'
			pageRef = '/categories'
			weight = 2
		[[languages.zh.menus.main]]
			name = '标签'
			pageRef = '/tags'
			weight = 3
		[[languages.zh.menus.main]]
			name = '联系我们'
			pageRef = '/contact'
			weight = 4
        
[params]
[params.google]
	google_ad_client = "ca-pub-******"  //Optional, replace ca-pub-****** with your content
	ga4 = "G-******" //Optional, replace G-****** with your Google Analytics GA4
[params.bannershowcase]
	categories = ["AI"]
	limit = 2
[params.email]
	contact = "angularcorp@outlook.com"
[params.ignore]
	categories = ["privacy", "terms", "archives", "cookie"]
[params.license]
	copyright = "&#xA9; 2019 ~ 2025 by guangmean. All Rights Reserved."
[params.share]
	sharethis = "******"  // Optional, Add hou ShareThis appid here
[params.disqus]
	shortname = "******"	// Optional, Disqus Comment Short Name

[outputs]
  home = ["HTML", "JSON"]
```
In order to change some translation key such as site title, slogan, etc you must create the following directory at the top level your project: `i18n`. 

- create an appropiate language file (e.g `en.toml` for english)

```
   # File path: i18n/en.toml
   [sitename]
   other = "Site Name"

   [siteslogan]
   other = "Site Slogan"

   [siteseokeywords]
   other = "Site Home SEO Keywords"

   [siteseodescription]
   other = "Site Home SEO Description"
```



---

### Step 3: Creating Your First Post 📝

Now, the fun part—your first post! Hugo makes adding new content effortless. Just run:

```sh
hugo new content posts/my-first-post.md
```

This creates a fresh Markdown file (`my-first-post.md`). Markdown is super intuitive and easy-to-read. Just open the file, start writing, and save—Hugo handles everything else. Trust me; you'll love how straightforward it feels.

---

### Why the Niello Theme? 🎨✨

I specifically picked [Niello](https://github.com/guangmean/Niello) because of its clean, minimalist design. The layout puts the spotlight firmly on your content without distractions. Plus, it's fully responsive—your posts look fantastic on desktops, tablets, and mobile phones alike.

---

### Automating the Deployment Process ⚙️🔄

This blog is deployed using the default github actions for hugo static sites. Under settings/pages enable github actions under the build and deploy section:

![screenshot](/image/Screenshot_20250326_145149.png)


```
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["main"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.128.0
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
        run: |
          hugo \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4

```

---

### Search

To enable the search functionality, you need to configure JSON output in the hugo.toml file by adding the following:

```
[outputs]
  home = ["HTML", "JSON"]
```

And a search folder with an _index.md file under the content directory is required, for example: content/en/search/_index.md

```
+++

title = "Search Results"
date = 2024-12-13T15:00:00+08:00
draft = false
layout = "search"

+++

```

With this setup, the search URL will be /en/search/?q=keywords

---