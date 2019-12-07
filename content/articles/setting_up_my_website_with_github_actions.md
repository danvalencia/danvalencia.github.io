---
title: "Setting up my website with Github Actions"
date: 2019-12-06T13:55:20-08:00
draft: true
comments: false
images:
---

I've had this website for 6 years now. Like most things in life, a website needs constant care and things break and rot if you disdain them. It's not that uncommon to lose interest in side projects after a while, it's happened to me time and time again.


### My setup 

I built this website with Hugo, and I've been very happy with the tool, it's easy to setup, it's fast and uses Markdown which is my favorite approach to writing. 

For some reason I genuinely can't recall I initially set my website in 2 repositories: one where I kept all the Hugo structure and my articles in Markdown and a separate repository for the published site, with the built HTML pages. It worked well for a while, but this approach required too much manual work, as I had to build the site on one hand and basically copy over the generated HTML to the published repo. 

Over time I stopped writing, and eventually I even forgot how the flow from writing the text to publishing the site worked altogether. 

Enough time happened that I just left my site rot.

Recently, I've felt the need to write more, but didn't have an outlet to do so. I decided to revamp my site, and implement it in a way that writing and publishing a post is just as easy as pushing a Markdown file to Github. Having a frictionless blogging setup is key to not lose interest in the important part, the writing. 

### Github Pages

Github Pages is a feature that allows all repositories to host a static HTML site directly within the Github platform. 

In the context of Github Pages there are 2 kinds of repos:
- Personal or Organization repositories: they have a name of _\<user-or-org\>_.github.io. GH Pages sites built from a personal or org erpo are published from the master branch by default.
- Project repositories: they can have any name. GH Pages sites built from a project repo are published by default from the _gh-pages_ branch. 

Given that mine is personal site I created the _danvalencia.github.io_ repo. 

The _master_ branch needs to have the published site with the pre-rendered HTML, and this time I'm automating the build and publishing process. 
I created an _author_ branch to host the content. I setup this branch as the default branch, via the Github settings. 

Ok, so by now, I had my repo with 2 branches: _author_ and _master_. 

### How the build process works

Building the site locally is as easy as running:

```
> hugo
```

Assumming `hugo` is already installed in your system, of course, which from a Mac is as simple as running `brew install hugo`.

I'd heard about Github Actions when it first was released a few months ago, and thought it was a super nice feature which could be used to run your CI pipeline. 

Within Github Actions you can create Workflows which are a series of steps that perform several tasks. You can choose to build your own actions or you can choose from existing actions that are published in the Github marketplace. You can pretty much do anything you want with this framework.

Github Actions end up being a combination of Docker image and scripts that will execute within the Docker image. The cool thing about this approach is that because it's built using Docker they can contain virtually any base software, and thus you can automate build for any programming stack. I'm not sure about the constraints of storage and memory, though.

### Workflow Configuration 

Like most configurations nowadays, Github Actions workflows are defined with YAML files added to your repository's _.github/workflows_ directory.

A workflow gets triggered by an event, such as a push or pull request creation. In the following example the workflow get's triggered on a _push_ event from the _author_ branch:

```
on:
  push:
    branches:
    - author
```

Then you specify jobs that need to run for this workflow. For each job you specify the docker image to use, like so:
```
jobs:
  build-deploy:
    runs-on: ubuntu-18.04
```

 Now you specify a list of steps to run, which can be external actions or scripts:

```
    steps:
    - uses: actions/checkout@v1
      with:
        submodules: true

    - name: Setup Hugo
      uses: peaceiris/actions-hugo@v2
      with:
        hugo-version: '0.59.1'
        # extended: true

    - name: Build
      run: hugo --minify

    - name: Deploy
      uses: peaceiris/actions-gh-pages@v2.5.0
      env:
        ACTIONS_DEPLOY_KEY: ${{ secrets.ACTIONS_DEPLOY_KEY }}
        PUBLISH_BRANCH: master
        PUBLISH_DIR: ./public
```

In order to build automatically 