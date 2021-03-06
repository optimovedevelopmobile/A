

-  [Introduction](#Introduction)
 - [Setup](#Setup)
	 - [Pre-Requisites](pre-reqs)
	 - [Deep Linking](deep%20linking)
	 - [Enabling Test Mode](test%20mode) 
 - [Post-Setup](Post%20setup)
	 - [Create & test notification templates](notification%20template) 
	 - [Set up an Optipush campaign](Optipush%20campaign) 


# <a id="Introduction"></a>Introduction

_*Optipush*_ is Optimove’s mobile push notification delivery add-in module, powering all aspects of preparing, delivering and tracking mobile push notification communications to customers, seamlessly from within Optimove.</br>
 _*Optimove SDK*_ for iOS includes built-in functionality for receiving push messages, presenting notifications in the app UI and tracking user responses.


# <a id="Setup"></a>Setup

## <a id="pre-reqs"></a>Pre-Requisites 

### 1. [Optimove Mobile SDK for Android](https://github.com/optimove-tech/A) implemented 

### 2.  App uses Java 8
Optimove's Android SDK uses the `Java 8`. If the hosting app still uses `Java 7` or `Jack` some build errors might occur. In that case migrate the app to `Java 8`. It is fast, simple and enables new and useful language features.

>For more information about _Android Official Java 8 Support_ checkout [this Developer's Guide](https://developer.android.com/studio/write/java8-support) article

Add the following to the app module's `build.gradle` file, under the `android` object.

```javascript
compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
}
```
## <a id="deep linking"></a>Deep Linking

### Setting up Optipush Deep Linking
In order to route end users back to the application from the notification, you must support *Deep Linking*.
Other than _UI attributes_, an **_Optipush Notification_** can contain metadata that can lead the user to a specific screen within the hosting application, alongside custom (screen specific) data.<br>
To support deep linking, update application's `manifest.xml` file to reflect which screen can be targeted. Each `Activity` the can be targeted must have the following _**`intent-filter`**_:

```xml
<intent-filter>
  <action android:name="android.intent.action.VIEW"/>

  <category android:name="android.intent.category.DEFAULT"/>
  <category android:name="android.intent.category.BROWSABLE"/>

  <data
    android:host="replace.with.the.app.package" 
    android:pathPrefix="/replace_with_a_custom_screen_name"
    android:scheme="http"/>
</intent-filter>
```

To support **_custom deep linking data_** pass a `LinkDataExtractedListener` to an instance of `DeepLinkHandler` inside the targeted `Activity`.<br>
The `deepLinkHandler` calls either the _**`onDataExtracted(Map<String, String> data)`**_ with the deep linking data in case of successful extraction, or the _**`onErrorOccurred(LinkDataError error)`**_ if any error occurred. 

```java
public class MyTargetedActivity extends AppCompatActivity implements LinkDataExtractedListener {

  public static final String EMPLOYEE_EXTRA_KEY = "employee";

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_employee_page);
    new DeepLinkHandler(getIntent()).extractLinkData(this);
  }

  @Override
  public void onDataExtracted(Map<String, String> data) {
    //Do any custom behavior according to the provided data Map
  }

  @Override
  public void onErrorOccurred(LinkDataError error) {
    //Handle errors in any way fitted
  }
}
```
<br>

### Deep Linking to the Main Activity

If the **_Main Activity_** (i.e. has `<intent-filter>` with `<action android:name="android.intent.action.MAIN"/>` and `<category android:name="android.intent.category.LAUNCHER"/>`) needs to be targeted by a **_deep link_**, add `android:launchMode="singleInstance"` to the activity's declaration. `singleInstance` ensures that if an _Optipush_ notification is open while the _application_ is running (either in the **foreground** or **background**), **_Android_** will not start a new `Task`, nor will it kill the current one, but will call the `onNewIntent(Intent intent)` with the notification's `Intent`.

`manifest.xml`
```xml
<activity android:name=".MainActivity"
  android:launchMode="singleInstance">
  <intent-filter>
    <action android:name="android.intent.action.MAIN"/>
    <category android:name="android.intent.category.LAUNCHER"/>
  </intent-filter>
  <intent-filter>
    <action android:name="android.intent.action.VIEW"/>

    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>

    <data
      android:host="replace.with.the.app.package" 
      android:pathPrefix="/replace_with_a_custom_screen_name"
      android:scheme="http"/>
  </intent-filter>
</activity>
```

`MainActivity.java`
```java
  public class MainActivity extends AppCompatActivity {

    @Override
    protected void onNewIntent(Intent intent) {
      super.onNewIntent(intent);
      new DeepLinkHandler(intent).extractLinkData(this);
  }

  @Override
  public void onDataExtracted(Map<String, String> data) {
    //Do any custom behavior according to the provided data Map
  }

  @Override
  public void onErrorOccurred(LinkDataError error) {
    //Handle errors in any way fitted
  }
}
```
<br>

## <a id="test mode"></a>Enabling Test Mode
 
You can test an **Optipush template** on your device *before* having to create an **Optipush campaign**.
To **enable** _"test Optipush templates"_ on one or more devices, call the<br>_**`Optimove.getInstance().startTestMode(@Nullable SdkOperationListener operationListener);`**_<br>method.
To **disable** _"test Optipush templates"_ call the<br>_**`Optimove.getInstance().stopTestMode(@Nullable SdkOperationListener operationListener);`**_.
```java
public class MainActivity extends AppCompatActivity implements OptimoveSuccessStateListener {

  public void startTestModeClickListener() {
    Optimove.getInstance().startTestMode(success -> {
      if (success) {
        Toast.makeText(this, "Test Mode is ON", Toast.LENGTH_SHORT).show();
      } else {
        Toast.makeText(this, "Failed to start Test Mode", Toast.LENGTH_SHORT).show();
      }
    });
  }

  @Override
  protected void onStop() {

    super.onStop();
    //Can be called even if test mode was not really started
    Optimove.getInstance().stopTestMode(success -> {
      if (success) {
        Toast.makeText(this, "Test Mode is OFF", Toast.LENGTH_SHORT).show();
      } else {
        Toast.makeText(this, "Failed to stop Test Mode", Toast.LENGTH_SHORT).show();
      }
    });
  }
}
```

>**Notes:**
>- It is recommended to maintain 2 apps - one with test mode enabled (for internal purposes ONLY) and one without test mode enabled (for the general public).
>- The app that is published to the App Store should NOT have the test mode enabled.
<br>

# <a id="Post setup"></a>Post-Setup


## <a id="notification template"></a>Create & Test notification templates
Once Optimove has enabled Optipush as an execution channel for your Optimove instance, you can begin creating and testing your first Optipush template.
>Note: In order to be able to test your templates, the test mode must be [enabled](https://github.com/optimove-tech/A/tree/master/O/O%20for%20A#enabling-test-mode) within your mobile app.<br>


### Create  an Optipush Template

 1. Go to the Manage Templates page and choose 'Optipush' from the Channel drop-down menu.
 2. Enter values for the following fields:
	 - Template Name- Name the template 
	 - Template Title - Title of push template
	 - Message - Message of the template
![](https://raw.githubusercontent.com/optimove-tech/A/master/O/O%20for%20iOS/images/3.png)

 3. Personalization - you can personalize the notification by adding dynamic tags and emojis to the notification.
 4. Preview - you can preview the push template notification before sending it out.
 5. **Deep links** (Optional) - choose the app (Android) and select the target screen in your app that you want your customers to be directed to when opening the message. <br>
 >Notes:
 >- In order to add Deep Links to the drop-down menu, please send the list of screen names to your CSM that you have configured in your app as described [here](https://github.com/optimove-tech/A/tree/master/O/O%20for%20A#deep-linking).
 >- *If a Deep Link is not chosen, the customer will be directed to the main screen when opening the message.* 
 >- When creating templates for Optipush, if the template is targeted for a specific device (iOS/Android), it is recommended to add the device name to the template naming convention. This way it will be identifiable when choosing a template for a campaign targeting a specific device.<br>
 

### Test an Optipush Template

1. **Validate** - validates the push notification template to make sure all mandatory fields are completed and contain no errors. 
2. **Send Test**  - clicking this link will send the push notification template to all devices that have the app installed with the test mode enabled.<br>

## <a id="Optipush campaign"></a>Set up an Optipush campaign

### Run Campaign

Please follow these steps in order to run a pre-scheduled campaign via execution channel **Optipush**.
1. From the main menu go on *One-to-One Campaigns* --> click on *More* from the drop-down menu --> click on ***Run Campaign***.  
2. Go through Steps 1 & 2 of the Run Campaign wizard as you would for any campaign.
3. In Step 3 (Execution Details) choose from the *Channel* drop-down menu *Optipush*. This action will open the **Optipush Options** window.<br>
![](https://raw.githubusercontent.com/optimove-tech/A/master/O/O%20for%20iOS/images/2_a.png)

4. Choose from the *App* drop-down menu if you would like to run the campaign for your iOS app, Android app, or both by selecting the relevant box(es).
5. Choose the relevant template for the *Template* drop down menu that you would like the targeted audience to receive.
6. Continue through the remaining steps of the Run Campaign wizard to schedule the campaign for your preferred dates and times.
