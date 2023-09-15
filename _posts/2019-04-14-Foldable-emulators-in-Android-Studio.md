---
slug: "2019-04-14-Foldable-emulators-in-Android-Studio"
title:  "Foldable emulators in Android Studio - or as I like to call it, the beginning of the end"
date:   2019-04-14 19:30 +1100
categories: "blog"
---


import foldable_emulators from './images/foldable_emulators.mp4'


## Foldable emulators in Android Studio - or as I like to call it, the beginning of the end

I have been thinking about how one can test apps on foldables - there's no way I am going to invest in $2000+ on a handset - in fact I distinctly recall my first macbook pro was only really about $1600 (after some work discounts of course).

Anyway, looks like foldable emulators are available on Android Studio now (requires time investment), so I don't have to. Plus Samsung eww.

Steps:
- Get AS 3.5 canary 10 or above.

- Update the emulator to 29+

- Get the latest Android Q beta

- Create a new AVD by using the available 7.3” or 8” foldables.

- Profit.

- Bonus! apply changes (the new instant run) looks available in this version of AS also! (But that might warrant a different post)

<br/>
<br/>

<div align="center">
<h3> Here's what it looks like: </h3>
    <video width="30%" controls autostart autoPlay src={ foldable_emulators } type="video/mp4" />
</div>