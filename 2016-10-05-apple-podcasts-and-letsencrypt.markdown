---
layout: post
title: "Apple, podcasts and letsencrypt"
date: 2016-10-05 22:13:44 +0200
comments: true
categories:
 - apple
 - rss
 - web
 - webhosting
 - letsencrypt
---

Recently a friend wanted me to setup a simple podcast on iTunes for him. When he asked me I though "How hard can this be?". After all, it is just an RSS feed. I looked up a sample, wrote the basic outline and validated it on [http://castfeedvalidator.com/](http://castfeedvalidator.com/). The document looked something like this:

<!-- more -->

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<rss xmlns:itunes="http://www.itunes.com/dtds/podcast-1.0.dtd" xmlns:atom="http://www.w3.org/2005/Atom" version="2.0">
<channel>
    <title>Name</title>
    <description>Why apple podcasts are terrible, awful, no good very bad things.</description>

    <link>https://foo.com/feed</link>
    <ttl>1</ttl>

    <atom:link href="https://foo.com/" rel="self" type="application/rss+xml" />

    <itunes:author>Friend</itunes:author>
    <copyright>© 2016 Friend Inc.</copyright>

    <language>en-us</language>
    <itunes:category text="Education"/>
    <itunes:explicit>No</itunes:explicit>
    <itunes:image href="http://foo.com/img.png"/>

</channel>
</rss>
```

This passed every validator so I put it into itunes' own validator and I get the almost totally unhelpful error `Can’t read your feed.` Nothing more. Nothing to tell me if it was a 504, 404, parsing error, nothing. I spent ages mucking about with the file location, file name, little tweaks to the document. I was getting very frustrated at this point so I gave it a "Hail-Mary" google and went digging. It turns out that on about page 3 of a tangentally related google search someone mentions that Java doesn't support lets encrypt ssl certs.

My whole site forces SSL using these amazing things, so I was loathe to remove it on this subdomain, but I had to give it a go. So I disabled force ssl on static.vogonjeltz.com and thank god it works.

My message to apple: Think about your users. Give us real error messages so things like this take 2 mins not 2 hours.
