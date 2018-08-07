---
layout: post
title: "Setting up Octopress"
date: 2016-08-19 15:25
comments: true
categories: 
 - linux
 - open-source
 - blogging
 - debian/ubuntu
---

Octopress, based on Jekyll, is a great way to run a simple, static blog with posts written in markdown. I use octopress to run this blog because it is light weight and easy to use. Setting it up, however, took me a number of google searches and I had to use multiple tutorials to get it working. This tutorial is for a debian based system but should work on most linux distros with a few minor tweaks.

<!-- more -->
## Requirements
For this we will need ruby and RubyGems
``` bash
sudo apt install ruby rubygems
```

## Installation
To start let's create a directory for our blogging activities and download octopress from github:

``` bash
mkdir ~/blogging
cd blogging
git clone git://github.com/imathis/octopress.git octopress
cd octopress
```
Now we install octopress and its default theme

``` bash
gem install bundler
rbenv rehash # Only if you use rbenv as your ruby installation
bundle install
rake install
```

## Configuration
The basic config for your blog is in `_config.yml` which allows you to set simple options for you blog. The most simple (and important options are right at the top of the file:

```YAML
url: https://blog.mysite.com
title: Mysite's Blog
subtitle: Nothing to see here.
author: Freddie Poser
simple_search: https://www.google.com/search
description:
```
You *have* to change the `url` option for links to work and you will obviously want to change the title and other attributes. I suggest having a look through the file to find things to change.

## Your first post
So you have your blog set up we can write an acutal blog post! Octopress(/Jekyll) makes this really easy with one command:

```
# rake new_post["<title>"]
cd source/_post
rake new_post["Test Post"]
```

This will create a file in the format `YYYY-MM-DD-<title>.markdown` so today it would create the file: `2016-08-18-test-post.markdown`  which will become the post. Now edit the file with your favourite text editor. The file should look something like this:

```yaml
---
layout: post
title: "Test Post"
date: 2011-07-03 5:59
comments: true
external-url:
categories:
---
```

You can change any of these settings if you want but most likely you will only want to set the title and the categories. You can set categories in multiple ways but this is my favourite:

```yaml
categories:
 - cat1
 - cat2
---
```
This will set the categories for this articles to `cat1` and `cat2`. You can now actually write the blog post in the same markdown used on github.

## Generating the Blog
Now we can actually generate the blog with the simple command below:

```
rake generate
```

This will create a whole blog in the `octopress/public` directory. This will be static HTML and self contained. To serve this up I use Apache by symlinking it to my `/var/www/html/blog` directory and then using a VirtualHost with a subdomain pointing to it. Now whenever I write a post I `rake generate` and its live!

Thanks for reading,

Freddie
