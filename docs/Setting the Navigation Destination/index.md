## Setting the Navigation Destination
Setting a Navigation Destination allows you to send a GPS location that you would like to prompt that user to navigate to using their embedded navigation. When using the `SendLocation` RPC, you will not receive a callback when issuing a SendLocation as to how the user interacted with this location, only if it was successfully sent to Core and received.

!!! note
	This currently is only supported for Embedded Navigation. This does not work with Mobile Navigation Apps at this time.
!!!

!!! note
SendLocation is an RPC that is usually restricted by OEMs. As a result, the OEM you are connecting to may limit app functionality if not approved for usage.
!!!

### Determining the Result of SendLocation
SendLocation has 3 possible results that you should expect:

1. SUCCESS - SendLocation was successfully sent.
2. INVALID_DATA - The request you sent contains invalid data and was rejected.
3. DISALLOWED - Your app does not have permission to use SendLocation.

### Detecting if SendLocation is Available
`SendLocation` is a newer RPC, so there is a possibility that not all head units will support it, especially if you are connected to a head unit that does not have an embedded navigation. In order to see if SendLocation is available, take a look at `SDLManager`'s `registerResponse` property once in the completion handler for `startWithReadyHandler`.

!!! note 
	If you need to know how to create and setup `SDLManager`, please see [Getting Started > Integration Basics]().
!!!

#### Objective-C
```objc
__weak typeof (self) weakSelf = self;
[self.sdlManager startWithReadyHandler:^(BOOL success, NSError * _Nullable error) {
    if (!success) {
        NSLog(@"SDL errored starting up: %@", error);
        return;
    } 

    SDLHMICapabilities *hmiCapabilities = weakSelf.sdlManager.registerResponse.hmiCapabilities;
    BOOL isNavigationSupported = NO;
    if (hmiCapabilities != nil) {
        isNavigationSupported = hmiCapabilities.navigation.boolValue;
    } 
}];
```

#### Swift
```swift
sdlManager.start { (success, error) in
    if success == false {
        print("SDL errored starting up: \(error)")
        return
    }
    
    var isNavigationSupported = false
    if let hmiCapabilities = self.sdlManager.registerResponse?.hmiCapabilities {
        isNavigationSupported = hmiCapabilities.navigation.boolValue
    }
}
```

### Using Send Location
To use SendLocation, you must at least include the Longitude and Latitude of the location.

#### Objective-C
```objc
SDLSendLocation *sendLocation = [[SDLSendLocation alloc] initWithLongitude:-97.380967 latitude:42.877737 locationName:@"The Center" locationDescription:@"Center of the United States" address:@[@"900 Whiting Dr", @"Yankton, SD 57078"] phoneNumber:nil image:nil];
[self.sdlManager sendRequest:sendLocation withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error) {
        NSLog(@"Encountered Error sending SendLocation: %@", error);
        return;
    }
    
    if (![response isKindOfClass:SDLSendLocationResponse.class]) {
        return;
    }
    
    SDLSendLocationResponse *sendLocation = (SDLSendLocationResponse *)response;
    SDLResult *resultCode = sendLocation.resultCode;
    if (![resultCode isEqualToEnum:SDLResult.SUCCESS]) {
        if ([resultCode isEqualToEnum:SDLResult.INVALID_DATA]) {
            NSLog(@"SendLocation was rejected. The request contained invalid data.");
        } else if ([resultCode isEqualToEnum:SDLResult.DISALLOWED]) {
            NSLog(@"Your app is not allowed to use SendLocation");
        } else {
            NSLog(@"Some unknown error has occured!");
        }
        return;
    }
    
    // Successfully sent!
}];
```

#### Swift
```swift
let sendLocation = SDLSendLocation(longitude: -97.380967, latitude: 42.877737, locationName: "The Center", locationDescription: "Center of the United States", address: ["900 Whiting Dr", "Yankton, SD 57078"], phoneNumber: nil, image: nil)!
sdlManager.send(sendLocation) { (request, response, error) in
    guard let response = response as? SDLSendLocationResponse,
        let resultCode = response.resultCode else {
            return
    }
    
    if let error = error {
        print("Encountered Error sending SendLocation: \(error)")
        return
    }
    
    if !resultCode.isEqual(to: SDLResult.success()) {
        if resultCode.isEqual(to: SDLResult.invalid_DATA()) {
            print("SendLocation was rejected. The request contained invalid data.")
        } else if resultCode.isEqual(to: SDLResult.disallowed()) {
            print("Your app is not allowed to use SendLocation")
        } else {
            print("Some unknown error has occured!")
        }
        return
    }
    
    // Successfully sent!
}
```