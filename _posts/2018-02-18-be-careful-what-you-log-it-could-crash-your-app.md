---
layout: post
title: Be Careful What You Log — It Could Crash Your App
date: 2018-01-30
canonical_url: https://proandroiddev.com/be-careful-what-you-log-it-could-crash-your-app-5fc67a44c842
description: Diving into a crash caused by a log statement with mixed parameter styles 
---
_If you are using Timber to log in your Android app you should be careful about logging variables you don’t fully control. If the input is formatted in a certain way, your app can crash._

```kotlin
Timber.w("The URL is %s - visited $count times", url)
Timber.w("The URL is $url - visited %d times", count)
```

One of these will log a URL quite happily, while the other could crash your app at runtime.

<img
    src="/images/timber-careful-logging-crash-scream.png"
    alt="scream emoji"
    height="100"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

## Overview
When using `Timber` and `Kotlin`, you can use [Kotlin String Templates](https://kotlinlang.org/docs/reference/basic-types.html#string-templates) to inject variables into your log statements or you can use the standard [Timber parameter formatting](https://github.com/JakeWharton/timber/blob/master/timber/src/main/java/timber/log/Timber.java#L57).

## Timber’s Approach
```kotlin
Timber.w("#1 - The URL is %s", url)
```

## Kotlin’s Approach
```kotlin
Timber.w("#2 - The URL is $url")
```

⚠️ You are advised to **use only the Timber parameter formatting style**.

When you try to mix both styles things can go wrong, as is detailed below. But even if you keep your style consistent the Timber approach has an advantage; lazy string concatenation. Timber will only build your parameters into a string for logging if the log will be used (if there is a logging tree configured).

If you use the Kotlin string template approach, those strings will always be built even if logging isn’t enabled for that particular build of your app.

## Logging URLs
I added some debug-only log statements to log the `URL`s a `WebView` was visiting to help track down a problem I was facing. I visited a few sites and all was fine. And then I visited another site and my app crashed. 😱

In my examples below, I use `Timber`, however, the problem really happens in the calls to [`String.format(String message, Object... args`)](https://developer.android.com/reference/java/lang/String.html#format(java.lang.String,%20java.lang.Object...)).

## Examples

In all the examples below, I am using the same URL: `example.com/%2F`

Although this looks contrived, it’s just a simplified example of one I saw in the wild. `%2F` is the code for a URL-encoded `/` (forward slash).

Ultimately, this is the problem. `%2F` itself is a string format placeholder. This is using an explicit index to specify that you will provide a float in the second parameter.

```kotlin
// one variable
Timber.w("#1 - The URL is %s", url)
Timber.w("#2 - The URL is $url")

// two variables - both given in same style
Timber.w("#3 - The URL is %s - visited %d times", url, count)
Timber.w("#4 - The URL is $url - visited $count times")

// two variables - mixing and matching styles
Timber.w("#5 - The URL is %s - visited $count times", url)
Timber.w("#6 - The URL is $url - visited %d times", count)
```

All 6 of these statements look like they will work at first glance. However, one will crash; one will work even though `Lint` will show it as an error; leaving the other 4 to work as expected.

<img
    src="/images/timber-careful-logging-stacktrace.png"
    alt="stacktrace showing an unknown format conversion exception"
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

## 1— This is ok ✅
`Timber.w("#1 - The URL is %s", url)`

This is fine; using the standard `Timber` mechanism for formatting strings.

## 2 — This is ok ✅
`Timber.w("#2 - The URL is $url")`

This is using Kotlin’s [String Templates](https://kotlinlang.org/docs/reference/basic-types.html#string-templates) which will handle replacing the `$url` with the variable. This works as expected; nothing to see here.


## 3 — This is ok ✅
`Timber.w(“#3 — The URL is %s — visited %d times”, url, count)`

We’re providing two variables here, one string and one int and providing both in the standard `Timber` manner.


## 4 — This is ok ✅
`Timber.w(“#4 — The URL is $url — visited $count times”)`

As above, two parameters. This time we’re using Kotlin string templates to provide the substitutions.

## 5 — This is a bit funny 🤡
`Timber.w(“#5 — The URL is %s — visited $count times”, url)`

<img
    src="/images/timber-careful-logging-code-example.png"
    alt="code example showing lint warning"
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

We’re providing two parameters again, but this time mixing the style between standard `Timber` and `Kotlin String Templates` . Things are starting to go a bit off here. The lint check is giving me an error here that I’ve provided the wrong number of arguments. But there is only one `%s` and one argument is provided so there should be no lint error here at all.

Despite the red squigglies, this runs fine. By the time it gets to the `String.format` method, the message parameter is

`#5 — The URL is %s — visited 4 times`

Here, the count value has already been substituted, but the URL has not.

## 6 — And here’s the crash 💥
`Timber.w(“#6 — The URL is $url — visited %d times”, count)`

Same as #5 above, except we’ve switched it so that `$url` is provided as a `Kotlin String Template` and `%d` is provided using the standard `Timber` arguments for passing an int.

This time, lint is perfectly happy with this line but it crashes when given our URL `example.com/%2F`.

This time, when it gets to the `String.format` line, the message is:

`#6 — The URL is http://example.com/%2F — visited %d times`

The URL has been substituted in place but the count has not. This now means it expects two parameters to be provided as arguments to the format method: `%2F` and `%d`. However, we only satisfy one of those, and therefore it throws an exception:

```
java.util.UnknownFormatConversionException: Conversion = 'F'
    at java.util.Formatter$FormatSpecifier.conversion(Formatter.java:2781)
    at java.util.Formatter$FormatSpecifier.<init>(Formatter.java:2811)
    at java.util.Formatter$FormatSpecifierParser.<init>(Formatter.java:2624)
    at java.util.Formatter.parse(Formatter.java:2557)
    at java.util.Formatter.format(Formatter.java:2504)
    at java.util.Formatter.format(Formatter.java:2458)
    at java.lang.String.format(String.java:2770)
    at timber.log.Timber$Tree.formatMessage(Timber.java:561)
    at timber.log.Timber$Tree.prepareLog(Timber.java:547)
    at timber.log.Timber$Tree.w(Timber.java:457)
    at timber.log.Timber$1.w(Timber.java:296)
    at timber.log.Timber.w(Timber.java:68)
```


# Summary
It’s fine to use `Timber` and `Kotlin` together. And while you can technically use either Kotlin string templates or Timber’s standard formatting, use of Timber’s method of providing parameters for logging is preferred as it is more efficient when logging is disabled.

Even if the performance benefit of using Timber’s approach doesn’t bother you and you prefer Kotlin’s, you should never try to mix and match using both approaches.

## Miscellaneous
This is questionable, but won’t crash

```kotlin
Timber.i("%2f")
```

Even though Timber will helpfully warn you that you haven’t provided the right number of arguments for the given format, nothing will crash. Nothing will crash because Timber will know there are no arguments, and therefore won’t call through to the `formatMessage` method.

```kotlin
if (args != null && args.length > 0) {
  message = formatMessage(message, args);
}
```