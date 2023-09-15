---
slug: "2019-09-28-Relasing-an-app"
title:  "App releases on the Play console"
date:   2019-09-28 14:00 +1100
categories: "blog"
---

import internal_app1 from './images/internal-app1.png';
import internal_app2 from './images/internal-app2.png';
import internal_app3 from './images/internal-app3.png';
import internal_app4 from './images/internal-app4.png';

import internal_test1 from './images/internal-test1.png';
import internal_test2 from './images/internal-test2.png';
import internal_test3 from './images/internal-test3.png';
import internal_test4 from './images/internal-test4.png';


import phasing from './images/phasing-rollouts.png';
import in_review from './images/in-review.png';
import beta from './images/beta.png';
import beta_test from './images/beta-test.png';


<h1 align="center"> App releases on the Play Console </h1>
<br/>
<br/>

Recently, there has been much conjecture around releases on the Play Console for Android apps. Between releasing for internal testing, sharing apps internally, running an open test, running a closed test, doing a staged rollout in testing or releasing to production - it can get confusing. With releases developers are also reporting an app store style app review taking place, so let's try and break down the plethora of options on the Play console.

## Internal App sharing.

This is a very similar way of distributing apps and app bundles to your internal testers as other facilities such as Hockeyapp.

- Doesn't require a version increment.

- Doesn't need a production APK - you can share debug versions too.

- Add internal testers and use the internal testing list for distributing an APK or app bundle. 

- Alternatively, you can generate a link to share with testers.

As a pre-requisite, you will need to add a list of internal testers - to do this, go to Play console > Manage Email lists and add your testers to this list.

- To get started with internal app sharing go to your app, go to Development tools > Internal App sharing and drop an APK or app bundle.
<img src= { internal_app1 } width="50%" />

- Once the APK is uploaded, you get an option to either share it using a link or with your internal testing list (screenshot below shows the latter selected)
<img src={ internal_app2 } width="50%" />

- When you proceed it will generate a link for distribution. Tap on "copy link" to distribute this link.
<img src= { internal_app3 } width="50%"/>

- When a user with a tester email uses this link on an Android device, it should present a link to open the app up in the Play store app.
<img src= { internal_app4 } width="50%"/>

## Internal app testing.

- Does require a version increment.

- Will need a signed production APK or app bundle.

- Up to a 100 testers in the tester email distribution list.

- Testers will need to be pre-registered, using an opt-in link.

- Should not go through app review in this track.

### To setup an internal test:

- You will need a list of testers registered and opted-in. For every internal test you setup it generates an opt-in URL that the testers list will need to use to opt-in. To proceed with this, go to your app on the Play console and go to "App releases". Ensure the checkbox next to your testers' list is checked and click on save! The opt-in URL in the screenshot has been blanked out of course.
<img src={ internal_test1 } width="50%"/>

- Now is the time to upload your APK or app bundle. Once this is complete, add your release notes and click "save".
<img src={ internal_test2 } width="50%"/>

- This should be the final screen wherein you can review all the information you've provided and then go "Start rollout to internal test". Please note there is no option here to do a phased release!
<img src={ internal_test3 } width="50%"/>

- The official documentation does say that the APK or app bundle would be available in minutes, however in the case of my internal test it took about 15-20 minutes!
<img src={ internal_test4 } width="50%"/>


## Closed track (Alpha) releases

- Does require an increment in the app's version code.

- Test within your organisation or within a tester email distribution list.

- Will need a signed production APK or app bundle.

- Testers will need to be pre-registered, using an opt-in link (this link is differnt to all the other tracks)

- A phased rollout can be performed for this track.

- You can promote to production after testing is complete.


This track's steps are pretty well the same as the internal testing track, including testers having to opt-in before testing can occur. To setup a closed alpha test, go to "App releases > closed alpha"

The only difference here really is a phased rollout is allowed, this is done by specfying a percentage in the review screen.

## Open track (Beta) releases

- This is an open testing track because, unlike the closed track users on Play can opt-in for beta versions of your app.

- If you want to limit the amount of users for this, you can specify a number up to 1000, if left blank this is as many testers as those that opt-in.

- The APK or app bundle version will need to be incremented.

- Only really accepts a signed production APK or app bundle.

- Anyone can pre-register to become a tester for your app.

- Phased rollouts are allowed.

- You should ensure that the store listing of your app is up to scratch because this is not your internal / closed testing track - the people that sign-up to be testers will be external and that's their window into your app!
<img src= { beta_test } width="50%" />


## Whoa, that's a lot of options.
Ok, so this got really quite detailed and fast. I think the best way to think about the different ways of doing releases with the Play console is probably working out each of the use-cases.

## In what situation would you use internal app sharing?

- I recommend, perhaps all the time! Sharing an APK via the official way of sharing an APK (y'know, because its on the Play console!) rather than relying on a 3rd party is probably advisable - because many of the steps are similar to doing an internal, closed or open track of testing.

- Its really convenient because once your testers' distribution list is added, it can be reused across other tracks.

- It becomes a central way of distributing your APKs or app bundles (no more managing users across 3rd parties just for testing)

### In what situation would you use internal app testing?

- Internal app testing requires a prod signed APK and a version increment, so you'd want to use it if you have a small team of internal testers once you have a release candidate, whom you could release to, first.

- It is easy to get feedback from your testers and if all goes well, this option complements the closed (alpha) track really well. 

- When used in conjunction with the closed (alpha) track, you can use staged rollouts.

- Its quicker than the closed or open tracks and doesn't require an app review.

### In what situation would you use closed (alpha) testing?

- The closed track complements the internal testing track really well, so I recommend promotion from the internal track to the closed track.

- An app review could occur at this stage so, basically this is a good exercise into addressing any issues that could arise here.

- If all goes well, you can promote your APK or app bundle to production!

### In what situation would you use open (beta) testing?

- If you have done all your homework and setup a nice listing and probably can promote an open beta test to real-world users this is a great way to kick start receiving any feedback.

- An app review could occur, so basically this is a good exercise into addressing any issues that arise here.

- You will need to ensure that all the store pre-reqs are met including the listing and possibly ratings for users. Not recommended for a WIP.

- Once you feel ready after all the feedback has been collected and addressed, you can promote the app directly to production!

## My recommendation for a release approach:

- Use internal app sharing testing during the normal development cycle, and any regression testing before picking an RC.

- Internal testing with an RC over a timebox.

- Promoting to the closed track and testing in a timebox, as well as having a review occur.

- Promote to production once everything's a-OK. I recommend using a phased rollout in production for any large scale releases so you can monitor for scale issues or new crashes.

- If you do any internal only releases between production releases, I highly recommend doing at least an internal test and promotion to alpha so any issues during a review can be addressed. You don't want to only kick-off reviews for production-only releases!

## Some gotchas to look out for:

- When you're on a test distribution list, you will see releases as well as test APKs or app bundles with a "(Beta)" at the end of the name.
<img src={ beta } width="50%" />


- When your app is being "reviewed", it will say so on the "Dashboards" page.
<img src= { in_review } width="50%"/>

- Phased rollouts in any track that allow them, are manual. This affords you the time to keep assessing any crash logs and / or any other stats to make decisions on proceeding with higher percentages for rollouts.
<img src= { phasing } width="50%" />
<br/>
<br/>

## Conclusion
The Play console’s testing tracks offer a host of options and leveraging these to your advantage can help you release your app in the most optimal way possible! Just like Sydney skies in Autumn!
What do you reckon? I’d love to hear your thoughts on how you do releases on the Play console!