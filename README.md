# All-In-One Guide to Using Eleventy and GitHub Codespaces to Create GitHub Pages
Modified from Eleventy website to GitHub Pages with GitHub Actions by Andres Lopez

## Intro to GitHub and repositories
1. Create GitHub account
1. Open your browser, log in to GitHub and navigate to github.new 
1. Name repository `eleventy-github`
1. Add a README file
1. Click green "Create Repository" button
1. Click the green "<>Code" dropdown and select the right "Codespaces" tab and the green "Create codespace on main" button
1. Your codespace will open in a new browser tab with Explorer to the left, Preview to the right, and Terminal at the bottom. You can toggle this view as needed with the icons in the top right
1. In your Terminal, copy and paste `npm init -y` and hit enter. This will create a `package.json` file
1. Next copy and paste `npm install @11ty/eleventy@latest sass@latest --save-exact` and hit enter. This will install Eleventy, create a `node_modules` folder, and a `package-lock.json` file. (Note: Explain what the three views are doing and the files in the explorer so far)
1. In the Explorer, select the new `package.json` file.
1. Highlight line 7, the "test" script and replace with the following (check indenting)

        `"watch:sass": "sass src/static/scss:public/static/css --watch",
        "build:sass": "sass src/static/scss:public/static/css",
        "watch:eleventy": "eleventy --serve",
        "build:eleventy": "ELEVENTY_ENV=development eleventy",
        "start": "npm run watch:eleventy & npm run watch:sass",
        "build": "npm run build:eleventy & npm run build:sass"`

1. Create a new `src` folder by clicking on the folder+ button in Explorer
1. Inside your new `src` folder create two subfolders: `_includes` and `static`
1. Inside your new `static` folder, create a new `scss` folder
1. Next create a new file `.eleventy.js` in the root directory with the following code:

`module.exports = function (eleventyConfig) {
  eleventyConfig.setBrowserSyncConfig({
    files: './public/static/**/*.css',
  });

  return {
    dir: {
      input: 'src',
      output: 'public',
    },
  };
};`  
1. Create an empty file named `main.scss` in the `static/scss` folder
1. Create a template file named `base.njk` in the `_includes` folder
1. Paste the following code in `base.njk`

`<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>{{ title }}</title>
  </head>
  <body>
    {% block content %} {{ content | safe }} {% endblock %}
  </body>
</html>`
    
1. Create an `index.md` file in your `src` folder
1. Paste the following code in your new `index.md` file      
`---
layout: 'base.njk'
permalink: /
title: 'Our Eleventy page'
---`

`# Hello Price Lab`
1. Permanently delete `README.md`
1. Open the Terminal and paste `npm run start`
1. Hit Enter and it will run the start script
1. Open the http://localhost:8080/ in a new browser tab
1. Go to line 6 in the <head> of `base.njk`, press enter, and paste the line of code

`<link rel="stylesheet" type="text/css" href="{{ 'static/css/main.css' | url }}" media="screen" />`
     
1. Add this code to `main.scss`
    `h1 {
    color: green;
    background-color: black;
    text-align: center;
    padding: 50px;
  }`
1. Refresh Local host page and see CSS changes

## GitHub Pages
1. Create new `.gitignore` file in root directory and paste the following code:

`# Dependencies
`/node_modules

`# Misc
`/public

`# Intellij
`/.idea

`# Sass
`.sass-cache/
`*.css.map
`*.sass.map
`*.scss.map

`# Visual Studio Code
`/.vscode
`.history`
1. Then go to GitHub repository Settings>Pages in left sidebar
1. Click the Branch none dropdown menu and switch to "main" and then hit "Save"
1. By default, GitHub pages utilizes Jekyll to generate your website. However, in our case, we’re using Eleventy, so we need to notify GitHub Pages about this. To achieve this, make an empty file named `.nojekyll` and place it in the root directory.
1. Then, create a `.github` folder in the root directory and within that folder, create another folder named `workflows`.
1. Create a `build.yml` file in workflows and paste this code
`name: 
      Build PR

on:
  pull_request:
    branches: ['main']

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: ['20']

    steps:
      - uses: actions/checkout@v4

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name: Install packages
        run: npm ci

      - name: Run npm build
        run: npm run build`
1. Create a `build-and-deploy.yml` file in workflows and paste this code
`name: 
      Build and Deploy

on:
  push:
    branches: ['main']

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: ['20']

    permissions:
      contents: write

    steps:
      - uses: actions/checkout@v4

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name: Install packages
        run: npm ci

      - name: Run npm build
        run: npm run build:prod

      - name: Deploy to gh-pages
        uses: peaceiris/actions-gh-pages@v4
        with:
          deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}`  
1. Control+C to stop current terminal command.
1. Generate your deploy key with the following command in the Terminal
 `ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""`
You will get 2 files: `gh-pages.pub` is a public key and `gh-pages` is a private key
1. Now go to Repository Settings>Deploy Keys 
1. Select the green button "add deploy key"
1. Give it the title “Public key of ACTIONS_DEPLOY_KEY”, copy and paste your public key (gh-pages.pub) with the Allow write access checked 
1. Then go to Secrets and Variables>Actions> and select the green button "New repository secret"
1. Name your private key ACTIONS_DEPLOY_KEY and copy and paste your private key (gh-pages) and hit the green "Add secret" button
1. Then delete `gh-pages` and `gh-pages.pub` from your Explorer
1. In your Terminal, run clear to clear the public key image
1. Add more scripts to `package.json` file (be sure to add a comma after every script and check indentation)
 `"build:sass:prod": "sass src/static/scss:public/static/css --style compressed",
      "build:eleventy:prod": "ELEVENTY_ENV=production eleventy",
      "build:prod": "npm run build:eleventy:prod & npm run build:sass:prod"`
1. Then paste the following commands to Terminal (This is a shortcut to add the current directory, then commit with a message, then push)
`git add .
git commit -m "build and deploy"
git push`
1. Go to your GitHub repository and click on the Actions tab at the top. You’ll usually find the Build & Deploy workflow running, followed by the pages-build-deployment workflow which deploys the website to GitHub Pages once the first one is completed
1. Once the Build & Deploy workflow runs, a new branch named "gh-pages" is automatically generated. This branch will contain the output of your Eleventy site. To ensure the website can be hosted correctly, we must notify GitHub Pages. 
1. Go to the Settings tab, choose Pages from the menu on the left, switch the branch for your GitHub Pages site to "gh-pages", and click save.
1. Once the pages-build-deployment workflow is completed, it will automatically be triggered again. After it’s done, you can check out the final result by visiting the URL where your website is published.
1. Make changes in Codespaces to `index.md` and then redo git add/commit/push
