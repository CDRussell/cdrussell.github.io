---
layout: post
title: Android WebView — Downloading Images
date: 2018-02-18 21:15 +0000
description: How to download an image from a WebView triggered by the user long pressing the image
---

_This blog post details how to download an image from a WebView which is triggered by the user long pressing on the image._

# Contents
There are a few parts of the problem needing solved
1. Capturing the long press event
2. Identifying if the user has long pressed on an image
3. Naming the image
4. Downloading the image

## Capturing the Long Press Event in a WebView
To capture a long press event on a WebView, you need to handle the overridable method in your `Activity` subclass called `onCreateContextMenu`.

```kotlin
override fun onCreateContextMenu(
    menu: ContextMenu, view: View, menuInfo: ContextMenu.ContextMenuInfo?
) { }
```

This method will be invoked upon the user long pressing on something within the `WebView`.

## Identifying if the User has Selected an Image
Now that we have a method which is invoked on a long press, we need to determine what the user has actually long pressed on.

```kotlin
 override fun onCreateContextMenu(menu: ContextMenu, view: View, menuInfo: ContextMenu.ContextMenuInfo?) {
        webView.hitTestResult?.let {
            when (it.type) {
                WebView.HitTestResult.IMAGE_TYPE,
                WebView.HitTestResult.SRC_IMAGE_ANCHOR_TYPE -> {
                    menu.setHeaderTitle(R.string.imageOptions)
                    menu.add(0, CONTEXT_MENU_ID_DOWNLOAD_IMAGE, 0, R.string.downloadImage)
                }
                else -> Timber.v("App does not yet handle target type: ${it.type}")
            }
        }
    }
```
_`CONTEXT_MENU_ID_DOWNLOAD_IMAGE` is just an int — you will use this soon to determine which menu item the user selected from the context menu_

We can obtain the `HitTestResult` from the `WebView` noting that it can be null. We interrogate the type of result to ensure the user has long pressed on an image by limiting hit types to only:
- `WebView.HitTestResult.IMAGE_TYPE`
- `WebView.HitTestResult.SRC_IMAGE_ANCHOR_TYPE`

If it is either of these types, we add a menu item to the context menu to give the user the option of downloading the image or not.

## Network Requests Only
You might want to add one additional check here to ensure you are dealing with an http or https request.

We can use our good friend `URLUtil` to help with that. The method `URLUtil.isNetworkUrl(string)` will return true if the URL begins `http://` or `https://`.

If it isn’t a network URL, we can refuse to add the download image option to the context menu.

<img
    src="/images/webview-download-context-menu.png"
    alt="Android emulator showing a context menu floating over a WebView with a single selectable option: download image"
    width="500"
    style="display: block; margin-left: auto; margin-right: auto;"
/>

We’ve now added a context menu which lets the user choose to download the image, and so we have to handle the user selecting that context menu item.

```kotlin
override fun onContextItemSelected(item: MenuItem): Boolean {
    
    webView.hitTestResult?.let {
        val url = it.extra

        if (CONTEXT_MENU_ID_DOWNLOAD_IMAGE == item.itemId) {
            pendingFileDownload = PendingFileDownload(url, Environment.DIRECTORY_PICTURES)
            downloadFileWithPermissionCheck()
            return true
        }
    }

    return super.onContextItemSelected(item)
}
```

By overriding the `onContextItemSelected` menu, we can respond to the user selecting our "Download Image" option from the context menu, using `CONTEXT_MENU_ID_DOWNLOAD_IMAGE` that we defined above.

## Runtime Permissions
Can’t forget about those pesky runtime permissions. If you’re planning to download the image to somewhere like `Environment.DIRECTORY_PICTURES` in the external storage, you’ll need the user’s permission.

For my app, there’s a chance I haven’t asked for this permission before and so I have to keep a variable which encapsulates what the user is about to download, before possibly segueing down the permission request/response flow. I encapsulate this intention in the `pendingFileDownload` variable above.

```kotlin
private fun downloadFileWithPermissionCheck() {
     if (hasWriteStoragePermission()) {
         downloadFile()
     } else {
         requestStoragePermission()
     }
}
```

If I already have the required permissions, I go ahead and instigate the download. Otherwise, I have to ask for the permission first and only upon being granted the permission can I then retrieve the `pendingFileDownload` and attempt the download.

## Naming the File
In testing, I found that some images on the web downloaded with a sensible file name and others did not. As such, I would recommend manually applying a filename to your download if you cannot fully control which images will be downloaded.

```kotlin
val guessedFileName = URLUtil.guessFileName(pending.url, null, null)
```

It would be great if there was a utility method which took a String of a URL and tried to guess a suitable filename for it. Oh hello `URLUtil.guessFileName` 👋. Glad you could join us. This provides a good guess for the name of the image we are downloading.


## Downloading the File
Now we know that the user has long pressed on an image, we know the location of that image, and we have a sensible filename to use when downloading the image, the only thing left to do is ... actually download the image.

One way of doing that is to use the built-in Android `DownloadManager`.

```kotlin
private fun downloadFile() {
    val pending = pendingFileDownload
    pending?.let {
        val uri = Uri.parse(pending.url)
        val guessedFileName = URLUtil.guessFileName(pending.url, null, null)
        Timber.i("Guessed filename of $guessedFileName for url ${pending.url}")
        val request = DownloadManager.Request(uri).apply {
            allowScanningByMediaScanner()
            setDestinationInExternalPublicDir(pending.directory, guessedFileName)
            setNotificationVisibility(VISIBILITY_VISIBLE_NOTIFY_COMPLETED)
        }
        val manager = getSystemService(Context.DOWNLOAD_SERVICE) as DownloadManager
        manager.enqueue(request)
        pendingFileDownload = null
    }
}
```

After guessing a filename, we can build a `DownloadManager.Request` object which encapsulates what we want to download, and how it should be downloaded.

We pass the request onto the `DownloadManager` using its `enqueue()` method, and then finally set the `pendingFileDownload` to null to indicate we have handled it.
