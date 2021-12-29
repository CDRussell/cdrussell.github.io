---
layout: post
title: "Unit Testing Coroutine Suspend Functions using TestCoroutineDispatcher"
date: 2019-11-08
permalink: 2019/11/unit-testing-coroutine-suspend-functions-using-testcoroutinedispatcher/

description: Kotlin coroutines provide an elegant way to write asynchronous code, but sometimes coroutines make it difficult to write unit tests. This post describes how to use TestCoroutineDispatcher to write efficient and stable unit tests for code written with coroutines.
keywords:
  - unit testing coroutines
  - kotlin
  - kotlin coroutines
  - override default coroutine dispatchers
  - runBlocking
  - runBlockingTest
  - TestCoroutineDispatcher
  - kotlinx-coroutines-test
  - customize Dispatchers.IO
  - Dispatchers.Main
  - android
---
*Kotlin coroutines provide an elegant way to write asynchronous code, but __sometimes coroutines make it difficult to write unit tests__.*

*This post describes how to use `TestCoroutineDispatcher` to write efficient and stable unit tests for code written with coroutines.*

# UPDATE: 2021-12-08
A new recommended way to test coroutines now exists. See [Testing Coroutines using runTest](https://craigrussell.io/2021/12/testing-android-coroutines-using-runtest/) for more info on the latest recommended practices for testing coroutines.


## Unit testing a Suspend function
In order to reliably unit test a `suspend` function written with Kotlin Coroutines, there are a few things we need to know.

At a minimum, we need to know how to build a coroutine from our unit tests and how to make our unit tests wait until all the jobs in the coroutine have finished.

Ideally beyond that, we will want to know how to make our unit test run as fast as possible, and not sit around waiting for a coroutine `delay` to finish.

This post will describe how to achieve this so you can quickly and reliably unit test `suspend` functions.


## Setup
### Gradle Dependencies
We are going to use the [`kotlinx-coroutines-test`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/) library, so we'll need to add that to our dependendies in `build.gradle`.

```kotlin

ext {
    coroutines = "1.3.1"
}

dependencies {
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:$coroutines"
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:$coroutines"

    // testImplementation for pure JVM unit tests
    testImplementation "org.jetbrains.kotlinx:kotlinx-coroutines-test:$coroutines"

    // androidTestImplementation for Android instrumentation tests
    androidTestImplementation "org.jetbrains.kotlinx:kotlinx-coroutines-test:$coroutines"
}
```

### CoroutineTestRule
Add the following class to your project - it can be included alongside your unit tests.
```kotlin

@ExperimentalCoroutinesApi
class CoroutineTestRule(val testDispatcher: TestCoroutineDispatcher = TestCoroutineDispatcher()) : TestWatcher() {
    override fun starting(description: Description?) {
        super.starting(description)
        Dispatchers.setMain(testDispatcher)
    }

    override fun finished(description: Description?) {
        super.finished(description)
        Dispatchers.resetMain()
        testDispatcher.cleanupTestCoroutines()
    }
}

```

This class is a unit test rule which watches for tests starting and finishing. It contains a reference to a `TestCoroutineDispatcher`, and as tests are starting and stopping it overrides the default `Dispatchers.Main` dispatcher and replaces the default with our test dispatcher.


### Apply the CoroutineTestRule
Apply the unit test rule to your unit test class.

```kotlin

@ExperimentalCoroutinesApi
class HeavyWorkerTest {
    
    @get:Rule
    var coroutinesTestRule = CoroutineTestRule()

    // todo - write some tests
}

```


### Onwards
With that setup done, we're now ready to proceed.


## Unit Testing a Suspend Function
### Example
Let's take the following `HeavyWorker` class as an example. It has a `heavyOperation()` function which is `suspendable`. Our goal here is to write unit tests for it that are stable and run as quickly as possible.

```kotlin

class HeavyWorker {

    suspend fun heavyOperation(): Long {
        return withContext(Dispatchers.Default) {
            return@withContext doHardMaths()
        }
    }

    // waste some CPU cycles
    private fun doHardMaths(): Long {
        var count = 0.0
        for (i in 1..100_000_000) {
            count += sqrt(i.toDouble())
        }
        return count.toLong()
    }
}

```

### What are my options?
There are multiple approaches to unit testing a `suspend` function. But crucially, since you are calling a `suspend` function, you'll need to use a coroutine builder. There's a few to choose from:

1. `kotlinx.coroutines.runBlocking` ❌
2. `kotlinx.coroutines.test.runBlockingTest` ❌
3. `kotlinx.coroutines.test.TestCoroutineDispatcher.runBlockingTest` ✅

🤔 If you're looking at the three options and wondering why they have such similar names, what the differences are and why they don't all work, I don't blame you. It's confusing! And worse still, some will work for some `suspend` functions but not for others.

Let's go through each in turn and highlight the problem with that approach before finally explaining why the `TestCoroutineDispatcher.runBlockingTest` approach is a good one.


### `runBlocking` ❌
```kotlin

@Test
fun useRunBlocking() = runBlocking<Unit> {
   val heavyWorker = HeavyWorker()
   val expected = 666666671666
   val result = heavyWorker.heavyOperation()
   assertEquals(expected, result)
}

```

In this example, even with the long operation, the unit test will patiently wait for the coroutine's completion before successfully passing. Depending on your hardware, this might take a few seconds to complete.

But crucially, it does pass. 🎉

OK, that was too easy. Let's make things a bit trickier and see where `runBlocking` begins to fail us. Let's see what happens if the heavy worker code had reason to `delay()` for 30 seconds during its execution.

```kotlin

suspend fun heavyOperation(): Long {
    return withContext(Dispatchers.Default) {
        delay(30_000)
        return@withContext doHardMaths()
    }
}

```

In this case, `runBlocking<Unit>` waits for the entirety of the `delay` period, meaning **your unit test takes an additional 30s to run** on top of the time it actually takes to crunch the numbers. 

- Unit test still passes 👍 
- Adds 30s to the unit test execution time 👎 

## runBlockingTest ❌

```kotlin

@Test
fun useRunBlockingTest() = runBlockingTest {
    val heavyWorker = HeavyWorker()
    val expected = 666666671666
    val result = heavyWorker.heavyOperation()
    assertEquals(expected, result)
}

```

[`runBlockingTest`](https://github.com/Kotlin/kotlinx.coroutines/tree/master/kotlinx-coroutines-test#runblockingtest) was introduced as a newer coroutine builder than `runBlocking`, specifically to help improve in areas such as not having to wait for the full `delay` period. Other features listed:

- Auto-advancing of time for regular suspend functions
- Explicit time control for testing multiple coroutines
- Eager execution of launch or async code blocks
- Pause, manually advance, and restart the execution of coroutines in a test
- Report uncaught exceptions as test failures

This is all sounding really great, and likely to improve some areas of testing massively. However, it doesn't work for this code 😢

Specifically, the test will fail with the following reason:

    java.lang.IllegalStateException: This job has not completed yet

	at kotlinx.coroutines.JobSupport.getCompletionExceptionOrNull(JobSupport.kt:1128)
	at kotlinx.coroutines.test.TestBuildersKt.runBlockingTest(TestBuilders.kt:53)
	at kotlinx.coroutines.test.TestBuildersKt.runBlockingTest$default(TestBuilders.kt:45)
	at com.cdrussell.coroutines.testing.HeavyWorkerTest.useRunBlockingTest(HeavyWorkerTest.kt:25)
	...

`runBlockingTest` is smart enough to realise that you have a job started during your unit testing and that it hasn't finished yet.

The fact that the unit test finishes before the invoked `Job` finishes is bad. Your unit test which was stable albeit a bit slow using `runBlocking` now breaks. That's not much of an improvement 😞.

### Why Doesn't This Work? 😕
This feels like it _should_ work, and I don't know why it doesn't. Maybe it's a bug in the current implementation of `runBlockingTest`. Maybe I'm trying to get it to do do something it isn't designed to do. I've reached out to some folks who are building this to clarify.

**UPDATE:**
Manuel confirmed that the above example _should_ work, and indeed, `runBlockingTest` should work in all cases where you currently use `runBlocking`.

{% twitter https://twitter.com/trionkidnapper/status/1193827122357448709 maxwidth=500 %}
{% twitter https://twitter.com/trionkidnapper/status/1193831513797922816 maxwidth=500 %}

Injecting dispatchers shouldn't be required for this scenario; `runBlockingTest` should work on its own. However, injecting dispatchers will still be required when calling a function which launches a coroutine; there will be a follow-up blog post on testing this scenario.

So watch https://github.com/Kotlin/kotlinx.coroutines/issues/1204 for updates on when that is resolved.

## TestCoroutineDispatcher.runBlockingTest ✅
This approach is very similar to the one we just tried (and failed) to use, but with a crucial difference; we'll provide alternative coroutine dispatchers when running in a unit test to those we'd use in production.

And after providing an alternative coroutine dispatcher, used only when unit testing, we'll make use of the dispatcher's `runBlockingTest` builder.

Let's see what's required.

### Injecting our Dispatchers
The conclusion I've drawn is that **we need to inject our dispatchers**. I tried to find alternatives to doing this, as I don't love having to inject dispatchers into production code, but I can't find an alternative; it seems necessary.

![Image from Android Dev Summit 2019 showing a slide with "You should ALWAYS inject Dispatchers"](/images/always-inject-dispatchers.png)

And it seems Sean McQuillan and Manuel Vivo agree ☝️; this was taken from their 2019 Android Dev Summit talk ["Testing Coroutines on Android"](https://www.youtube.com/watch?v=KMb0Fs8rCRs)

What does it mean to inject our dispatchers? Let's revisit our production code.

```kotlin

suspend fun heavyOperation(): Long {
    return withContext(Dispatchers.Default) {
        return@withContext doHardMaths()
    }
}

```

We have a hardcoded dispatcher here, `Dispatchers.Default`, which cannot be changed. We cannot provide an alternative dispatcher under any condition, not even when unit testing. And it turns out that providing an alternative dispatcher during unit testing is _exactly_ what we need.

In order to change the dispatcher that is used when unit testing, we need a way to _inject_ the dispatcher into our `HeavyWorker` production class.

### DispatcherProvider
In the example above, we used `Dispatchers.Default` but we know there are other common dispatchers that we might encounter. As such, let's define an interface which will allow classes to obtain whichever dispatcher they require.

```kotlin

interface DispatcherProvider {

    fun main(): CoroutineDispatcher = Dispatchers.Main
    fun default(): CoroutineDispatcher = Dispatchers.Default
    fun io(): CoroutineDispatcher = Dispatchers.IO
    fun unconfined(): CoroutineDispatcher = Dispatchers.Unconfined

}

class DefaultDispatcherProvider : DispatcherProvider

```

We define the interface and provide functions for the dispatchers, each of which have a default implementation.

- `main()` --> `Dispatchers.Main`, 
- `default()` --> `Dispatchers.Default`, 
- `io()` --> `Dispatchers.IO`
- `unconfined()` --> `Dispatchers.Unconfined`

Additionally, we define a `DefaultDispatcherProvider` default implementation of the interface; this is what we'll use in our production code. 

In production, we'll always use this default dispatcher provider, which will always result in using `Dispatchers.Main`, `Dispatchers.IO`  etc..., and the production behavior will be as it always was.

In unit testing, however, we will not use this default dispatcher provider; we will provide an alternative version.

### Injecting DispatcherProvider
Now that we have interface defined, we need to modify `HeavyWorker` class to make use of the interface instead of fetching the dispatchers in its current hardcoded fashion. To do this, we can inject the interface into the constructor. 

```kotlin
class HeavyWorker(private val dispatchers: DispatcherProvider = DefaultDispatcherProvider()) {

    suspend fun heavyOperation(): Long {
        return withContext(dispatchers.default()) {
            delay(30_000)
            return@withContext doHardMaths()
        }
    }
    
}
```

#### Constructor parameter
Since you are specifying the default implementation of the provider should be used in production, you never have to actually pass a parameter when instantiating this class in production. `HeavyWorker()` is still a valid way to construct the class, just as it was before. 


#### No longer hardcoding dispatchers
We no longer hardcode dispatchers to use `Dispatchers.Default` and instead use the provided `dispatchers.default()`. To be clear, the production behaviour here would use `DefaultDispatcherProvider` by default, and therefore no behavioural changes have happened in production 😌.


#### Providing alternative dispatcher provider during unit testing
 We now define an alternative dispatcher provider that we'll use during unit testing.

 ```kotlin

 val testDispatcherProvider = object : DispatcherProvider {
    override fun default(): CoroutineDispatcher = testDispatcher
    override fun io(): CoroutineDispatcher = testDispatcher
    override fun main(): CoroutineDispatcher = testDispatcher
    override fun unconfined(): CoroutineDispatcher = testDispatcher
}

```

You can define this wherever you like so long as it has access to the test dispatcher, but I chose to add it inside the `CoroutineTestRule` itself.

```kotlin
@ExperimentalCoroutinesApi
class CoroutineTestRule(val testDispatcher: TestCoroutineDispatcher = TestCoroutineDispatcher()) : TestWatcher() {

    val testDispatcherProvider = object : DispatcherProvider {
        override fun default(): CoroutineDispatcher = testDispatcher
        override fun io(): CoroutineDispatcher = testDispatcher
        override fun main(): CoroutineDispatcher = testDispatcher
        override fun unconfined(): CoroutineDispatcher = testDispatcher
    }

    override fun starting(description: Description?) {
        super.starting(description)
        Dispatchers.setMain(testDispatcher)
    }

    override fun finished(description: Description?) {
        super.finished(description)
        Dispatchers.resetMain()
        testDispatcher.cleanupTestCoroutines()
    }
}

```

### Writing the test
```kotlin

@ExperimentalCoroutinesApi
class HeavyWorkerTest {

    @get:Rule
    var coroutinesTestRule = CoroutineTestRule()

    @Test
    fun useTestCoroutineDispatcherRunBlockingTest() = coroutinesTestRule.testDispatcher.runBlockingTest {
        val heavyWorker = HeavyWorker(coroutinesTestRule.testDispatcherProvider)
        val expected = 666666671666
        val result = heavyWorker.heavyOperation()
        assertEquals(expected, result)
    }

}

```

Note:

- We use `runBlockingTest` that we obtain from the `TestCoroutineDispatcher`, which is inside the `CoroutineTestRule`.
- We pass the `testDispatcherProvider` from inside the `CoroutineTestRule` to the `HeavyWorker`'s constructor.
- The test passes ✅
- It doesn't wait 30s for the delay to finish! 🙏

Using `TestCoroutineDispatcher.runBlockingTest` as our coroutine builder, and injecting the test dispatcher, allows us full control over the coroutine jobs created when unit testing. 

It allows us to achieve the reliability we need, combined with the speed in not having to wait for delays to end.

## Summary
In this post, we saw how to write a reliable and fast unit test for code which uses Kotlin coroutines; specifically, how to unit test a `suspend` function.

We saw that:

- 😞 `runBlocking` worked for us, but could result in slow tests.  
- 😢 `runBlockingTest` didn't work at all.
- 😎 `TestCoroutineDispatcher.runBlockingTest` worked, so long as we can inject coroutine dispatchers to our class under test.


## Links

- [Source code](https://github.com/CDRussell/testing-coroutines)
- [Testing Coroutines on Android - Android Dev Summit](https://www.youtube.com/watch?v=KMb0Fs8rCRs)
- [runBlockingTest](https://github.com/Kotlin/kotlinx.coroutines/tree/master/kotlinx-coroutines-test#runblockingtest)
- [kotlinx-coroutines-test](https://github.com/Kotlin/kotlinx.coroutines/tree/master/kotlinx-coroutines-test)