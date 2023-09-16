---
slug: '2022-04-05-Notification-permissions-android-13'
title: 'Notification permissions in Android 13'
date: 2022-04-05 21:30 +1100
categories: 'blog'
---

# Android 13 Preview 2

Last weekend I finally got some time to have a play with the Android 13 emulator and I have to admit,
its a bit rough around the edges.

For example all the icons in the launcher are humongo 😮

![Android 13 icons](/images/android_13_icons.png)

Anyway nothing ventured, nothing gained I figured.

# Notifications and runtime permissions.

The main goal I had in mind when I downloaded the new emulator was checking out the shiny new `POST_NOTIFICATIONS`
permission being introduced in Android 13.

So, what is it?

Well, iOS has had it for a while and essentially it is a permission prompt for Android apps targeting Android 13 (SDK level 33 and above)
asking the user whether they want to receieve notifications. I think this is a great win not only for user experience, but also a win from
the privacy perspective.

Anyway, here's how I went about testing it 📝

- Downloading Android 13 (Codename Tiramisum) from the SDK manager.
- In my app's `build.gradle` changing the `compileSdkVersion` to  `android-Tiramisu` and `targetSdkVersion` to `Tiramisu`.
- Downloading Android Studio Dolphin Canary 7.
- Downloading Android build tools version `33.0.0 rc2` and updating the `buildToolsVersion` in my app's `build.gradle` to `33.0.0-rc2`
- Downloading the emulator `arm64-v8a` image for `Android Tiramisu` in the emulator creation wizard in Android Studio.


Ok, now we're cooking with gas 🔥

Because this is a runtime permission, you also need to follow the general steps 📝
- Call the `checkSelfPermission` API for `POST_NOTIFICATIONS`,
- If this does not return `PERMISSION_GRANTED`,
- Then check if the permission rationale (ie. why you need this permission) dialog.
- If it returns false, you can call `requestPermissions` for `POST_NOTIFICATIONS` directly.
- If in the the permission rationale dialog, the user clicks the `positiveButton`, call `requestPermissions` for `POST_NOTIFICATIONS`.

unfortunately, checking for `SDK_INT >= 33` does not work currently, so I had to make do with calling `SDK_INT >= 32` instead.

```
        if (Build.VERSION.SDK_INT >= 32) {
            String[] permission = {POST_NOTIFICATIONS};
            if (checkSelfPermission(POST_NOTIFICATIONS) != PackageManager.PERMISSION_GRANTED) {
                if (!shouldShowRequestPermissionRationale(POST_NOTIFICATIONS)) {
                    requestPermissions(permission, 1001);
                } else {
                    showNotificationRationale();
                }
            }
        }
```

# Here's what it looks like 👌
<p align="center">
    <video width="320" controls autoplay loop muted src="/images/notification_permissions.webm" type="video/webm" />
</p>