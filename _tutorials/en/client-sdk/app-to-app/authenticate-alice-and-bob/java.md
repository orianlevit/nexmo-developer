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

Replace `ViewModel.java` file content with below code snippet:

```java
package com.vonage.tutorial.voice;

import androidx.lifecycle.LiveData;
import androidx.lifecycle.MutableLiveData;
import androidx.lifecycle.ViewModel;
import com.nexmo.client.NexmoClient;
import com.nexmo.client.request_listener.NexmoConnectionListener.ConnectionStatus;

public class LoginViewModel extends ViewModel {

    private NexmoClient client = null; // TODO: Retrieve NexmoClient instance

    NavManager navManager = NavManager.getInstance();

    User user = null;

    private MutableLiveData<ConnectionStatus> _connectionStatusMutableLiveData = new MutableLiveData<>();
    public LiveData<ConnectionStatus> connectionStatusLiveData = _connectionStatusMutableLiveData;

    public LoginViewModel() {
        // TODO: Add client connection listener
    }

    void onLoginUser(User user) {
        // TODO: Login user
    }
}
```

### Get NexmoClient instance

To retrieve client instance inside `LoginViewModel` class. Usually, it would be provided it via injection, but for tutorial purposes you will retrieve instance directly using static method. Replace the `client` property in the `LoginViewModel` class:

```java
private NexmoClient client = NexmoClient.get();
```

### Login user

Your user must be authenticated to be able to participate in the Conversation. Replace the `onLoginUser` method inside `LoginViewModel` class:

```java
void onLoginUser(User user) {
    this.user = user;

    if (!user.jwt.trim().isEmpty()) {
        client.login(user.jwt);
    }
}
```

> **NOTE:** Inside `LoginFragment` class, explore the `loginUser` method. This method is called when one of the two `Login ...` buttons are clicked. This method calls the above `onLoginUser` method. 

### Monitor connection state

When a successful connection is established you need to navigate user to `MainFragment`. Locate the `LoginViewModel` constructor and replace its body:


```java
public class LoginViewModel extends ViewModel {

    // ...

    public LoginViewModel() {
        client.setConnectionListener((connectionStatus, connectionStatusReason) -> {
            if (connectionStatus == ConnectionStatus.CONNECTED) {
                NavDirections navDirections = LoginFragmentDirections.actionLoginFragmentToMainFragment(user.getName());
                navManager.navigate(navDirections);
                return;
            }

            _connectionStatusMutableLiveData.postValue(connectionStatus);
        });
    }

    // ...
}
```

The above code will monitor connection state and if the user is authenticated (`ConnectionStatus.CONNECTED`) it will navigate the user to the `MainFragment`, otherwise it will emit connection status to the UI (`Loginfragment`).

## Update `LoginFragment`

Replace `LoginFragment.java` file content with below code snippet:

```java
package com.vonage.tutorial.voice;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.ProgressBar;
import android.widget.TextView;
import android.widget.Toast;
import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;
import androidx.lifecycle.ViewModelProvider;
import com.nexmo.client.request_listener.NexmoConnectionListener.ConnectionStatus;

public class LoginFragment extends Fragment {

    private Button loginAsAliceButton;
    private Button loginAsBobButton;
    private ProgressBar progressBar;
    private TextView connectionStatusTextView;

    private LoginViewModel viewModel;

    public LoginFragment() {
        super(R.layout.fragment_login);
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        viewModel = new ViewModelProvider(requireActivity()).get(LoginViewModel.class);

        viewModel.connectionStatusLiveData.observe(getViewLifecycleOwner(), connectionStatus -> {
            connectionStatusTextView.setText(connectionStatus.toString());

            if (connectionStatus == ConnectionStatus.DISCONNECTED) {
                setDataLoading(false);
            }
        });

        loginAsAliceButton = view.findViewById(R.id.loginAsAliceButton);
        loginAsBobButton = view.findViewById(R.id.loginAsBobButton);
        progressBar = view.findViewById(R.id.progressBar);
        connectionStatusTextView = view.findViewById(R.id.connectionStatusTextView);

        loginAsAliceButton.setOnClickListener(it -> loginUser(Config.getAlice()));
        loginAsBobButton.setOnClickListener(it -> loginUser(Config.getBob()));
    }

    private void setDataLoading(Boolean dataLoading) {
        loginAsAliceButton.setEnabled(!dataLoading);
        loginAsBobButton.setEnabled(!dataLoading);

        int visibility;

        if (dataLoading) {
            visibility = View.VISIBLE;
        } else {
            visibility = View.GONE;
        }

        progressBar.setVisibility(visibility);
    }

    private void loginUser(User user) {
        if (user.jwt.trim().isEmpty()) {
            Toast.makeText(getActivity(), "Error: Please set Config." + user.name + " jwt", Toast.LENGTH_SHORT).show();
        } else {
            viewModel.onLoginUser(user);
            setDataLoading(true);
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
