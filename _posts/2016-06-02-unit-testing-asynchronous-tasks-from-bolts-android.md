---
layout: post
title: Unit Testing Asynchronous Tasks from Bolts-Android
date: 2016-06-02
description: How to write unit tests for asynchronous code using Bolts
---
# Unit Testing Asynchronous Tasks from Bolts-Android
_How to write unit tests for asynchronous code using Bolts_

## What is Bolts?
What is Bolts-Android? Bolts essentially consists of two libraries; one for supporting app linking and another for asynchronous tasks. It is an open source library from Facebook and you can read all about it [here](https://github.com/BoltsFramework/Bolts-Android).

If you are new to Bolts, this is not the article for you, as there are better articles out there for describing how to get started with Bolts; this article assumes a good deal of familiarity with it already. For me, adopting Bolts has made many complicated tasks much easier and has become a regular in my Android developer toolbox.

This article describes a means of testing Bolts Tasks.

<img
    src="/images/bolts-and-tools.jpeg"
    alt="various hardware tools laid out on a table"
    width="100%"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

## I’m in a hurry, just give me the code!!

Sure, here you go. [Sample Project](https://github.com/CDRussell/BoltsExample) on Github.

## Why is testing difficult?
Why is testing asynchronous `Tasks` difficult? In essence, it is because the thread executing the unit tests will complete before the thread doing the background work. The unit tests won’t wait around for the task to finish by default. What we need is a way of notifying when a unit test should perform its assertion.

In order to do that we need a way of blocking the unit test from completing.

There are two distinct scenarios here:
1. The method we are testing directly returns a `Task`
2. The method we are testing uses a `Task` under the hood, but doesn't expose it.

### Testing methods returning a Task
This is the easiest case. Let’s consider the following method:

```java
public Task<Boolean> doWorkTask() {
    return Task.callInBackground(() -> {

        // simulate doing 'hard' work
        Thread.sleep(1000);
        return true;

    });
}
```

This method returns immediately with a `Task` which runs a background thread for 1 second before eventually having a result of true.

To unit test this, a naive approach would be to try:

```java
// THIS TEST WILL FAIL ❌
@Test
public void directTaskAccess() {
    Task<Boolean> task = new Worker().doWorkTask();
    assertTrue(task.getResult());
}
```

This will fail with a `NullPointerException` as `task.getResult()` isn’t populated yet (and won’t be for another ~1 second).

A simple approach for handling this when you have direct access to the `Task` object, like in this case, is to use the method `waitForCompletion()` method, which blocks until the `Task` completes.

```java
// THIS TEST WILL PASS ✅
@Test
public void directTaskAccess() throws InterruptedException {
    Task<Boolean> task = new Worker().doWorkTask();
    task.waitForCompletion();
    assertTrue(task.getResult());
}
```

### Testing a method which uses a background Task but doesn't return it
Let’s get more complicated then. How do you manage the situation whereby you don’t have access to the `Task` directly from the unit tests? For example, consider the following setup:

```java
public void doWorkCallback(final WorkCompletionListener callback) {
    Task.call((Callable<Void>) () -> {

        // simulate doing 'hard' work
        Thread.sleep(1000);

        if (callback != null) {
            callback.finished(true);
        }
        return null;
    }, Task.BACKGROUND_EXECUTOR);
}
public interface WorkCompletionListener {
    void finished(boolean successful);
}
```

This time around, we aren’t exposing the `Task` directly, but instead, inside the method we are testing, we utilise an asynchronous `Task` and return the result later using a provided callback.

Our naive attempt this time might be to try this:

```java
// WARNING: THIS WILL PASS BUT THE ASSERTION ISN'T CHECKED! ❌
@Test
public void noDirectTaskAccess() throws InterruptedException {
    new Worker().doWorkCallback(Assert::assertTrue);
}
```

By the way, `Assert::assertTrue` is the same as writing the following but uses `Retrolambda` to make it much more succinct.

```java
new Worker().doWorkCallback(new Worker.WorkerCompletionListener() {
    @Override
    public void finished(boolean successful) {
        assertTrue(successful);
    }
});
```

The problem here is that the unit testing is finishing without an issue but our assertTrue never even runs. Go ahead and change the assertion to check for false instead of true; still passes!

The solution this time is to set a custom executor for unit testing that doesn't actually run in the background; instead we use an executor which runs immediately.

```java
public class Worker {

    @VisibleForTesting
    Executor bgExecutor = Task.BACKGROUND_EXECUTOR;

    public void doWorkCallback(final WorkerCompletionListener listener) {
        Task.call((Callable<Void>) () -> {

            // simulate doing 'hard' work
            Thread.sleep(1000);

            if (listener != null) {
                listener.finished(true);
            }
            return null;
        }, bgExecutor);
    }
}
```

This code is functionally the same as before, but this time we use a variable which points to the executor to use, defaulting to `Task.BACKGROUND_EXECUTOR` as before. Now, though, we can set that variable during testing to an immediately executing executor. What does that look like?

#### with Retrolambda
```java
Runnable::run;
```

#### without Retrolambda (same as previous; just more verbose)
```java
new Executor() {
    @Override
    public void execute(Runnable command) {
        command.run();
    }
};
```

### Finishing off
```java
@Test
public void noDirectTaskAccess() throws InterruptedException {
    Worker worker = new Worker();
    worker.bgExecutor = Runnable::run;
    worker.doWorkCallback(Assert::assertTrue);
}
```

Success! ✅ 

For the extra paranoid, ensure it fails when you `Assert::assertFalse` and passes when you use `Assert:assertTrue`.