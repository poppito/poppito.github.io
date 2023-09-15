---
slug: "2020-12-26-A-neat-trick-for-testing-on-the-Play-Store"
title:  "A neat trick for testing on the Play store"
date:   2020-12-26 14:00 +1100
categories: "blog"
---

import  tester_distribution_list from './images/tester_distribution_list.png';
import tester_link from './images/tester_link.png';
import switch_accounts from './images/switch_accounts.png';
import play_settings1 from './images/play_store_settings_1.png';
import play_settings2 from './images/play_store_settings_2.png';
import play_store from './images/play_store_1.png';
import play_store_gotcha from './images/play_store_gotcha.png';

# A neat trick for testing on the Play store

<br/>
<br/>

Hope you've had a Merry Christmas 🎄 and kept safe in these COVID-19 times!

<br/>
<br/>

## Recap

Firstly, let's do a quick recap of how test apk and aab distribution works on the Play store:

Usually when we setup Internal and Alpha testers (testers within our own organisation), we add said testers' Google emails to a 
distribution list on the Play Console, like so

<img className="image" src= { tester_distribution_list } width="500"/>

Once this is set up, the user then goes to the internal or alpha testing link generated for the said track on the Play console:

<img className="image" src= { tester_link } width="500"/>

When the user goes to the opt-in URL on their device, they can opt into testing the app like so

For more information about tracks, distribution lists and opt-in urls, I previously wrote [this post](../2019-09-28-Relasing-an-app)

<br/>
<br/>

## But what if...

Ok so your tester distribution list is receiving your binaries and testing's going real smooth. Glad to hear it. Because the tester distribution list
uses the Play Store app to download test binaries there is a bit of a conundrum. Suppose they want to do a quick comparison with the current production app.
Does that mean they now have to go through the opt-out process? So everytime they want to switch between the alpha/beta tracks and the production track, this
would be super tedious, right? Sure would!

Well, here's a neat trick.

You might already know that the Play Store app allows multiple login accounts. These can be switched by tapping the Avatar picture of your account:

<img className="image" src= { switch_accounts } width="500"/>
<br/>
<br/>

Before doing this though, you need to clear the cache of the Play Store app. This is done by going to the Play Store's app's settings.

<img className="image" src={ play_settings1 } width="500" />

<br/>
<br/>

<img className="image" src={ play_settings2 } width="500" />

<br/>
<br/>

Now if you switch accounts to an account that is NOT on the tester distribution list, you should be able to download the production binary!

<img className="image" src={ play_store } width="500" />

<br/>
<br/>

# A quick gotcha

If you get a warning such as the one below -- the Play store cache has not been cleared properly! This means Play Store is remembering that one
of your accounts is on the tester distribution list! Just follow the steps above to clear the cache and data for the Play Store app!

<img className="image" src= { play_store_gotcha } width="500" />