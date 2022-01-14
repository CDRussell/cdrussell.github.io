---
layout: post
title: Kotlin — Measure and Execute a block of code
date: 2019-02-14
description: Kotlin utility function to execute code and measure how long that took
---

## Kotlin — Measure and Execute a block of code

<img
    src="/images/stock-image-timer.jpeg"
    alt="stock image of a white kitchen timer"
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

### Overview

This post illustrates a utility function which executes a block of code. It also measures how long it took to execute, and logs the duration.

This is useful for generating crude benchmarks of operations so you can see roughly how long a block of code takes to execute. It’s not scientific, but sometimes it’s enough to make decisions on where bottlenecks might lie.

It records the start time, executes the function and prints out the time it took to execute.

### **Measure and Execute**

```kotlin
inline fun <T> measureExecution(logMessage: String, logLevel: Int = Log.DEBUG, function: () -> T): T {
    val startTime = System.nanoTime()
    return function.invoke().also {
        val difference = System.nanoTime() - startTime
        Timber.log(logLevel, "$logMessage; took ${TimeUnit.NANOSECONDS.toMillis(difference)}ms")
    }
}
```

- We define the function measureExecution to accept a log message that will be printed after execution is finished. 
- We can also provide a log level to use though it will default to DEBUG if you omit it. 
- And finally it accepts the action code you want to measure, defined as a function: () -> T; this function accepts no parameters but returns *something* if required.

### Usage

For the simplest usage case, call the measureExecution function passing in the log statement to accompany the measurement. For example:

```kotlin
measureExecution("Finished doing work") {
  foo.doSomeWork()
}
```

This will output Finished doing work; took 123ms at DEBUG log level. If you wanted to log to a different log level, and perhaps return a value, that’s cool too.

```kotlin
val result = measureExecution("Finished doing work", Log.INFO) {
  foo.doSomeWork()
  return foo.getResult()
}
```

You don’t even need to specify the type; the compiler will figure it out 👌

*Image Attribution: [Marcelo Leal](https://unsplash.com/photos/vZawEq0Eexo?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/timer?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*
