## Adding the Lock Screen
The lock screen is a vital part of SmartDeviceLink, as this does not allow the user to use your application while the vehicle is in motion. Prior to SDL 4.3, creating and maintaining the lock screen was up to the developer to do. Now, SDL takes care of the lock screen for you. It still allows you to use your own view controller if you prefer your own look, but still want the recommended logic that SDL provides for free.

A benefit to using the provided Lock Screen, is that we also handle support for retrieving a lock screen icon for versions of Core that support it, so that you do not have to be concerned with what car manufacturer you are connected to.

If you would not like to use any of the following code, you may use the `SDLLockScreenConfiguration` class function `disabledConfiguration`, and manage the entire lifecycle of the lock screen yourself. 

To see where the `SDLLockScreenConfiguration` is used, refer to the [Getting Started > Integration Basics](Getting Started/Integration Basics).

### Using the Provided Lock Screen
Using the default lock screen is simple. Using the lock screen this way will automatically load an automaker's logo, if supported, to show alongside your logo. If it is not, the default lock screen will show your logo alone.

![Generic Lock Screen](/assets/GenericLockScreen.png)

To do this, instantiate a new `SDLLockScreenConfiguration`:

#### Objective-C
```objc
SDLLockScreenConfiguration *lockScreenConfiguration = [SDLLockScreenConfiguration enabledConfiguration];
```

** Swift **
```swift
let lockScreenConfiguration = SDLLockScreenConfiguration.enabled()
```

### Customizing the Provided Lock Screen
If you would like to use the provided lock screen, however would like to add your own appearance to it, we provide that as well. `SDLLockScreenConfiguration` allows you to customize the background color as well as your app's icon. If the app icon is not included, we will use the SDL logo.

![Custom Lock Screen](/assets/CustomLockScreen.png)

#### Objective-C
```objc
UIImage *appIcon = <# Retreive App Icon #>
UIColor *backgroundColor = <# Desired Background Color #>
SDLLockScreenConfiguration *lockScreenConfiguration = [SDLLockScreenConfiguration enabledConfigurationWithAppIcon:appIcon backgroundColor:backgroundColor];
```

#### Swift
```swift
let appIcon = <# Retrieve App Icon #>
let backgroundColor = <# Desired Background Color #>
let lockScreenConfiguration = SDLLockScreenConfiguration.enabledConfiguration(withAppIcon: appIcon, backgroundColor: backgroundColor)
```

### Using Your Own Lock Screen
If you would like to use your own lock screen instead of the provided SDL one, but still use the logic we provide, you can use a new initializer within `SDLLockScreenConfiguration`:

#### Objective-C
```objc
UIViewController *lockScreenViewController = <# Initialize Your View Controller #>;
SDLLockScreenConfiguration *lockScreenConfiguration = [SDLLockScreenConfiguration enabledConfigurationWithViewController:lockScreenViewController];
```

#### Swift
```swift
let lockScreenViewController = <# Initialize Your View Controller #>
let lockScreenConfiguration = SDLLockScreenConfiguration.enabledConfiguration(with: lockScreenViewController)
```

### Retrieving Make and Model
If you have decided to create your own lock screen, a best practice is to display the OEM you are connect to's logo, as this adds a more personal touch for the end user.

#### Objective-C
```objc
__weak typeof (self) weakSelf = self;
[self.sdlManager startWithReadyHandler:^(BOOL success, NSError * _Nullable error) {
    if (!success) {
        NSLog(@"SDL errored starting up: %@", error);
        return;
    } 

    SDLVehicleType *vehicleType = weakSelf.sdlManager.registerResponse.vehicleType;
    NSString *make = vehicleType.make;
    NSString *model = vehicleType.model;
}];
```

#### Swift
```swift
sdlManager.start { (success, error) in
    if success == false {
        print("SDL errored starting up: \(error)")
        return
    }
    
    guard let vehicleType = self.sdlManager.registerResponse?.vehicleType,
    	let make = vehicleType.make,
    	let model = vehicleType.model else {
    	// No make/model found.
    	return
    } 
}
```

### Retrieving Lock Screen URL
A newer feature that some OEMs may adapt is the ability for the head unit to provide `SDLManager` a URL for a logo to display on the Lock Screen. If you are creating your own lock screen, it is a best practice to utilize this feature. This notifcation comes through after we have downloaded the icon for you, and sent it in the `SDLDidReceiveLockScreenIcon` notification.

**First**, register for the `SDLDidReceiveLockScreenIcon` Notification:

#### Objective-C
```objc
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(lockScreenIconReceived:) name:SDLDidReceiveLockScreenIcon object:nil];
```

#### Swift
```swift
NotificationCenter.default.addObserver(self, selector: #selector(lockScreenIconReceived(_:)), name: SDLDidReceiveLockScreenIcon, object: nil)
```

**Then**, act on the notification:

#### Objective-C
```objc
- (void)lockScreenIconReceived:(NSNotification *)notification {
    if (![notification.userInfo[SDLNotificationUserInfoObject] isKindOfClass:[UIImage class]]) {
        return;
    }

    UIImage *icon = notification.userInfo[SDLNotificationUserInfoObject];
}
```

#### Swift
```swift
func lockScreenIconReceived(_ notification: NSNotification) {
	guard let image = notification.userInfo[SDLNotificationUserInfoObject] as? UIImage else {
		return
	}
}
```