# Expanse Community

This repository is the home of the [Expanse](https://expanse.sh) community:

- **[Discussions](https://github.com/expanse-labs/community/discussions)** — questions, ideas, show-and-tell, and anything about running GPU and HPC workloads with Expanse. Open to everyone.
- **`blog/`** — the source of the official [Expanse blog](https://expanse.sh/blog). Posts are markdown files, published after merge to `main` and the next landing-site deployment.

Comments and reactions on blog posts live in Discussions — each post links to its discussion thread, and the blog renders that thread's comments and reactions.

## Writing a blog post

Official posts are authored by the Expanse team via pull request. One markdown file per post in `blog/`, named after its URL slug (`blog/<slug>.md` → `expanse.sh/blog/<slug>`).

Frontmatter contract:

```yaml
---
title: The post title
description: One-sentence summary, used for the meta description and index cards.
date: 2026-02-28
author: Full Name
github: github-username   # optional, links the byline
tags: [hpc, slurm]
draft: false              # true = in repo, not published
featured: false           # true = ordered before other posts on the blog index
---
```

### Images

New posts can supply responsive local images. Paths are relative to `blog/`, so
keep a post's assets under `blog/images/<slug>/` and use this frontmatter:

```yaml
images:
  alt: A concise description of the image
  social: images/<slug>/social.png # optional PNG/JPEG, exactly 1200x630, max 5 MB
  card:
    mobile: images/<slug>/card-mobile.webp # 2:1
    desktop: images/<slug>/card-desktop.webp # 6:5
  header:
    mobile: images/<slug>/header-mobile.webp # 5:4
    desktop: images/<slug>/header-desktop.webp # 12:5
```

Card and header files may be `.avif`, `.jpeg`, `.jpg`, `.png`, or `.webp`.
The landing build validates the paths, copies the assets to `expanse.sh`, and
uses content-hashed URLs so a replaced asset is not held by browser caches.
Do not combine `images` with the deprecated `image` and `imageAlt` fields,
which remain only for posts published before this contract.

YouTube embeds: put the video URL on its own line and the blog renders it as an embedded player.

## Community posts

Anyone can write in the [Community Posts](https://github.com/expanse-labs/community/discussions) category. Posts the team marks with the `featured` label are surfaced on the Expanse blog index, linking back to the discussion.
