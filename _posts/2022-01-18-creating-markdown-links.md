---
layout: post
title: Creating Markdown Links
date: 2022-01-18 23:11 +0000
description: How to create links in markdown, both those which display just the URL and those which display custom text
---
# Creating Markdown Links
_There are two types of links you can create in Markdown:_
- _those which display just the URL. e.g., <https://craigrussell.io>_
- _those which display custom text. e.g., [My Blog](https://craigrussell.io)_

_This post details how to use both types._

## Creating a Markdown Link with Custom Text
The standard syntax for creating links in markdown is providing text to display in square brackets `[ ]`, followed by the URL in parentheses `( )`. For example, to link to this blog, I could do this:

`[My Blog](https://craigrussell.io)`, which renders as [My Blog](https://craigrussell.io)

### 💡 Tips
- It is important there are no spaces between `[ ]` and `( )`
- It is important the `[ ]` appears before the `( )`

## Creating a Markdown Link which just shows the URL
Instead of adding display text for the link you can have it displayed as just the URL, and there are 2 ways to do this.

### Option 1: repeating URL as the title ❌
One way to create a link with no title is to duplicate the URL into both parts of the link: e.g., 

`[https://craigrussell.io](https://craigrussell.io)`, which renders as [https://craigrussell.io](https://craigrussell.io).

<ul style="list-style-type:none;padding-left:20px">
    <li> ➕ You probably already know the syntax</li>
    <li> ➖ It's annoying to provide the same URL twice</li>
    <li> ➖ It is error-prone if you update one part and forget to update the other</li>
</ul>


### Option 2: using angle brackets ✅
The best way to create a link which just shows the URL and no custom display text is to wrap it inside angle brackets `< >`.

`<https://craigrussell.io>`, which renders as <https://craigrussell.io>

<ul style="list-style-type:none;padding-left:20px">
    <li> ➕ Less typing required </li>
    <li> ➕ No chance for display text and URL to mismatch </li>
    <li> ➖ Need to remember the alternative syntax style </li>
</ul>
