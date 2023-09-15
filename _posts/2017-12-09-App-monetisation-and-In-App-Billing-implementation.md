---
slug: "2017-12-09-App-monetisation-and-In-App-Billing-implementation"
title:  "App monetisation and In-App Billing in Android."
date:   2017-12-08 19:30 +1100
categories: "blog"
---

import inAppProducts from "./images/billing/in-app-products.png";
import addProducts from "./images/billing/add-product.png";

# Android apps and monetisation.
<br/>
<br/>

Some devs make apps for fun, some make them for learning things. But if you've got a good app idea, chances are you want to be able to monetise it. With Android, there are several ways of going about it i.e. putting in ads - which relies on not only high volumes of downloads but large amounts of ad screen time - or you could sell a paid app - but that's not without its challenges. A lot of mobile users will want to be able to try your app just so they can work out if it's worth their while. This would perhaps mean publishing a trial version of your app and then subtly diverting these users to a paid version - but this does mean you'd have to then publish two versions of your app - however, you're still splitting your audience across two apps - this means splitting your ratings, downloads and app indexation on the Play store!

# Best of both worlds.

There is a better way. You can combine these two monetisation models by building a single version of your app and introducing paid features in your app - by using the [Android's In-App Billing API](https://developer.android.com/google/play/billing/index.html) - and its super easy to implement. There are a couple of things you'll want to work out first though - for example whether you want to charge your users with a subscription model i.e. something a media service would charge its users every month or over some other pre-determined amount of time, or have a consumable feature - i.e. something like ammo in a game - but in this particular example we're going to be focussing on having a perpetually unlockable feature which really is a baseline implementation of the lot.

# Let's give it a go.

The `BillingClient` convenience class provides a gateway into the [Play Billing Library](https://developer.android.com/google/play/billing/billing_library.html) and abstracts away calls to the [In-App billing API](https://developer.android.com/google/play/billing/api.html) for us. Here's how to leverage it:

* First we need to get an `instance` of the `BillingClient`. There is a convenience `Builder` class that requires the calling Activity's `Context`. We can also set the `PurchasesUpdatedListener` callback here which is important because we want to be notified of when the billing flow is complete and the user has been charged of course!

  ```
  mBillingClient = BillingClient.newBuilder(mContext)
                 .setListener(this)
                 .build();
  ```
* Let's take a quick detour and also update our permissions to use `In-App Billing` in our `AndroidManifest.xml` file.

  ```
  <uses-permission android:name="com.android.vending.BILLING" />
  ```

* We set up the `PurchasesUpdatedListener` previously - so, when the `onPurchasesUpdated` callback gets notified - there are a couple of things to check. Firstly, there is an integer value `responseCode`. You can read about what [responseCode](https://developer.android.com/reference/com/android/billingclient/api/BillingClient.BillingResponse.html) means but for now, we're looking for `BillingResponse.OK`. Secondly, a `@nullable List<Purchase>` value - we can iterate over all of its values and look for particular `skus` - more on this later. Because `List<Purchas>` is nullable, its always a good idea to guard it with a null check of course and then iterate over to see if the required `sku` has been returned.

  ```
  if (purchases.size() > 0) {
                  for (Purchase result : purchases) {
                      if (result.getSku().equals(mContext.getResources().getString(skuOfInterest))) {
                          //the billing flow has completed successfully.
                      }
                  }
              }
  ```

* Sweet. Now, we're Ok to go ahead and connect up our `mBillingClient`, we've previously built and added a callback to. We are required to pass in a `BillingClientStateListener`. Basically, this callback comes back with a integer value `BillingResponse` code which helps us determine if connnectivity was successful or not. There is also a `onBillingServiceDisconnected` method which is needed to be queried and consequently we need to return some meaningful feedback error to the user if required.

    ```
    mBillingClient.startConnection(new BillingClientStateListener() {
            @Override
            public void onBillingSetupFinished(int responseCode) {
                if (responseCode == BillingClient.BillingResponse.OK) {
                    mIsBillingClientConnected = true;
                    //do other interesting things here - as we're connected now.
                }
            }

            @Override
            public void onBillingServiceDisconnected() {
               mIsBillingClientConnected = false;
               //handle this failure gracefully.
           }
        }
    ```
  now we know that we can query the In-App billing API when `mIsBillingClientConnected` is `true` and gracefully try either reconnecting or show an error response of sorts when it is `false`.

* Now we've mentioned `sku`s a few times in previous steps. This is just a `String` value that an **In-App product** gets given when we generate one. Let's explore this here - we'll add an **In-app product** that we can bill against.

  On the **Play Console**, under **Store Presence**, there is an option to create **In-App products** click on **Add a Managed Product** - then give it an id (only underscores, lowercase letters and digits allowed!) - which will become the `sku` for your **In-App Product**

  Next, give it a title and a description as well as price. This will show up when a user tries to buy your In-app product through your app, so make it short and simple. The Play console does automatic currency conversions for countries your app is supported in - so if you need to tweak that for individual countries - you can it here as well. Finally, click  on save and it will add your `In-App product`.

  <p align="center">
  <img src= { inAppProducts } width="30%" />
  </p>
  <p align="center">
  <img src= { addProducts } width="50%" />
  </p>

  Now that you've got an `sku`, you can create a `@StringRes` for it.

  ```
  <string name="id_sku_my_product">01_my_first_product</string>
  ```

* Nearly there! Now you can query for if the user has purchased your **In-App Product**. This is determined by running the `queryPurchaseHistoryAsync` callback on your `mBillingClient`.

  ```
  mBillingClient.queryPurchaseHistoryAsync(BillingClient.SkuType.INAPP,
               new PurchaseHistoryResponseListener() {
                   @Override
                   public void onPurchaseHistoryResponse(@BillingClient.BillingResponse int responseCode,
                                                         List<Purchase> purchasesList) {
                       if (responseCode == BillingClient.BillingResponse.OK) {
                           if (purchasesList != null) {
                               if (purchasesList.size() > 0) {
                                   for (Purchase result : purchasesList) {
                                       if (result.getSku().equals(mContext.getResources().getString(R.string.01_my_first_product))) {
                                           //success! your user has bought your In-App product. Woohoo!
                                       }
                                   }

                               }
                           }
                       }
                   }
               });
    ```

* Although, Android gives you some more convenience on top of this. You do not need to call `queryPurchaseHistoryAsync` everytime. Purchases are cached by the Play store app - so you can just run `queryPurchases`. So, what naturally follows is that you would use this call to see if it comes back with legit `PurchaseResult`, if not you can run  `queryPurchaseHistoryAsync`.

  ```
  Purchase.PurchasesResult purchasesResult = mBillingClient.queryPurchases(BillingClient.SkuType.INAPP);
                    if (purchasesResult != null) {
                        if (purchasesResult.getPurchasesList() != null) {
                            if (purchasesResult.getPurchasesList().size() > 0) {
                                for (Purchase result : purchasesResult.getPurchasesList()) {
                                    if (result.getSku().equals(mContext.getResources().getString(R.string.01_my_first_product))) {
                                        //success! your user has bought your In-App product. Wowie!
                                    }
                                }
                            }
                        }
                    }
  ```

* Ok, so that's the basic flow. Let's recap:
  1. Build the `BillingClient`.
  2. Connect said `BillingClient`.
  3. On success, pass in the `purchasesUpdatedListener` callback.
  4. When purchases are updated query for your `sku` by running first running `queryPurchases` or `queryPurchaseHistoryAsync`.

  But what about users that have already bought your product? Well, you'll have to run this process at the start of your app too! Users that have already bought your app don't need to be run through a purchase process of course!

# Optimisation tips.

 * It almost seems like you're going to have to follow the billing process twice. You're just better off using a `Singleton` instance of the `BillingClient`. If you're using a DI framework like the excellent [Dagger](https://google.github.io/dagger/), you're better off creating a `Singleton` component on your dependency graph and then injecting it as needed.

 * One of the biggest challenges for me was the fact that `In-App Billing` was not showing up on the Play console. That'd be because my production APK didn't have the In-App billing permissions. This is easily remedied by creating an Alpha release of your app - and actually its highly recommended because you can then use test accounts to test the purchase flows. As long as your test users are added as alpha testers for your app!