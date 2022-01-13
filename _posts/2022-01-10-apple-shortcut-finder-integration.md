---
layout: post
title: Apple Shortcut Finder Integration
date: 2022-01-13 12:27 +0000
description: Apple Shortcut which integrates with Finder, for an easier way of adding images to a blog post
categories: 
- Productivity
---

# The Problem
All images that will be included in my blog posts live in the same directory, `Images`, which is a subfolder of my root blog project repo. Once I have an image I want to include in a blog post, I noticed it being a point of friction to copy it to the correct directory and to add the desired image markup to reference it. 

<img
    src="/images/blogging-images-directory.png"
    alt="File structure showing location of images directory within blog project"
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>


## Too Many Manual Steps
The manual steps were the same each time. From the point of having found the image I want to include in Finder:
1. Choose to copy the image
2. Open the `Images` subdirectory
3. Paste the image
4. In the blog post, add an image tag (remembering the correct syntax somehow)
5. Manually type out the full image name
6. _(usually) forget the image name, and have to return to `Finder` to check again_

Ripe for automation! 🍇

# Automating with Apple Shortcuts
The steps automated by this shortcut are:
1. Integrate with Finder to run the shortcut, passing the selected image file as an argument
2. Copy the image to the target `Image` subdirectory, required for the blog
3. Copy the image name to the clipboard so it's ready to paste into the article

## Apple Shortcuts can integrate with Finder
Apple Shortcuts can integrate with `Finder` so that we can right-click on any image, and invoke our Shortcut, passing in that image as input to our script. 

<img
    src="/images/apple-shortcuts-integrate-with-quick-actions.png"
    alt=""
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

We can configure the shortcut to be available as a "Quick Action", and configure the acceptable input types to only allow "Images", as you can see in the screenshot above. For extra laziness, which is always good, you can add a keyboard shortcut.

<img
    src="/images/apple-shortcuts-finder-menu-integration.png"
    alt="Finder integration showing a menu action called 'Add image to blog'"
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

## Copy the selected image to the blog image subdirectory

I dropped back to the command line to make copying the image easier.

<img
    src="/images/apple-shortcuts-copying-file-using-command-line.png"
    alt="apple script step using command line to copy the selected file to the desired blog image directory"
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>


## Copy image name to the clipboard

With the image in the correct place, the last step is to reference the image using its filename. To streamline this, the image name is copied to the clipboard for easy pasting.

<img
    src="/images/apple-shortcuts-copying-filename-clipboard.png"
    alt="apple script steps to get the filename, plus file extension, and copy to clipboard"
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

## Demo

<img
    src="/images/apple-shortcut-full-flow-animation.gif"
    alt="animation showing the full flow. Right click on an image file and choose the `Add image to blog` menu item. The image immediately shows in the correct image subdirectory. And the filename is immediately pasted into the blog post editor."
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

## How you can do this, too
Shortcut available here: [add-image-to-blog-post.shortcut](/downloads/add-image-to-blog-post.shortcut) (on MacOS device). Open downloaded file, and import into `Apple Shortcuts`

