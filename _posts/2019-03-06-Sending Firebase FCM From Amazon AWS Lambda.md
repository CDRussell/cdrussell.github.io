---
layout: post
title: "Send Firebase FCM Push Notification from AWS Lambda"
date: 2019-03-06T23:50:21Z
description: An example of how to configure an Amazon AWS Lambda to send a push notification to Firebase FCM, written with the Node.js 8.10 runtime.
permalink: 2019/03/send-firebase-fcm-push-notification-from-aws-lambda/
keywords:
  - FCM
  - Firebase
  - Firebase cloud messaging
  - AWS Lambda
  - Amazon Lambda
  - Node.js
  - Push notification
  - Send push
  - Trigger push
  - Send message
images:
- /images/firebase-lambda.png
featured_image: /images/firebase-lambda.png
list_image: /images/firebase-lambda.png
---

*This posts provides an example of how to configure an `Amazon AWS Lambda` to send a push notification to `Firebase FCM`, written with the `Node.js 8.10` runtime.*

## What is AWS Lambda
This post assumes you are familiar with AWS Lambda already. If not, you can read more about it staight from [the docs](https://aws.amazon.com/lambda/).

## Configure the lambda
Create a new AWS lambda using the AWS Console. Configure these two options:

```shell
    Runtime: Node.js 8.10
    Handler: index.handler

    // 'index' refers to the file you are about to create
    // 'handler' refers to the function you are about to define
```

Create a single file called `index.js`, and in it, paste the following:


```js
// index.js

const http = require('http');

exports.handler = async(event, context) => {
    return new Promise((resolve, reject) => {
       const options = {
           host: "fcm.googleapis.com",
           path: "/fcm/send",
           method: 'POST',
           headers: {
               'Authorization': process.env.authorization,
               'Content-Type': 'application/json'
           }
       };
       
       const req = http.request(options, (res) => {
          resolve('success') ;
       });
       
       req.on('error', (e) => {
           reject(e.message);
       });
       
       const reqBody = '{"to":"' + process.env.deviceToken + '", "priority" : "high"}'
       
       req.write(reqBody);
       req.end();
    });
}

```

## Environment Variables
Note the two highlighted lines above: they both make use of `process.env` to access environment variables.

This means you'll need to configure the following environment variables for the lamba to be authorized to send a Firebase push notification.

`authorization`

*This is the authorization header you receive from your Firebase account console.*

It will likely have the form `'key=ABCDEFG....'` and you need to ensure you keep the `key=` prefix.

`deviceToken`

*This is the device token you receive from the device that will receive your Firebase push notification. You'll receive this directly on the device as it registers for FCM.*

## Done
That's it. You should be able to save that and hit the `test` button. When you do, your AWS Lambda will execute and send an HTTP POST request to Firebase FCM, triggering a push notification to your intended device.