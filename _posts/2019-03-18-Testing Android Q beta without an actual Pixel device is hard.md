---
slug: "2019-03-18-Testing Android Q beta without an actual Pixel device is hard"
title:  "Testing Android Q beta without a Pixel device is hard."
date:   2019-03-18 19:30 +1100
categories: "blog"
---

import below from './images/elbow.webm';

## Android Q Beta.

Its super easy to just [opt-in](https://www.google.com/android/beta) into the Android Beta program and test out the Android Q Beta! But if you're without a Pixel device, pretty much good luck. Nah, I sussed it out.

Steps:

- Get the Android Studio 3.4 (current RC is ok).

- Get the Android Q SDK (it will probably say Android Q Preview).

- In your app level `build.gradle`, change your `compileSdkVersion 'android-Q'` and your 
`targetSdkVersion 'Q'` (Why are these TWO DIFFERENT VALUES!?!? SUPER CONFUSING!)

- Update the emulator to 29+

- Profit.

- Now you're ready to test out some of the new features. Let's try `SettingsPanel`s - a new feature that lets users change OS level settings within your app.

In an activity, define a button of sorts. In the `onClickListener` go:

```
Intent intent = new Intent(Settings.Panel.ACTION_VOLUME);
startActivity(intent);
```

Here's what it looks like. Basically, the sound is not turned on, so when you tap on the pronunciation button, it doesn't work. Settings panels to the rescue! (even though the collapsible animation is super dodgey), you turn the volume on within the app and voila! it works. Sweet as!

<div align="center">
    <video width="30%" controls autostart autoPlay src={below} />
</div>