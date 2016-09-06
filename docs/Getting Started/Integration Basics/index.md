## Integration Basics
### How SDL Works
SmartDeviceLink works by sending remote procedure calls (RPCs) back and forth between a smartphone application and the SDL Core. These RPCs allow you to build the user interface, detect button presses, play audio, and get vehicle data, among other things. You will interact with the SDL library in order to have your app's functionality available on the remote system.

### Set Up a Proxy Manager Class
You will need a class that manages the RPCs sent back and forth between your app and SDL Core. Since  there should be only one active connection to the SDL Core, you may wish to implement this proxy class using the singleton pattern.
```swift
class ProxyManager: NSObject {
  // Singleton
  static let sharedInstance = ProxyManager()
  private override init() {
      super.init()
    }
}
```

Your app should always start passively watching for a connection with a SDL Core as soon as the app launches. The easy way to do this is by instantiating the *Proxy Class* in the `didFinishLaunchingWithOptions()` method in your *AppDelegate* class.
```swift
class AppDelegate: UIResponder, UIApplicationDelegate {
  func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
    // Initialize the Proxy Manager class
    ProxyManager.sharedInstance
    return true
  }
}
```

### Import the SDL Library
At the top of the *ProxyManager* class, import the SDL for iOS library.
```swift
import SmartDeviceLink_iOS
```

### Implement the Proxy Manager
The `SDLManager` is a helper class that will handle setting up the initial connection with the SDL Core. It will also help you upload images and send RPCs.
```swift
class ProxyManager: NSObject {
  // The Manager
  var sdlManager: SDLManager?

  // Singleton
  static let sharedInstance = ProxyManager()
  private override init() {
    super.init()
  }
}
```

#### 1. Create a Lifecycle Configuration   
In order to instantiate the `SDLManager` class, you must first configure an `SDLLifecycleConfiguration` instance with the application name and application id. During the development stage, a dummy app id is usually sufficient. For more information about obtaining an application id, please consult "SDK Configuration". You must also decide which network configuration to use to connect the app to the SDL Core. Optional but recommended configuration properties include short app name, app icon, and app type.  

##### Network Connection Type
 There are two different ways to connect your app to a SDL Core: with a TCP network connection or with an iAP network connection. Use TCP for debugging and use iAP for production level apps.

 1. iAP
    ```swift
let lifecycleConfiguration = SDLLifecycleConfiguration.defaultConfigurationWithAppName(
"your app name",
appId: "your app id")
    ```

 2. TCP
 ```swift
 let lifecycleConfiguration = SDLLifecycleConfiguration.debugConfigurationWithAppName(
 "your app name"
 appId: "your app id",
 ipAddress: “123.4.567.982”,
 port: 12345))
 ```  

!!! NOTE
If you are using an emulator, the IP address is your computer or virtual machine’s IP address, and the port number is usually 12345.
!!!  

