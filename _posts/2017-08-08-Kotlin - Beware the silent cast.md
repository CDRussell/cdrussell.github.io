---
title: Kotlin - Beware the Silent Cast
date: 2017-08-08
layout: post
description: Beware a silent cast that can cause unexpected behavior
---
_Beware a silent cast that can cause unexpected behavior_

# Kotlin - Beware the Silent Cast

## Background
Kotlin has a `rangeTo` function which uses the operators in and .. and allows you to achieve some nice things with a concise syntax.

```kotlin
// Define a simple range of numbers, and determine if an input `i` is within that range

if (i in 1..10) {
    println("$i is within the range of 1-10")
} else {
    println("$i is not in range of 1-10")
}
```


Here, if we set `val i = 5` then we’ll see the output 5 is within the range of 1-10 ; this seems entirely intuitive.

It’s good to check the extremes, too, so setting `val i = 1` and `val i = 10` also results in it determining that 1 and 10 are both within the range of 1–10 (so both sides of the range are inclusive).

## The Problem
All is working well, so let’s meet the problem. Instead of setting `i` to an integer, let’s use a float instead and see what happens.

If we set `val i = 5.5` then we’ll see that 5.5 is within the range of 1–10 which is, again, expected.

But what happens if we set the value of i to 10.5? I expect it to tell me that 10.5 is not within range of 1–10.

```kotlin
val i = 10.5

if (i in 1..10) { 
    println("$i is within the range of 1-10")
} else {
    println("$i is not in range of 1-10")     
}

// outputs "10.5 is within the range of 1-10"
```

`val i = 10.5` gives the output 10.5 is within the range of 1–10 — uh oh! That’s not right! ❌

## The Explanation
I’m still new to learning Kotlin and while most things seem entirely intuitive and it does its best not to surprise me; this surprised me. Through further experiment I realised it was silently casting the float 10.5 down to an integer of 10 and therefore passing the range check.

I shared on Twitter and received a few responses from people offering explanations.

{% twitter https://twitter.com/trionkidnapper/status/893846014376935427 maxwidth=500 %}

Thanks to those who helped me understand. I was at the point where I sort of understood why it was doing that; because the range was an `intRange` then the input value needed to also be an integer. It sort of makes sense if you know that’s how it works, though it certainly isn’t obvious from looking at just the code above.

## Plot Twist
Just when I thought I understood the problem and was ready to move on, [Michael Bailey](https://twitter.com/yogurtearl) pointed out a further oddity.

The implementation changed, in a breaking way, between Kotlin 1.0.7 and 1.1.3.

```kotlin
val i = 10.5

if (i in 1..10) { 
    println("$i is within the range of 1-10")
} else {
    println("$i is not in range of 1-10")     
}

// outputs "10.5 is not in range of 1-10"
```

In Kotlin 1.0.7, that actually outputs that 10.5 is not in range, which is what I would expect to happen. But the same code run under Kotlin 1.1.3 results in the opposite, as described above.

## Source Code
Digging into the source code helps explain the problem.
### 1.0.7

```kotlin
@kotlin.jvm.JvmName("intRangeContains")
public operator fun ClosedRange<Int>.contains(value: Float): Boolean {
    return start <= value && value <= endInclusive
}
```

Here, you can see for an input value of type Float on an Int range, it simply compares the value to see if it is >= the start value and if it is also <= the end value. This is exactly the implementation I was expecting when I was using the range operator in this way.

### 1.1.3
```kotlin
@kotlin.jvm.JvmName("intRangeContains")
public operator fun ClosedRange<Int>.contains(value: Float): Boolean {
    return value.toIntExactOrNull().let { if (it != null) contains(it) else false }
}

internal fun Float.toIntExactOrNull(): Int? {
    return if (this in Int.MIN_VALUE.toFloat()..Int.MAX_VALUE.toFloat()) this.toInt() else null

```

With 1.1.3, it silently casts the float to an integer first. This means that 10.5 becomes 10 and then is determined to be in the range.

If you want to try this out for yourself, check out https://try.kotl.in which is an online code playground where you write some Kotlin and get immediate output. It also lets you set the Kotlin version that’ll be used to run the code.

Thanks to Pavlos Petros Tournaris for finding the commit in Kotlin source code where the change was made: <https://github.com/JetBrains/kotlin/commit/5773594412660af11357ac9adc6ffcb45138b840>

# Summary
While I’m assured that Kotlin does its best not to surprise you with silent casts like this, it does happen.

Oddly, it feels like the implementation of 1.0.7 is more intuitive, and simple. With 1.1.3, if it isn’t going to use a float, and instead silently cast to an integer, I feel like the API shouldn’t support the float as input in the first place. This would eliminate the chance for error.

If the float as an input is supported, then it should be honoured, and not silently cast to an integer.

This is a breaking change between Kotlin versions that could easily catch anyone out. If you use the `rangeTo` operator in this way, you’d be best sanity checking you aren’t passing it floats as an input value anywhere.

# Bug Tracker
Seems like there is already a bug filed for this. It will be interesting to see if it will be marked as intended behaviour or not. <https://youtrack.jetbrains.com/issue/KT-18938>

# Links
- <https://kotlinlang.org/docs/reference/ranges.html>
- <https://try.kotl.in>
