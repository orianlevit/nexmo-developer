---
title: Authenticate User
description: In this step you authenticate your users via the JWTs you created earlier
---

# Authenticate User

You perform this authentication using the `JWTs` generated in previous steps. Users must be authenticated to be able to participate in the Conversation. You will now build login screen (`LoginFragment` and `LoginViewModel` classes) responsible for authenticating the users.

## Update `fragment_login` layout

Open `fragment_login.xml` file.

> **NOTE** You can open any file by using `Go to file...` action. Press `Shift + Cmd + O` and enter file name.

Click `Code` button in top right corner to display layout XML code:

```screenshot
image: public/screenshots/tutorials/client-sdk/android-shared/show-code-view.png
```

Replace file content with below code snippet:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:padding="10dp">

    <Button
            android:id="@+id/loginAsAliceButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Login as Alice"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

    <Button
            android:id="@+id/loginAsBobButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Login as Bob"
            android:layout_marginTop="30dp"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toBottomOf="@id/loginAsAliceButton"
            app:layout_constraintVertical_bias="0.1" />

    <androidx.core.widget.ContentLoadingProgressBar
            android:id="@+id/progressBar"
            style="?android:attr/progressBarStyleLarge"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:visibility="invisible"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toBottomOf="@id/loginAsBobButton" />

    <TextView
            android:id="@+id/connectionStatusTextView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textColor="@color/colorPrimary"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toBottomOf="@id/progressBar"
            tools:text="Connection status" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

## Update `LoginViewModel`

Replace `LoginViewModel.kt` file content with below code snippet:

```kotlin
package com.vonage.tutorial.voice

import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import com.nexmo.client.NexmoClient
import com.nexmo.client.request_listener.NexmoConnectionListener.ConnectionStatus

class LoginViewModel : ViewModel() {

    private val navManager = NavManager

    private val _connectionStatus = MutableLiveData<ConnectionStatus>()
    val connectionStatus = _connectionStatus as LiveData<ConnectionStatus>

    private val client: NexmoClient = TODO("Retrieve NexmoClient instance")

    init {
        TODO("Add client connection listener")
    }

    fun onLoginUser(user: User) {
        TODO("Login user")
    }
}
```

### Get NexmoClient instance

To retrieve client instance inside `LoginViewModel` class. Usually, it would be provided it via injection, but for tutorial purposes you will retrieve instance directly using static method. Replace the `client` property in the `LoginViewModel` class:

```kotlin
private val client = NexmoClient.get()
```

### Login user

Your user must be authenticated to be able to participate in the Conversation. Replace the `onLoginUser` method inside `LoginViewModel` class:

```kotlin
fun onLoginUser(user: User) {
    if (user.jwt.isNotBlank()) {
        client.login(user.jwt)
    }
}
```

> **NOTE:** Inside `LoginFragment` class, explore the `loginUser` method. This method is called when one of the two `Login ...` buttons are clicked. This method calls the above `onLoginUser` method. 

### Monitor connection state

When a successful connection is established you need to navigate user to `MainFragment`. Locate the `init` block inside `LoginViewModel` class and replace it's body:


```kotlin
class LoginViewModel : ViewModel() {
    init {
        client.setConnectionListener { newConnectionStatus, _ ->

            if (newConnectionStatus == ConnectionStatus.CONNECTED) {
                val navDirections = LoginFragmentDirections.actionLoginFragmentToMainFragment()
                navManager.navigate(navDirections)
                
                return@setConnectionListener
            }

            _connectionStatus.postValue(newConnectionStatus)
        }
    }

    // ...
}
```

The above code will monitor connection state and if the user is authenticated (`ConnectionStatus.CONNECTED`) it will navigate the user to the `MainFragment`, otherwise it will emit connection status to the UI (`Loginfragment`).

## Update `LoginFragment`

Replace `LoginFragment.kt` file content with below code snippet:

```kotlin
package com.vonage.tutorial.voice

import android.os.Bundle
import android.view.View
import android.widget.Button
import android.widget.ProgressBar
import android.widget.TextView
import android.widget.Toast
import androidx.core.view.isVisible
import androidx.fragment.app.Fragment
import androidx.fragment.app.viewModels
import androidx.lifecycle.Observer
import com.nexmo.client.request_listener.NexmoConnectionListener.ConnectionStatus
import kotlin.properties.Delegates

class LoginFragment : Fragment(R.layout.fragment_login) {

    private val viewModel by viewModels<LoginViewModel>()

    private lateinit var loginAsAliceButton: Button
    private lateinit var loginAsBobButton: Button
    private lateinit var progressBar: ProgressBar
    private lateinit var connectionStatusTextView: TextView

    private var dataLoading: Boolean by Delegates.observable(false) { _, _, newValue ->
        loginAsAliceButton.isEnabled = !newValue
        loginAsBobButton.isEnabled = !newValue
        progressBar.isVisible = newValue
    }

    private val stateObserver = Observer<ConnectionStatus> {
        connectionStatusTextView.text = it.toString()

        if (it == ConnectionStatus.DISCONNECTED) {
            dataLoading = false
        }
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        viewModel.connectionStatus.observe(viewLifecycleOwner, stateObserver)

        loginAsAliceButton = view.findViewById(R.id.loginAsAliceButton)
        loginAsBobButton = view.findViewById(R.id.loginAsBobButton)
        progressBar = view.findViewById(R.id.progressBar)
        connectionStatusTextView = view.findViewById(R.id.connectionStatusTextView)

        loginAsAliceButton.setOnClickListener {
            loginUser(Config.alice)
        }

        loginAsBobButton.setOnClickListener {
            loginUser(Config.bob)
        }
    }

    private fun loginUser(user: User) {
        if (user.jwt.isBlank()) {
            Toast.makeText(context, "Error: Please set Config.${user.name.toLowerCase()}.jwt", Toast.LENGTH_SHORT)
        } else {
            viewModel.onLoginUser(user)
            dataLoading = true
        }
    }
}
```

# Run the app

You can either launch the app on the physical phone (with [USB Debugging enabled](https://developer.android.com/studio/debug/dev-options#enable)) or create a new [Android Virtual Device](https://developer.android.com/studio/run/managing-avds). When device is present press `Run` button: 

```screenshot
image: public/screenshots/tutorials/client-sdk/android-shared/launch-app.png
```

You should see login screen with `Login Alice` button. After clicking user will login and empty main screen will open.
