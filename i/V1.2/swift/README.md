
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
3. Cocoapods version 1.5 and above
4. Optimove SDK is dependent on Firebase version 5.4.0


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
	1. Enable `push notifications` capabilities 
	2. Enable`remote notifications` capabilities in`Background Modes`

 
[![apple_dashboared.png](https://s9.postimg.cc/9ln5sfxe7/apple_dashboared.png)](https://postimg.org/image/itfe954gb/)

----
### Optipush configuration
1.  In application target capabilities -> App Groups - add the following group using the naming convention:
	group.[*your Bundle Id*].optimove
2. Add **Notification Service Extension** to your project under Targets
3. Under Notification service extension capabilities -> App Groups, select the group you created in #1.
------

In order to work with the Optimove SDK for your iOS native app, need to download its module from CocoaPods.

1. In your Podfile, add the following:

`pod OptimoveSDK`

If you are using Optipush, add the folllowing under the extension target:

  `pod 'OptimoveNotificationServiceExtension`'

Example:
```ruby

platform :ios, '10.0'

target 'iOSDemo' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  pod 'OptimoveSDK'

end

target 'NotificationExtension' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  pod 'OptimoveNotificationServiceExtension'

end
```

2. `OptimoveSDK` relies on other modules as infrastructure, such as `Firebase`, so when you download `OptimoveSDK` you get the following frameworks:

* `Firebase/Core` version `5.4.0`

* `Firebase/Messaging`

* `Firebase/DynamicLinks`


#### Notification Service Extension

In order to enable `Optimove` to track its push notifications, you'll need to add a `Notification Extension` to your project.

Since the `Notification Extension` is a different target, it must use an additional, lean, SDK of `Optimove`. so in your `Podfile` add an additional dependency:

  `pod 'OptimoveNotificationServiceExtension`'
  
`Note` that the extension versioning must be aligned with the application.

Make sure that the extension's `Deployment Target` (found in the project's settings page) is the **same** as the app's `Deployment Target

  In order to enable communication between the extension and the application, add an `App group` capability in both of the targets.

The group name convension should be: `group.<the application bundle id>.optimove`

Notication Extension group is automatically generated.
In the group, go to `NotificationService.swift`  and add
 `import OptimoveNotificationServiceExtension`
 
Inside `class NotificationService`, add
```swift
var optimoveExtensionService:OptimoveNotificationServiceExtension!
```
Inside the `override func didReceive( request: contentHandler:)` initialize the notification extension tenant info:


```swift
let info = NotificationExtensionTenantInfo(endpoint: "htts://www.endpoint.com",
                                                   token: "ios-demo-token",
                                                   version: "ios.demo.version.1",
                                                   appBundleId: "com.optimove.sdk.demo")
        optimoveExtensionService = OptimoveNotificationServiceExtension(tenantInfo: info)
  ```

>Notes: 
>- The values should correspond to the tenant info provided by the Product Integration team.
>- The `appBundleId` refers to the app bundle Id from the application target (not from the Notification extension target)

2 functions to be defined:
1. didReceive
2. serviceExtensionTimeWillExpire

In order to identify if the message should be handled by Optimove, add the following code snippet:
```swift
optimoveExtensionService.didReceive(request, withContentHandler: contentHandler)
        if !optimoveExtensionService.isHandledByOptimove 
```
In order to notify Optimove that your extension is about to be terminated, add the following code snippet:
```swift
override func serviceExtensionTimeWillExpire() {
        if optimoveExtensionService.isHandledByOptimove {
            optimoveExtensionService.serviceExtensionTimeWillExpire()
        } else {
            if let contentHandler = contentHandler, let bestAttemptContent =  bestAttemptContent {
                contentHandler(bestAttemptContent)
            }
        }
    }
```

Example for full implementation:
```swift
override func didReceive(_ request: UNNotificationRequest, withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
        
        let info = NotificationExtensionTenantInfo(endpoint: "htts://www.endpoint.com",
                                                   token: "ios-demo-token",
                                                   version: "ios.demo.version.1",
                                                   appBundleId: "com.optimove.sdk.demo")
        optimoveExtensionService = OptimoveNotificationServiceExtension(tenantInfo: info)
        
        optimoveExtensionService.didReceive(request, withContentHandler: contentHandler)
        if !optimoveExtensionService.isHandledByOptimove {
            self.contentHandler = contentHandler
            bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)
            
            if let bestAttemptContent = bestAttemptContent {
                // Modify the notification content here...
                bestAttemptContent.title = "\(bestAttemptContent.title) [modified]"
                
                contentHandler(bestAttemptContent)
            }
        }
    }
    
    override func serviceExtensionTimeWillExpire() {
        if optimoveExtensionService.isHandledByOptimove {
            optimoveExtensionService.serviceExtensionTimeWillExpire()
        } else {
            if let contentHandler = contentHandler, let bestAttemptContent =  bestAttemptContent {
                contentHandler(bestAttemptContent)
            }
        }
    }
```
      

### Run the SDK

In your _`AppDelegate`_ class, inside the application

  

_`application (_: didFinishLaunchingWithOptions:)`_ method, create a new `OptimoveTenantInfo` object. This object should contain:

  

1. The end-point URL

2. Unique Optimove token

3. The configuration version provided by the Optimove Integration Team

4. Indication for existing Firebase module inside your application

5. Indication for using your own `Firebase/Messaging`

Use this object as an argument for the SDK function,

  

`Optimove.sharedInstance.configure(info:)`

  

This call initializes the OptimoveSDK singleton.

6. You should register for notification certificate from APN using `UIApplication.shared.registerForRemoteNotifications()`

For Optipush
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

  

## Important Installation and Usage Notes

The SDK initialization process occurs asynchronously, off the `Main Thread`.

Before calling the Public API methods, make sure that the SDK has finished initialization by calling the `registerSuccessStateListener(listener:)` method with an instance of _`OptimoveSuccessStateListener`_.

  

When the SDK finished its initializtion process, it'll call the callback `optimove(_:didBecomeActiveWithMissingPermissions:) ` and provide the missing permission that the SDK must have in order to work properly.

  

## Analytics

  

#### set user id

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

### Report Screen Event

To track which screens the user has visited in your app, send a _setScreenEvent_ message to the shared Optimove instance:.<br>

  

````swift

Optimove.sharedInstance.reportScreenVisit(viewControllersIdentifiers:url)

````

The _`viewControllersIdentifiers`_ argument should include an array that represents the path to the current screen.<br>

To support more complex view hierarchies, you may also specify a screen URL in the second parameter.<br>

  
  

### Report Custom Event

To report a _**`Custom Event`**_ (one defined by you and the Optimove Integration Team and already configured in your Optimove site), you must implement the `OptimoveEvent` protocol.<br>

The protocol defines 2 properties:

1. `name: String` - Declares the custom event's name

2. `parameters: [String:Any]` - Defines the custom event's parameters.<br>

Then send that event through the `reportEvent(event:)` method of the `Optimove` singleton.

  

````swift

override func viewDidAppear(_ animated: Bool) {

super.viewDidAppear(animated)

Optimove.sharedInstance.reportEvent(event: MyCustomEvent())

}

````

>Note:<br>

* All _**`Custom Event`**_ must be declared in the _Tenant Configurations_. <br>

* _**`Custom Event`**_ reporting is only supported when OptiTrack Feature is enabled.<br>

>**Notes**:
>- As already mentioned, all [custom events](https://github.com/optimove-tech/SDK-Custom-Events-for-Your-Vertical) must be pre-defined in your Tenant configurations by the Optimove Product Integration Team.
>- Reporting of custom events is only supported if you have the Mobile SDK implemented.
 >- Events use snake_case as a naming convention. Separate each word with one underscore character (_) and no spaces. (e.g., Checkout_Completed)
 >- The usage of the `reportEvent` function depends on your needs. This function may include a completion handler that will be called once the report has finished. The default value for this argument is nil.

## Optipush:

_*Optipush*_ is Optimove’s mobile push notification delivery add-in module, powering all aspects of preparing, delivering and tracking mobile push notification communications to customers, seamlessly from within Optimove.<br>  _*Optimove SDK*_ for iOS includes built-in functionality for receiving push messages, presenting notifications in the app UI and tracking user responses.

All of `OptimoveSDK` method for notification should be implemented in the setup steps.

###Notification Extension

.

.

.

..

  


  

## Deep Link:

  

Other than _UI attributes_, an **_Optipush Notification_** can contain metadata linking to a specific screen within your application, along with custom (screen specific) data.<br>

  

To support deep linking, you should:

  

* Enable Associated Domains:

In your project capabilities, add the dynamic link domain with `applinks:` prefix and without any `https://` prefix

  

![associated_domain.png](https://s9.postimg.cc/hqrw4eqm7/associated_domain.png)

<br>

Any _`ViewControler`_ should recieve DynamicLink data callback, should implement _`didReceive(dynamicLink:)`_, thus conforming to _`OptimoveDeepLinkCallback`_ protocol .<br>

_`OptimoveDeepLinkCallback`_ protocol has one method:

````swift

didReceive(deepLink:)

````

that receives an `OptimoveDeepLinkComponents` as argument

  

A `OptimoveDeepLinkComponents` entity contains two properties:

  

· ScreenName: for the required _*viewcontroller*_ you need to segue to.

  

· Query: That contain the content that should be included in that view controller.

  

example:

````swift

func didReceive(deepLink: OptimoveDeepLinkComponents?)

{

let vc = self.storyboard!.instantiateViewController(withIdentifier: deepLink!.screenName)

self.navigationController?.pushViewController(vc, animated: true)

}

````

  

### Test Optipush Templates

t is usually desirable to test an **_Optipush Template_** on an actual device before sending an **_Optipush Campaign_** to users.<br>

To enable _"test campaigns"_ on one or more devices, call the _**`Optimove.sharedInstance.startTestMode()`**_ method.<br>

To stop receiving _"test campaigns"_ call _**`Optimove.sharedInstance.stopTestMode()`**_.<br>

  

````swift

class ViewController: UIViewController {

override func viewDidAppear(_ animated: Bool) {

super.viewDidAppear(animated)

Optimove.sharedInstance.startTestMode()

}

}

````
