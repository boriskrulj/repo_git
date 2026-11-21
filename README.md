# Personal site and blog (Jekyll, for GitHub Pages)

A one-page portfolio (Introduction, About, Projects, Blog, Contact, Resume) plus a blog. Plain CSS, no external fonts, light and dark mode.

## Put it online (about 10 minutes)

This copy is set up for the repository `boriskrulj/repo_git`; the site will be at https://boriskrulj.github.io/repo_git/ .

1. Open https://github.com/boriskrulj/repo_git, click **Add file -> Upload files**, and drag in everything from inside this folder (keep the folder structure and include the hidden `.gitignore`). Commit.
2. Open `_config.yml` on GitHub (pencil icon) and fill in the lines at the top: your name, tagline, bio, email, LinkedIn, resume link and the interests list. Commit.
3. Go to **Settings -> Pages**, choose **Deploy from a branch**, branch `main`, folder `/ (root)`, and save.
4. After a minute or two the site is live. If it looks broken, open the repository's **Actions** tab to see the build error.

The files already in the repository (`app.txt`, `update.txt`) will also be served on the site.

If you later want the shorter address https://boriskrulj.github.io , create a repository named `boriskrulj.github.io`, put these files there and set `baseurl: ""` in `_config.yml`.

## Where things are

- `_config.yml`: your details (top of the file).
- `index.html`: the home page sections. Edit the text there.
- `_data/projects.yml`: one block per project card.
- `assets/css/style.css`: colours and layout (the colours are the variables at the top).
- `_posts/`: published posts. `_drafts/`: posts that are not published.
- `assets/images/`: pictures; show one in a post with `![alt text]({{ '/assets/images/file.png' | relative_url }})`.

## Write a post

Create a file in `_posts/` named `YYYY-MM-DD-short-title.md` starting with:

```
---
layout: post
title: "Your title"
tags: [world-models]
---
```

Add `math: true` to the top block of a post to turn on equations (`$x^2$` inline, `$$ ... $$` on its own line).

## The draft already included

`_drafts/2026-11-21-door-checkpoints-lab-notebook.md` is a first post written from the project notes. It is **not published**: Jekyll ignores `_drafts/`. To publish, move it to `_posts/`. It contains no paths, names of employers, or code from the project. Read it through before you publish.

## Before you publish anything about work done at your employer

Check your employer's policy on publishing, and whether the work needs approval. If in doubt, publish the idea and the results and leave out code and internal details. (Delete this section from the README before you make the repository public, since the README is visible there.)

## Preview on your computer (optional)

Install Ruby, then in this folder:

```
bundle install
bundle exec jekyll serve --drafts
```

and open http://localhost:4000/repo_git/ . The `--drafts` flag shows files from `_drafts/` too.
