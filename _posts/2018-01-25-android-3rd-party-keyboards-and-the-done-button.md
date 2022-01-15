---
layout: post
title: Android 3rd party keyboards and the done button
date: 2018-01-25 22:21 +0000
description: Exploration of a bug report about a third party keyboard not submitting when done button pressed
---
# Android 3rd party keyboards and the done button
_Capturing the done button on an Android software keyboard is hard, as each keyboard might implement this differently. This post describes an interesting bug report about a user’s third party keyboard not submitting when they hit their done button, and what I had to do to fix it._

# Capturing the done button on an Android software keyboard
Say you have an `EditText` and when the user is done typing, you’d like them to be able to use their keyboard’s done button to submit the input.

## Part #1 of the Solution — onEditorActionListener
You search around and find a post on StackOverflow that recommends setting an `onEditorActionListener` on the `EditText` which will allow you to capture the done or enter buttons on the keyboard. You test on your devices and it works well.

```kotlin
editText.setOnEditorActionListener(TextView.OnEditorActionListener { _, actionId, keyEvent ->
    if (actionId == EditorInfo.IME_ACTION_DONE || keyEvent?.keyCode == KeyEvent.KEYCODE_ENTER) {
        userEnteredQuery(editText.text.toString())
        return@OnEditorActionListener true
    }

    false
})
```

And then you actually speak to your users. So here’s what you see when you’re testing your app day to day.

<img
    src="/images/android-3p-keyboards-typing-view.png"
    alt="what you see testing the app; the stock keyboard and everything looking normal and expected"
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

And here’s what your users are actually doing! Third party keyboards… 🤣

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/vSul-JCzfXY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

The user hits the done button and instead of our app getting notified of this as an editor action, it just adds a space character to the `EditText`. Well that’s not what we want!

Android’s customisability is a blessing for power users, but it is also a curse for developers.

Digging into this particular bug report, I had to install some third party keyboards and see how each handle the done button press. It isn’t pretty.

## New Line Character — \n
It turns out that some keyboards don’t submit an editor action for the done button, but instead submit a `\n` new line character. 🙄


## Part #2 of the solution — TextWatcher
To catch this `\n` character as it arrives, we need to make use of a [`TextWatcher`](https://developer.android.com/reference/android/text/TextWatcher.html). We can hook into the `onTextChanged` method to be notified that the text is being changed, and use this an opportunity to detect if the change is actually the user typing normal characters or if the user has pressed their done button.

If we detect the `\n` character we first remove that character, as we don’t want it to be shown to the user as a space character (which is how `EditText` tries to render it), and then use this as an opportunity to submit the user’s query.

```kotlin
override fun onTextChanged(
        charSequence: CharSequence,
        start: Int,
        before: Int,
        count: Int
) {

    // some 3rd party keyboards submit \n instead of an IME action
    if (before == 0 && count == 1 && charSequence[start] == '\n') {
        editText.text.replace(start, start + 1, "")
        userEnteredQuery(editText.text.toString())
    }
    
}
```


## Final Solution
We need to use both an [`onEditorActionListener`](https://developer.android.com/reference/android/widget/TextView.OnEditorActionListener.html) and a [`TextWatcher`](https://developer.android.com/reference/android/text/TextWatcher.html). It is a bit hacky, but at least it gives us an opportunity to react to the user’s done button press.

```kotlin
    editText.setOnEditorActionListener(TextView.OnEditorActionListener { _, actionId, keyEvent ->
        if (actionId == EditorInfo.IME_ACTION_DONE || keyEvent?.keyCode == KeyEvent.KEYCODE_ENTER) {
            userEnteredQuery(editText.text.toString())
            return@OnEditorActionListener true
        }
        false 
    })

    editText.addTextChangedListener(object : TextWatcher {

        override fun onTextChanged(
            charSequence: CharSequence,
            start: Int,
            before: Int,
            count: Int
        ) {
            // some 3rd party keyboards submit \n instead of an IME action
            if (before == 0 && count == 1 && charSequence[start] == '\n') {
                editText.text.replace(start, start + 1, "")
                userEnteredQuery(editText.text.toString())
            }
        }

        override fun beforeTextChanged(s: CharSequence?, start: Int, count: Int, after: Int) {}
        override fun afterTextChanged(editable: Editable) {}

    })
```

# Update
It turns out this particular behaviour is only happening with this keyboard when the `EditText` has a `textShortMessage` set as an input type option.

```
android:inputType="textUri|textShortMessage|textNoSuggestions"
```

Removing `textShortMessage` means the done button is routed to the editor action listener as we would expect, and the special `\n` handling isn’t necessary. 🤷‍♂️

Whether that is the case for all 3rd party keyboards remains to be seen. 😅

