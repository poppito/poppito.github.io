---
slug: "2019-04-06-In-App-Billing-implementation-v1.2"
title:  "In-App-Billing-implementation-v1.2"
date:   2019-04-06 19:30 +1100
categories: "blog"
---

import video from './images/billing/strayamate-demo.webm';
import in_app_products from './images/billing/in-app-products.png';
import add_product from './images/billing/add-product.png';

//[previously](/2017-12-09-App-monetisation-and-In-App-Billing-implementation.md) 

# A continuation.
I wrote a post about In-App Billing in Android - this is really just an update to that original post. Before we start (again) though - lets get the semantics out of the way: the API is called In-App Billing however, whenever you sell a product leveraging this API, its an In-App product (IAP).

# First things first.

IAPs are setup in the Play console - under store presence > In-App Products. But, I wasn't able to initially set this up at all. After digging a little bit I found my production APK didn't have the In-App billing permissions. (Wouldn't this be the case for everyone?) This is easily remedied by creating an Alpha release of your app - and actually its highly recommended because you can then use test accounts to test the purchase flows. As long as your test users are added as alpha testers for your app!

Now we set up an `SKU` - a stock keeping unit - This is just a `String` value that an **In-App product** gets given when we generate one. Let's explore this here - we'll add an **In-app product** that we can bill against.

On the **Play Console**, under **Store Presence**, there is an option to create **In-App products** click on **Add a Managed Product** - then give it an id (only underscores, lowercase letters and digits allowed!) - which will become the `sku` for your **In-App Product**
Next, give it a title and a description as well as price. This will show up when a user tries to buy your In-app product through your app, so make it short and simple. The Play console does automatic currency conversions for countries your app is supported in - so if you need to tweak that for individual countries - you can it here as well. Finally, click  on save and it will add your `In-App product`.

<p align="center">
<img src= { in_app_products } width="30%" />
</p>
<p align="center">
<img src= { add_product } width="30%" />
</p>


Now that you've got an `sku`, you can create a `@StringRes` for it.

```
    <string name="id_sku_my_product">01_my_first_product</string>
```

Update our permissions to use `In-App Billing` in our `AndroidManifest.xml` file.

```
    <uses-permission android:name="com.android.vending.BILLING" />
```

Also, to start using IAB, you will need to pull in the IAB depedency into your app. Add this to the `dependencies` closure of your app's `build.gradle`:

```
    implementation 'com.android.billingclient:billing:1.2.2'
```

# The anatomy of IAB / IAPs in Android.

Use the `BillingClient`, its your friend. When you use the `BillingClient` builder, you set a `PurchasesUpdatedListener`.

  ```
    billingClient = BillingClient.newBuilder(context)
                    .setListener(this)
                    .build();
  ```

In order to start doing anything interesting, you call `startConnection()` and pass an instance of a `ConnectionStateListener`. This ensures connectivity of your app with the In-App Billing API. If it comes back with `OnBillingSetupFinished()`, which is the success scenario, you're ready for the next step. If it fails, it comes back with `OnBillingServiceDisconnected()`, this might warrant an error message.

```
    billing.startConnection(new BillingClientStateListener() {
            @Override
            public void onBillingSetupFinished(int responseCode) {
                if (responseCode == BillingClient.BillingResponse.OK) {
                    isBillingClientConnected = true; //this boolean value persists connection state.
                    //do other interesting things here - as we're connected now.
                }
            }

            @Override
            public void onBillingServiceDisconnected() {
               isBillingClientConnected = false;
               //handle this failure gracefully.
           }
        }
```

Now, we've got two things to worry about - what are the localised SKUs and their costs? So first, we query for all SKUs first. This is done by running:

```
    List<String> skuList = new ArrayList<> ();
    skuList.add("premium_upgrade");
    skuList.add("gas");
    SkuDetailsParams.Builder params = SkuDetailsParams.newBuilder();
    params.setSkusList(skuList).setType(SkuType.INAPP);
    billingClient.querySkuDetailsAsync(params.build(),
        new SkuDetailsResponseListener() {
            @Override
            public void onSkuDetailsResponse(int responseCode, List<SkuDetails> skuDetailsList) {
                if (responseCode == BillingResponse.OK && skuDetailsList != null) {
                    for (SkuDetails skuDetails : skuDetailsList) {
                        String sku = skuDetails.getSku();
                        String price = skuDetails.getPrice();
                        if ("premium_upgrade".equals(sku)) {
                            premiumUpgradePrice = price;
                        } else if ("gas".equals(sku)) {
                            gasPrice = price;
                        }
                    }
                }
            }
        }
    });
```

Now that user has a list of SKUs and they want to purchase one, we start what's called a `BillingFlow`.

```
    BillingFlowParams flowParams = BillingFlowParams.newBuilder()
        .setSkuDetails(skuDetails)
        .build();
    int responseCode = billingClient.launchBillingFlow(flowParams);
```

Now, recall `OnPurchasesUpdatedListener` we set in the first step! That should call back if the purchase is successful!

```
    @Override
    void onPurchasesUpdated(@BillingResponse int responseCode, List<Purchase> purchases) {
            if (responseCode == BillingResponse.OK
                    && purchases != null) {
                for (Purchase purchase : purchases) {
                    handlePurchase(purchase);
                }
            } else if (responseCode == BillingResponse.USER_CANCELED) {
                // Handle an error caused by a user cancelling the purchase flow.
            } else {
                // Handle any other error codes.
            }
    }
```

Nearly there! Now you can query for if the user has purchased your **In-App Product**. This is determined by running the `queryPurchaseHistoryAsync` callback on your `mBillingClient`.

```
    billingClient.queryPurchaseHistoryAsync(BillingClient.SkuType.INAPP, new PurchaseHistoryResponseListener() {
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

Although, Android gives you some more convenience on top of this. You do not need to call `queryPurchaseHistoryAsync` everytime. Purchases are cached by the In-App Billing library - so you can just run `queryPurchases`. So, what naturally follows is that you would use this call to see if it comes back with legit `PurchaseResult`, if not you can run  `queryPurchaseHistoryAsync`.

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


# Optimisation tips.
It almost seems like you're going to have to follow the billing process twice. You're just better off using a `Singleton` instance of the `BillingClient`. If you're using a DI framework like the excellent [Dagger](https://google.github.io/dagger/), you're better off creating a `Singleton` component on your dependency graph and then injecting it as needed.


<p align="center">
 <h2> Demo </h2>
 <video width="30%" controls autostart autoPlay src= { video } frameborder="0" allowfullscreen></video>
 </p>