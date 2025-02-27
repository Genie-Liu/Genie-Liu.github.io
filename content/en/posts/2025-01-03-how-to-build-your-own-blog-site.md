---
date: '2025-01-03T13:41:47+08:00'
draft: false
title: 'Build Your Own Blog Site using Hugo & PaperMod'
---

## Install Hugo

Make sure you have met the requirement:

**Install hugo**

```sh
brew install hugo
```

Some useful commands

```sh

# create new site resource
hugo new site <your site name>

# deploy the site locally, help check before deploy to github
hugo server

# Generate a new blog, a simple suggestion is that add the date before your file name,
# so that you can easily sort your posts in the directory.
# Remember to set the draft field to true when you want to publish it.
hugo new content content/posts/<datetime>-my-first-post.md

```

## PaperMod Theme

[Github Project](https://github.com/adityatelange/hugo-PaperMod)

**Change CSS style**,  take blockquote as example:

- Create css assets: /static/css/styles.css
  ```css
  blockquote {
    color: gray !important;  /* Overrides any existing color */
    border-left: 4px solid #ccc;  /* Optional: Adds a subtle left border */
    padding-left: 10px;  /* Optional: Adds some padding to the left */
  }
  ```
- Add path to config.yml
  ```yml
  params:
    customCSS:
      - "/css/styles.css"
  ```
- Copy and modify layout: copy `themes/PaperMod/layouts/_default/baseof.html` to your layouts direcotry
  ```html
  <head>
    <!-- Other head content like meta tags, title, etc. -->

    {{ with .Site.Params.customCSS }}
        {{ range . }}
            <link rel="stylesheet" href="{{ . | relURL }}">
        {{ end }}
    {{ end }}

  </head>
  ```

## Deploy to Github

[Ref](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

1. Create personal page repository. `<github-name>/<github-name>.github.io`

2. Link the site direcotry to github repository.

    `git remote add origin git@github.com:<github-name>/<github-name>.github.io.git`

3. Add github action to automatically deploy once the repository is updated.

   - Create action file: `./github/workflows/hugo.yaml`
   - Copy the [content](https://github.com/Genie-Liu/Genie-Liu.github.io/blob/main/.github/workflows/hugo.yaml) to the file

Now you can enjoy testing your webpage: `https://\<github-name\>.github.io/`

## Reference

- [quick-start of hugo](https://gohugo.io/getting-started/quick-start/)
- [Example site of PaperMod theme](https://adityatelange.github.io/hugo-PaperMod/)
- [hugo-PaperMod/wiki](https://github.com/adityatelange/hugo-PaperMod/wiki)
- [Deployment on github](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
- [ramsayleung's blog code](https://github.com/ramsayleung/ramsayleung.github.io)
