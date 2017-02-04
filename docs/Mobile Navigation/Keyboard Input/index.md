## Keyboard Input
Keyboard input is available via the `SDLPerformInteraction` RPC. For a general understanding of how this RPC works, reference the [Displaying Information > Menus](Displaying Information/Menus). As opposed to the normal process for using `SDLPerformInteraction` with a required `SDLCreateInteractionChoiceSet`, using the keyboard requires no interaction choice sets to be created beforehand. It does, however, require an empty array to be passed in. To show the perform interaction as a keyboard, we modify the `interactionLayout` property to be `KEYBOARD`. Note that while the vehicle is in motion, keyboard input will be unavailable (resulting in a grayed out keyboard).

#### Swift
```swift
let performInteraction = SDLPerformInteraction()!
performInteraction.initialText = "Find Location"
performInteraction.interactionChoiceSetIDList = []
performInteraction.timeout = 100000
performInteraction.interactionMode = .manual_ONLY()
performInteraction.interactionLayout = .keyboard()
sdlManager.send(performInteraction) { (request, response, error) in
    if response?.resultCode.isEqual(to: SDLResult.success()) == false {
        print("Error sending perform interaction.")
        return
    }

    guard let performInteractionResponse = response as? SDLPerformInteractionResponse,
        let textEntry = performInteractionResponse.manualTextEntry else {
        return
    }

    // text entered
}
```

#### Objective-C
```objc
SDLPerformInteraction* performInteraction = [[SDLPerformInteraction alloc] init];
performInteraction.initialText = @"Find Location";
performInteraction.interactionChoiceSetIDList = [@[] mutableCopy];
performInteraction.timeout = @(100000);
performInteraction.interactionMode = SDLInteractionMode.MANUAL_ONLY;
performInteraction.interactionLayout = SDLLayoutMode.KEYBOARD;
[self.sdlManager sendRequest:performInteraction withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if (![response.resultCode isEqualToEnum:SDLResult.SUCCESS]) {
        NSLog(@"Error sending perform interaction.");
        return;
    } else if (![response isKindOfClass:SDLPerformInteractionResponse.class]) {
        return;
    }

    SDLPerformInteractionResponse* performInteractionResponse = (SDLPerformInteractionResponse*)response;

    if (performInteractionResponse.manualTextEntry.length) {
        // text entered
    }
}];
```

!!! note
In Ford's current SYNC 3 implementation of SmartDeviceLink, there is a bug resulting in the need for an interaction choice array to be set in the RPC call.
!!!