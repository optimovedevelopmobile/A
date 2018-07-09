
-  [Introduction](Introduction)
 - [Setup](Setup)
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

### 1. [Optimove Mobile SDK for iOS](https://github.com/optimove-tech/A) implemented 


### 2. Optipush Configuration

#### Notification Service Extension <br>

In order to enable Optimove to track the push notifications, you'll need to add a **Notification Extension** to your project.

1. Since the `Notification Extension` is a different target, it must use an additional, lean, SDK of Optimove. To do this, in your `Podfile` add an additional dependency:

  `pod 'OptimoveNotificationServiceExtension`'
  
>Notes: 
>- The extension versioning must be aligned with the application.
>- Make sure that the extension's `Deployment Target` (found in the project's settings page) is the **same** as the app's `Deployment Target

See following example containing both `OptimoveSDK` & `OptimoveNotificationServiceExtension`:

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

2. In order to enable communication between the extension and the application, add an `App group` capability in both of the targets.

The group name convention should be: `group.<the application bundle id>.optimove`

3. Add **Notification Service Extension** to your project under Targets
4. Under Notification service extension capabilities -> App Groups, select the group you created in #1.

5. Once you created the group name in #2, *Notification Extension group* is automatically generated.
In this group, go to `NotificationService.swift`  and add
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

6. Within *Notification Extension group* 2 functions need to be defined:
-  `func didReceive`
-  `func serviceExtensionTimeWillExpire()`

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


#### Notification of APN token

In order for Optipush to be able to deliver push notifications to your iOS app, Optimove SDK for iOS must receive an APN token from your app. This is accomplished by the following  steps:
Inside the application `AppDelegate` class </br>

````swift
application(_:didRegisterForRemoteNotificationsWithDeviceToken:)
````

call </br>

````swift
 Optimove.sharedInstance.application(didRegisterForRemoteNotificationsWithDeviceToken:)
````

And  in </br>

````swift
application(_:didReceiveRemoteNotification:fetchCompletionHandler:)
````

 call </br>

````swift
 Optimove.sharedInstance.handleRemoteNotificationArrived(userInfo:fetchCompletionHandler)
````

 example:</br>

````swift
  func application(_ application: UIApplication,
                     didReceiveRemoteNotification userInfo: [AnyHashable : Any],
                     fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void)
    {
        Optimove.sharedInstance.handleRemoteNotificationArrived(userInfo: userInfo,
                                                        fetchCompletionHandler: completionHandler)
    }
    func application(_ application: UIApplication,
                     didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data)
    {
        Optimove.sharedInstance.application(didRegisterForRemoteNotificationsWithDeviceToken: deviceToken)
    }
````
<br>

## <a id="deep linking"></a>Deep Linking

In order to route end users back to the application from the notification, you must support *Deep Linking*.

Other than _UI attributes_, an **_Optipush Notification_** can contain metadata linking to a specific screen within your application, along with custom (screen specific) data.<br>

  To support deep linking, you should:

  * Enable Associated Domains:

In your project capabilities, add the dynamic link domain with `applinks:` prefix and without any `https://` prefix that will be **provided to you by Optimove Product Integration team**

  
![associated_domain.png](https://s9.postimg.cc/hqrw4eqm7/associated_domain.png)

<br>

Any _`ViewControler`_ should recieve DynamicLink data callback, should implement _`didReceive(dynamicLink:)`_, thus conforming to _`OptimoveDeepLinkCallback`_ protocol .<br>

_`OptimoveDeepLinkCallback`_ protocol has one method:

````swift
didReceive(deepLink:)
````

that receives an `OptimoveDeepLinkComponents` as argument
  
A `OptimoveDeepLinkComponents` entity contains two properties:

   - **ScreenName**: for the required _*viewcontroller*_ you need to segue to.
  - **Query**: That contain the content that should be included in that view controller. <br>

  
Example:

````swift
func didReceive(deepLink: OptimoveDeepLinkComponents?)
{
let vc = self.storyboard!.instantiateViewController(withIdentifier: deepLink!.screenName)
self.navigationController?.pushViewController(vc, animated: true)
}
````

  
## <a id="test mode"></a>Enabling Test Mode
 
You can test an **Optipush template** on your device *before* having to create an **Optipush campaign**.

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


>**Notes:**
>- It is recommended to maintain 2 apps - one with test mode enabled (for internal purposes ONLY) and one without test mode enabled (for the general public).
>- The app that is published to the App Store should NOT have the test mode enabled.

<br>

# <a id="Post setup"></a>Post-Setup




## <a id="notification template"></a>Create & Test notification templates
Once Optimove has enabled Optipush as an execution channel for your Optimove instance, you can begin creating and testing your first Optipush template.
>Note: In order to be able to test your templates, the test mode must be [enabled](https://github.com/optimove-tech/A/tree/master/O/O%20for%20A#enabling-test-mode) within your mobile app.<br>


### Create  an Optipush Template

 1. Go to the Manage Templates page and choose 'Optipush' from the Channel drop-down menu. <br>
 2. Enter values for the following fields:
	- Template Name- Name te template 
	 - Template Title - Title of push template
	 - Message - Message of the template <br>
![](https://raw.githubusercontent.com/optimove-tech/A/master/O/O%20for%20iOS/images/3.png)
<br>
 3. Personalization - you can personalize the notification by adding dynamic tags and emojis to the notification. <br>
 4. Preview - you can preview the push template notification before sending it out. <br>
 5. **Deep links** (Optional) - choose the app (iOS) and select the target screen in your app that you want your customers to be directed to when opening the message. <br>
 >Notes:
 >- In order to add Deep Links to the drop-down menu, please send the list of screen names to your CSM that you have configured in your app as described [here](https://github.com/optimove-tech/A/tree/master/O/O%20for%20iOS#deep-linking).
 >- *If a Deep Link is not chosen, the customer will be directed to the main screen when opening the message.* 
 >- When creating templates for Optipush, if the template is targeted for a specific device (iOS/Android), it is recommended to add the device name to the template naming convention. This way it will be identifiable when choosing a template for a campaign targeting a specific device.<br>
 

### Test an Optipush Template

1. **Validate** - validates the push notification template to make sure all mandatory fields are completed and contain no errors. <br>
2. **Send Test**  - clicking this link will send the push notification template to all devices that have the app installed with the test mode enabled.<br>

## <a id="Optipush campaign"></a>Set up an Optipush campaign

### Run Campaign

Please follow these steps in order to run a pre-scheduled campaign via execution channel **Optipush**.<br>
1. From the main menu go on *One-to-One Campaigns* --> click on *More* from the drop-down menu --> click on ***Run Campaign***.  <br>
2. Go through Steps 1 & 2 of the Run Campaign wizard as you would for any campaign. <br>
3. In Step 3 (Execution Details) choose from the *Channel* drop-down menu *Optipush*. This action will open the **Optipush Options** window.<br>
![](https://raw.githubusercontent.com/optimove-tech/A/master/O/O%20for%20iOS/images/2_a.png)
<br>
4. Choose from the *App* drop-down menu if you would like to run the campaign for your iOS app, Android app, or both by selecting the relevant box(es).<br>
5. Choose the relevant template for the *Template* drop down menu that you would like the targeted audience to receive.<br>
6. Continue through the remaining steps of the Run Campaign wizard to schedule the campaign for your preferred dates and times.
