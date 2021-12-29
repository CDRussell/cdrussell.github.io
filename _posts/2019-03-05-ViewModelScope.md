---
layout: post
date: 2019-03-05
title: Coroutine Support in ViewModels using the new ViewModelScope Extension Property
description: This post describes how to use Coroutines in ViewModels, making use of the new ViewModelScope extension property. This allows coroutines to be cancelled automatically when the ViewModel is being cleared.
keywords:
  - kotlin
  - coroutine
  - viewModelScope
  - android architecture components
  - CoroutineScope
  - Cancel coroutine automatically ViewModel cleared
  - Cancel coroutine when activity finishes
  - Cancel coroutine onCleared()

permalink: 2019/03/coroutine-support-in-viewmodels-using-the-new-viewmodelscope-extension-property/
---

*This post describes how to use `Coroutines` in `ViewModels`, making use of the new `ViewModelScope` extension property. This allows coroutines to be cancelled automatically when the `ViewModel` is being cleared.*

# Overview
As of [lifecycle 2.1.0-alpha01](https://developer.android.com/jetpack/androidx/releases/lifecycle#2.1.0-alpha01), there is now easy support for coroutines in `ViewModels`, using an extension property called `ViewModel.viewModelScope`.

# Example
`viewModelScope` is likely best explained by starting with an example. Let's imagine we are following the standard Android Architecture Components setup, and we have a `ViewModel` which extends `androidx.lifecycle.ViewModel`

## Existing Setup

If you wanted to make use of coroutines from your `ViewModel`, you might currently do something like this:

```kotlin
class LoginViewModel : ViewModel(), CoroutineScope {

  // Define job and coroutine context  
  private val job = SupervisorJob()
  override val coroutineContext: CoroutineContext
      get() = job + Dispatchers.Main

  // cancel the job when the Activity is being destroyed for good
  override fun onCleared() {
      super.onCleared()
      job.cancel()
  }

  fun onUserDoingAThing() {
      launch {
        // get your work done
      }
  }

}

```

By doing this, you would be able to use `launch{ }` inside the `ViewModel` to run coroutines. 

By overriding the `onCleared()` function, you can cancel the coroutine job to ensure you aren't continuing to do work that is no longer necessary.

Hopefully this looks fairly familiar to you, and you largely understand why all the code is needed.

Now, let's look at the new way of doing it and let's see what code we can remove while keeping the same functionality.

##  New Setup Using ViewModelScope
```kotlin

class LoginViewModel : ViewModel() {    

  fun onUserDoingAThing() {
      viewModelScope.launch {
        // get your work done
      }
  }

}

```

Wait... where did all the code go? It just isn't needed! 🎉

### No longer need to implement `CoroutineScope`
It's no longer required that you specify that your `ViewModel` implements `CoroutineScope`. You can use `viewModelScope` which is already available inside your `ViewModel`. 

Any time you want to launch a coroutine, therefore, you can use `viewModelScope.launch { }`.

Note, by default it will run the coroutine on `Dispatchers.Main`, but you can override that as normal to use any dispatcher. For example, `viewModelScope.launch(Dispatchers.IO) { }`.

### No longer need to define a `Job`
You no longer have to create a job to use for cancellation. The latest `ViewModel` already comes equipped under the hood with a `SupervisorJob` which is likely what you want for this scenario anyway. 

Use of a `SupervisorJob` means that you can launch multiple coroutines and have it so that one failing will not affect any of the others. 

>> _A failure or cancellation of a child does not cause the supervisor job to fail and does not affect its other children_


### No longer need to override `onCleared()` function and cancel the job
Job cancellation now happens automatically when the `onCleared()` function is called. 

ℹ️ As a reminder, this **is not called** when the `Activity` is being destroyed during a configuration change, such as the user rotating their device. 

However, it is called when the `Activity` is being destroyed for good, such as when the user hits the back button to navigate away from the `Activity`.

# Looks Great! How Do I Get It?
`viewModelScope` was added in `2.1.0-alpha01` of the lifecycle extensions dependency, so make sure you have at least that version in your `build.gradle` file, and make sure your project is configured for using coroutines.

For example:

    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.1.0-alpha02"
    implementation "androidx.lifecycle:lifecycle-extensions:2.1.0-alpha02"
    annotationProcessor "androidx.lifecycle:lifecycle-compiler:2.1.0-alpha02"
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.1.1'
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.1.1'

# Summary
You can now use coroutines in your `ViewModels` with the minimum of effort. The Architecture Components libraries will now give you a very sensible default setup without writing any code at all.

Note, if your work isn't tied to a particular screen and should continue even if the user navigates away, this isn't what you should be using; you should be using something with a lifecycle that isn't tied to the screen.

Coroutines that are tied to a particular screen, however, can now be launched easily, and cancelled automatically when the screen is going away.


# Links
[AndroidX Lifecycle Release Notes](https://developer.android.com/jetpack/androidx/releases/lifecycle#2.1.0-alpha01)
  
## Thanks

_The original article was ahead of its time in suggesting a `SupervisorJob` was used under the hood; at the time of writing it was using `Job`; thanks to [@fabioCollini](https://twitter.com/fabioCollini) and [@geoffreymetais](https://twitter.com/geoffreymetais) for correcting me._

_Since publishing, it has now changed to use the `SupervisorJob` which always felt like the best choice. Thanks to [@sindrenm](https://twitter.com/sindrenm) for letting me know they'd updated it._