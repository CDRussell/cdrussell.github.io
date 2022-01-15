---
title: 'Android MVP: Keeping Presenters Alive'
description: >-
  This article outlines a solution for allowing Presenters to survive a View’s
  configuration change in an MVP architecture.
date: '2016-08-15T00:02:31.722Z'
categories: []
keywords: []
layout: post
slug: /@trionkidnapper/android-mvp-keeping-presenters-alive-a91b9e080761
---
_This article outlines a solution for allowing Presenters to survive a View’s configuration change in an MVP architecture_

# Allowing a Presenter to Survive Activity configuration changes

<img
    src="/images/mvp-surviving-presenters-turn-to-clear.jpeg"
    alt=""
    width="100%"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

## I’m in a Hurry — Show Me:

### The Solution

`onRetainCustomNonConfigurationInstance()`   
`getLastCustomNonConfigurationInstance()`

Use these methods to store, and later retrieve, any object you want to survive an Activity’s configuration change; perfect for a Presenter in an app using an MVP architecture.

### The Source Code

The source code for a sample project showing this in action is available on GitHub: [https://github.com/CDRussell/SurvivingPresenters](https://github.com/CDRussell/SurvivingPresenters)

You’ll find a full explanation of the source below.

# About MVP

The Model View Presenter (MVP) design pattern dictates that you separate your presentation (the View) from the business logic and the data sources (the Presenter and the Model). Translated into Android terminology, this means having no logic in your Activity and making these as lightweight as possible. The Activity remains passive and is instructed on what to do and when to do it by the Presenter; also known as a Passive View design.

Android views, however, are complicated and have lifecycles that need carefully managed. We know that in Android if your Views are Activities then these will be destroyed and rebuilt on configuration changes (like when the user rotates the device).

If you have a long running operation which is a concern of your Presenter and Model, like posting data to a server or saving data into a DB, how do you keep that executing whilst your View is being destroyed and rebuilt?

## Best Practice

If you’ve been reading up on MVP, you might have noticed that there are many opinions on how it should be implemented. There is no absolute right or wrong here, and whilst that provides enough flexibility to find something right for your project, it can be daunting not to have a _best practice_ implementation to use as a reference.

I therefore present to you, if I may be so bold, a _best practice_ for allowing the Presenter to outlive the View. We don’t just want the Presenter to live forever though. We want the Presenter to survive configuration changes whilst the view is being destroyed and rebuilt. But we also want the Presenter itself to be destroyed when the user has no more need of that View.

In other words, if the user rotates their device, we want the Presenter to stick around. But if the user presses the back button and the Activity is destroyed _for good_, we want the Presenter to be destroyed here too.

## Isn’t There a Library For  That?

Libraries and frameworks have been created to solve this solution. If you are interested in using one, you can check out [Nucleus](https://github.com/konmik/nucleus) or [Mosby](https://github.com/sockeqwe/mosby). However, my reservation with libraries/frameworks like these is that you must place a lot of faith in them to use them. Not only do you have to trust they play nice with a typical Android project setup today, you must trust that those developers will keep the project compatible tomorrow, and the next day. In other words, it is 3rd party magic that is crucial to the project. You might be ok with that.

When the use of a library will be pervasive throughout my project (every Activity will depend on it) I don’t want there to be magic involved. I want to know how to solve the problem myself in case the magician were ever to… disappear. At some point along the journey to understanding 3rd party libraries, you have enough knowledge to solve the problem yourself and keep it simple in doing so.

## The Solution

The solution lies in the combination of two simple methods:

```java
public Object onRetainCustomNonConfigurationInstance()  
public Object getLastCustomNonConfigurationInstance ()
```

[onRetainCustomNonConfigurationInstance](https://developer.android.com/reference/android/support/v4/app/FragmentActivity.html#onRetainCustomNonConfigurationInstance%28%29)(), defined in [FragmentActivity](https://developer.android.com/reference/android/support/v4/app/FragmentActivity.html), is provided as part of [Android Support Library](https://developer.android.com/topic/libraries/support-library/index.html). New apps created in recent versions of Android Studio default to using AppCompatActivity as a subclass for all your defined Activities. AppCompatActivity in turn inherits from [FragmentActivity](https://developer.android.com/reference/android/support/v4/app/FragmentActivity.html).

In other words, chances are high that you already have access to this method.

### Saving an  Object

The purpose of [onRetainCustomNonConfigurationInstance()](https://developer.android.com/reference/android/support/v4/app/FragmentActivity.html#onRetainCustomNonConfigurationInstance%28%29) is to allow your Activity the ability to return a single object which can be re-used across configuration changes. This object will not serialised and deserialised, nor will it be destroyed and recreated; this object will _survive_ the configuration change altogether. The exact same instance of the object that was used by the old View can be re-attached to the new View.

This is the perfect mechanism for storing our Presenter in MVP; when a View (Activity) is being destroyed due to a configuration change, we can return the Presenter instance from the [onRetainCustomNonConfigurationInstance()](https://developer.android.com/reference/android/support/v4/app/FragmentActivity.html#onRetainCustomNonConfigurationInstance%28%29) method and when the new View has been recreated, we can re-attach the Presenter.

### Retrieving a Saved Object

To retrieve the object stored by [onRetainCustomNonConfigurationInstance](https://developer.android.com/reference/android/support/v4/app/FragmentActivity.html#onRetainCustomNonConfigurationInstance%28%29)() we use its opposite

`getLastCustomNonConfigurationInstance();`

[getLastCustomNonConfigurationInstance()](https://developer.android.com/reference/android/support/v4/app/FragmentActivity.html#getLastCustomNonConfigurationInstance%28%29) returns the value stored using [onRetainCustomNonConfigurationInstance](https://developer.android.com/reference/android/support/v4/app/FragmentActivity.html#onRetainCustomNonConfigurationInstance%28%29)(), or null if nothing was stored.

As such, to meaningfully use the value returned from this method, we need to check to ensure it isn’t null and cast it to the right type of object; you’ll know the type you need to use as it has to match the object saved from [onRetainCustomNonConfigurationInstance](https://developer.android.com/reference/android/support/v4/app/FragmentActivity.html#onRetainCustomNonConfigurationInstance%28%29)().

### Sample Code

The sample project is available on GitHub: [https://github.com/CDRussell/SurvivingPresenters](https://github.com/CDRussell/SurvivingPresenters)

In this sample, we have a simple Activity which contains two buttons, one to increment a counter and one to decrement the counter. We have a TextView which is used to display the current count.

<img
    src="/images/mvp-surviving-presenters-activity-view.png"
    alt="activity view showing a number labeled counter, and buttons to increment and decrements"
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>
We use the MVP pattern to separate concerns. The Presenter is in charge of holding the current count and changing its value and is able to instruct the view to show the user a new value. The View is in charge of responding to a user clicking on either the increment or decrement button and informing the Presenter about it.

We use Paul Blundell’s excellent suggested structure of the required MVP classes ([https://www.novoda.com/blog/better-class-naming/](https://www.novoda.com/blog/better-class-naming/)) to define an interface as follows:

```java

interface IntroMvp {  
  
    interface View {  
  
        void updateCount(int count);  
    }  
  
    interface Presenter {  
  
        void attachView(View view);  
  
        void detachView();  
  
        void incrementValue();  
  
        void decrementValue();  
    }  
}

```

Note, the Model is omitted here for brevity.

Our Activity must implement our IntroMvp.View interface and extend from AppCompatActivity

```java
public class IntroActivity extends AppCompatActivity implements IntroMvp.View
```

The Activity holds a reference to an instance of the IntroMvp.Presenter which it attaches to in the onCreate() method.

```java
private IntroMvp.Presenter presenter;

@Override  
protected void onCreate(Bundle savedInstanceState) {  
    super.onCreate(savedInstanceState);  
    setContentView(R.layout._activity\_intro_);  
    attachPresenter();  
}

private void attachPresenter() {  
    presenter = (IntroMvp.Presenter) getLastCustomNonConfigurationInstance();  
    if (presenter == null) {  
        presenter = new IntroPresenter();  
    }  
    presenter.attachView(this);  
}
```

We retrieve an instance from the getLastCustomNonConfigurationInstance() method if it exists. If it doesn’t exist, we need to create an instance of the Presenter.

Finally we call presenter.attachView passing _this_ which allows the Presenter to gain access to the newly created View.

```java

@Override  
protected void onDestroy() {  
    presenter.detachView();  
    super.onDestroy();  
}  
  
@Override  
public Object onRetainCustomNonConfigurationInstance() {  
    return presenter;  
}

```

The system will automatically invoke `onRetainCustomNonConfigurationInstance()` at the correct time and ask you which object you want to survive the current configuration change. You simply need to return the reference to the Presenter.

When the View is being destroyed, we inform the Presenter so that it doesn’t try to update an invalid View.

The rest of the Activity is simply handling the user pressing buttons and updating the view when told to by the Presenter.

```java

public void incrementButtonPressed(View view) {  
    presenter.incrementValue();  
}  
  
public void decrementButtonPressed(View view) {  
    presenter.decrementValue();  
}  
  
@Override  
public void updateCount(final int count) {  
    runOnUiThread(new Runnable() {  
        @Override  
        public void run() {  
            output.setText(getString(R.string._updated\_count_, count));  
        }  
    });  
}
```

Running the sample app, you can see that the Presenter continues to exist and hold the current value of the counter, even after the device is rotated. Additionally, if you hit the back button to properly destroy the Activity and relaunch it, the Presenter’s count will be reset to 0 as we’re dealing with a brand new instance of it in this scenario.

### Summary

There are a large number of words in this article, but they all lead to one simple point. Use

```java
onRetainCustomNonConfigurationInstance()   
getLastCustomNonConfigurationInstance()
```

to store, and later retrieve, any object you want to survive an Activity’s configuration change; perfect for a Presenter in an app using an MVP architecture.