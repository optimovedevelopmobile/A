

-   [Introduction](#Introduction)
-   [Android SDK Setup](#Android%20SDK%20Setup)
	- [Basic Setup](#Basic%20Setup)
	- [Advanced Setup](#Advanced%20Setup)
-   [Track](#Track)
	-   [Linking App Visitors to Registered User IDs](#Linking%20Visitors%20to%20Users)
	-   [Tracking a Screen Visit Event](#Tracking%20a%20Screen%20Visit)
	-   [Reporting Custom Events](https://docs.optimove.com/optimove-sdk/#Web_Reporting_Events)
-   [Trigger](#Trigger)
 	- [Executing via Optimail](#trigger-optimail)
	- [Executing via Optimove APIs](#trigger-api)

<br>

# <a id="Introduction"></a>Introduction
Marketers use the [Optimove Relationship Marketing Hub](https://www.optimove.com/product) to automate the execution of highly-personalized customer communications. Optimove offers its clients an efficient way to report data from their websites and trigger campaigns accordingly.

This guide will show you how to setup the Android SDK in order to:

 - Track visitor and customer actions and events
 - Trigger Realtime campaigns

The SDK is supported by Android native applications.<br>
The SDK supports a **minimum version of Android 4.4**

<br>

# <a id="Android SDK Setup"></a>Android SDK Setup
Use the Basic Setup (required) in order to:

 - Track visitor and customer actions and events
 - Execute Push Notifications ([OptiPush](https://github.com/optimove-tech/A/blob/master/O/O.md))

## <a id="Basic Setup"></a>Basic Setup


### **1. Request a Mobile SDK from Optimove**

Before implementing the Optimove Track & Trigger to report visitor / customer activities or perform other functions ([OptiPush](https://github.com/optimove-tech/A/blob/master/O/O.md)), you will need to contact your Optimove Customer Success Manager (CSM) or Optimove point of contact and send the below details with a request for your Tenant and Mobile SDK configuration details in order incorporate into your Android app.

Send the following information with your request: 
1.	The app's package
2.	The SHA256 cert fingerprint (can be retrieved using: `keytool -list -v -keystore my-release-key.keystore`)

Receive a *tenant_information_suite* from the Optimove Integration Team that contains: 
1.	**End-point URL** – The URL where the tenant configurations reside
2.	**Unique Optimove token** – The actual token, unique per tenant
3.	**Configuration name** – The version of the desired configuration

For additional technical details, please use our [Android GitHub repository](https://github.com/optimoveintegrationmobile/android-sdk).


### **2. Add the Optimove Repository to Your Project**

1.  Open the project’s `build.gradle` file (located under the application’s root folder).
2.  Under `allprojects`, locate the `repositories` object.
3.  Add the **optimove-sdk** repository:

```javascript
maven {
  url  "https://mobiussolutionsltd.bintray.com/optimove-sdk"
}
```
### **3. Download the SDK**

1.	Open the app’s `build.gradle` file (located under the application’s app module folder).
2.	Under `dependencies`, add the following:

```javascript
compile 'com.optimove.sdk:optimove-sdk:1.0.0'
```
### **4. Run the SDK**

>**Note**: Skip this step if the application already has a working subclass of the `Application` object.

Create a new subclass of `Application` and override the `onCreate` method. Then, add the new object’s name to the ***manifest*** under the `application` ***tag*** as the `name` ***attribute***:

```java
public class MyApplication extends Application {

  @Override
  public void onCreate() {
    super.onCreate();
  }
}
```
```xml
<application
  android:name=".MyApplication">
  .
  .
  .
</application>
```
___
Using the provided Tenant token, a `Context` instance and a flag indicating whether the hosting application has its own **_Firebase SDK_** create a new `TenantInfo` object, initialize the `Optimove` singleton via `Optimove.configure`. 
<br>
The initialization must be called *as soon as possible*, preferably after the call to `super.onCreate()` in the `onCreate` callback.

```java
public class MyApplication extends Application {

  @Override
  public void onCreate() {

    super.onCreate();
    TenantInfo tenantInfo = new TenantInfo("https://optimove.mobile.demo/sdk-configs/", //The initEndPointUrl
                                              "abcdefg12345678", //The token
                                              "myapp.android.1.0.0", //The config name
                                              false); //Has Firebase
    Optimove.configure(this, tenantInfo);
  }
}
```
### **5. Important Installation and Usage Notes**
If Your App Already Uses Firebase

The Optimove SDK for Android is dependent upon the Firebase Android SDK. If your app already uses Firebase SDK or has a dependency with Firebase SDK, a build conflict or runtime exception may occur, due to backward compatibility issues. Therefore, it is highly recommended to match the application’s Firebase SDK version to Optimove’s Firebase SDK:

| Optimove SDK Version | Firebase SDK Version |
|----------------------|----------------------|
| 1.0.7                | 11.8.0               |

Multiple FirebaseMessagingServices

If your app utilizes Firebase Cloud Messaging and implements the **_`FirebaseMessagingService`_** Android's **_Service Priority_** kicks in, then, the app developer **must** explicitly call the `OptipushMessagingHandler` like this:

```java
public class MyMessagingService extends FirebaseMessagingService {

    @Override
    public void onMessageReceived(RemoteMessage remoteMessage) {
        super.onMessageReceived(remoteMessage);
        new OptipushMessagingHandler(this).onMessageReceived(remoteMessage);
    }
}
```
Initializing the Default FirebaseApp Manually

Firebase usually takes care of its own initialization. Usually when using **_Firebase_** usually takes care of its own initialization. However, there are cases in which it is desired to initialize the **_default FirebaseApp_** manually. In these special cases, be advised that calling the `Optimove.configure` before the `FirebaseApp.initializeApp` leads to a `RuntimeException` since the **_default FirebaseApp_** must be initialized before any other **_secondary FirebaseApp_**, which in this case would be triggered by the _Optimove Android SDK_.
<br>
**State Registration**
The SDK initialization process occurs asynchronously, off the `Main UI Thread`.<br>
Before calling the Public API methods, make sure that the SDK has finished initialization by calling the `registerStateListener` method with an instance of `OptimoveStateListener`.<br>
>**Note**: If the object implementing the `OptimoveStateListener` is a component with a _"Lifecycle"_ (i.e. `Activity` or `Fragment`), **_always_** unregister that object at the `onStop()` callback to prevent memory leaks:<br>

```java
public class MainActivity extends AppCompatActivity implements OptimoveStateListener {

  @Override
  protected void onStart() {

    super.onStart();
    Optimove.getInstance().registerStateListener(this);
  }

  @Override
  protected void onStop() {

    super.onStop();
    Optimove.getInstance().unregisterStateListener(this);
  }

  @Override
  public void onConfigurationStarted() {
    
  }

  @Override
  public void onConfigurationSucceed(MissingPermissions... missingPermissions) {

    //If appropriate, ask for permissions here
    //Do any call to the Optimove SDK safely in here
  }

  @Override
  public void onConfigurationFailed(OptimoveStateListener.Error... errors) {

    Log.d("OptimoveSDK", Arrays.deepToString(errors));
  }
}
```
### **6. Reporting Visitor and Customer activity**
You will also need to include the following steps to complete the basic setup:

 - Reporting User Activities and Events
 - [Linking App Visitors to Registered User IDs](#Linking%20Visitors%20to%20Users)
<br>
## <a id="Advanced Setup"></a>Advanced Setup

Use the Advanced Setup (optional) in order to track visitor and customer customized actions and events.
As described in [Reporting Custom Events](https://github.com/optimove-tech/SDK-Custom-Events-for-Your-Vertical), this step requires collaboration between you and Optimove’s Integration Team. Please contact your Optimove Customer Success Manager (CSM) or Optimove point of contact to schedule a meeting with the Product Integration team.

>**Note**: You can deploy the basic setup, followed by adding the advanced setup at a later stage. The Basic Setup is a pre-requisite.

<br>

# <a id="Track"></a>Track

## <a id="Linking Visitors to Users"></a>Linking App Visitors to Registered User IDs

Once the user has downloaded the application and the *OptimoveSDK* for Android has run for the first time, the user is considered a *visitor*, i.e., an unidentified person.

Once the user authenticates and becomes identified by a known `PublicCustomerId`, then the user is considered a *customer*. As soon as this happens for each individual user, call the `SetUserId` function to pass the `CustomerId` to the Optimove singleton:

```java
Optimove.getInstance().setUserId("a-unique-user-id");
```
       
>**Note:** 
>
 >- The `CustomerId` is usually provided by the server application that manages customers, and is the same ID provided to Optimove during the daily customer data transfer. 
 >- Due to its high importance, `set (userId:)` may be called at any time, regardless of the SDK’s state
> - If you will be sending encrypted userId, please follow the steps in [Reporting encrypted CustomerIDs](https://github.com/optimove-tech/Reporting-Encrypted-CustomerID)
 
<br>

## <a id="Tracking a Screen Visit"></a>Tracking a Screen Visit Event

To track which screens the user has visited in your app, call the `reportScreenVisit` method of the Optimove singleton. It can accept either the current `Activity` or, for more finely tuned screen hierarchy reporting, a `String` describing the Screen's hierarchy.

```java
public class MainActivity extends AppCompatActivity {
  @Override
  public void onConfigurationSucceed() {

    Optimove.getInstance().reportScreenVisit(this, "Main");
  }
}
```
```java
public class CheckoutFragment extends Fragment {
  @Override
  public void onConfigurationSucceed() {

    Optimove.getInstance().reportScreenVisit("Main/Footwear/Boots/ConfirmOrder", "Checkout");
  }
}
```


<br>

## Reporting Custom Events

Optimove clients may use the Optimove Mobile SDK to track specific customer actions and other custom events to Optimove (beyond the OOTB events such as visits). This data is used for tracking visitor and customer behavior, targeting campaigns to specific visitor and/or customer segments and triggering campaigns based on particular visitor and/or customer actions/events.

Each Optimove client has a tailored set of customer actions that may be reported via the SDK. As mentioned above, you will collaborate with the Optimove Integration Team to define the particular set of custom events that your website will be able to report. This approach allows you to define any event and its associated parameters.

Once you and the Optimove Integration Team have together defined the custom events supported by your app, the Integration Team will implement your particular functions within your Optimove site, while you will be responsible for implementing the `OptimoveEvent` protocol of the individual events within your app using the appropriate function calls.

To see examples of Custom Events, please visit [Defining the Set of Custom Tracking Events](https://github.com/optimove-tech/SDK-Custom-Events-for-Your-Vertical) that You Will Report for more information.

>**Note**: While you can always add/change the custom events and parameters at a later date (by speaking with the Optimove Integration Team), only the particular custom events that you and the Optimove Integration Team have already defined together will be supported by your Optimove site.

This `OptimoveEvent` protocol defines two properties:

1.	**String getName()** – Declares the custom event’s name
2.	**Map<String, Object> getParameters()** – Specifies the custom event’s parameters.

Then, send that event through the `reportEvent` method of the `Optimove` singleton:

```java

public class MainActivity extends AppCompatActivity implements OptimoveStateListener {

  public void onClick(View view) {
    MyCustomEvent event = new MyCustomEvent(12, "diamond");
    Optimove.getInstance().reportEvent(event);
  }
}

class MyCustomEvent implements OptimoveEvent {

  private int prizeValue;
  private String itemOfInterest;

  public MyCustomEvent(int prizeValue, String itemOfInterest) {

    this.prizeValue = prizeValue;
    this.itemOfInterest = itemOfInterest;
  }

  @Override
  public String getName() {

    return "my_custom_event";
  }

  @Override
  public Map<String, Object> getParameters() {

    Map<String, Object> params = new HashMap<>();
    params.put("prize_value", prizeValue);
    params.put("item_of_interest", itemOfInterest);
    return params;
  }
}
```


>**Notes**:
> - As already mentioned, all [custom events](https://github.com/optimove-tech/SDK-Custom-Events-for-Your-Vertical) must be pre-defined in your Tenant configurations by the Optimove Integration Team.
> - Reporting of custom events is only supported if you have the Mobile SDK implemented.
 >- Events use snake_case as a naming convention. Separate each word with one underscore character (_) and no spaces. (e.g., Checkout_Completed)

<br>

# <a id="Trigger"></a>Trigger

## <a id="trigger-optimail"></a>Executing via Optimail
Ability to execute Realtime campaigns using Optimove’s Optimail email service provider (ESP) add-on product - **Coming Soon**.

For more information on how to add Optimail to your account, please contact your CSM or your Optimove point of contact.

## <a id="trigger-api"></a>Executing via Optimove APIs
Ability to execute Realtime campaigns for mobile native app using Optimove’s APIs -**Coming Soon**

For more information on how to acquire an API key to use Optimove APIs, please request one from your CSM or your Optimove point of contact.



