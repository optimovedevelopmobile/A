
-  [Introduction](Introduction)
 - [Setup](Setup)
	 - [Pre-Requisites ](pre-reqs)
	 - [Optipush Configuration](configuration)
	 - [Deep Linking](deep%20linking)
	 - [Enabling Test Mode](test%20mode) 
 - [Post-Setup](Post%20setup)
	 - [Create & test notification templates](notification%20template) 
	 - [Set up an Optipush campaign](Optipush%20campaign) 


# <a id="Introduction"></a>Introduction

**Optipush** is Optimove’s mobile push notification delivery add-in module, powering all aspects of preparing, delivering and tracking mobile push notification communications to customers, seamlessly from within Optimove.</br>
 **Optimove SDK** for iOS includes built-in functionality for receiving push messages, presenting notifications in the app UI and tracking user responses.


# <a id="Setup"></a>Setup

## <a id="pre-reqs"></a>Pre-Requisites 

 1. [Optimove Mobile SDK for iOS](https://github.com/optimove-tech/A) implemented 


## <a id="configuration"></a>Optipush Configuration

### Notification Service Extension <br>

In order to enable Optimove to track the push notifications, you'll need to add a **Notification Extension** to your project.

>Skip items 1-6 if you already have Notification Service Extension implemented in your project.

A notification service app extension ships as a separate bundle inside your iOS app. To add this extension to your app: 

1.  Select File > New > Target in Xcode.
2.  Select the Notification Service Extension target from the iOS > Application section.
3.  Click Next.
4.  Specify a name for your app extension.
5.  Click Finish.
6. In your `Podfile` add a new target matching the extension's name.

7. To that target, add the `pod 'OptimoveNotificationServiceExtension`' 

Example for the updated `Podfile`:

```ruby
platform :ios, '10.0'

target 'My Application' do #Your app target
  use_frameworks!

  pod 'OptimoveSDK'
end

target 'NotificationExtension' do #Your new extension target
  use_frameworks!

  pod 'OptimoveNotificationServiceExtension'
end
``` 
>Notes: 
> The extension versioning must be aligned with the application, so make sure that the extension's `Deployment Target` (found in the project's settings page) is the **same** as the app's `Deployment Target


8. In order to enable communication between the extension and the application, add an `App group` capability in both the app and the extension targets.

The group name convention should be: `group.<the application bundle id>.optimove`

![\[Screenshot\]](https://raw.githubusercontent.com/optimove-tech/A/master/O/O%20for%20iOS/images/Screen%20Shot%202018-07-02%20at%2018.06.21.png)

To implement the service's logic open the `NotificationService.swift` file.
Inside, you'll find 2 callbacks that are defined by iOS:
- `didReceive( request: contentHandler:)` - Called with the content of the push notification
- `func serviceExtensionTimeWillExpire()` - Called when the OS is about to _force kill_ the extension's process

Both of these callbacks must be implemented by **your** extension and forwarded to the **Optimove Notification Extension Module**.

9. Inside the `didReceive` callback create a `NotificationExtensionTenantInfo` with the same information that was provided by the PI team to create the `OptimoveTenantInfo` object in the app target.
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

10. Forward the callback to the `OptimoveNotificationServiceExtension`. In order to identify if the message was processed by Optimove, add the following:
```swift
optimoveExtensionService.didReceive(request, withContentHandler: contentHandler)
        if !optimoveExtensionService.isHandledByOptimove 
```
11. Finally, if the message was processed by Optimove let it know that the extension is about to be terminated by the OS through the `serviceExtensionTimeWillExpire` callback:

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
override func didReceive(_ request: UNNotificationRequest, withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void)
{
        let info = NotificationExtensionTenantInfo(endpoint: "htts://www.endpoint.com",
                                                   token: "ios-demo-token",
                                                   version: "ios.demo.version.1",
                                                   appBundleId: "com.optimove.sdk.demo")
		// Store as a property of the class to implement the serviceExtensionTimeWillExpire callback
        self.optimoveExtensionService = OptimoveNotificationServiceExtension(tenantInfo: info)
        
        self.optimoveExtensionService.didReceive(request, withContentHandler: contentHandler)
        if !optimoveExtensionService.isHandledByOptimove {
	        // Reach this section if the notification was not processed by Optimove
            self.contentHandler = contentHandler
            self.bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)
            
            if let bestAttemptContent = bestAttemptContent {
                // Modify the notification content to YOUR app's needs here...
                bestAttemptContent.title = "\(bestAttemptContent.title) [modified]"
                
                contentHandler(bestAttemptContent)
            }
        }
    }
    
    override func serviceExtensionTimeWillExpire() {
		// Pass the callback to Optimove only if the notifcation was process by it (adva fix)
        if optimoveExtensionService.isHandledByOptimove {
            self.optimoveExtensionService.serviceExtensionTimeWillExpire()
        } else {
            if let contentHandler = contentHandler, let bestAttemptContent =  bestAttemptContent {
                contentHandler(bestAttemptContent)
            }
        }
    }
```

Optipush notification center delegate
```swift
//
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

``` swift
// Request permission to show notifications (Relevant for Optipush only)
	UNUserNotificationCenter.current().requestAuthorization(options: [.alert]) { (grant, error) in }
UNUserNotificationCenter.current().delegate = self
```

Apply items 7-8 if using **Optipush**: <br>
7. Also, if you'd like to present the notifications to the user, request authorization by calling:

`UNUserNotificationCenter.current().requestAuthorization(options:completionHandler:)`

8. Finally declare the `AppDelegate` as the `UserNotificationCenter` delegate

9. For steps 6 - 8 you must import `UserNotifications`  


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
 >Notes: <br>
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
