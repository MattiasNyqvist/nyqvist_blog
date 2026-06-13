# Astro Starter Kit: Blog

```sh
npm create astro@latest -- --template blog
```

> 🧑‍🚀 **Seasoned astronaut?** Delete this file. Have fun!

Features:

- ✅ Minimal styling (make it your own!)
- ✅ 100/100 Lighthouse performance
- ✅ SEO-friendly with canonical URLs and Open Graph data
- ✅ Sitemap support
- ✅ RSS Feed support
- ✅ Markdown & MDX support

## 🚀 Project Structure

Inside of your Astro project, you'll see the following folders and files:

```text
├── public/
├── src/
│   ├── assets/
│   ├── components/
│   ├── content/
│   ├── layouts/
│   └── pages/
├── astro.config.mjs
├── README.md
├── package.json
└── tsconfig.json
```

Astro looks for `.astro` or `.md` files in the `src/pages/` directory. Each page is exposed as a route based on its file name.

There's nothing special about `src/components/`, but that's where we like to put any Astro/React/Vue/Svelte/Preact components.

The `src/content/` directory contains "collections" of related Markdown and MDX documents. Use `getCollection()` to retrieve posts from `src/content/blog/`, and type-check your frontmatter using an optional schema. See [Astro's Content Collections docs](https://docs.astro.build/en/guides/content-collections/) to learn more.

Any static assets, like images, can be placed in the `public/` directory.

## 🧞 Commands

All commands are run from the root of the project, from a terminal:

| Command                   | Action                                           |
| :------------------------ | :----------------------------------------------- |
| `npm install`             | Installs dependencies                            |
| `npm run dev`             | Starts local dev server at `localhost:4321`      |
| `npm run build`           | Build your production site to `./dist/`          |
| `npm run preview`         | Preview your build locally, before deploying     |
| `npm run astro ...`       | Run CLI commands like `astro add`, `astro check` |
| `npm run astro -- --help` | Get help using the Astro CLI                     |

## ✍️ Publishing a new post

A quick checklist for adding a new article to the blog.

### 1. Write the text

Create a Markdown file, for example `my-content.md`.

### 2. Prepare the hero image

Export a `.jpg` at 1200 × 630 px and give it the same name as the post, for example `my-content.jpg`. Compress it (e.g. with [Squoosh](https://squoosh.app/)) to keep it under 200 KB.

### 3. Create a folder for the image

Add a folder named after the post under `src/assets/images/blog/`:

```text
src/assets/images/blog/my-content/my-content.jpg
```

The hero image must live in `src/assets/` because it is handled by Astro's `<Image />` component at build time.

### 4. Add the frontmatter

Place this at the top of your `.md` file:

```markdown
---
title: "My Content"
description: "This is my post about ..."
pubDate: 2026-05-17
heroImage: "../../assets/images/blog/my-content/my-content.jpg"
tags: ["ai", "claude"]
---
```

### 5. Move the post into the collection

```text
src/content/blog/my-content.md
```

### 6. Preview locally

```sh
npm run dev
```

Open `http://localhost:4321` and check that the hero image renders and that the post shows up in the list.

### 7. Push to GitHub

```sh
git add .
git commit -m "Post about..."
git push
```

Azure picks up the push and deploys automatically. The post is live within a couple of minutes.

**Troubleshooting**

- **Image not showing?** Check that the `heroImage` path matches exactly (it's case-sensitive) and restart the dev server.
- **Post missing from the list?** Make sure `pubDate` is not in the future — Astro hides future-dated posts automatically.

## 👀 Want to learn more?

Check out [our documentation](https://docs.astro.build) or jump into our [Discord server](https://astro.build/chat).

## Credit

This theme is based off of the lovely [Bear Blog](https://github.com/HermanMartinus/bearblog/).