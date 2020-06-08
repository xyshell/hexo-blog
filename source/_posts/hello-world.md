---
title: Hexo & Theme Usage
tags: Hexo
---

[Hexo](https://hexo.io/), [documentation](https://hexo.io/docs/), [troubleshooting](https://hexo.io/docs/troubleshooting.html), [GitHub Issues](https://github.com/hexojs/hexo/issues).

# Hexo

## Install & Setup

```bash
$ npm install -g hexo-cli
```

## Initialize

``` bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```

File Structure:

``` python
|-- _config.yml # configuration file for hexo
|-- source
    |-- _posts
        |-- one_blog.md # blog file
    |-- _drafts
        |-- one_draft.md # draft file
|-- themes
    |-- landscape # default theme
    |-- ocean 
        |-- _config.yml # configuration file for ocean
```

## Create a new post

``` bash
$ hexo new "My New Post" / $ hexo n "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

## Run server

``` bash
$ hexo server / $ hexo s
```

More info: [Server](https://hexo.io/docs/server.html)

## Generate static files

``` bash
$ hexo generate / $ hexo g
```

More info: [Generating](https://hexo.io/docs/generating.html)

## Deploy to remote sites

``` bash
$ hexo deploy / $ hexo d
```

More info: [Deployment](https://hexo.io/docs/deployment.html)

## Deploy to github

``` bash
$ yarn add hexo-deployer-git
```

Then, go to hexo's _config.yml, add this to deploy session:

``` yml
deploy:
  type: git
  repo: https://github.com/xyshell/xyshell.github.io.git
  branch: master
```

# Theme

## Install & Setup & Update

``` bash
$ git clone https://github.com/zhwangart/hexo-theme-ocean.git themes/ocean
```

Modify theme setting in hexo's _config.yml to ocean:

``` yml
theme: ocean
```

Further update:

``` bash
cd themes/ocean
git pull
```

# Theme Reference

## Ocean

https://github.com/zhwangart/hexo-theme-ocean

## Minos

https://github.com/ppoffice/hexo-theme-minos