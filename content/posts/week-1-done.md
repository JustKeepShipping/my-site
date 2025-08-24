---
title: "Week-1-done"
date: 2025-08-24T12:00:00+10:00
draft: false
---

What I Did
1. Setup WSL & Ubuntu

Installed Ubuntu inside Windows Subsystem for Linux (WSL).

Learned about the Linux filesystem (~/projects) and created a dedicated folder for experiments.

Troubleshot some quirks like quotes in Bash and directory permissions.

2. Installed Hugo

Installed Hugo (static site generator).

Upgraded to the extended build to support themes like PaperMod.

Created my first Hugo site with hugo new site my-site.

3. Added PaperMod Theme

Cloned PaperMod as a Git submodule.

Edited the hugo.toml config to properly enable the theme.

Learned TOML syntax basics and fixed errors caused by duplicate or malformed entries.

4. Made My First Content

Used hugo new posts/hello-world.md to create a post.

Edited the markdown to publish it (setting draft: false).

Previewed the site locally at http://localhost:1313.

5. Set Up Git & GitHub

Initialized a Git repo, committed changes, and connected to a new GitHub repo.

Learned about Personal Access Tokens (PATs), since GitHub no longer supports password pushes.

Fixed a mistake where I was tracking the generated public/ folder (added .gitignore).

6. Built CI/CD with GitHub Actions

Wrote a GitHub Actions workflow (.github/workflows/deploy.yml).

Debugged YAML indentation issues and missing configure-pages step.

Fixed authentication scopes so Actions could push deployments.

Finally saw the pipeline run end-to-end: push code → GitHub builds site → Pages deploys live.

7. Went Live 🎉

Site published at https://JustkeepShipping.github.io/my-site/.

Confirmed the menu, theme, and first post all rendered correctly.

Proved CI/CD by adding a new post and watching it go live automatically.

What I Learned

The basics of Linux commands and file paths inside WSL.

Static site generation with Hugo and TOML configuration.

How Git and GitHub interact (repos, remotes, branches).

Authentication with GitHub PATs and why workflow permissions matter.

Debugging real-world problems (YAML indentation, Hugo extended, Pages deployment).

How to wire together a minimal but modern DevOps pipeline.
