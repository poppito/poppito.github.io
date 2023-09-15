---
slug: "2020-08-22-In-App-Reviews-PlayCore-API"
title:  "Finally! In-App Reviews for Android using Playcore"
date:   2020-08-22 14:00 +1100
categories: "blog"
---

import  review_test from './images/review_test.mp4';

# Finally! In-App Reviews for Android using Playcore

<br/>
<br/>

One of the coolest things about writing iOS apps are it's high-level APIs, for example one of my favourite things is how you can prompt the user to leave a review
without getting them to ever leave your app. The best part of this? The code to request a review is ALSO super concise, like so:

```
import StoreKit

class SomeViewController: UIViewController {
        override func viewDidLoad() {
        super.viewDidload()
        if (shouldShowRatingDialog) {
            //does anyone even still run iOS 10.3? 🤔
            if #available( iOS 10.3,*){
                SKStoreReviewController.requestReview()
            }
        }
    }
}
```

See how easy that is 🤩

Well, until just recently this was not even possible in Android. But with the new Playcore API, this is totally doable and even though not as easy as it's iOS counterpart, it's not super difficult. ✅

To get started, we first pull in the ```PlayCore``` library into our app level ```build.gradle```'s dependencies block. This can of course be different if you define your dependencies elsewhere.

### Gradle dependency

```
implementation 'com.google.android.play:core:1.8.0'
```

Next steps include thinking about how this screen can be presented to the end users. For example - presenting this option excessively might drive users away from reviewing your app, or worse yet, away from your app. The good thing about the StoreKit API on iOS is, you can just make the call to present the `SKStoreReviewController` and it automatically caps how often it appears for the user. Well, fret not, even the PlayCore In-app reviews API does this for us! This however, is a double-edged sword because when we first implement areas in our apps or games to prompt the user for a review, we want to ensure that it works! Well, on iOS the `SKStoreReviewController` actually always appears in a test build of the app, however on Android it's a little bit more complicated. More on that when we get to ```Testing``` below.


### Implementation

To show users the prompt, I create a blocking `ProgressBar`, contained within a ```Framelayout``` like so. This is just an example, but it makes sense because the API is asynchronous, so there is a little bit of waiting for the user before the prompt can appear.

```    
<FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <ProgressBar
            android:id="@+id/progress_vocab_term"
            android:layout_width="48dp"
            android:layout_height="48dp"
            android:layout_gravity="center"
            android:visibility="gone" />
    </FrameLayout>
```

As a rule of thumb, this can be presented when the user is already waiting for a screen to load within a game - such as a transition between level ups, or in my case, a slang term to finish loading! 

The Playcore API gives us a ```ReviewManagerFactory``` to create an instance of the ```ReviewManager```.

Called within an activity or a fragment, we can either use `this` (Activity) or `requireContext()` (Fragment)

Once available, we can `requestReviewFlow()` and add a `completionListener` as this is a deferred `Task`, because of asynchronous nature. This is also about the time you would show a blocking `ProgressBar`.

This can be abstracted into a function that takes a boolean like so:

```
fun toggleProgressBar(visible: Boolean) {
    if (visible) {
        progress_bar.visibility = View.Visible
        content_view.visibility = View.Gone
    } else {
        progress_bar.visibility = View.Gone
        contente_view.visibility = View.Visible
    }
}
```
Now we can start our In-app Review flow, first by grabbing the `ReviewManager` then requesting a `ReviewInfo` object by calling `requestReviewFlow()`, then passing it to `launchReviewFlow()`

```
        val reviewManager = ReviewManagerFactory.create(this)
        toggleProgressBar(visible = true)
        reviewManager.requestReviewFlow().addOnCompleteListener {
            if (it.isComplete) {
                val info = it.result //your reviewInfo object
                reviewManager.launchReviewFlow(this, info).addOnCompleteListener { 
                    //the flow completed successfully. The user either left you a review 👍🏽 or dismissed the review prompt 🙀
                    toggleProgressBar(visible = false) 
                }
            } else {
                //something went wrong, the review flow couldn't be completed
                toggleProgressBar(visible = false)
            }
        }
```

### Testing

- Testing this behaviour is a little bit tricky. The easiest way is to create an `Internal Testing Track` build and rollout to your internal testers. I know this sounds like a bit of work but in practice this is super quick! Internal testing builds are available to testers within a few minutes of uploading (unless it's the first ever build, which can take up to 48 hours!). 👋

- Once the tester gets the build, you can **run local builds**! 👏

- In the testing track builds the review prompt has no quotas so test away! 🚗

- If a testing user leaves a review during the test, the prompt **won't** show again. But the tester can just as easily go to the Play store `Internal Testing` build and remove their review to re-enable the prompt! 🤝

- I would not recommend creating an `Alpha` track build! These go through reviews! And of course `Beta` track is open to everyone in the world, so definitely also not recommended. 🚫



<div align="center">
    <video width="30%" controls autostart autoPlay src={review_test} type="video/mp4" />
</div>