---
title: Build your own blog with Hexo.js
date: 2020-07-21 09:00:00
---

Since lots of my classmates/friends have their own blogs, I decided to build one too with [Hexo.js](https://github.com/hexojs/hexo).

# Install Hexo.js

1. Install [Node.js](https://nodejs.org/).

2. Install [Git](https://git-scm.com/).

3. Install [Hexo.js](https://hexo.io/) using npm:
```bash
$ npm install hexo-cli -g
```
 Hexo is a blog framework. It generates a [static website](https://en.wikipedia.org/wiki/Static_web_page) from your Markdown files. It's easy to use, extremely extendable.

# Initialize your blog

1. Run `hexo init` at where you want to create your blog. It creates a project structure like this:
```
.
├── _config.yml
├── package.json
├── node_modules
├── scaffolds
│   ├── draft.md
│   ├── page.md
│   └── post.md
├── source
│   └── _posts
│       └── hello-world.md
├── themes
│   └── landscape
└── yarn.lock
```
 - `_config.yml` is a configuration file for your blog. You can edit it as you like. (https://hexo.io/docs/configuration)
 - `package.json` holds various metadata relevant to the project. (Hexo's dependencies)
 - `node_modules` is Hexo's dependencies.
 - `scaffolds` when you create a new post, Hexo bases the new file on the scaffold.
 - `source` holds your posts, etc.
 - `themes` contains your themes for your site.
 More info @ https://hexo.io/docs/setup.

2. Initialize a git repository as you like. The `.gitignore` file is already created.

3. To see what you have created, run `hexo server`. This will start a server at localhost (port 4000 by default).

4. Use `hexo new post` to create a new post. (e.g. `hexo new post "Hello world"`) `hexo new page` to create a new page. (e.g. `hexo new page "about"`)

# Themes

Open https://hexo.io/themes/. Most themes offer a demo website and an installation guide. Following them is ok most of the time. (I use [even](https://github.com/ahonn/hexo-theme-even).)

# Optional suggested plugins

1. RSS feed is provided by [hexo-generator-feed](https://github.com/hexojs/hexo-generator-feed).
2. Sitemap is provided by [hexo-generator-sitemap](https://github.com/hexojs/hexo-generator-sitemap).

Following their instructions is ok.

# Deploying

To make others see your blog, we have to deploy it somewhere. Here I use [GitHub Pages](https://pages.github.com/) as an example.

First install [hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git).

## Manually

1. Modify the `deploy` section at the vey bottom of your `_config.yml` file like this:
```yml
deploy:
  type: git
  repo: https://github.com/<username>/<project>
  branch: <branch>
```

2. Whenever you want to deploy to GitHub, run `hexo generate && hexo deploy`. This generates the static files and deploy them to GitHub Pages. For personal homepage (`*.github.io`), branch should be `master`.

## CD supported by [Travis-ci](https://travis-ci.org/)

It's a bit tricky to use CD with Travis-ci since there is a bug in hexo-deployer-git.

1. Generate a token for Travis-ci to access to your repo to update it. Follow this: https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token#creating-a-token

2. Modify the `deploy` section like this, note that we use an environment variable to store the token, but not hardcoded into the code:
```yml
deploy:
  type: git
  repo:
    github:
      url: https://github.com/<username>/<username>.github.io
      branch: <branch>
      token: $GH_TOKEN
  name: <github-username>
```
Due to a [bug](https://github.com/hexojs/hexo-deployer-git/issues/159), we cannot use a similar structure in the `Manually` section.

3. Create a `.travis.yml` at the root of your blog's source code. It goes like this:
```yml
language: node_js

node_js: stable

install:
  - npm i # Install Hexo and it's dependencies.

script:
  - hexo generate
  - hexo deploy # As what you do manually.
```
The script tells Travis-ci how to build and deploy your site.

4. Go to [Travis-ci](https://travis-ci.org/). Login with your GitHub account.

5. Active your blog's repository by clicking the small + at the left. Build starts every time you push to the GitHub repo.

6. Go to your repository's settings. add an environment variable named `GH_TOKEN` and paste your GitHub token generated at Step #1 as the value.

7. Now push. Your blog should be deployed.
