---
layout: post
title: Representing View State with Kotlin Data Classes
date: 2017-11-30
description: Using Kotlin data classes to represent view state
categories: 
- Android
---

# Representing View State with Kotlin Data Classes
<img
    src="/images/kotlin-logo.png"
    alt="kotlin logo"
    width="100%"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

_This article describes how to use Kotlin’s [data classes](https://kotlinlang.org/docs/reference/data-classes.html) to represent the view state of an [Android Architectures Component](https://developer.android.com/topic/libraries/architecture/index.html) ViewModel_

## Background
Using a [`ViewModel`](https://developer.android.com/topic/libraries/architecture/viewmodel.html), we can represent the current state of the UI and pass this to an Activity which can render it. Using this architecture, an Activity will react to changes in the view state as they happen.

Since this view state is typically only changed by the `ViewModel`, we can make it immutable as far as the Activity is concerned.

## Representing the UI State
We can define the UI state as a Kotlin data class within our `ViewModel`; we take this as a convenient place to add default values for each of the fields.

```kotlin
data class ViewState(
    val isLoading: Boolean = false,
    val progress: Int = 0,
    val url: String? = null,
    val isEditing: Boolean = false,
    val browserShowing: Boolean = false,
    val showClearButton: Boolean = false
)
```

## Exposing observable state
Once defined like this, we can expose a [LiveData](https://developer.android.com/topic/libraries/architecture/livedata.html)<ViewState> to which an Activity can observe changes.

```kotlin
val viewState: MutableLiveData<ViewState> = MutableLiveData()
init {
    viewState.value = ViewState()
}

private fun currentViewState(): ViewState = viewState.value!!
```

Above, we define the `viewState` value which is observable, and we push a default value to it using the `init` block. For convenience, we also define a private function that can be used in the `ViewModel` to obtain the current view state at any time.

Technically `viewState.value` is nullable, even though we know it won’t be since we’ve provided a default value upon creating, so this function saves the rest of the class from thinking about `viewState.value` from ever being null. We need to use the ugly `!!` syntax here, but it’s isolated to this one function and saves the rest of the view model from needing to handle nulls.

## Observing UI Changes
In the Activity we can subscribe to observing any changes to this view state.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
  super.onCreate(savedInstanceState)
  setContentView(R.layout.activity_browser)
  
  viewModel.viewState.observe(this, Observer<BrowserViewModel.ViewState> {
      it?.let { render(it) }
  })
}

private fun render(viewState: BrowserViewModel.ViewState) {
  when (viewState.browserShowing) {
      true -> webView.show()
      false -> webView.hide()
  }

  when (viewState.isLoading) {
      true -> pageLoadingIndicator.show()
      false -> pageLoadingIndicator.hide()
  }

  when (viewState.showClearButton) {
      true -> showClearButton()
      false -> hideClearButton()
  }

  pageLoadingIndicator.progress = viewState.progress
}
```
Every time a change is pushed from the `ViewModel`, our Activity immediately gets notified, and renders the UI accordingly.

## Making changes to the UI state
So how do changes to the view state happen? Since we’ve defined the view state as being immutable, how do we change anything?

When the `ViewModel` wants to make a change that the Activity will render, it copies the existing view state and changes only the values it needs to, preserving the existing values for the other fields.

```kotlin
override fun progressChanged(newProgress: Int) {
    viewState.value = currentViewState().copy(progress = newProgress)
}

override fun loadingStarted() {
    viewState.value = currentViewState().copy(isLoading = true)
}

override fun loadingFinished() {
    viewState.value = currentViewState().copy(isLoading = false)
}
```

## The copy() method
To update one or more view state properties, the `ViewModel` grabs the current view state, as returned by `currentViewState()`, and uses the copy method to set a new value on one or more fields.

The copy method is provided as part of the Kotlin `data class`. When we use copy, all fields in the object remain as their original value unless you provide a new value.

It’s perfectly fine to define multiple fields that have changed in your view state as part of that copy method too, making it a convenient way of capturing changes. For example:

```kotlin
viewState.value = currentViewState().copy(isEditing = hasFocus, showClearButton = false)
```

Because `viewState` is `LiveData`, any time a new value is set using `viewState.value` the Activity will immediately be notified that the UI state has changed and it will render the new UI.

# Links
- For more source code as a reference, you can check out the [DuckDuckGo browser](https://github.com/duckduckgo/Android), which is fully open source.