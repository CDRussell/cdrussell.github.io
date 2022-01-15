---
title: Disable Crashlytics Crash Reporting (Fabric) on DEBUG Builds
description: >-
  How to disable crash reporting for your BuildConfig.DEBUG
  builds when using Crashlytics, part of the Fabric tool suite.
date: '2017-07-03T12:07:43.730Z'
categories: []
keywords: []
layout: post
slug: >-
  /@trionkidnapper/disabling-crashlytics-crash-reporting-fabric-on-debug-builds-aee6fdc34a57
---

_This post details how to disable crash reporting for your `BuildConfig.DEBUG` builds when using `Crashlytics`, part of the `Fabric` tool suite_

# Disabling Crashlytics Crash Reporting on DEBUG builds

In your `Application` subclass’s `onCreate()` method, you will set up your `Crashlytics` crash reporting.

```java
public class YourApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        configureCrashReporting();
    }
    
    private void configureCrashReporting() {
        CrashlyticsCore crashlyticsCore = new CrashlyticsCore.Builder()
                .disabled(BuildConfig.DEBUG)
                .build();
        Fabric.with(this, new Crashlytics.Builder().core(crashlyticsCore).build());
    }
    
}
```

You can use `CrashlyticsCore.Builder()` to customise the crash reporting. In this case, you say that crash reporting is disabled when `BuildConfig.DEBUG` is true.

With this, there will be no more annoying crash reports which happened during your day-to-day development. Now, why this isn’t the default, I have no idea.