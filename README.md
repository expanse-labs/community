# Expanse Community

This repository is the home of the [Expanse](https://expanse.sh) community:

- **[Discussions](https://github.com/expanse-labs/community/discussions)** — questions, ideas, show-and-tell, and anything about running GPU and HPC workloads with Expanse. Open to everyone.
- **`blog/`** — the source of the official [Expanse blog](https://expanse.sh/blog). Posts are markdown files, published when merged to `main`.

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
featured: false           # true = pinned to the top of the blog index
---
```

YouTube embeds: put the video URL on its own line and the blog renders it as an embedded player.

## Community posts

Anyone can write in the [Community Posts](https://github.com/expanse-labs/community/discussions) category. Posts the team marks with the `featured` label are surfaced on the Expanse blog index, linking back to the discussion.
