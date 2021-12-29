---
layout: post
date: "2019-02-15 21:14:36.058Z"
description: If you launch a coroutine using launch(Dispatchers.Main) while already
  on the main thread, will the code execute immediately?
keywords:
  - kotlin
  - coroutine
  - main thread
  - immediate
  - launch immediately
  - isDispatchNeeded
  - dispatchers
title: Launching a Kotlin Coroutine for immediate execution on the Main thread
permalink: 2019/02/launching-a-kotlin-coroutine-for-immediate-execution-on-the-main-thread/
---

If you launch a coroutine using `launch(Dispatchers.Main)` while already on the main thread, will the code execute immediately?

_No_, is the short answer. This post explains _why_.

Consider the following code. As this is the `onCreate` function of an `Activity`, we know it is running on the main thread.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    launch(Dispatchers.Main) {
        log("A")
    }

    log("B")
}
```

We expect that two things will be printed to the logs; “A” and “B”. But in which order?

**OUTPUT:**  
```
// B  
// A
```

Does it surprise you that it prints “B” first? It surprised me.

### Why This Happens

The answer lies here: [`CoroutineDispatcher.isDispatchNeeded()`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-dispatcher/is-dispatch-needed.html)

> Returns `true` if execution shall be dispatched onto another thread. The default behaviour for most dispatchers is to return `true`.

OK, that makes sense. Each dispatcher can choose if execution should happen on another thread or not. And the default is for it to be `true`. But what about a UI dispatcher like `Dispatchers.Main`? Shouldn’t it return `false` so that a coroutine is executed immediately if already on the main thread? Wouldn’t that be more efficient?

Turns out… yes, that would be more efficient but even so, no, it shouldn’t do that.

> UI dispatchers _should not_ override `isDispatchNeeded`, but leave a default implementation that returns `true`.

The `Main` dispatcher _could_ check if it was already running on the main thread and, if it were, execute the code immediately instead of queuing it up. However, as the documentation for that method highlights, this could introduce pernicious bugs which are hard to debug.

If a background thread called the code, it would execute _asynchronously_, but if it were called on the main thread it would execute _synchronously_. That’s not going to be easy to debug if something goes wrong because of it.

### If Not Immediately, When?

We can see it doesn’t execute immediately; so when exactly does it execute? Under the hood, the Main dispatcher uses a `Handler` to post a `Runnable` to the `MessageQueue`. Basically, it’ll get added to the end of the event queue.

This means it will execute _soon_, but not _immediately_. Hence why “B” gets printed before “A”.

### Executing Immediately

What if you do need it to execute immediately? For this, you can use [`Dispatchers.Main.immediate`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-main-coroutine-dispatcher/immediate.html)

> Executes coroutines immediately when it is already in the right context

In other words, if your code is called from the main thread and you use this dispatcher, it will execute immediately.

**OUTPUT**:   
```
// A  
// B
```

If this introduces subtle bugs into your code, well, you can’t blame coroutines for that. By making the default behaviour _safe_, they protect against accidentally introducing unexpected behaviour. If it’s really the behaviour you want though, you can have it, but you have to be explicit about it.

ℹ️ `Dispatchers.Main.immediate` is currently marked as _experimental._