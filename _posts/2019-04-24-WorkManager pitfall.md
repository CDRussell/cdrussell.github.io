---
layout: post
title: "A WorkManager Pitfall: Modifying a Scheduled Worker"
date: 2019-04-24T00:59:50+01:00
description: A potential WorkManager pitfall when refactoring or deleting your Worker subclasses.
keywords:
  - Android
  - WorkManager
  - ClassNotFoundException
  - WorkerFactory
  - deleting worker
  - refactoring worker

images:
- /images/factory.jpg

permalink: 2019/04/a-workmanager-pitfall-modifying-a-scheduled-worker/
---
*This post describes a potential `WorkManager` pitfall when refactoring or deleting your `Worker` subclasses.*

## Overview
When using [`WorkManager`](https://developer.android.com/topic/libraries/architecture/workmanager/), you define a `Worker` subclass, optionally add some constraints and enqueue it with `WorkManager` so that your work will happen sometime later.

### How bad things might happen
But what happens if:

1. you schedule a job for sometime in the future
2. before the job runs, you update your app to rename, move or delete the `Worker` class?

`WorkManager` will try to create the `Worker` you told it to use but will be unable to find that class.

### Why would anyone do that?
Perhaps your worker was part of an experimental feature you were building, but later decided to pull. Perhaps you refactored a worker to have a different name and/or package.

### Isn't that a really silly thing to do?
Refactoring most code shouldn't be dangerous and if you are never going to need a `Worker` again, there's a downside to leaving unused code in your APK. So refactoring or removing a `Worker` isn't a silly mistake; it might genuinely be something you need to do.

This is a scenario that might bite you. This post details what happens in this scenario and why, and offers particular advice for handling this situation when using a custom `WorkerFactory`.

### Scheduling work to happen
You tell `WorkManager` which `Worker` should run as part of building the work request.

```kotlin

val workRequest = OneTimeWorkRequestBuilder<MyWorker>().build()
WorkManager.getInstance().enqueue(workRequest)

```

At the stage of enqueuing the work, an instance of your `Worker` **is not yet** created. It won't be created until `WorkManager` decides your work should run.

### When it's time to work
When the time comes for your work to run, an instance of your `Worker` will be instantiated. By default, this will be done by [WorkerFactory](https://developer.android.com/reference/androidx/work/WorkerFactory#createWorker(android.content.Context,%2520java.lang.String,%2520androidx.work.WorkerParameters)).

The problem is that if you have removed the `Worker` subclass from your APK (or moved/renamed it), then the `WorkerFactory` will be handed the class name as it was when you scheduled the work, and asked to create a new instance of it. This will fail.

### Example
**Monday**

  - You release v1 of your app. 
  - You have a worker called `com.foo.MyWorker` 
  - You schedule work for that worker to happen in 2 days' time

**Tuesday**
  
  - You move `MyWorker` class to `com.bar.MyWorker`
  - You release v2 of your app

**Wednesday**

  - `WorkManager` will try to create an instance of `com.foo.MyWorker` which doesn't exist.

Another scenario is if instead of moving `com.foo.MyWorker`, you delete it altogether.

In this case, you might be surprised to see `WorkManager` still attempting to create an instance of `com.foo.MyWorker`.
  
#### What will happen?
Your job won't run as expected. How major a bug that is will depend on your app and what the job was supposed to do.

As well as not running, there might be another consequence: the app might crash. The default `WorkerFactory` will handle this scenario safely enough to not crash, but the job still won't run.

If you are using a custom `WorkerFactory`, then you will have to ensure you are creating `Worker` instances in a safe way that is resilient to these types of changes.

### Custom WorkerFactory
In some use cases you might find yourself registering a [custom `WorkerFactory` implementation](https://stackoverflow.com/a/53377279/1654145). This is particularly useful if you want to use `Dagger` to inject objects into your `Worker` subclass.

#### Naive Implementation of CustomWorkerFactory

```kotlin

⚠️ This implementations has flaws ⚠️

class CustomWorkerFactory : WorkerFactory() {

  override fun createWorker(appContext: Context, workerClassName: String, workerParameters: WorkerParameters): ListenableWorker? {
    val workerClass = Class.forName(workerClassName).asSubclass(ListenableWorker::class.java)
    val constructor = workerClass.getDeclaredConstructor(Context::class.java, WorkerParameters::class.java)
    return constructor.newInstance(appContext, workerParameters)
  }

}

```

What's the problem here? As in the above example, if you originally enqueued a work request using `com.foo.MyWorker` then that will be passed into the `createWorker()` function as the `workerClassName: String` parameter.

In the naive implementation above, `com.foo.MyWorker` no longer exists, and attempts to find the class for it will result in a `ClassNotFoundException` being thrown.

#### Safer Implementation of CustomWorkerFactory
We need to wrap the attempts at finding the class in a `try/catch` block, ensuring we catch `ClassNotFoundException` since we now know how and why that might happen.

```kotlin
class CustomWorkerFactory : WorkerFactory() {

  override fun createWorker(appContext: Context, workerClassName: String, workerParameters: WorkerParameters): ListenableWorker? {
    try {
        val workerClass = Class.forName(workerClassName).asSubclass(ListenableWorker::class.java)
        val constructor = workerClass.getDeclaredConstructor(Context::class.java, WorkerParameters::class.java)
        return constructor.newInstance(appContext, workerParameters)
    } catch (e: ClassNotFoundException) {
        return null
    }
  }

}

```

After catching the `ClassNotFoundException`, we return `null`. 

As per the documentation, you should return `null` if the worker could not be created.