!!! IMPORTANT
If you are using a head unit or TDK, and are using the [relay app](https://github.com/smartdevicelink/relay_app_ios) for debugging, the IP address and port number should be set to the same IP address and port number as the app. This information appears in the relay app once the server is turned on in the relay app.
!!!  

##### Short app name (optional)
This is a shortened version of your app name that is substituted when the full app name will not be visible due to character count constraints

```swift
lifecycleConfiguration.shortAppName = "a shortened name for you app"
```

##### App Icon (optional)
This is a custom icon for your application.

```swift
let appIcon = SDLArtwork.persistentArtworkWithImage(
UIImage(named: "your app name"),
name: "you app icon name",
asImageFormat: SDLArtworkImageFormat.JPG)  // Change to the correct image format
lifecycleConfiguration.appIcon = appIcon
```

##### App Type (optional)
The app type is used by car manufacturers to decide how to categorize your app. Each car manufacturer has different categorization system. For example, if you set your app type as media, your app will also show up in the audio tab as well as the apps tab of Ford’s SYNC3 head unit. The app type options are: default, communication, media (i.e. music/podcasts/radio), messaging, navigation, information, and social.

```swift
lifecycleConfiguration.appType = SDLAppHMIType.MEDIA()
```

#### 2. Set the Configuration
The `SDLConfiguration` class is used to set the lifecycle and lock screen configurations for the app. Use the lifecycle configuration settings above to instantiate a `SDLConfiguration` instance.
```swift
let configuration = SDLConfiguration(
    lifecycle: lifecycleConfiguration,
    lockScreen: SDLLockScreenConfiguration.enabledConfiguration())
```

##### Lock screen
 A lock screen is used to prevent the user from interacting with the app on the smartphone while they are driving. When the vehicle starts moving, the lock screen is activated. Similarly, when the vehicle stops moving, the lock screen is removed. You must implement the lock screen in your app for safety reasons. Any application without a lock screen will not get approval for release to the public.
 The SDL SDK takes care of the lock screen implementation for you, and even includes a default lock screen. You can choose to implement your own lock screen or you can use the default lock screen.

 1. Use the default lock screen
    ```swift
SDLLockScreenConfiguration.enabledConfiguration()
    ```

 2. Modify the default lock screen with your own icon and background color
    ```swift
SDLLockScreenConfiguration.enabledConfigurationWithAppIcon(
UIImage(named: "yourCustomImageName") ?? UIImage(),
backgroundColor: UIColor.redColor())
    ```

 3. Create a custom view controller for the lock screen
    ```swift
SDLLockScreenConfiguration.enabledConfigurationWithViewController(
UIViewController(nibName: "your view controller's nib name", bundle: NSBundle.mainBundle()))
    ```

!!! IMPORTANT
If you used CocoaPods to install the SDL SDK, you must complete the following steps to add the default lock screen resources to your project.
 1. Select your application's build target, go to **Build Phases**, **Copy Bundle Resources**.
 2. Then in the Navigator window of Xcode, go to **Target’s Support Files**, **Pods-YourProjectName**, and drag and drop the **SmartDeviceLink.bundle** file into **Copy Bundle Resources**.
 3. After the bundle is dropped into **Copy Bundle Resources** check “copy items if need” from the popup box and click “Finish.”
!!!

#### 3. Create a SDLManager
Now you can use the `SDLConfiguration` instance to instantiate the `SDLManager`.

```swift
sdlManager = SDLManager(configuration: configuration, delegate: self)
```

#### 4. Start the SDLManager
The manager should be started as soon as the class is instantiated, so you should configure and start the manager in the `ProxyManager` class’ initializer. Once the manager has been started, it will immediately begin passively watching for a connection with the remote system. The manager will continuously search for a connection with a SDL Core during the entire lifespan of the app. When the manager connects with a SDL Core, the `startWithReadyHandler` will be called.

!!! NOTE  
In production, your app will be watching for connections using IAP, which will not use any additional battery power than normal.  
!!!

If the connection is successful, you can start sending RPCs to the SDL Core. However, if the SDL Core HMI is not ready to accept RPCs, your requests will be ignored. If you want to make sure that the SDL Core will not ignore your RPCs, use the `SDLManagerDelegate` methods in the next step.

```swift
sdlManager?.startWithReadyHandler({ (success, error) in
    if success {
      // Your app has successfully connected with the SDL Core.
    }
})
```

#### 5. Example Implementation of a Proxy Class  
The following code snippet has an example of setting up both a TCP and iAP connection.

```
class ProxyManager: NSObject {
    var sdlManager: SDLManager?
    static let sharedInstance = ProxyManager()

    private override init( ) {
      super.init()
      self.startTCP()	// or use self.startIAP()
    }

    private func startIAP() {
      let lifecycleConfiguration = self.dynamicType.setLifecycleConfigurationPropertiesOnConfiguration(
        SDLLifecycleConfiguration.defaultConfigurationWithAppName(“MyAppName”, appId: “MyAppId”))
        startSDLManager(lifecycleConfiguration)
      }

    private func startTCP() {
      let ipAddress = “192.168.1.000”	// Set to your own IP address and port number
      let port = “2776”
      let lifecycleConfiguration: SDLLifecycleConfiguration =    
        self.dynamicType.setLifecycleConfigurationPropertiesOnConfiguration(
          SDLLifecycleConfiguration.debugConfigurationWithAppName(
            “MyAppName",
            appId: “MyAppId",
            ipAddress: ipAddress,
            port: port))
      startSDLManager(lifecycleConfiguration)
    }

    class func setLifecycleConfigurationPropertiesOnConfiguration(configuration: SDLLifecycleConfiguration) -> SDLLifecycleConfiguration {
      // App icon image
      let appImage = UIImage(named: "default")
      let appIconArt: SDLArtwork = SDLArtwork.persistentArtworkWithImage(appImage,
        name: "MyAppIconName",
        asImageFormat: SDLArtworkImageFormat.JPG)

      configuration.shortAppName = "ShortNameForApp"
      configuration.appIcon = appIconArt
      configuration.appType = SDLAppHMIType.MEDIA()

      return configuration
    }

    private func startSDLManager(lifecycleConfiguration: SDLLifecycleConfiguration) {
      let configuration: SDLConfiguration = SDLConfiguration(lifecycle: lifecycleConfiguration,
        lockScreen: SDLLockScreenConfiguration.enabledConfiguration())
      sdlManager = SDLManager(configuration: configuration, delegate: self)

      // Start watching for a connection with a SDL Core
      sdlManager?.startWithReadyHandler({ (success, error) in
        if success {
          // Your app has successfully connected with the SDL Core
        }
      })
    }
}
```

### Implement the SDL Manager Delegate
The *Proxy* class should conform to the `SDLManagerDelegate` protocol. This means that the *Proxy* class must implement the following required functions:
  1. `managerDidDisconnect()` This function is called only once, when the proxy disconnects from the SDL Core. Do any cleanup you need to do in this function.
  2. `hmiLevel(oldLevel: SDLHMILevel!, didChangeToLevel newLevel: SDLHMILevel!)` This function is called when the HMI level changes for the app. The HMI level can be `FULL`, `LIMITED`, `BACKGROUND`, or `NONE`. It is important to note that any RPCs sent while the app is in `BACKGROUND` or `NONE` mode will be ignored by the SDL Core.  

##### Different HMI Levels:**
* `FULL` - The app has full use of the SDL Core's HMI. The app may output via text-to-speech, display, or streaming audio and may gather input via voice recognition, touch-screen button presses, and hard-button presses
* `LIMITED` - This HMI level is only defined for a media app using an HMI with an 8 inch touchscreen system. The application's `SDLShow` RPC text is displayed and it receives button presses from media-oriented buttons (SEEKRIGHT, SEEKLEFT, TUNEUP, TUNEDOWN, PRESET_0-9).
* `BACKGROUND` - The app has been discovered by a SDL Core, but the app cannot send any requests or receive any notifications.
* `NONE` - This means that there is no existing HMI or that user has exited the application by saying "exit yourAppName" or selecting "exit" from the HMI menu. When this happens, the application still has an active interface registration with SDL and all SDL resources the application has created (e.g. choice sets, subscriptions, etc.) still exist, however the application cannot send any RPCs to SYNC, except `UnregisterAppInterface`.
