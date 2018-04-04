

-   [Introduction](#Introduction)
-   [iOS SDK Setup](#iOS%20SDK%20Setup)
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

This guide will show you how to setup the iOS (Swift) SDK in order to:

 - Track visitor and customer actions and events
 - Trigger Realtime campaigns

The SDK is supported by iOS native applications.

<br>

# <a id="iOS SDK Setup"></a>iOS (Swift) SDK Setup
Use the Basic Setup (required) in order to:

 - Track visitor and customer actions and events
 - Execute Push Notifications ([OptiPush](https://github.com/optimove-tech/A/blob/master/O/O.md))

## <a id="Basic Setup"></a>Basic Setup


### **1. Request a Mobile SDK from Optimove**

Before implementing the Optimove Track & Trigger to report visitor / customer activities or perform other functions ([OptiPush](https://github.com/optimove-tech/A/blob/master/O/O.md)), you will need to contact your Optimove Customer Success Manager (CSM) or Optimove point of contact and send the below details with a request for your Tenant and Mobile SDK configuration details in order incorporate into your iOS app.

Send the following information with your request: 
1.	Acquire a paid development account and valid certificates for remote notifications or APN Auth key.
2.	APN Auth key (p8)
3.	Team ID
4.	Appstore ID of the application

Receive a *tenant_information_suite* from the Optimove Integration Team that contains: 
1.	**end-point URL** – The URL where the tenant configurations reside
2.	**Unique Optimove token** – The actual token, unique per tenant
3.	**configuration name** – The version of the desired configuration

Enable push notifications and remote notification capabilities in your project (*this step is required only for sending push notifications using [OptiPush](https://github.com/optimove-tech/A/blob/master/O/O.md)*).
 
 [![apple_dashboared.png](https://s9.postimg.org/9ln5sfxe7/apple_dashboared.png)](https://postimg.org/image/itfe954gb/)
 
For additional technical details, please use our [iOS GitHub repository](https://github.com/optimoveintegrationmobile/ios-sdk).


### **2. Setting Up the iOS SDK**

Optimove SDK for iOS is provided as a group of files within a folder named, 'OptimoveSDK'. This folder can be found in this GitHub repository. To install the SDK, drag this folder into your project. If not all files inside the folder are members of the target application, add them.

In order to work with the Optimove SDK, you also need to download some modules from CocoaPods. In your Podfile, add the following:

 ````swift
pod 'Firebase','~> 4.8.0'
pod 'FirebaseMessaging', '~> 2.0.8'
pod 'FirebaseDynamicLinks', '~> 2.3.1'
pod 'XCGLogger', '~> 6.0.2'
pod 'OptimovePiwikTracker'
````

In your `AppDelegate` class, inside the application 
````swift
application (_: didFinishLaunchingWithOptions:)
```` 
method, create a new `OptimoveTenantInfo` object. This object should contain:

1. The end-point URL
2.	Unique Optimove token
3.	The configuration version provided by the Optimove Integration Team
4.	Indication for existing Firebase module inside your application

Use this object as an argument for the SDK function,

```swift
 Optimove.sharedInstance.configure(info:) 
```     

This call initializes the `OptimoveSDK` singleton. For example:

       
````swift
func application(_ application: UIApplication,
didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
let info = OptimoveTenantInfo(url:"https://optimove.mobile.demo/sdk-configs/", // end-point url
token: "abcdefg12345678", // token
version: "myapp.ios.1.0.0", //config name
hasFirebase: false) // indication for tenant autonomous firebase
Optimove.sharedInstance.configure(info: info)
````

The initialization must be called **as soon as possible**, unless the tenant has its own Firebase SDK. In this case, start the initialization right after calling `FirebaseApp.configure()`.

**If Your App Already Uses Firebase**

The Optimove iOS SDK is dependent upon the Firebase iOS SDK. If your app already uses Firebase SDK or has a dependency with Firebase SDK, a build conflict or runtime exception may occur, due to backward compatibility issues. Therefore, it is highly recommended to match the application’s Firebase SDK version to Optimove’s Firebase SDK:

| Optimove SDK Version | Firebase Core Version | Firebase Messaging Version | FirebaseDynamicLinks |
|----------------------|-----------------------|----------------------------|----------------------|
| 1.0.4.3              | 4.8.0                 | 2.0.8                      | 2.3.1                |

<br>

**State Registration**

The SDK initialization process occurs asynchronously, off the `Main Thread`.
Before calling the Public API methods, make sure that the SDK has finished initialization by calling the _`register(stateDelegate:)`_ method with an instance of _`OptimoveStateDelegate`_.

```swift
class AppDelegate: UIResponder,
    UIApplicationDelegate, OptimoveStateDelegate
  {
    var window: UIWindow?
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions:
                     [UIApplicationLaunchOptionsKey: Any]?) -> Bool
    {    
        let info = OptimoveTenantInfo(url:"https://optimove.mobile.demo/sdk-configs/",
                                      token: "abcdefg12345678",
                                      version: "myapp.ios.1.0.0",
                                      hasFirebase: false)
        
        Optimove.sharedInstance.configure(info: info)
        Optimove.sharedInstance.register(stateDelegate: self)
        return true
    }
  }
```
Do not forget to implement the _`OptimoveStateDelegate`_ methods, and provide a unique Int id to any enitity that conform to the protocol.

<br>

## <a id="Advanced Setup"></a>Advanced Setup

Use the Advanced Setup (optional) in order to track visitor and customer customized actions and events.
As described in [Reporting Custom Events](https://github.com/optimove-tech/SDK-Custom-Events-for-Your-Vertical), this step requires collaboration between you and Optimove’s Integration Team. Please contact your Optimove Customer Success Manager (CSM) or Optimove point of contact to schedule a meeting with the Product Integration team.

>**Note**: You can deploy the basic setup, followed by adding the advanced setup at a later stage. The Basic Setup is a pre-requisite.

<br>

# <a id="Track"></a>Track

## <a id="Linking Visitors to Users"></a>Linking App Visitors to Registered User IDs

Once the user has downloaded the application and the *OptimoveSDK* for iOS has run for the first time, the user is considered a *visitor*, i.e., an unidentified person.

Once the user authenticates and becomes identified by a known `PublicCustomerId`, then the user is considered a *customer*. As soon as this happens for each individual user, pass the `CustomerId` to the `Optimove` singleton like this:

```swift
Optimove.sharedInstance.set(userId:)
```
       
>**Note:** 
>
 >- The `CustomerId` is usually provided by the server application that manages customers, and is the same ID provided to Optimove during the daily customer data transfer. 
 >- Due to its high importance, `set (userId:)` may be called at any time, regardless of the SDK’s state
> - If you will be sending encrypted userId, please follow the steps in [Reporting encrypted CustomerIDs](https://github.com/optimove-tech/Reporting-Encrypted-CustomerID)
 
<br>

## <a id="Tracking a Screen Visit"></a>Tracking a Screen Visit Event

To track which screens the user has visited in your app, send a *setScreenEvent* message to the shared Optimove instance:

  ````swift
Optimove.sharedInstance.setScreenEvent(viewControllersIdentifiers:url)
````
The `viewControllersIdentifiers` argument should include an array that represents the path to the current screen. To support more complex view hierarchies, you may also specify a screen URL in the second parameter.

<br>

## Reporting Custom Events

Optimove clients may use the Optimove Mobile SDK to track specific customer actions and other custom events to Optimove (beyond the OOTB events such as visits). This data is used for tracking visitor and customer behavior, targeting campaigns to specific visitor and/or customer segments and triggering campaigns based on particular visitor and/or customer actions/events.

Each Optimove client has a tailored set of customer actions that may be reported via the SDK. As mentioned above, you will collaborate with the Optimove Integration Team to define the particular set of custom events that your website will be able to report. This approach allows you to define any event and its associated parameters.

Once you and the Optimove Integration Team have together defined the custom events supported by your app, the Integration Team will implement your particular functions within your Optimove site, while you will be responsible for implementing the `OptimoveEvent` protocol of the individual events within your app using the appropriate function calls.
To see examples of Custom Events, please visit [Defining the Set of Custom Tracking Events](https://github.com/optimove-tech/SDK-Custom-Events-for-Your-Vertical) that You Will Report for more information.

>**Note**: While you can always add/change the custom events and parameters at a later date (by speaking with the Optimove Integration Team), only the particular custom events that you and the Optimove Integration Team have already defined together will be supported by your Optimove site.

This `OptimoveEvent` protocol defines two properties:

1.	**name: String** – Declares the custom event’s name
2.	**parameters: [String:Any]** – Specifies the custom event’s parameters.

Then, send that event through the `reportEvent(event:)` method of the `Optimove` singleton:

````swift
override func viewDidAppear(_ animated: Bool) {
super.viewDidAppear(animated)
Optimove.sharedInstance.reportEvent(event: MyCustomEvent())
}
````

**Notes**:

> - As already mentioned, all custom events must be pre-defined in your Tenant configurations by the Optimove Integration Team.
> - Reporting of custom events is only supported if you have the Mobile SDK implemented.
 >- Events use snake_case as a naming convention. Separate each word with one underscore character (_) and no spaces. (e.g., Checkout_Completed)
 >- The usage of the `reportEvent` function depends on your needs. This function may include a completion handler that will be called once the report has finished. The default value for this argument is nil.

<br>

# <a id="Trigger"></a>Trigger

## <a id="trigger-optimail"></a>Executing via Optimail
Ability to execute Realtime campaigns using Optimove’s Optimail email service provider (ESP) add-on product - **Coming Soon**.

For more information on how to add Optimail to your account, please contact your CSM or your Optimove point of contact.

## <a id="trigger-api"></a>Executing via Optimove APIs
Ability to execute Realtime campaigns for mobile native app using Optimove’s APIs -**Coming Soon**

For more information on how to acquire an API key to use Optimove APIs, please request one from your CSM or your Optimove point of contact.

