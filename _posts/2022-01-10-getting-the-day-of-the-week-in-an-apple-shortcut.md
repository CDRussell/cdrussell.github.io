---
layout: post
title: Getting the day of the week in an Apple Shortcut
date: 2022-01-10 07:53 +0000
description: How to get the day of the week when using Apple Shortcuts
categories: 
- Productivity
---

_This post describes how you could structure your Shortcut to get the current day of the week, and how you'd do something different depending on which day it is._

Apple Shortcuts provides only basic built-in functionality for working with Dates. If you want to do something like treating weekdays differently from a weekend, or only performing an action on Tuesdays and Thursday, it isn't obvious how you'd achieve this in Shortcuts.

## Nested if-statements ❌
Shortcuts doesn't support `if-else if` statements. At best, you could use an `if`, and in the `else` block nest another `if`. Then in _its_ `else` block, nest another `if`... 🔁.

```java
If day is Monday {
   // do something
}
else {
  if day is Tuesday {
    // do something
  }
  else {
    if day is Wednesday {
      // do something
    } 
    else {
       .. and so on
    }
  }
}
```
_yuk_.

## Using a dictionary ✅

<img
    src="/images/apple-shortcut-days-of-week-dictionary.png"
    alt="dictionary where there are 7 keys, matching the names of the day of the week. values are empty for now as it is just to help describe generally how this helps"
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

We can define a dictionary where the keys are the days of the week. The general idea is that we can parse the day of the week from the date as a string, and then use that string to lookup a value from the dictionary. In the example above, the values are empty, but we could add in a meaningful value like a string or a boolean (there's an example of this at the end of the post).

⚠️ The format of the days matters here; we need to make sure that the names we use for a key match the correct custom date formatting.

## Date formats
We can use the `Get Dictionary Value` action, and apply a custom date format which will return only the day of the week.

<img
    src="/images/apple-shortcuts-custom-date-format-day-of-week.png"
    alt="Apple Shortcut custom date formatting to return only name of day of the week"
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

With a date format set to `custom`, we can apply a format based on the [standard date formats](https://www.unicode.org/reports/tr35/tr35-31/tr35-dates.html#Date_Format_Patterns). For our case, `EEEE` which returns the full name of the day of the week:
 - `E` meaning to return the day of the week
 - That there are 4 `E`s meaning to use the "full name" format

⚠️ You have to make sure the date format you use produces the exact same names as the keys you use in the dictionary. My set of keys works for me in English, but would need adjusted if running on a machine set to another locale. You could decide to use numerical days of the weeks instead, but I prefer the named days for my use case as it's more readable.

# Example: Detecting If It Is a Weekend

<img
    src="/images/apple-shortcuts-is-it-a-weekend-example.png"
    alt="Example of an Apple Shortcut script showing if it's a weekend or not"
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>