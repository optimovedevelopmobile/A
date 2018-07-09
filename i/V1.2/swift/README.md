
- [Introduction](#Introduction)
- [Basic Setup](#Basic%20Setup)
- [Advanced Setup](#Advanced%20Setup)
-   [Track](#Track)
	-   [Stitching App Visitors to Registered Customer IDs](#Stitching%20Visitors%20to%20Users)
	-   [Tracking Screen Visits](#Tracking%20a%20Screen%20Visit)
	-   [Tracking Custom Events](https://docs.optimove.com/optimove-sdk/#Web_Reporting_Events)
-   [Trigger](#Trigger)
 	- [Executing via Optimail](#trigger-optimail)
	- [Executing via Optimove APIs](#trigger-api)
	- [Executing via Optipush](trigger-optipush)
<br>

# <a id="Introduction"></a> Introduction

Marketers use the [Optimove Relationship Marketing Hub](https://www.optimove.com/product) to automate the execution of highly-personalized customer communications. Optimove offers its clients an efficient way to report data from their websites and trigger campaigns accordingly.

This guide will show you how to setup the iOS (Swift) SDK in order to:

 - Track visitor and customer actions and events
 - Trigger Realtime campaigns


# <a id="Basic Setup"></a> Basic Setup

## **Request a Mobile SDK from Optimove**

Before implementing the Optimove Track & Trigger to track visitor / customer activities or perform other functions ([OptiPush](https://github.com/optimove-tech/A/blob/master/O/O.md)), you will need to contact your Optimove Customer Success Manager (CSM) or Optimove point of contact. <br>
To get started, please follow these instructions: 

### 1. Pre-requisites <br>

1. You have a paid development account for your iOS app, and valid certificates for remote notifications or APN Auth key.
2. The app's Deployment Target is at least iOS 10.0
3. Your Cocoapods version is 1.5 or above
4. You have a Firebase version of 5.4.0 (Optimove SDK is dependent on Firebase version 5.4.0)


### 2. Provide your iOS app details: <br>

Send the following information to your CSM or Optimove POC with your request for your Mobile SDK configuration details in order to incorporate into your iOS app :<br>

1.	***Auth key*** (with its key id) P8 format
2.	***Bundle ID*** (If you are using multiple apps for development/testing purposes, please provide a list of all bundle IDs being used for all environments.)
3.	***Team ID*** (from apple developer dashboard)
4.	***App Store ID*** (from itunesconnect)<br>
  

### 3. Retrieve *tenant_information_suite* details: <br>


After providing the info above, you will receive a *tenant_information_suite* from the Optimove Product Integration Team that contains:<br>
1.	***End-point URL*** – The URL where the tenant configurations reside
2.	***Unique Optimove token*** – The actual token, unique per tenant
3.	***Configuration name*** – The version of the desired configuration

For a demo application containing the iOS SDK, please use our [iOS GitHub repository](https://github.com/optimove-tech/iOS-SDK-Integration-Guide/tree/master/DemoApplication/SwiftDemoApp).

    

## Setting up the iOS SDK

### 1. Install the SDK 
In your Application Target Capabilities: 
1. Enable `push notifications` capabilities <br>
2. Enable`remote notifications` capabilities in`Background Modes`<br> 
[![apple_dashboared.png](https://s9.postimg.cc/9ln5sfxe7/apple_dashboared.png)](https://postimg.org/image/itfe954gb/)



### 2.  Download *OptimoveSDK* pod
In order to work with the Optimove SDK for your iOS native app, need to download its module from CocoaPods.

1. In your Podfile, add the following:

`pod OptimoveSDK`


Example:
```ruby

platform :ios, '10.0'

target 'iOSDemo' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  pod 'OptimoveSDK'

end
```

2. `OptimoveSDK` relies on other modules as infrastructure, such as `Firebase`, so when you download `OptimoveSDK` you get the following frameworks:

* `Firebase/Core` version `5.4.0`

* `Firebase/Messaging`

* `Firebase/DynamicLinks`

### 3. If you are using Optipush, see additional configurations [here](https://github.com/optimove-tech/A/tree/master/O/O%20for%20iOS/O%20for%20iOS%20V1.2#2-optipush-configuration)  
    
### Run the SDK

In your _`AppDelegate`_ class, inside the application

  
_`application (_: didFinishLaunchingWithOptions:)`_ method, create a new `OptimoveTenantInfo` object. This object should contain:

  
1. The end-point URL

2. Unique Optimove token

3. The configuration version provided by the Optimove Integration Team

4. Indication for existing Firebase module inside your application

5. Indication for using your own `Firebase/Messaging`

Use this object as an argument for the SDK function,

  
```swift
   Optimove.sharedInstance.configure(info:)
```
  
This call initializes the OptimoveSDK singleton.

6. You should register for notification certificate from APN using `UIApplication.shared.registerForRemoteNotifications()`

Apply items 7-8 if using **Optipush**:
7. Also, if you'd like to present the notifications to the user, request authorization for that using:

`UNUserNotificationCenter.current().requestAuthorization(options:completionHandler:)`

8. Finally declare the `AppDelegate` as the `UserNotificationCenter` delegate

9. For steps 6 - 8 you must import `UserNotifications` and `OptimoveSDK` (the last import is necessary in any class that uses `OptimoveSDK` API )

For example:

  

```swift

import UIKit

import UserNotifications

import OptimoveSDK

  
  

@UIApplicationMain

class AppDelegate: UIResponder,

UIApplicationDelegate,

UNUserNotificationCenterDelegate

{

var window: UIWindow?

func application(_ application: UIApplication,

didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {

  

let info = OptimoveTenantInfo(url: "https://optimove.mobile.demo",

token: "abcdefg12345678",

version: "myapp.ios.1.0.0",

hasFirebase: false,

useFirebaseMessaging: false)

Optimove.sharedInstance.configure(for: info)

UNUserNotificationCenter.current().requestAuthorization(options: [.alert]) { (grant, error) in

}

UIApplication.shared.registerForRemoteNotifications()

UNUserNotificationCenter.current().delegate = self

return true

}

```

Note : The initialization must be called as soon as possible, unless you have your own _`Firebase SDK.`_ In this case, start the initialization right after calling `FirebaseApp.configure().

  

Still in `AppDelegate`, please implement the following callbacks:

```swift

func application(_ application: UIApplication,

didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data)

{

Optimove.sharedInstance.application(didRegisterForRemoteNotificationsWithDeviceToken: deviceToken)

}

func application(_ application: UIApplication,

didReceiveRemoteNotification userInfo: [AnyHashable : Any],

fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void)

{

if !Optimove.sharedInstance.didReceiveRemoteNotification(userInfo: userInfo,didComplete: completionHandler)

{

completionHandler(.newData)

	}
}

func userNotificationCenter(_ center: UNUserNotificationCenter,

didReceive response: UNNotificationResponse,

withCompletionHandler completionHandler: @escaping () -> Void) {

if !Optimove.sharedInstance.didReceive(response: response,withCompletionHandler: completionHandler)

{

completionHandler()

	}
}

func userNotificationCenter(_ center: UNUserNotificationCenter,

willPresent notification: UNNotification,

withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {

if !Optimove.sharedInstance.willPresent(notification: notification, withCompletionHandler: completionHandler)

{

completionHandler(.alert)

	}
}
```

All of `Optimove` methods (except the `didRegisterForRemoteNotificationsWithDeviceToken` ) return a boolean value that indicate if the received notification should be handled by `OptimoveSDK` or it is part of the application services and should be handle by you.

  

### 3. Important Installation and Usage Notes

The SDK initialization process occurs asynchronously, off the `Main Thread`.

Before calling the Public API methods, make sure that the SDK has finished initialization by calling the `registerSuccessStateListener(listener:)` method with an instance of _`OptimoveSuccessStateListener`_.
  

When the SDK finished its initializtion process, it'll call the callback `optimove(_:didBecomeActiveWithMissingPermissions:) ` and provide the missing permission that the SDK must have in order to work properly.

### 4. Tracking Visitor and Customer activity
 
You will also need to include the following steps to complete the basic setup:

 - Tracking User Activities and Events
 - [Stitching App Visitors to Registered Customer IDs](Stitching%20Visitors%20to%20Users)
<br>


# <a id="Advanced Setup"></a>Advanced Setup
Use the Advanced Setup (optional) in order to track visitor and customer customized actions and events.
As described in [Tracking Custom Events](https://github.com/optimove-tech/SDK-Custom-Events-for-Your-Vertical), this step requires collaboration between you and Optimove’s Product Integration Team. Please contact your Optimove Customer Success Manager (CSM) or Optimove point of contact to schedule a meeting with the Product Integration team.

>**Note**: You can deploy the basic setup, followed by adding the advanced setup at a later stage. The Basic Setup is a pre-requisite.

<br>
  
# <a id="Track"></a>Track

## <a id="Stitching Visitors to Users"></a>Stitching App Visitors to Registered Customer IDs

Once the user has downloaded the application and the _Optimove SDK_ run for the first time, the user is considered a _Visitor_, i.e. an unidentified person.<br>

Once the user authenticates and becomes identified by a known `PublicCustomerId`, then the user is considered _Customer_.<br>

As soon as this happens for each individual user, pass the `CustomerId` to the `Optimove` singleton like this: <br>

```swift
Optimove.sharedInstance.set(userId:)
```

>**Notes:** 
>
 >- The `CustomerId` is usually provided by the server application that manages customers, and is the same ID provided to Optimove during the daily customer data transfer. 
 >- Due to its high importance, `set (userId:)` may be called at any time.
> - If you will be sending encrypted userId, please follow the steps in [Reporting encrypted CustomerIDs](https://github.com/optimove-tech/Reporting-Encrypted-CustomerID)
 
<br>

## <a id="Tracking a Screen Visit"></a>Tracking Screen Visits
To track which screens the user has visited in your app, send a _setScreenEvent_ message to the shared Optimove instance:.<br>

  
````swift
Optimove.sharedInstance.reportScreenVisit(viewControllersIdentifiers:url)
````

The _`viewControllersIdentifiers`_ argument should include an array that represents the path to the current screen.<br>

To support more complex view hierarchies, you may also specify a screen URL in the second parameter.<br>

## Tracking Custom Events  

Optimove clients may use the Optimove Mobile SDK to track specific customer actions and other custom events to Optimove (beyond OOTB events such visits). This data is used for tracking visitor and customer behavior, targeting campaigns to specific visitor and/or customer segments and triggering campaigns based on particular visitor and/or customer actions/events.  
  
To see examples of Custom Events, please visit [Defining the Set of Custom Tracking Events](https://github.com/optimove-tech/SDK-Custom-Events-for-Your-Vertical) that you will report for more information.

>**Note**: While you can always add/change the custom events and parameters at a later date (by speaking with the Optimove Product Integration Team), only the particular custom events that you and the Optimove Product Integration Team have already defined together will be supported by your Optimove site.

### How to Track a Custom Event from within your iOS app

Once you and the Optimove Product Integration Team have together defined the custom events supported by your app, the Product Integration Team will implement your particular functions within your Optimove site, while you will be responsible for implementing the `OptimoveEvent` protocol of the individual events within your app using the appropriate function calls.<br>

The protocol defines 2 properties:
1.	**name: String** – Declares the custom event’s name
2.	**parameters: [String:Any]** – Defines the custom event's parameters.

Then send that event through the `reportEvent(event:)` method of the `Optimove` singleton.
  
````swift
override func viewDidAppear(_ animated: Bool) {
super.viewDidAppear(animated)
Optimove.sharedInstance.reportEvent(event: MyCustomEvent())
}
````


>**Notes**:
>- As already mentioned, all [custom events](https://github.com/optimove-tech/SDK-Custom-Events-for-Your-Vertical) must be pre-defined in your Tenant configurations by the Optimove Product Integration Team.
>- Reporting of custom events is only supported if you have the Mobile SDK implemented.
 >- Events use snake_case as a naming convention. Separate each word with one underscore character (_) and no spaces. (e.g., Checkout_Completed)
 >- The usage of the `reportEvent` function depends on your needs. This function may include a completion handler that will be called once the report has finished. The default value for this argument is nil.

# <a id="Trigger"></a>Trigger

## <a id="trigger-optimail"></a>Executing via Optimail
Ability to execute campaigns using Optimove’s Optimail email service provider (ESP) add-on product. With Optimail you will be able to:
* Send HTML email campaigns
* Set personalized tags (first name, last name, and more)
* These Tags are retrieved from both your daily data transfer, as well as the SDK events you are tracking.
* Preview campaign email before sending
* Send realtime marketing campaigns based on your website SDK activity triggering rules

For more information on how to add Optimail to your account, please contact your CSM or your Optimove point of contact.

## <a id="trigger-api"></a>Executing via Optimove APIs
You can also trigger Optimove realtime campaigns using Optimove’s APIs:
* Register listener to receive realtime campaign notifications, please refer to RegisterEventListener (where eventid = 11)
* To view your realtime API payload, please refer to [Optimove Realtime Execution Channels](https://docs.optimove.com/optimove-realtime-execution-channels/) (see Method 3: Realtime API) 
For more information on how to acquire an API key to use Optimove APIs, please request one from your CSM or your Optimove point of contact.


## <a id="trigger-optipush"></a>Executing via Optipush:
Ability to execute campaigns using Optimove’s push notification add-on product, Optipush.
_*Optipush*_ is Optimove’s mobile push notification delivery add-in module, powering all aspects of preparing, delivering and tracking mobile push notification communications to customers, seamlessly from within Optimove.<br>  _*Optimove SDK*_ for iOS includes built-in functionality for receiving push messages, presenting notifications in the app UI and tracking user responses.
For instruction on setting up Optipush for iOS click [here](https://github.com/optimove-tech/A/tree/master/O/O%20for%20iOS/O%20for%20iOS%20V1.2).
