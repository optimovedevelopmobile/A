# Executing Push Notifications (Optipush)

Optipush is Optimove’s mobile push notification delivery add-in module, powering all aspects of preparing, delivering and tracking mobile push notification communications to customers, seamlessly from within Optimove.

Optimove Mobile SDK includes built-in functionality for receiving push messages, presenting notifications in the app UI and tracking user responses. Once the Optimove SDK is running, you do not need to implement anything else in your code for these functions to operate.

Additional functionality, such as the use of deep linking and testing Optipush templates, is also supported. Implementing these features in your code is described in the following sections.

For more information on how to add Optipush to your account, please contact your CSM or your Optimove point of contact.

In order for Optipush to be able to deliver push notifications to your iOS app, Optimove SDK for iOS must receive an APN token from your app. This is accomplished by the following two steps:

1.	Inside the application’s `AppDelegate` class `application(_:didRegisterForRemoteNotificationsWithDeviceToken:)`, call `Optimove.sharedInstance.application(didRegisterForRemoteNotificationsWithDeviceToken:)`
2.	In `application(_:didReceiveRemoteNotification:fetchCompletionHandler:)`, call `Optimove.sharedInstance.handleRemoteNotificationArrived(userInfo:fetchCompletionHandler)`

For example:

    Copy
    
    func application(_ application: UIApplication, didReceiveRemoteNotification userInfo: [AnyHashable : Any],
    
         fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
                 	Optimove.sharedInstance.handleRemoteNotificationArrived(userInfo:userInfo, fetchCompletionHandler: completionHandler)
         }
         
         func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
         	
         	Optimove.sharedInstance.application(didRegisterForRemoteNotificationsWithDeviceToken: deviceToken)
         }
     

## Deep Linking Notifications to a Particular App Screen

Other than UI attributes, an Optipush notification may also contain metadata linking to a specific screen within your app, along with custom (screen-specific) data.

To support deep linking, enable Associated Domains. To do so: In your project capabilities, add the deep link domain (provided by your Optimove CSM) with the applinks: prefix and without the https:// prefix.
 
 *image
 
Any ViewController should receive a DeepLink data callback, and should implement `didReceive(deepLink:)`, thus conforming to the `DeepLinkCallback` protocol.

The `DeepLinkCallback` protocol has one method:

    Copy
    
    didReceive(deepLink:)

The `deepLinkComponent` structure contains two properties:

 - **ScreenName** – for the required ViewController you need to segue to
- **Query** – contains the content that should be included in that view controller

For example:

        Copy
        
        func didReceive(deepLink: OptimoveDeepLinkComponents?) {
            if let deepLink = deepLink {
            DispatchQueue.main.asyncAfter(deadline: .now()+2.0)
            {
                if let vc = self.storyboard?.instantiateViewController(withIdentifier: deepLink.screenName) {
                  self.navigationController?.pushViewController(vc, animated: true)
                }
             }
          }
        }

## Testing Push Notification Templates

It is usually desirable to test an Optipush push notification template on an actual device before sending an Optipush campaign to users. 

To enable “test campaigns” on one or more devices, call the Optimove.sharedInstance.subscribeToTestMode() method.

To stop receiving “test campaigns,” call `Optimove.sharedInstance.unSubscribeFromTestMode()`.

    Copy
    
    class ViewController: UIViewController {
      override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        Optimove.sharedInstance.subscribeToTestMode()
      }
    }

**Notes**:
In order to test a push notification template, the marketer can send a test push notification campaign from within Optimove (using the Optipush UI). Notifications from such a campaign will only be delivered to devices running the version of your app in which the above “test campaigns” method was called.

