---
layout: post
title: Comparing StandardTestDispatcher and UnconfinedTestDispatcher to test coroutines
date: 2022-01-19 16:19 +0000
description: Describes the two provided coroutine test dispatchers, standard and unconfined, the difference between them and when to use each
---

# StandardTestDispatcher vs UnconfinedTestDispatcher
_This articles describes the two provided coroutine test dispatchers, standard and unconfined, the difference between them, and when to use each_

## Some background

### What is a coroutine dispatcher?
A `coroutine dispatcher` determines which thread (or thread pool) should be used for running the coroutine. You are likely familiar with standard dispatchers, such as `Dispatchers.IO` and `Dispatchers.Default` (and `Dispatchers.Main` if working in Android). When using these, we can ensure that a coroutine is performed on the correct type of thread (e.g., threads meant for heavy CPU work, or IO work, or main thread etc...)

> **Coroutine Dispatchers** determine what thread or threads the corresponding coroutine uses for its execution. The coroutine dispatcher can confine coroutine execution to a specific thread, dispatch it to a thread pool, or let it run unconfined.
- Source: [kotlinlang](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html#dispatchers-and-threads)

```kotlin
launch(Dispatchers.Default) { 
    // do work that might be computationally expensive for the CPU
    // uses a thread pool
    //   - typically sized to have one thread for every CPU core the device has
    //   - minimum of 2 threads
}

launch(Dispatchers.IO) {
    // do IO work, like reading from disk or sending data over the network
    // uses a thread pool
    //   - the number of threads can grow and shrink on demand up to a pre-configured max size
    //   - default max thread pool is 64
}

launch (Dispatchers.Main) {
    // runs on the Android main thread. e.g., used for when interacting with Views
}
```

### What's the relevance of coroutine dispatchers when unit testing?
When writing unit tests, it is recommended to use a [`TestDispatcher`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-test-dispatcher/index.html) instead of a real coroutine dispatcher. 

Doing so allows you to have more control over how and when coroutines are launched in your classes under test, and can therefore result in more deterministic and reliable tests. You can control time, like skipping calls to `delay` to make tests run faster than they otherwise would, or advancing virtual time forward by a certain amount and asserting on the state of your coroutines at that moment. 

In order to dispatch coroutines using a test dispatcher, you should [inject coroutine dispatchers](https://craigrussell.io/2021/12/testing-android-coroutines-using-runtest/#injecting-coroutine-dispatchers).

## Which test dispatcher to use?
If you've read the above, you now know:
1. why a test dispatcher is useful
2. how to inject one into your class under test

What remains now is deciding which of the two available test dispatchers to use. Let's see what the options are:
- [StandardTestDispatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-standard-test-dispatcher.html)
- [UnconfinedTestDispatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-unconfined-test-dispatcher.html)

### StandardTestDispatcher
A standard test dispatcher does not execute any tasks right away. Instead, it has a test coroutine scheduler which exposes functions like `advanceTimeBy`, `runCurrent` which control when tasks get executed.

When coroutines are launched with this dispatcher, instead of executing immediately they are left in a pending state. What this means is that when testing code which launches coroutines (e.g., using `launch { }` ), **they will not be launched automatically**. This gives you full control over what is executed and when, so that you use that fine control to assert your code is working as expected.

#### If coroutines don't automatically run when using StandardTestDispatcher, how do you make them run?
You can access the `TestCoroutineScheduler` linked to the `StandardTestDispatcher` which provides methods for controlling the execution state of pending tasks. The following functions can be called directly from inside your `runTest` block.

[`runCurrent()`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-test-coroutine-scheduler/run-current.html)
- _runs the pending tasks that have been scheduled until current virtual time_  

[`advanceUntilIdle()`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-test-coroutine-scheduler/advance-until-idle.html)
- _runs all pending tasks_

[`advanceTimeBy(ms)`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-test-coroutine-scheduler/advance-time-by.html)
- _advances virtual time by given number of milliseconds, executing any pending coroutines that are now scheduled to run_


### UnconfinedTestDispatcher
An unconfined test dispatcher offers **no guarantees on the order** in which coroutines will be launched. 

In return for giving up this control, however, you are **not** required to manually call `runCurrent()` and `advanceUntilIdle()` in your tests as **coroutines are eagerly launched** with this dispatcher. 

>Using this TestDispatcher can greatly simplify writing tests where it's not important which thread is used when and in which order the queued coroutines are executed. Another typical use case for this dispatcher is launching child coroutines that are resumed immediately, without going through a dispatch; this can be helpful for testing Channel and StateFlow usages. ([Source](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-unconfined-test-dispatcher.html))


## Comparing standard and unconfined test dispatchers

| **Test Dispatcher Type** | **Full control over execution order** | **Coroutines Run Automatically** | 
|===|===|===|
|Standard|✅|❌|
|Unconfined|❌|✅|

- If you care about testing the order of coroutine execution, use a **Standard** test dispatcher
- If you have particularly tricky coroutine code, where you need fine control over what is launched and when, use a **Standard** test dispatcher
- Otherwise, give **Unconfined** test dispatcher a go as it can simplify every test you write and make them more concise 


# Links
- [Testing Android Coroutines using runTest](https://craigrussell.io/2021/12/testing-android-coroutines-using-runtest/)
- [Official docs](https://github.com/Kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-test/README.md)
- [StandardTestDispatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-standard-test-dispatcher.html)
- [UnconfinedTestDipatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-unconfined-test-dispatcher.html)