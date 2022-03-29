# MFM Agent

## Overview

MFM Agent is android library that will be used to track Dimensions (Network, battery, Session ...)
periodically (default interval frequency is 900000 ms) using The WorkManager Android api.

The agent fetched the System (os name, version...) and Device(brand, width...) information.

All the processed data will be saved in a local database. all fetched Data will be synchronized with a remote Web server
using apiKey that must be provide when we initialize the module.

## Features

An Android library to track the below dimensions

| Dimension                                | Summary                                     |
|----------------------------------------|-----------------------------------------------|
| Session                   | Managing active user sessions |
| Battery                   | Fetch battery information |
| Location                  | Locate device (latitude, longitude) |
| Network(Wifi, Mobile data)| Detect if device is online and fetch Network info |
| Memory                    | Collect device memory (RAM, internal and external storage) |
| Reboot                    | Count numbers of reboots |
| Fall down                 | Count numbers of fall downs. |

## Getting Started

Add this to your module's `build.gradle` file

```gradle
dependencies {
	...
	implementation project(path: ':MFMAgent')
}
```

### Configuration

Add this to you Activity or Application class, you can pass the interval to fetch dimensions data or use the default
interval frequency 15 minutes (900000 in ms)

```kotlin
private fun initMFMAgent(context: Context) {
    try {
        MFMAgent
            .Builder(context = context, apiKey = "bf98399f-6db4-4d7f-8af9-2be5f3295d68")
            .setPeriodicDimensionsState(state = Dimension.State.ENABLED, interval = 900000)
            .setSystemDataState(state = Dimension.State.ENABLED)
            .setDeviceDataState(state = Dimension.State.ENABLED)
            .enableSessionTracking(state = Dimension.State.ENABLED)
            .build()
    } catch (exception: Exception) {
        exception.printStackTrace()
    }
}
```

to enable debug you can use this line of code:

```kotlin
    MFMAgent
    .Builder(context = context, apiKey = "your_api_key")
    .setDebugEnabled(true)
```

the function ``` .setPeriodicDimensionsState(state = Dimension.State.ENABLED, interval = 900000) ```
enable fetching data for all dimensions so if you want to enable dimension by name you use the specific function:

```kotlin
    MFMAgent()
    .Builder()
    .setMemoryDataState(Dimension.State.ENABLED, interval = 90000)
    .setNetworkDataState(Dimension.State.ENABLED, interval = 90000)
    .setBatteryDataState(Dimension.State.ENABLED, interval = 90000)
    .build()
 ```

We have the possibility to cancel a worker by the name of the dimension.

```kotlin
    MFMAgent
    .getInstance()
    .cancelWorker(context, Dimension.NETWORK)
```

We can also cancel all workers

```kotlin
    MFMAgent
    .getInstance()
    .cancelAllWorkers(context)
```

Check if worker is running (running or enqueue)

```kotlin
   MFMAgent
    .getInstance()
    .workerIsRunning(context, Dimension.NETWORK)
```

### Requirements

Do not forget to add the below permissions in manifest if already not present:

 ```xml

<uses-permission android:name="android.permission.INTERNET" />
```

 ```xml

<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

```xml

<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```

 ```xml

<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

 ```xml

<uses-permission android:name="android.permission.READ_PHONE_STATE" />
```

 ```xml

<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
 ```

To detect reboot event we need to declare a broadCastReceiver, Add this tag to AndroidManifest.xml

```xml

<receiver android:name="com.hubone.mfmagent.services.BootListenerBroadcastReceiver" android:enabled="true"
    android:exported="false">
    <intent-filter>
        <category android:name="android.intent.category.DEFAULT" />
        <action android:name="android.intent.action.BOOT_COMPLETED" />
        <action android:name="android.intent.action.QUICKBOOT_POWERON" />
        <!--For HTC devices-->
        <action android:name="com.htc.intent.action.QUICKBOOT_POWERON" />
    </intent-filter>
</receiver>
```

This permission is required to detect device reboot

```xml

<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
 ```

### Health check
A health check is a monitoring service that indicate the state of the microservices .
we can say it's monitoring method that checks your API and alerts you when it notices something's amiss

We can check the server status using Lambda function (anonymous function)

```kotlin
MFMAgent.getInstance().setHealthCheckListener { serverStatus ->
    if (serverStatus == ServerStatus.ONLINE) {
        // handle result of online
    } else if (serverStatus == ServerStatus.OFFLINE) {
        // handle result of offline
    } else {
        //ServerStatus.UNKNOWN
    }
}
```

Also we can use another approach: implementing `HealthCheckListener` by our Activity or Fragment

```kotlin
class MyActivity : ComponentActivity(), HealthCheckListener {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MfmTheme {
                MFMView()
            }
        }
        //Register listener
        MFMAgent.getInstance().setHealthCheckListener(this)

        //Verify mfm server and database
        MFMAgent.getInstance().requestHealthCheck()
    }

    override fun onHealthCheckCompleted(status: ServerStatus) {
        Log.e("onHealthCheckCompleted", status.toString())
    }
}
```
