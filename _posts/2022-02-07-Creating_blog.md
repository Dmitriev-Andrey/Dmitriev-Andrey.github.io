---
layout: post
title: Creating blog
excerpt: Create a blog on Github Pages. Simple guide.
---

## Why is Github?

If you want create blog then Github Pages is a good choice:
1. It's free
2. It's fast
3. It's easy

One of the main problems when creating a blog is a long choice of hosting and engine.
Of course, creating your own website is a so cool, but it's wrong way. You will work a lot of time and when you've done (maybe or maybe not), you will hate blogging, programming and sometimes your live. I felt it, I know it. You need to make it fast, while you don't lost your motivation.

Github gives you this. Also you get some benefits:
1. Version control
2. Markdown syntax
3. And good indexing

## How create blog on Github Pages

You need to make some simple steps:
1. Sign up on Github.
2. Create your first repo with name github-nickname.github.io. For instance, dmitriev-andrey.github.io.
3. Find interesting template [here](https://github.com/topics/jekyll-template).
4. Copy template in local repository.
Honestly, you can start write article and push it to Github. But I advise testing locally.

1. Install [ruby](https://www.ruby-lang.org) if necessary.
2. Use `jekyll serve` for start local server.
3. Very often you will get some errors, because you need to install some ruby packages (gems). Install them - use `gem install "package"`.
4. When all will be all right, you see something like that:
```sh
Configuration file: /home/marbok/projects/home-projects/Dmitriev-Andrey.github.io/_config.yml
            Source: /home/marbok/projects/home-projects/Dmitriev-Andrey.github.io
       Destination: /home/marbok/projects/home-projects/Dmitriev-Andrey.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
       Jekyll Feed: Generating feed for posts
                    done in 0.234 seconds.
 Auto-regeneration: enabled for '/home/marbok/projects/home-projects/Dmitriev-Andrey.github.io'
    Server address: http://127.0.0.1:4000
  Server running... press ctrl-c to stop.
```
5. Go to http://127.0.0.1:4000 in your browser.
6. If template work, push it in Github.
7. Profit!!!

Github will build your blog and you can see it on https://githubname.github.io/.
But your journey is just beginning!!! Customize template, create post and find interesting idea!!!