---
layout: post
title: "Adding Author tag to RSS Feed using Hugo static site generator"
date: 2019-03-14 01:34:15Z
description: This post details how to include the author tag in the generated RSS feed for a Hugo-generated site.
keywords:
  - hugo
  - rss feed
  - add author
  - add author to rss feed
  - include author

list_image: /images/rss-icon.png
images:
- /images/rss-icon.png

permalink: 2019/03/adding-author-tag-to-rss-feed-using-hugo-static-site-generator/
---

*This post details how to include the author tag in the generated RSS feed for a Hugo-generated site.*

## Default RSS Template
Hugo comes equipped with a template by default. In fact you might be pleasantly surprised to discover your site has an RSS feed already. You can try it out by visiting your site and adding `/index.xml` on the end of either of your home page, or your section containing your blog posts. e.g.,
  
  - [https://craigrussell.io/index.xml](https://craigrussell.io/index.xml)
  - [https://craigrussell.io/posts/index.xml](https://craigrussell.io/posts/index.xml)

## Customising The Template
While the default template might be enough for some, others are going to want to customise it. For me, I found that I wanted to include an `<author>` tag in my posts.

You can either grab the default template and add the `<author>` tag into it (recommended approach) or just skip ahead and grab my one from the gist below. I would recommend using the default one as it may contain newer changes from Hugo that my gist won't have over time.

To do this, we must start by grabbing a copy of the embedded RSS template. We'll drop that into your project and customise it to include the author tag.

1. Get the [default Hugo RSS template](https://gohugo.io/templates/rss/#the-embedded-rss-xml)
2. Save the template into your Hugo directory as `layouts/_default/rss.xml`
3. Add the `<author>` tag in. You'll probably want to add it under the `<item>` tag, and you may additionally want to add it at the top level.
4. Ensure you have your name detailed in the site config (i.e.,  `config.yaml`, `config.toml` or `config.json` file)

When you're done, it should look like this 👇
{% gist 55a019f576eb10802543e5d1dcc65743 %}


```
// config.yaml

author: 
  name: Craig Russell
```

## Final RSS
Now when you publish the site and access the RSS XML, you'll see the `<author>` is now included.

![RSS XML showing the included author tag](/images/author-tag-in-rss-xml.png)