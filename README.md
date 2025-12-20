# My blog

![Hugo](https://img.shields.io/badge/Hugo-000000?style=for-the-badge&logo=hugo&logoColor=FF4088)
![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-222222?style=for-the-badge&logo=github&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)
![Brewm Theme](https://img.shields.io/badge/Theme-Brewm-informational?style=for-the-badge)

## Introduction

This is a blog created using Hugo, a fast and flexible static site generator, and the modern, responsive "Brewm" theme. The blog is designed to be easily deployable via GitHub Pages with automated workflows using GitHub Actions.

## Features

*   **Fast & Modern:** Built with Hugo for speed and a contemporary look and feel.
*   **Responsive Design:** Adapts seamlessly to various screen sizes, from mobile to desktop.
*   **Dark Mode Support:** Automatically switches between light and dark themes based on system preferences.
*   **Automated Deployment:** Leverages GitHub Actions for continuous deployment to GitHub Pages.
*   **Customizable:** Easily extendable for content and certain styles.

## Getting Started

### Prerequisites

Before you begin, ensure you have Hugo installed on your local machine. Follow the official Hugo installation guide for your operating system: [https://gohugo.io/getting-started/installing/](https://gohugo.io/getting-started/installing/)

### Local Development

To run the blog locally and see your changes in real-time:

1.  **Clone the Repository:**
    ```bash
    git clone https://github.com/adnair2024/blog-source.git # Replace with your repository URL
    cd blog-source
    ```
2.  **Initialize Theme Submodule:**
    ```bash
    git submodule update --init --recursive
    ```
3.  **Run Hugo Server:**
    ```bash
    hugo server -D --baseURL "http://localhost:1313/"
    ```
    Open your browser to `http://localhost:1313/`. The `-D` flag ensures that draft content is also rendered. The `--baseURL` flag ensures correct asset loading during local development.

### Creating Content

#### New Blog Post

To create a new blog post:

```bash
hugo new post/my-new-post.md
```

This will create a new markdown file in `content/post/my-new-post.md`. Open this file, add your content, and make sure to change `draft: true` to `draft: false` in the front matter to publish it.

Example `content/post/my-new-post.md` front matter:

```yaml
---
date: '2025-12-20T11:56:44-06:00'
draft: false
title: 'My New Post'
type: 'post'
tags: ['example', 'blogging']
categories: ['Tutorials']
---
```

#### New Slides

To create a new slide for the homepage carousel:

```bash
hugo new slide.md
```

This will create a new markdown file in `content/slide.md`. Rename it to something unique like `content/slide-welcome.md` if you plan to have multiple. Open this file, add your content, and ensure the `type` is `slide` and `weight` is unique for ordering.

Example `content/slide-welcome.md` front matter:

```yaml
---
type: 'slide'
title: 'Welcome to My Blog!'
cover: 'https://raw.githubusercontent.com/foxihd/hugo-et-hd/master/static/svg/flowlines/your-image.svg'
weight: 1
params:
    headless: true
---
```

### Deployment

This blog is set up for automated deployment to GitHub Pages using GitHub Actions.

1.  **Workflow File:** The deployment is managed by the `.github/workflows/deploy.yml` file.
2.  **Personal Access Token (PAT):** A GitHub Personal Access Token with `repo` scope is required to push changes to the deployment repository (`adnair2024.github.io`). This PAT must be stored as a repository secret named `GH_PAGES_DEPLOY` in your `blog-source` repository.
3.  **Automated Process:** Every time you push changes to the `main` branch of this `blog-source` repository, the GitHub Action will automatically:
    *   Checkout your code.
    *   Build the Hugo site.
    *   Deploy the generated static files to the `blog` subdirectory of your `adnair2024.github.io` repository.

Your live blog will be accessible at `https://adnair2024.github.io/blog/`.

## Customizations and Notes

*   **Theme:** This blog uses the [Brewm Theme](https://github.com/foxihd/hugo-brewm).
*   **Vertical Text Fix:** The original theme displayed some section titles vertically. This was resolved by overriding the theme's default `list.html` partial (`layouts/_default/list.html`) and removing the `section-title` class from the relevant header elements.
*   **Dark Mode:** The theme inherently supports dark mode based on system preferences. Custom CSS overrides were found to interfere with this functionality and were removed.
*   **Images:** Images in the hero section and slides are configured via the `cover` parameter in the respective content files (`_index.md`, `slide.md`).
*   **Scrollbars:** Attempts to customize scrollbars with custom CSS led to conflicts with the theme's dark mode. Currently, scrollbars revert to browser defaults to maintain dark mode functionality.
*   **Safari Compatibility:** Some minor display issues were noted on Safari but tabled for future investigation to maintain progress.

## Future Enhancements

*   Investigate and resolve Safari rendering issues.
*   Find a method to customize scrollbars without breaking dark mode.
*   Add more detailed styling and custom components.
*   Expand content with more posts and slides.
