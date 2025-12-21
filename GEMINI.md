
# Gemini Guide: Setting Up Your Hugo Blog

This guide continues our conversation and provides the next steps for setting up and deploying your Hugo blog.

## Current Status
- **Hugo is installed** on your local machine.
- You have **created and cloned a new, empty repository** for your blog's source code (this `blog-source` repository).

---

## Part 1: Create and Configure Your Hugo Site

First, we'll create the basic Hugo site structure, add a theme, and configure it.

### Step 1: Create the Hugo Site
Navigate into your `blog-source` directory in your terminal and run:
```bash
hugo new site . --force
```
This command populates the current directory with Hugo's folder structure (`archetypes`, `content`, `static`, etc.).

### Step 2: Add a Theme
Every Hugo site needs a theme. We'll use the `ananke` theme as an example. It's a good starting point.
```bash
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
```

### Step 3: Configure Your Site (`hugo.toml`)
Hugo now defaults to `hugo.toml` for configuration. Open this file and replace its contents with the following. **This is a critical step.**

```toml
# hugo.toml
baseURL = "https://adnair2024.github.io/blog/"
languageCode = "en-us"
title = "My New Blog"
theme = "ananke"

# Important for subdirectory deployment:
relativeURLs = true
uglyURLs = true 
```
*   `baseURL` tells Hugo where the site will live.
*   `relativeURLs = true` is essential for links to work correctly when served from a `/blog/` subdirectory.

### Step 4: Create Your First Post
Generate a new post to have some content on your blog.
```bash
hugo new posts/my-first-post.md
```
Open `content/posts/my-first-post.md`, add some text, and make sure to change `draft: true` to `draft: false` at the top of the file so it will be published.

### Step 5: Test Locally
Before deploying, preview your site locally.
```bash
hugo server -D
```
Open your browser to the URL provided (usually `http://localhost:1313`). You should see your new blog with your first post. Press `Ctrl + C` in the terminal to stop the server.

### Step 6: Commit Your Work
Save all the new files to your `blog-source` git repository.
```bash
git add .
git commit -m "feat: Initial Hugo blog setup"
git push origin main
```

---

## Part 2: Automated Deployment with GitHub Actions

This is the recommended way to publish your blog. Every time you `git push` new changes to this `blog-source` repository, a GitHub Action will automatically build your site and deploy it to the `blog` folder in your `adnair2024.github.io` repository.

### Step 1: Generate a Personal Access Token (PAT)
This token will allow your `blog-source` repository to write to your `adnair2024.github.io` repository.

1.  Go to GitHub -> Settings -> Developer settings -> Personal access tokens -> **Tokens (classic)**.
2.  Click **Generate new token**.
3.  Give it a descriptive name (e.g., `HUGO_BLOG_DEPLOY`).
4.  Set the **Expiration** as desired.
5.  **Crucially, grant it the `repo` scope.**
6.  Click **Generate token** and **copy the token immediately**. You won't be able to see it again.

### Step 2: Add the PAT as a Repository Secret
1.  In your `blog-source` repository on GitHub, go to `Settings` -> `Secrets and variables` -> `Actions`.
2.  Click **New repository secret**.
3.  Name the secret exactly: `GH_PAGES_DEPLOY`.
4.  Paste the PAT you copied into the "Value" field.

### Step 3: Create the GitHub Actions Workflow
This file tells GitHub what to do when you push code.

1.  In your local `blog-source` repository, create the folders `.github/workflows/`.
2.  Inside the `workflows` folder, create a new file named `deploy.yml`.
3.  Paste the following content into `deploy.yml`:

```yaml
name: Deploy Hugo Blog to GitHub Pages

on:
  push:
    branches:
      - main # Trigger on pushes to the main branch of blog-source

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout blog source
        uses: actions/checkout@v4
        with:
          submodules: true  # Fetch Hugo themes
          fetch-depth: 0     # Fetch all history for .GitInfo and lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'

      - name: Build Hugo site
        run: hugo --minify --baseURL "https://adnair2024.github.io/blog/"

      - name: Deploy to adnair2024.github.io/blog/
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GH_PAGES_DEPLOY }} # Use the deploy token
          publish_dir: ./public
          destination_dir: blog # Deploy to the 'blog' subdirectory
          external_repository: adnair2024/adnair2024.github.io # Your main portfolio repo
          publish_branch: main # The branch of your main portfolio repo
          cname: adnair2024.github.io # Add your custom domain if you have one, otherwise remove
          force_orphan: false # Do not orphan the branch, just update the subdirectory
```

### Step 4: Commit and Push the Workflow
Save the workflow file and push it to your `blog-source` repository.
```bash
git add .github/workflows/deploy.yml
git commit -m "ci: Add GitHub Actions workflow for deployment"
git push origin main
```

---

## All Done!
Your blog is now live. After the workflow completes (check the "Actions" tab in your `blog-source` repository), you should be able to see your new blog at `https://adnair2024.github.io/blog/`.

From now on, just write new posts, commit, and push to this `blog-source` repository, and your blog will update automatically.

##  Images
-   Vertical long left image:
    -   https://raw.githubusercontent.com/foxihd/hugo-et-hd/master/static/svg/flowlines/28.svg
-   Hero Section Greeter image:
    -   https://raw.githubusercontent.com/foxihd/hugo-et-hd/master/static/svg/flowlines/22.svg

