---
layout: post
title: Lightweight Journal Built on Apple Shortcuts
date: 2022-01-07
description: Building a journaling tool using Apple Shortcuts and Notes
---
_This post details how to use **Apple Shortcuts** to build a lightweight journal to log events, which are timestamped and saved to **Apple Notes**._

# Apple Shortcuts
[Apple Shortcuts](https://support.apple.com/en-gb/guide/shortcuts/welcome/ios) provide a way to automate and extend the functionality of apps on Apple devices. It's like programming, except you don't need a compiler, an app store or to deploy a website. It's powerful, incredibly useful and a great tool to have in your toolbox. 🛠️🧰 

We can leverage an Apple Shortcut script which prompts the user for some details when it runs, and saves those details in a timestamped note in Apple Notes. 

# Lightweight Journal
I wanted to have a way to quickly log when events happen, that would automatically be grouped by date.

<img
src="/images/journalling-ios-shortcuts.gif"
alt="animation showing logging two life events for the current day"
height="500"
style="
display: block;
margin-left: auto;
margin-right: auto;
"
/>

## What do I journal?
I use this same system for a few things:

- Logging important life events. e.g., births, weddings, buying a car, vaccinations etc.. 
- Logging hilariously cute things my young kids say


## Creating Events Quickly
I've streamlined the process as much as I can for creating an event. It takes only two inputs and can be completed within a few seconds:
- text input to describe what happened
- the date the event happens (defaults to today)

## How Events are Saved
Events are saved in **Apple Notes**, in a folder called **Journal**. As they are saved in Apple Notes, it makes them accessible and searchable across all of my devices.

I wanted to group events by date, so a new note is created for each day which had an event recorded. If there are multiple events for a day they will be appended to the same file.


<img
src="/images/apple-notes-journal.png"
alt="Apples Notes list view, showing journal entries where each entry is in ISO-8601 date format"
width="300"
style="
display: block;
margin-left: auto;
margin-right: auto;
"
/>

<img
src="/images/journal-entry.png"
alt="Apples Notes note view for 6th January, showing multiple events separated by a dotted line break"
width="300"
style="
display: block;
margin-left: auto;
margin-right: auto;
"
/>

## Apple Shortcuts

<img
src="/images/apple-shortcuts.png"
alt="screenshot of Apple Shortcut for logging a life event"
height="500"
style="
display: block;
margin-left: auto;
margin-right: auto;
"
/>

The shortcut itself is a set of script events which ask the user for text describing what happened, then ask the user for the date the event happened. With these two inputs, it will then check if a note already exists for that day.
- If it already exists, the event details will be appended to the existing note
- If it doesn't exist, it will be created with the event details

For a link to the full shortcut, see "How You Can Use This" below


# How you can use this, too
1. Open **Apple Notes**, and create a folder for your journal. Let's assume you call it "Journal" 
2. Download [Journal.shortcut](/downloads/Journal.shortcut) (on an iOS or MacOS device). Open downloaded file, and import into `Apple Shortcuts` 
3. Launch `Apple Shortcuts`, and open the new `Journal` shortcut. You need to choose the Folder for your journal **in 2 places**. (You need to do this even if you are already using the name "Journal")

## If importing on MacOS
<img
src="/images/apple-shortcut-import-instructions-macos.png"
alt="Annotated MacOS screenshot of editing the new Apple Shortcut to choose destination folder in two places"
height="500"
style="
display: block;
margin-left: auto;
margin-right: auto;
"
/>

## If importing on iOS
<img
src="/images/apple-shortcut-import-instructions-ios.jpeg"
alt="Annotated iOS screenshot of editing the new Apple Shortcut to choose destination folder in two places"
height="500"
style="
display: block;
margin-left: auto;
margin-right: auto;
"
/>


## Lightweight journaling built on Apple Shortcuts and Apple Notes
If you've done it all correctly, whenever you run that shortcut you'll be prompted for the event details, followed by the date. The created note will then be shown which is handy for adding further details, pasting in images etc... 

💡 You can add the shortcut to your Home Screen for one-click triggering.  

Go forth and journal ✍️



