---
title: "Deploy an angular app on Github"
description: "Deploy an angular application on Github pages"
tags: [ "github", "deployment"]
lastmod: 2017-01-17
date: "2017-01-17"
categories:
  - "Development"
slug: "deploy-angular-app-on-github-pages"
image: "/images/github.png"
---

A cheap way to deploy your angular project is to use Github pages. For a long time you needed to have a `gh-pages` 
branch containing your dist code, but recently Github allowed the dist code to be in `docs` which makes our life a little 
easier. Bellow is a little script script to build the production version of the dist file in the docs folder.

```
ng build --target=production --base-href=/repository_name/ --output-path=docs
```

Once you ran this command, just push it on your github repo and you are done!

##### Explanation

- `--target=production` is running compiling your assets for production
- `--output-path=docs` is changing the default output from `/dist` to `/docs`
- `--base-href=/respository_name` is changing your `<base href='/'>` to `<base href='/repository_name/'`, this is necessary 
otherwise the web server will be looking for assets under your root github domain and our site is published on the repository subdomain.