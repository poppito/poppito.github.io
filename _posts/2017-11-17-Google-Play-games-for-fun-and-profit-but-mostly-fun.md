---
slug: "2017-11-17-Google-Play-games-for-fun-and-profit-but-mostly-fun"
title:  "Google play games API for fun and profit - but mostly only fun."
date:   2017-11-17 19:30 +1100
categories: "blog"
---

# Google play games API for fun and profit - but mostly only fun.

# The motivation

When I first built [StrayaMate](https://play.google.com/store/apps/details?id=strayanslangapp.noni.com.strayanslangapp), it was only really ever going to be a Australian slang dictionary. The need for this app was borne out of the fact that my partner - who teaches ESL (English as a second language) to overseas students, wanting to have something visual that her students could refer to. Now, as any dictionary goes - it can be quite dry. So, to make the app more engaging we gamified it!

One of my favourite word games is Hangman, so I figured I'd try and implement it for this app.

Conceptually, the game is really simple. A player gets a few chances to guess a word or a phrase (in this case, Australian Slang). The player of course gets given a hint that can aide in guessing said word and phrase. Points get awarded if the user is able to successfully guess the word or phrase - however, in my version of the game points also get taken away for each wrong guess. Users get 9 guesses in total, per word or phrase.

But there was still something missing. I wanted to implement a social aspect to the game - so as to increase user engagement. If you aren't playing for an achievement, for example, a game can quickly get boring! Also, a backend implementation can quickly get quick complicated - namely, you need to define a user identity and ensure there is some form of authorization. Then you award points per guessed word, and write a way to quantify how a user's performance compares to another's. Enter Google Play Games services.

Google Play provides some really cool facilities out of the box for Game developers. I am hoping I can illustrate some of these facilities I've gone ahead and utilised for this app, here, namely.

Leaderboards: Social leaderboards, where a user can compete against their friends or anyone else that plays a particular game!

Achievements: Think of these as a level up for a user. As a user progresses in a particular game, they can even level up in XP on Google Play games.

# But what exactly is Google play Games Services?

To explain it simply, it is a framework that utilises [Google Play Games](https://play.google.com/store/apps/details?id=com.google.android.play.games&hl=en) in Android. Users can create a Games Profile - an account that persists and saves their games data. It also introduces a social aspect wherein, users can add other users as friends. Games generally need to implement a points system i.e. a "unit" they award a user for carrying out a particular action successfully. This is the metric a game developer can use to place users a Leaderboard. It is also the same metric that can be used to level users up when they reach and progress through amounts of it, assigned to them.

# That's enough theory, how do I implement it?!

You start with creating a Game on Google play developer console. After signing in, go to Games Services.

![Play games](/images/play-games/play-games-app.png)

Click on "ADD NEW GAME", followed by selecting "I don't use any Google APIs in my game yet". Note, that here Games services creates an "app_id", which we will later use.

![Add game](/images/play-games/add-new-game.png)

Give your game a name, for example Slangman in my case and then choose a category - for example - Slangman is in the Educational category.

![Category](/images/play-games/add-game-choose-category.png)

Select Android.

Now add your existing app's packagename. It will conveniently provide you a dropdown of your current apps on the Play Console of course. Other options i.e. Real time multiplayer are not applicable in my case so use your discretion!

Next, authorise your app to create an OAUTH 2.0 token. Note: You can check a successfully created OAUTH token by logging on to Google cloud console and searching for “Google play game services”.

Now you are ready to start configuring your Android project! First, add the “Get Accounts” permission to your AndroidManifest.

```
<uses-permission android:name="android.permission.GET_ACCOUNTS" />
```

Add Play Services to your Android project's build.gradle.

```   
dependencies {
        ...
        classpath 'com.google.gms:google-services:3.1.0'
    }
```

Add Games services to your app's build.gradle.

```
dependencies {
    ...
    compile 'com.google.android.gms:play-services-games:10.2.0'
}
```

Configuring Leaderboards is easy. Basically, you need to implemement a "points" metric - something you reward the user when they carry a certain action. Then determine appropriate times in the user's gaming lifecycle to call the Leaderboards API and post the user's current points. That's it! To add a leaderboard, in the Play Console, under Games Services, just click on "Add a Leaderboard", give it a name for example. World Rankings, and then determine if larger points or smaller points is better. You can also define a limit i.e. not to award points above a certain limit. Note, every Leaderboard creates a leaderboard_id String. This needs to be added as a @StringRes String so we can reference it later!

![Leaderboards](/images/play-games/configure-leaderboards.png)

Configuring Achievements is even simpler. As I said previously, these are "level ups" for players. What is unique is, that as the play levels up in your game, Play games also levels them up in "XP", in that particular game's category! Achievements are quantified in terms of the points metric we spoke about previously. So, achievement-1 could be awarded when a user scores 50 points, achievement-2 awarded when a user scores 100 etc. To configure achievements, on Games services, just click on just click on “Add achievement”, give it an icon, description, XP points, whether it is visible to the user. Note, every achievement creates an achievement_id String. This needs to be added as a @StringRes String so we can reference it later!

![Achievements](/images/play-games/configure-achievements.png)

Now let's get back to the code! First of all, when we created the Game under Games services, it automatically created an app_id.
We need to add this to our AndroidManifest.xml's metadata. We can use a @StringRes for the app_id String and reference it directly.

```
<meta-data android:name="com.google.android.gms.games.APP_ID"
                      android:value="@string/app_id">
```

We now need to use the GoogleApiClient class. Please note that before doing this, its always a good idea to prompt the user that i.e. some users might not want to sign-in with their play games profile.

![Sign-in](/images/play-games/alert-to-sign-in.png)

To configure GoogleApiClient, first we use the convenient Builder class. Once built, we can then run "connect()", on it.

```
private GoogleApiClient buildGoogleApiClient() {
    return new GoogleApiClient.Builder(this)
            .addApi(Games.API)
            .addConnectionCallbacks(this)
            .addOnConnectionFailedListener(this)
            .addScope(Games.SCOPE_GAMES)
            .build();
}

mGoogleApiClient = buildGoogleApiClient();
            mGoogleApiClient.connect();
```


Now, you may have noted that we added some ConnectionCallbacks and also a ConnectionFailedListener. When we first run connect, this will always call onConnectionFailed(), as the user has either not signed-in already, or might not even have a  Games profile! No matter. When the onConnectionFailed() callback comes back, it has a ConnectionResult object. We can query it to see if it returns true for hasResolution() and if so, we run the startResolutionForResult() method on it and pass in an arbitrary integer, very similarly to startActivityForResult() in Android. Finally, we can track the result in "onActivityResult()", of course.

```
  connectionResult.startResolutionForResult(this, REQUEST_RESOLVE_ERROR);
```

```
if (requestCode == REQUEST_RESOLVE_ERROR) {
            if (!(resultCode == RESULT_OK)) {
                if (mGoogleApiClient != null) {
                    if (!mGoogleApiClient.isConnecting()
                    || !mGoogleApiClient.isConnected()) {
                        mGoogleApiClient.connect();
                    }
                }
            }
        }
```

Now, as I said previously, to submit the score to the Leaderboards API, you can use your judgement. I do it at arbitrary points in the game as well as when the user runs out of tries and its game over of course.

```
if (mGoogleApiClient.isConnected()) {
                Games.Leaderboards.submitScore(mGoogleApiClient,
                        getResources().getString(R.string.leaderboard_points_leaderboard),
                        points);
```

To submit an achievement, you will need to track the user's progress as they move through the game. To award an achievement just run (note here I am using a BuildConfig field and not a @StringRes)

```
 Games.Achievements.unlock(mGoogleApiClient, BuildConfig.achievement_getting_there);
```

That's it! But how do we display Leaderboards or Achievements to the user?! Simply by:

```
startActivityForResult(Games.Leaderboards.
                            getLeaderboardIntent(mGoogleApiClient,
                            getResources().getString(R.string.leaderboard_points_leaderboard)),
                            REQUEST_LEADERBOARD);
```

```
startActivityForResult(Games.Achievements.getAchievementsIntent(mGoogleApiClient),
                        REQUEST_ACHIEVEMENTS);
```

# Addendum

Well, looks like `GoogleApiClient` is soon to be deprecated as of Play Services v11. No matter, we will probably be able to explore this in my upcoming posts.

That's pretty much it from me, for now. I sure hope this post has added some value to someone out there. If you read this and have any questions you can feel free to tweet them through to me - my handle should just be below this post :D