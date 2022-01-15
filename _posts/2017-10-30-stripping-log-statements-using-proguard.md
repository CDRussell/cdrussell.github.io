---
layout: post
title: Stripping log statements using Proguard
date: 2017-10-30
---
# Stripping Log Statements using Proguard
_This post describes how to use Proguard to strip log statements from your Android app; good for both the standard android.util.Log class as well as when using Timber._

<img
    src="/images/strip-log-statements-axe.jpeg"
    alt="photo of an axe and wood"
    width="100%"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

## Logging only on DEBUG Builds
Sometimes you want to remove all log statements from your app and it is as simple as _not_ logging in your production releases.

You can wrap your calls to `Log` inside a check to ensure you are on a debug build.

```
if(BuildConfig.DEBUG) Log.i(TAG, "foo");
```

In `Timber` land, you’d only plant a `DebugTree` and not bother about planting anything else.

```
if(BuildConfig.DEBUG) Timber.plant(new Timber.DebugTree());
```

## Logging in Production Too
However, if you need to _sometimes_ log, it gets more complicated. Let’s say you use `Timber` and plant a debug tree which will handle your logging while in development, but you also want a logger which will log any messages which are a warning or greater when in production too.

### Logging in Production? 😱
You probably shouldn't actually log in production. So maybe you don’t actually log to logcat and instead send the most important logs to your crash reporting tool.

```java
public class ProductionLogger extends Timber.Tree {
    @Override
    protected void log(int priority, String tag, String message, Throwable t) {
        if(priority >= Log.WARN) {
            // do your thing
        }
    }
}
```

## Stripping Log Statements
In `debug` there is nothing to do; you want to see all logs statements. But in production, you want strip all logs except for `warn` and `error`. Proguard can help us here.

In `build.gradle` you can enable `Proguard`

```
minifyEnabled true
proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
```

Note, that’s **proguard-android-optimize.txt** ☝️

## Stripping Standard Log Statements
Modify the `proguard-rules.pro` file, which should live under your standard Android app directory:

``` 
proguard-rules.pro
==================

# This will strip `Log.v`, `Log.d`, and `Log.i` statements and will leave `Log.w` and `Log.e` statements intact.

-assumenosideeffects class android.util.Log {
    public static boolean isLoggable(java.lang.String, int);
    public static int v(...);
    public static int d(...);
    public static int i(...);
}
```

## Stripping Timber Log Statements
Modify the `proguard-rules.pro` file:

```
-assumenosideeffects class timber.log.Timber* {
    public static *** v(...);
    public static *** d(...);
    public static *** i(...);
}
```

## Testing it works
Add the following to a class you know will be executed (delete as appropriate depending on if you’re stripping standard Log statements or `Timber` logs)

```java
Log.v(TAG, "verbose");
Log.d(TAG, "debug");
Log.i(TAG, "info");
Log.w(TAG, "warning");
Log.e(TAG, "error");

Timber.v("verbose");
Timber.d("debug");
Timber.i("info");
Timber.w("warning");
Timber.e("error");
```

When you run in `debug` mode, you should see all the statements in logcat.

<img
    src="/images/strip-log-statements-all-logs-visible.png"
    alt="all logs visible in log output"
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

When running the app after proguarding, however, you should only see the output from warning and error log statements.

<img
    src="/images/strip-log-statements-only-subset-logs-visible.png"
    alt="only warn and error logs visible"
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>


# Bonus Content: Using Proguard Only to Remove Logging
If you don’t really want to use `Proguard`, but do quite like the idea of it stripping out your logs for you, you can disable all proguarding except for the log stripping.

Add the following to your `proguard-rules.pro` to achieve this:

```
-dontwarn **
-target 1.7
-dontusemixedcaseclassnames
-dontskipnonpubliclibraryclasses
-dontpreverify
-verbose
-optimizations !code/simplification/arithmetic,!code/allocation/variable
-keep class **
-keepclassmembers class *{*;}
-keepattributes *
```

# Attributions
- Cover photo by Malte Wingen on Unsplash
- StackOverflow — <https://stackoverflow.com/a/15593061/1654145>