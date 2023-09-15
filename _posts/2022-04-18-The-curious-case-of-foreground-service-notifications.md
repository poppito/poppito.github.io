---
slug: '2022-04-18-The-curious-case-of-foreground-service-notifications'
title: 'The curious case of Foreground service notifications'
date: 2022-04-18 10:30 +1100
categories: 'blog'
---

import notifications from './images/copy_pasta.webm';

# The curious case of Foreground service notifications

Ok, let's start with looking at some history.

# A background on background work in Android

When I first got into Android development, background services were kinda [unrestricted](https://android-developers.googleblog.com/2010/02/service-api-changes-starting-with.html)

Because the Services API is so **powerful** and being able to do things in the background while persisting the app's [process](https://proandroiddev.com/android-internals-101-how-android-os-starts-you-application-e1c98a014c05#:~:text=According%20to%20dictionary%20definition%3A%20Zygote,thus%20achieving%20fast%20app%20launches.) can
actually affect the device's performance due to contention (Memory, CPU) and usage (Battery), developers certainly have a responsibility to use services wisely.

One more thing to note is progressive restrictions from Android 8.0's [background execution limits](https://developer.android.com/about/versions/oreo/background)

My favourite API to do long running background processing in Android is thusly [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager/advanced/long-running)

# Foreground services and notifications

Of course Android supports long running operations that the user explicitly requests. These are run by posting status bar notifications and using [foreground services](https://developer.android.com/guide/components/foreground-services#notification-immediate)

The rationale behind this is that

- The user has explicitly requested a long running operation that can persist outside of the app being killed.

- The notification is visible to the user throughout the duration of this long running operation.

So, I suppose the question here to ask is **what would happen if the user has explicitly turned off ALL notifications in the app's settings?**

Let's find out by creating a sample app that does this.

- First we create a Service and declare it in the app's `AndroidManifest.xml`

```
    <service
            android:name=".SomeService"
            android:enabled="true"
            android:exported="false" />
```

- Create `SomeService`

```
    class SomeService : Service() {

        companion object {
            const val CHANNEL_NAME = "serviceChannel"
            const val CHANNEL_ID = "channelId"
            const val TITLE = "SomeService up"
            const val MESSAGE = "SomeService is now running"
            val targetActivity = HomeActivity::class.java
            const val NOTIFICATION_ID = 0
        }

        override fun onBind(intent: Intent): IBinder? = null

        override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
            val target = Intent(this, targetActivity)
            val targetIntent =
                PendingIntent.getActivity(this, 1, target, PendingIntent.FLAG_UPDATE_CURRENT)

            val notificationManager =
                getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
            val notification = NotificationCompat.Builder(this, CHANNEL_ID)
                .setContentTitle(TITLE)
                .setContentText(MESSAGE)
                .setNotificationSilent()
                .setSmallIcon(R.drawable.ic_launcher_foreground)
                .setContentIntent(targetIntent)
                .build()

            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                val channel = NotificationChannel(
                    CHANNEL_ID,
                    CHANNEL_NAME,
                    NotificationManager.IMPORTANCE_DEFAULT
                )
                notificationManager.createNotificationChannel(channel)
            }

            if (Build.VERSION.SDK_INT >= 29) {
                startForeground(startId, notification, ServiceInfo.FOREGROUND_SERVICE_TYPE_NONE)
            } else {
                startForeground(startId, notification)
            }

            serviceStarted = true
            return START_NOT_STICKY
        }

        override fun stopService(name: Intent?): Boolean {
            serviceStarted = false
            return super.stopService(name)
        }
    }
```

- `ServiceNotifier` is a `Singleton` that sets/unsets the `serviceStarted` flag. This is just for us to debug `SomeService` when notifications are turned off.

```
    object ServiceStartedNotifier {
        var serviceStarted = false   
    }
```

- `HomeActivity` is just the launcher activity.

```
    class HomeActivity : AppCompatActivity() {
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_main)

            if (!serviceStarted) {
                val serviceIntent = Intent(this, SomeService::class.java)
                startService(serviceIntent)
            }
        
        // poll for `SomeService` being up/ down.
        val scope = CoroutineScope(Dispatchers.IO)
        scope.launch {
            while (true) {
                delay(2000)
                withContext(Dispatchers.Main) {
                    txt_status.text = if (serviceStarted) "Service is up" else "Service is down"
                }
            }
        }
    }

    // Not strictly required, but saves me from killing the service manually.
    override fun onDestroy() {
        super.onDestroy()
        serviceStarted = false
        stopService(Intent(this, SomeService::class.java))
    }
```

- `activity_main.xml` is just an xml layout. I should probably have written this in Jetpack Compose!

```
    <androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".HomeActivity">

            <TextView
            android:id="@+id/txt_status"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:gravity="center"
            android:textIsSelectable="true"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />
    </androidx.constraintlayout.widget.ConstraintLayout>
```


<div align="center">
<h3> Ok, now for the big reveal</h3>
<video width="30%" controls autostart autoPlay src={ notifications } type="video/mp4" />

__That's right. For foreground services, if the user stops showing notifications, this just stops the service from posting notifications! However, the 
service and its processing still seems to be working as normal__


<img src="https://media.giphy.com/media/aLdiZJmmx4OVW/giphy.gif" />

</div>