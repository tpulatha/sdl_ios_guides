## Mobile Navigation

Mobile Navigation allows map partners to bring their applications into the car and display their maps and turn by turn easily for the customer. This feature has a different behavior on the head unit than normal applications. The main differences are:

* Navigation Apps don't use base screen templates. Their main view is the video stream sent from the device
* Navigation Apps can send audio via a binary stream. This will attenuate the current audio source and should be used for navigation commands
* Navigation Apps can receive touch events from the video stream

!!! note
In order to use SDL's Mobile Navigation feature, the app must have a minimum requirement of iOS 8.0. This is due to using iOS's provided video encoder.
!!!

### Connecting an app

The basic connection is the similar for all apps. Please follow the basic instructions in the "Preparation" section of the [mobile weather tutorial](https://github.com/smartdevicelink/sdl_mobileweather_tutorial_ios/wiki/Section%201%20Preparation).

The first difference for a navigation app is the `appHMIType` of Navigation that has to be set in the `SDLLifecycleConfiguration`. Navigation apps are also non-media apps.

The second difference is a property called `securityManagers` that needs to be set in the `SDLLifecycleConfiguration` if connecting to a version of Core that requires secure video & audio streaming. This property requires an array of classes of Security Managers, which will conform to the `SDLSecurityType` protocol. These security libraries are provided by the OEMs themselves, and will only work for that OEM. There is not a general catch-all security library.

!!! note

For more information relating to app registration, please reference the "App Registration" section of the [mobile weather tutorial](https://github.com/smartdevicelink/sdl_mobileweather_tutorial_ios/wiki/Section%201%20Preparation#app-registration).

!!!

#### Objective-C
```objc
SDLLifecycleConfiguration* lifecycleConfig = [SDLLifecycleConfiguration defaultConfigurationWithAppName:@"<#App Name#>" appId:@"<#App Id#>"];
lifecycleConfig.appType = SDLAppHMIType.NAVIGATION;
lifecycleConfig.securityManagers = @[OEMSecurityManager.class];
```

#### Swift
```swift
let lifecycleConfig = SDLLifecycleConfiguration.defaultConfiguration(withAppName: "<#App Name#>", appId: "<#App Id#>")
lifecycleConfig.appType = .navigation()
lifecycleConfig.securityManagers = [OEMSecurityManager.self]
```

!!! note
When compiling, you must make sure to include all possible OEM's security managers that you wish to support.
!!!

After being registered, the app will start receiving callbacks. One important callback is `hmiLevel:didChangeToLevel:`, which informs the app about the currently visible application on the head unit. Right after registering, the `hmiLevel` will be `NONE` or `BACKGROUND`. Streaming should commence once the `hmiLevel` has been set to `LIMITED` by the head unit.

### Moving Forward
Currently, the maintanence of knowing your phone's state, the head unit's state, and the streaming state, is all up to the developer. This has been noted by the maintainers of SDL, and strides have been moving forward to implementing all of this internally to SDL, so the developer no longer needs to worry about these states, and we can have customers expect a consistent experience across all mobile navigation applications.

You can track the current proposal on [SDL Evolution](https://github.com/smartdevicelink/sdl_evolution/pull/56).
And the work-in-progress code on the [sdl_ios repository](https://github.com/smartdevicelink/sdl_ios/pull/514).