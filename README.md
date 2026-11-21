# Blog starter (Jekyll + Minimal Mistakes, for GitHub Pages)

Write posts as Markdown files. GitHub builds and hosts the site for free.

## Put it online (about 10 minutes)

This copy is set up for the repository `boriskrulj/repo_git`; the site will be at https://boriskrulj.github.io/repo_git/ .

1. Open https://github.com/boriskrulj/repo_git, click **Add file → Upload files**, and drag in everything from inside this folder (keep the folder structure and include the hidden `.gitignore`). Commit.
2. Open `_config.yml` on GitHub and fill in `Your Blog Title`, `Your Name` and the bio. Commit.
3. Go to **Settings → Pages**, choose **Deploy from a branch**, branch `main`, folder `/ (root)`, and save.
4. After a minute or two the site is live. If it looks broken, open the repository's **Actions** tab to see the build error.

Note: the files already in the repository (`app.txt`, `update.txt`) will also be served on the site.

If you later want the shorter address https://boriskrulj.github.io , create a repository named `boriskrulj.github.io`, put these files there and set `baseurl: ""` in `_config.yml`.

## Write a post

Create a file in `_posts/` named `YYYY-MM-DD-short-title.md` starting with:

```
---
title: "Your title"
date: 2026-11-21
tags: [world-models]
---
```

Equations work: `$x^2$` inline, `$$ ... $$` on their own line. Images go in `assets/images/` and are shown with `![alt text](/assets/images/file.png)`.

## The draft already included

`_drafts/2026-11-21-door-checkpoints-lab-notebook.md` is a first post written from the project notes. It is **not published**: Jekyll ignores `_drafts/`. To publish, move it to `_posts/` (the date prefix is already there). It contains no paths, names of employers, or code from the project. Read it through before you publish.

## Before you publish anything about work done at your employer

Check your employer's policy on publishing, and whether the work needs approval. If in doubt, publish the idea and the results and leave out code and internal details. (Delete this section from the README before you make the repository public, since the README is visible there.)

## Preview on your computer (optional)

Install Ruby, then in this folder:

```
bundle install
bundle exec jekyll serve --drafts
```

and open http://localhost:4000. The `--drafts` flag shows files from `_drafts/` too.

## Changing the look

In `_config.yml` change `minimal_mistakes_skin` (`default`, `air`, `aqua`, `contrast`, `dark`, `dirt`, `neon`, `mint`, `plum`, `sunrise`). The navigation bar is in `_data/navigation.yml`. The theme's documentation is at https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/.
