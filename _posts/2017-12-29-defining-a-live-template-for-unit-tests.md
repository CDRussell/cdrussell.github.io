---
layout: post
title: Defining a Live Template for Unit Tests
date: 2017-12-29
description: How to define a live template in Intellij/Android Studio which can be used to quickly add a unit test in the exact style you prefer.
---
# Defining a Live Template for Unit Tests
_This post will describe how to define a live template in Intellij/Android Studio which can be used to quickly add a unit test in the exact style you prefer._

When I add a new unit test, I want it to be as quick as possible but I also need to adhere to the team’s style on unit test naming. I want to define my when/then conditions, and have the cursor ready to type with as little effort as possible.

## Creating the Live Template
Create a live template, from the `Preferences` menu.

`Editor -> Live Templates`

<img
    src="/images/unit-test-live-templating-defining.png"
    alt="defining the live template"
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

Create new template, and give it an abbreviation. I’ve used test in this example. This is the word you need to type, followed by `ENTER` key, to invoke the live template.

```
@org.junit.Test
fun when$WHEN$Then$THEN$() {
    $END$
}
```

Adapt the template body to suit your team’s unit test naming conventions. Note that `@org.junit.Test` will automatically resolve to simply `@Test` so long as **Shorten FQ names** is checked. This will also add the appropriate import statement for you if it isn’t already added.

For applicable contexts, I’ve chosen **Kotlin Classes**.

## Template in Action
1. I type out the word test and hit `ENTER` key. It automatically populates the surrounding boilerplate and sets my cursor ready to type the _when_ condition for my unit test name.
2. Hitting `ENTER` again sets the cursor ready to type the then condition.
3. A final `ENTER`, and the cursor is in the body of the test ready to go.
   
<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/jDL8f8H1uUg" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>