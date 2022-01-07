---
layout: post
title: Testing Android Coroutines using runTest
date: 2021-12-07 01:05 +0000
description: The latest tooling to help test Android Coroutines, provided as part of the kotlinx.coroutines test libraries, which aim to “provide utilities for efficiently testing coroutines”.
permalink: 2021/12/testing-android-coroutines-using-runtest/
---

*This post describes the latest tooling to help test Android Coroutines, provided as part of the `kotlinx.coroutines` test libraries, which aim to "provide utilities for efficiently testing coroutines".*

*This blog post covers how to test the following scenarios involving coroutines:*
1. _Writing a unit test for code that calls a suspend function_ 
1. _Writing a unit test for code that launches a new coroutine internally_ 

# Overview
Testing code which creates or uses coroutines has always been a challenge in Android. There have been a few official tools and libraries provided previously which sort of worked, but came with challenges and gotchas. Now, as of around December 2021, we have a new contender to simplify testing coroutines.

# kotlinx-coroutines-test Module
A module specifically to improve testing coroutines and code which interacts with coroutines. The [official docs](https://github.com/Kotlin/kotlinx.coroutines/tree/master/kotlinx-coroutines-test) are definitely worth reading, and this blog post serves to complement them with additional explanations as to why they're needed.

## Add Dependencies
Add one or both of the **test** dependencies below to get started, depending on whether you run JVM-based unit tests, instrumentation tests or both. These should be added to your `build.gradle` file.

```groovy
dependencies {
    
    // JVM-based unit tests (that don't need a real device or emulator)
    testImplementation "org.jetbrains.kotlinx:kotlinx-coroutines-test:1.6.0-RC"
    
    // Instrumentation unit tests (that will require a real device or emulator)
    androidTestImplementation "org.jetbrains.kotlinx:kotlinx-coroutines-test:1.6.0-RC"
    
    // Coroutines, and the much-recommended library to add lifecycle-awareness to coroutines
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.5.2"
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.4.0"
    
}
```

# Example Code using Coroutines

Let's pretend we have a class which does some hard work. This class adds a lot of numbers to a list, and sorts and shuffles them over and over for a while, before returning the first number of the sorted list, which is always 0. But really that's not important here; what is important is that it does too much work to be called on the main thread.

```kotlin
class NumberCruncher {

    fun getResult(): Int {
        return longRunningOperation()
    }

    private fun longRunningOperation(): Int {
        val list = mutableListOf<Int>()

        for (i in 0..1_000) {
            list.add(i)
        }
        for (i in 0..20_000) {
            list.shuffle()
            list.sort()
        }

        return list.first()
    }
}
```

We have a simple `Activity` containing a `TextView` and a `Button`. When the button is pressed, the `TextView` will show a temporary `calculating...` message, and then the `TextView` will show the result.


<img src="/images/coroutine-runTest-activity.png" alt="Screenshot of the activity described above" height="600">


```kotlin
class MainActivity : AppCompatActivity() {

    // UI references
    private lateinit var resultTextView: TextView
    private lateinit var calculateButton: Button

    // class which does a lot of hard work
    private val numberCruncher = NumberCruncher()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        resultTextView = findViewById(R.id.resultTextView)
        calculateButton = findViewById(R.id.calculateButton)

        calculateButton.setOnClickListener {
            resultTextView.text = getText(R.string.calculating)
            val result = numberCruncher.getResult()
            resultTextView.text = String.format("Got result %d", result)
        }
    }
}
```

## Adding coroutines into the example
If we try to call that code as is, the UI will completely freeze as we try to do too much work from the main thread. Let's modify `NumberCruncher` to make use of coroutines so that it delegates the CPU-intensive work to another thread.

```kotlin
class NumberCruncher {

    suspend fun getResult(): Int {
        return withContext(Dispatchers.Default) {
            longRunningOperation()
        }
    }

    // function body hidden as it's the same as before. We won't change this function at all in this blog post.  
    private fun longRunningOperation(): Int {...}
}
```
We've made `getResult()` a `suspend` function, and ensured the heavy CPU work is done away from the main thread using `withContext(Dispatchers.Default)`. When we run the app now, we can see the UI does not freeze, and we can see the temporary `calculating...` message. Huzzah! But let's not celebrate too quickly; we've still got to write unit tests for our `NumberCruncher` class.

# Writing a unit test for code that calls a suspend function


## Attempt 1, not using coroutines ❌

```kotlin
class NumberCruncherTest {
    
    private val numberCruncher = NumberCruncher()
    
    @Test
    fun test() {
        numberCruncher.getResult() // ❌ won't compile
    }
    
}
```

The compiler won't let us call `numberCruncher.getResult()` like this since it is a `suspend` function, meaning it can only be called from a coroutine.

## Attempt 2, launching a coroutine from the test ❌

```kotlin
@Test
fun test() {
    GlobalScope.launch {
        assertEquals(0, numberCruncher.getResult())
    }
}
```

If you try launching a new coroutine inside your test like this, you might be pleased to see the test passing. However, **this isn't working at all**, and if you were to change that assertion to expect any other value, the test would continue to pass. This is because the test is finished before the calculation can finish, and before the new coroutine can even start. 

💡 This is why you should always start with a failing test, then make it pass.


## Attempt 3, using `runBlocking` ❌
```kotlin
@Test
fun test() = runBlocking { 
    assertEquals(0, numberCruncher.getResult())
}
```

Previously, for testing coroutines, there were a few options including using `runBlocking`, and `runBlockingTest`. However promising these seemed, there were always scenarios where they didn't work as expected or were error-prone. In a [blog post](https://craigrussell.io/2019/11/unit-testing-coroutine-suspend-functions-using-testcoroutinedispatcher/) I wrote about this a few years back, I noted a scenario where `runBlockingTest` should have worked, including linking to a long-running [PR](https://github.com/Kotlin/kotlinx.coroutines/issues/1204) which promised a fix was coming. However, it never did. Instead, that PR was closed off in favor of the new coroutine testing tooling. 

In short, this isn't the solution you're looking for either. It might work in some cases and confusingly not work in others. However, don't despair, help is at hand.

## Attempt 4, using `runTest` ✅
As promised, the latest coroutine testing tooling offers a solution with the introduction of a new coroutine builder specifically to be used in tests, called [`runTest`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/run-test.html).

```kotlin
@Test
fun test() = runTest { 
    assertEquals(0, numberCruncher.getResult())
}
```

🎉 This time, we have success. The `runTest` coroutine builder means you can test your code which calls `suspend` functions, and doesn't come with the same problems that its predecessor `runBlockingTest` had.




# Writing a unit test for code that launches new coroutines under the hood
In the above example, our `NumberCruncher` exposed a `suspend` function. However sometimes you will be trying to test code that internally launches new coroutines. 

```kotlin
// 👇 We pass a `CoroutineScope` in to the constructor now to let us launch new coroutines
class NumberCruncher(private val coroutineScope: CoroutineScope) {

    // 👇 We have a `SharedFlow` of results now.
    private val _results = MutableSharedFlow<Int>()
    fun results() = _results.asSharedFlow()

    // 👇 We now allow a new result to be requested, but it isn't returned immediately.
    fun calculate() {

        // 👇 we have a `launch` in here now, where we had a `withContext` before
        coroutineScope.launch(Dispatchers.Default) {
            val result = longRunningOperation()

            // We've added a 5s delay here to make testing even harder. 
            // 👇 Ideally, production code would respect this delay, but unit tests would not it will slow down your test suite.
            delay(5_000)

            _results.emit(result)
        }
    }
    
    // unchanged
    private fun longRunningOperation(): Int {...}
}
```
We've made our code more reactive, since reactive code is in fashion 👔. Before, we could call `getResult()` and wait for the result to be returned. Now, we can request a new result be calculated but it won't be returned there and then; instead, it will be emitted from a `Flow` shortly afterwards when it's calculated.

Testing this kind of code is harder than before. Because this code internally calls `launch` to create a new coroutine (and doesn't expose the `Job` externally) ensuring the logic is executed while the unit test is running is important. We don't want to hit the problem from before when the unit test completes before the coroutine has been launched, as we aren't testing what we think we are testing if that happens.


## Attempt 1, testing this code using only `runTest` ❌

```kotlin
@Test
fun test() = runTest {
    val numberCruncher = NumberCruncher(this)
    numberCruncher.calculate()
    assertEquals(0, numberCruncher.results().first())
}
```

This looks like it should work, and indeed running it you'll find the test passing, but it will take a while. The reason it's so slow is because that `delay(5_000)` we added is being respected even in the unit test. Why isn't `runTest` doing what it claims to do in the docs: _"The calls to delay are automatically skipped"_? The answer is given in the docs in the section called [Virtual Time Support With Other Dispatchers](https://github.com/Kotlin/kotlinx.coroutines/tree/master/kotlinx-coroutines-test#virtual-time-support-with-other-dispatchers).

> Calls to withContext(Dispatchers.IO), withContext(Dispatchers.Default), and withContext(Dispatchers.Main) are common in coroutines-based code bases. Unfortunately, just executing code in a test will not lead to these dispatchers using the virtual time source, so delays will not be skipped in them. Tests should, when possible, replace these dispatchers with a TestDispatcher.


## Injecting coroutine dispatchers
We need to **stop hardcoding the dispatchers** using code like `Dispatchers.Default` and instead provide a way to inject dispatchers into classes. One simple mechanism I use for this is to define a `DispatcherProvider` interface.

```kotlin
interface DispatcherProvider {

    fun main(): CoroutineDispatcher = Dispatchers.Main
    fun default(): CoroutineDispatcher = Dispatchers.Default
    fun io(): CoroutineDispatcher = Dispatchers.IO
    fun unconfined(): CoroutineDispatcher = Dispatchers.Unconfined

}

class DefaultDispatcherProvider : DispatcherProvider

```

This interface defines defaults for each of the main dispatchers you'll already be familiar with, and also defines a ready-made class called `DefaultDispatcherProvider` for convenience. To use this, we pass a dispatcher provider into the constructor of a class, like this:

```kotlin
class NumberCruncher(private val coroutineScope: CoroutineScope,
                     private val dispatchers: DispatcherProvider = DefaultDispatcherProvider()) {
    
    ...

    fun calculate() {

        // 👇 we now use `dispatchers.default()` instead of hardcoding the dispatcher to `Dispatchers.Default'
        coroutineScope.launch(dispatchers.default()) {
            ... 
        }
} 
```

In production code, the default parameter value is used, meaning you don't have to explicitly provide it anywhere. But the value now is that while unit testing, you can provide an alternative version which uses a `TestDispatcher` instead of a real one. For convenience, some of the boilerplate required in each test can be encapsulated in a test rule.

```kotlin
@ExperimentalCoroutinesApi
class CoroutineTestRule(val testDispatcher: TestDispatcher = UnconfinedTestDispatcher(TestCoroutineScheduler())) : TestWatcher() {

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
    }
}
```

Now, you can apply that test rule and use its test dispatcher provider when creating your class under test.

# Final working attempt, using `runTest`, and providing test dispatchers ✅

Let's summarise what we have done:
1. Used `runTest` to create a coroutine to be used while unit testing
2. Used the coroutine scope provided by `runTest` by passing it to the class that launches a new coroutine
3. Provided alternative coroutine dispatchers while testing
4. Created a coroutine test rule to hold some boilerplate

```kotlin
class NumberCruncher(private val coroutineScope: CoroutineScope,
                     private val dispatchers: DispatcherProvider = DefaultDispatcherProvider()) {

    private val _results = MutableSharedFlow<Int>()
    fun results() = _results.asSharedFlow()

    fun calculate() {

        // 👇 using dispatcher provider avoids hardcoding dispatcher, allowing for us to use a `TestDispatcher` while testing
        coroutineScope.launch(dispatchers.default()) {
            val result = longRunningOperation()
            delay(5_000)
            _results.emit(result)
        }
    }

    private fun longRunningOperation(): Int {
        val list = mutableListOf<Int>()

        for (i in 0..1_000) {
            list.add(i)
        }
        for (i in 0..20_000) {
            list.shuffle()
            list.sort()
        }

        return list.first()
    }
}
```

```kotlin
@get:Rule
val coroutineTestRule: CoroutineTestRule = CoroutineTestRule()

@Test
fun test() = runTest {
    val numberCruncher = NumberCruncher(this, coroutineTestRule.testDispatcherProvider)
    numberCruncher.calculate()
    assertEquals(0, numberCruncher.results().first()) 
}
```

This test passes, and passes quickly as it now rightfully skips the `delay()`.

### Further reading
ℹ️ The new tooling offers lots of control over execution of coroutines which isn't covered in this post. For more details on that if required, you should check out the javadocs for `UnconfinedTestDispatcher` and `StandardTestDispatcher`, along with functions available inside of the `runTest` block to [control virtual time](https://github.com/Kotlin/kotlinx.coroutines/tree/master/kotlinx-coroutines-test#controlling-the-virtual-time). 

