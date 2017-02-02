## Displaying Turn By Turn Advices

Currently, to display a Turn by Turn advice, a combination of the `SDLShowConstantTBT` and `SDLAlertManeuver` RPCs must be used. The `SDLShowConstantTBT` RPC involves the data that will be shown on the head unit. The main properties of this object to set are `navigationText1`, `navigationText2`, and `turnIcon`. A best practice for navigation applications is to use the `navigationText1` as the advice to give (Turn Right) and `navigationText2` to provide the distance to that advice (3 mi). When an `SDLAlertManeuver` is sent, you may also include accompanying text that you would like the head unit to speak when an advice is displayed on screen (In 3 miles turn right.).

!!! note
If the connected device has received a phone call in the vehicle, the Alert Maneuver is the only way for your app to inform the user.
!!!

#### Swift
```swift
// Create SDLImage object for turnIcon.
let turnIcon = <#Create SDLImage#>

let turnByTurn = SDLShowConstantTBT()!
turnByTurn.navigationText1 = "Turn Right"
turnByTurn.navigationText2 = "3 mi"
turnByTurn.turnIcon = turnIcon

manager.send(turnByTurn) { (request, response, error) in
    if response?.resultCode.isEqual(to: SDLResult.success()) == false {
        print("Error sending TBT.")
        return
    }

    let alertManeuver = SDLAlertManeuver(tts: "In 3 miles turn right", softButtons: nil)!
    self.manager.send(alertManeuver) { (request, response, error) in
        if response?.resultCode.isEqual(to: SDLResult.success()) == false {
            print("Error sending AlertManeuver.")
            return
        }
    }
}
```

#### Objective-C
```objc

// Create SDLImage object for turnIcon.
SDLImage* turnIcon = <#Create SDLImage#>;

SDLShowConstantTBT* turnByTurn = [[SDLShowConstantTBT alloc] init];
turnByTurn.navigationText1 = @"Turn Right";
turnByTurn.navigationText2 = @"3 mi";
turnByTurn.turnIcon = turnIcon;

__weak typeof(self) weakSelf = self;
[self.manager sendRequest:turnByTurn withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if (![response.resultCode isEqualToEnum:SDLResult.SUCCESS]) {
        NSLog(@"Error sending TBT.");
        return;
    }

    typeof(weakSelf) strongSelf = weakSelf;
    SDLAlertManeuver* alertManeuver = [[SDLAlertManeuver alloc] initWithTTS:@"In 3 miles turn right" softButtons:nil];
    [strongSelf.manager sendRequest:alertManeuver withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
        if (![response.resultCode isEqualToEnum:SDLResult.SUCCESS]) {
            NSLog(@"Error sending AlertManeuver.");
            return;
        }
    }];
}];
```

> Remember when sending a SDLImage, that the image must first be uploaded to the head unit via a PutFile RPC.

To clear a navigation advice from the screen, we send an `SDLShowConstantTBT` with the `maneuverComplete` property as `YES`. This specific RPC does not require an accompanying `SDLAlertManeuver`.  

#### Swift
```swift
let turnByTurn = SDLShowConstantTBT()!
turnByTurn.maneuverComplete = true

manager.send(turnByTurn) { (request, response, error) in
    if response?.resultCode.isEqual(to: SDLResult.success()) == false {
        print("Error sending TBT.")
        return
    }
}

```

#### Objective-C
```objc
SDLShowConstantTBT* turnByTurn = [[SDLShowConstantTBT alloc] init];
turnByTurn.maneuverComplete = @(YES);

[self.manager sendRequest:turnByTurn withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if (![response.resultCode isEqualToEnum:SDLResult.SUCCESS]) {
        NSLog(@"Error sending TBT.");
        return;
    }

    // Successfully cleared
}];

```