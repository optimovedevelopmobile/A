
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


### 2. Push Notifications Enabled 
Enable push notifications and remote notification capabilities in your project (*this step is required only for sending push notifications using Optipush.
 
[![apple_dashboared.png](https://s9.postimg.cc/9ln5sfxe7/apple_dashboared.png)](https://postimg.org/image/itfe954gb/)

### 3. Setting Up Optipush

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
Other than _UI attributes_, an **_Optipush Notification_** can contain metadata linking to a specific screen within your application, along with custom (screen specific) data. </br>

To support deep linking, you should:

* Enable Associated Domains:

In your project capabilities, add the deep link domain with `applinks:` prefix and without any `https://` prefix that will be **provided to you by Optimove Product Integration team**.

[![associated_domain.png](https://s9.postimg.cc/hqrw4eqm7/associated_domain.png)](https://postimg.cc/image/3x3jfcy0r/)
</br>

Any  _`ViewControler`_ should recieve DeepLink data callback, should implement _`didReceive(deepLink:)`_, thus conforming to _`OptimoveDeepLinkCallback`_ protocol .</br>
_`OptimoveDeepLinkCallback`_ protocol has one method:

````swift
didReceive(deepLink:)
````

A deepLinkComponent struct contains two properties:

· **ScreenName**: for the required _*viewcontroller*_ you need to segue to.

· **Query**: That contain the content that should be included in that view controller.

example:

````swift
class ViewController: UIViewController,OptimoveDeepLinkCallback
{
    private var responder: OptimoveDeepLinkResponder!
    func didReceive(deepLink: OptimoveDeepLinkComponents?)
    {
            let vc = self.storyboard!.instantiateViewController(withIdentifier: deepLink!.screenName)
            self.navigationController?.pushViewController(vc, animated: true)
    } 
    override func viewDidLoad() {
        super.viewDidLoad()
        Optimove.sharedInstance.register(stateDelegate: self)
        responder = OptimoveDeepLinkResponder(self) 
    }
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        Optimove.sharedInstance.register(deepLinkResponder: responder)
    }
    override func viewDidDisappear(_ animated: Bool) {
        super.viewDidDisappear(animated)
        Optimove.sharedInstance.unregister(deepLinkResponder: responder)
    }
}
````

## <a id="test mode"></a>Enabling Test Mode
 
You can test an **Optipush template** on your device *before* having to create an **Optipush campaign**.
To **enable** _"test Optipush templates"_ on one or more devices, call the _**`Optimove.sharedInstance.subscribeToTestMode()`**_ method.</br>
To **disable** _"test Optipush templates"_ call  _**`Optimove.sharedInstance.unSubscribeFromTestMode()`**_.</br>

````swift
class ViewController: UIViewController {
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        Optimove.sharedInstance.subscribeToTestMode()
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

 1. Go to the Manage Templates page and choose 'Optipush' from the Channel drop-down menu.
 2. Enter values for the following fields:
	 - Template Name- Name the template 
	 - Template Title - Title of push template
	 - Message - Message of the template
 3. Personalization - you can personalize the notification by adding dynamic tags and emojis to the notification.
 >Note: Personalization tags may cause push message to be truncated at 129 characters.
 4. Preview - you can preview the push template notification before sending it out.
 5. **Deep links** - choose the app (iOS) and select the target screen in your app that you want your customers to be directed to when opening the message. 
 >Notes:
 >- In order to add Deep Links to the drop-down menu, please send the list of screen names to your CSM that you have configured in your app as described [here](https://github.com/optimove-tech/A/tree/master/O/O%20for%20iOS#deep-linking).
 

### Test an Optipush Template

1. **Validate** - validates the push notification template to make sure all mandatory fields are completed and contain no errors. 
2. **Send Test**  - clicking this link will send the push notification template to all devices that have the app installed with the test mode enabled.<br>

## <a id="Optipush campaign"></a>Set up an Optipush campaign

### Run Campaign


1. 
