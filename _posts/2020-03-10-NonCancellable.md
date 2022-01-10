---
layout: post
title: "Preventing coroutine cancellation for important actions"
description: A pattern for launching coroutines which cancel when the Activity or ViewModel is destroyed, but support allowing important parts of the coroutine to run uncancelled.

date: 2020-03-11
permalink: 2020/03/preventing-coroutine-cancellation-for-important-actions/

list_image: /images/keepcalm.jpg

categories:
  - Android
---

*This post describes a pattern of launching coroutines which cancel when the Activity or ViewModel is destroyed, but support allowing important parts of the coroutine to run uncancelled.*

# Overview
When you have an `Activity` and a `ViewModel`, you need to pay attention to the scopes in which you launch coroutines. Launching a coroutine from an `Activity` will result in it having a different lifecycle than if launched from `ViewModel`. And what happens if you have async work that you _don't_ want to cancel if the `Activity` or `ViewModel` are destroyed?

<img src="/images/keepcalm.jpg" style="width:300px; display:block; margin-left: auto; margin-right: auto;"  />

# Existing architecture
In our codebase, we typically marked our `ViewModel` functions with `suspend` when they would involve asynchronous work, and launched the coroutine from the `Activity`. Part of the rationale for this was that it made testing much easier. We knew [how to write a test for a suspend function](https://craigrussell.io/2019/11/unit-testing-coroutine-suspend-functions-using-testcoroutinedispatcher/).

Let's consider this example; we have the `TabSwitcherActivity` which is used to show users their open tabs and let them switch, add, and close tabs. 

<img src="/images/tab-switcher.png" alt="tab switcher view from the DuckDuckGo Privacy Browser" style="width:300px; display:block; margin-left: auto; margin-right: auto;" />

For brevity, let's just focus on the _close tab_ functionality.

```kotlin
class TabSwitcherActivity : CoroutineScope by MainScope() {
 
  fun onTabDeleted(tab: TabEntity) {
    launch { viewModel.onTabDeleted(tab) }
  }
  
}

class TabSwitcherViewModel {

   suspend fun onTabDeleted(tab: TabEntity) {
      tabRepository.delete(tab)
   }

}

class TabRepository {

   suspend fun delete(tab: TabEntity) {
      deleteOldPreviewImages(tab.tabId)
      tabsDao.deleteTabAndUpdateSelection(tab)
   }

}
```

## Key points to note

- User chooses to delete a tab from the `Activity`
- `Activity` launches a coroutine, and calls delete method on `ViewModel`
- `ViewModel` calls delete method on `TabRepository`
- `TabRepository` deletes the tab from the database

## Lifecycle problems
There is a problem looming here. The reason we are using coroutines at all in this flow is because deleting from the database is an asynchronous operation; it might not happen immediately since it involves disk IO. 

What happens if the user navigates away from the `Activity` immediately after hitting the delete button but before the tab is deleted? Consider this sequence 👇

<img src="/images/coroutine-cancellation-sad-scenario.svg" />

If the Activity is destroyed, the coroutine will be cancelled and we might fail to honor the user's decision to delete the tab. This is dangerous! ⚠️ If the user thinks the tab will be deleted, we can't disregard that just because the user hit the back button or rotated their phone.

# Solving the lifecyle problem
1. We could choose to **not cancel** the coroutine. 
  - This could cause leaks and crashes as code executes on an `Activity` which is now destroyed. 
  - This is bad ❌
2. We could choose to launch a coroutine from the `ViewModel` instead of the `Activity`. 
   - This moves the coroutine to a longer-lived component, but we haven't really solved the problem; merely kicked the can further down the road. 
   - If the user navigates back from the tab switcher `Activity`, the `Activity`, `ViewModel` and coroutine will all be destroyed and cancelled and we'll still fail to honor the user's action. 
   - This is bad ❌
3. We could use `GlobalScope.launch` from the `Repository`. 
   - This would ensure the deletion takes place
   - However, we've now disregarded structured concurrency and won't be able to tell when the operation has finished from the calling code. If we wanted to show a `Snackbar` when the tab was deleted for instance, it would be trickier to achieve with this approach.
   - This is little better than creating a new `Thread` and firing that off to do the deletion. ❌
4. In the `Repository`, we can use `NonCancellable` to ensure the deletion cannot be cancelled once started. ✅

## Introducing - `NonCancellable` ✨
>A non-cancelable job that is always active. It is designed for `withContext` function to prevent cancellation of code blocks that need to be executed without cancellation.

Well now, that sounds pretty handy for our scenario. Let's explore [NonCancellable](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-non-cancellable.html) further, and see what the code looks like for our `TabRepository`. As it turns out, it **requires just a tiny change** to our `TabRepository` to get this new behavior.

```kotlin
suspend fun delete(tab: TabEntity) {
  withContext(Dispatchers.IO + NonCancellable) {
      deleteTabImagePreview(tab)
      tabsDao.deleteTab(tab)
  }
}
```

# Summary
Within a coroutine, we can mark a block as `NonCancellable` to ensure it runs to completion once started.

- If the `Activity` which launched the coroutine is still around, it'll be able to take action to confirm deletion with the user (e.g., show a `Snackbar`) 
- If the `Activity` is destroyed, the coroutine it launched should be cancelled too. However, with `NonCancellable`, we ensure that the tab will be deleted (as long as the app itself isn't killed)
- If the `Activity` is already dead, nothing in the coroutine after the block marked with `NonCancellable` will execute; the refusal to be stopped applies only to code within the block. 
- It's all just as testable as before 😌