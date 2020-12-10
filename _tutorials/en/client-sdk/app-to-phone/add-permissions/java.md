---
title: Add permissions
description: In this step you add permissions
---

# Add permissions

## Declare permissions in Android Manifest

Add required permissions into `AndroidManifest.xml` file:

```screenshot
image: public/screenshots/tutorials/client-sdk/android-shared/android-manifest-file.png
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.vonage.tutorial">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
    <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    ...
```

## Request permissions at runtime

Open `MainActivity` class add `requestPermissions` medhod:

```java
private void requestPermissions() {
    String[] callsPermissions = { Manifest.permission.RECORD_AUDIO };
    ActivityCompat.requestPermissions(requireActivity(), callsPermissions, 123);
}
```

Call `requestPermissions` method from `onCrete` method:

```java
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {

    // ...

    requestPermissions();
}
```