---
title: Authenticate User
description: In this step you authenticate your users via the JWTs you created earlier
---

# Authenticate User

You perform this authentication using the `JWT` generated in previous steps. Login screen (`LoginFragment` and `LoginViewModel` classes) is responsible for authenticating the user.

```screenshot
image: public/screenshots/tutorials/client-sdk/android-shared/login-screen-user.png
```

## Get NexmoClient instance

You have to retrieve client instance inside `LoginViewModel` class. Usually, it would be provided it via injection, but for tutorial purposes you will retrieve instance directly using static method. Locate the `private val client` property in the `LoginViewModel` class and update its implementation:

```kotlin
private val client = NexmoClient.get()
```

Make sure to add missing import again.

## Login user

Your user must be authenticated to be able to participate in the Call. Locate the `onLoginUser` method inside `LoginViewModel` class and replace it with this code:

```java
void onLoginUser(User user) {
    if (!StringUtils.isBlank(user.jwt)) {
        this.user = user;
        client.login(user.jwt);
    }
}
```

> **NOTE:** Inside `LoginFragment` class, explore the `loginUser` method that was written for you. This method is called when one of the two `Login ...` buttons are clicked. This method calls the above `onLoginUser` method.

> **NOTE:** The `User` type is the `data class` that we've defined in the `Config.kt` file.

## Monitor connection state

When a successful connection is established you need to navigate user to `MainFragment`. Locate the `LoginViewModel` constructor inside `LoginViewModel` class and replace it with this code:

```java
public LoginViewModel() {
    client.setConnectionListener(new NexmoConnectionListener() {
        @Override
        public void onConnectionStatusChange(@NonNull ConnectionStatus connectionStatus, @NonNull ConnectionStatusReason connectionStatusReason) {
            if (connectionStatus == ConnectionStatus.CONNECTED) {
                navigate();
                return;
            }

            connectionStatusMutableLiveData.postValue(connectionStatus);
        }
    });
}
```

The above code will monitor connection state and if the user is authenticated (`ConnectionStatus.CONNECTED`) it will navigate the user to the `MainFragment`.

You're now ready to make the call within the app.
