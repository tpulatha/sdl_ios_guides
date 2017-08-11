## Get Vehicle Data

Use the SDLGetVehicleData RPC call to get vehicle data. The HMI level must be FULL, LIMITED, or BACKGROUND in order to get data.

Each vehicle manufacturer decides which data it will expose. Please check the `SDLRPCResponse` RPC to find out which data you will have access to in your head unit.

!!! note
You may only ask for vehicle data that is available to your appName & appId combination. These will be specified by each OEM separately.
!!!

| Vehicle Data | Parameter Name  |  Description |
| ------------- | ------------- | ------------- |
| GPS | gps | Longitude and latitude, current time in UTC, degree of precision, altitude, heading, speed, satellite data vs dead reckoning, and supported dimensions of the GPS |
| Speed | speed | Speed in KPH |
| RPM | rpm | The number of revolutions per minute of the engine |
| Fuel level | fuelLevel | The fuel level in the tank (percentage) |
| Fuel level state | fuelLevel_State | The fuel level state: unknown, normal, low, fault, alert, or not supported |
| Instant fuel consumption | instantFuelConsumption | The instantaneous fuel consumption in microlitres |
| External temperature | externalTemperature | The external temperature in degrees celsius |
| VIN | vin | The Vehicle Identification Number |
| PRNDL | prndl | The selected gear the car is in: park, reverse, neutral, drive, sport, low gear, first, second, third, fourth, fifth, sixth, seventh or eighth gear, unknown, or fault |
| Tire pressure | tirePressure | Tire status of each wheel in the vehicle: normal, low, fault, alert, or not supported. Warning light status for the tire pressure: off, on, flash, or not used |
| Odometer | odometer | Odometer reading in km |
| Belt status | beltStatus | The status of each of the seat belts: no, yes, not supported, fault, or no event |
| Body information | bodyInformation | Door ajar status for each door. The Ignition status. The ignition stable status. The park brake active status. |
| Device status | deviceStatus | Contains information about the smartphone device. Is voice recognition on or off, has a bluetooth connection been established, is a call active, is the phone in roaming mode, is a text message available, the battery level, the status of the mono and stereo output channels, the signal level, the primary audio source, whether or not an emergency call is currently taking place |
| Driver braking | driverBraking | The status of the brake pedal: yes, no, no event, fault, not supported |
| Wiper status | wiperStatus | The status of the wipers: off, automatic off, off moving, manual interaction off, manual interaction on, manual low, manual high, manual flick, wash, automatic low, automatic high, courtesy wipe, automatic adjust, stalled, no data exists |
| Head lamp status | headLampStatus | Status of the head lamps: whether or not the low and high beams are on or off. The ambient light sensor status: night, twilight 1, twilight 2, twilight 3, twilight 4, day, unknown, invalid |
| Engine torque | engineTorque | Torque value for engine (in Nm) on non-diesel variants |
| Acceleration pedal position | accPedalPosition | Accelerator pedal position (percentage depressed) |
| Steering wheel angle | steeringWheelAngle | Current angle of the steering wheel (in degrees) |
| E-Call infomation | eCallInfo | Information about the status of an emergency call |
| Airbag status | airbagStatus | Status of each of the airbags in the vehicle: yes, no, no event, not supported, fault |
| Emergency event | emergencyEvent | The type of emergency: frontal, side, rear, rollover, no event, not supported, fault. Fuel cutoff status: normal operation, fuel is cut off, fault. The roll over status: yes, no, no event, not supported, fault. The maximum change in velocity. Whether or not multiple emergency events have occurred |
| Cluster mode status | clusterModeStatus | Whether or not the power mode is active. The power mode qualification status: power mode undefined, power mode evaluation in progress, not defined, power mode ok. The car mode status: normal, factory, transport, or crash. The power mode status: key out, key recently out, key approved, post accessory, accessory, post ignition, ignition on, running, crank |
| My key | myKey | Information about whether or not the emergency 911 override has been activated |

### Single Time Vehicle Data Retrieval
Using `SDLGetVehicleData`, we can ask for vehicle data a single time, if needed. 

#### Objective-C
```objc
SDLGetVehicleData *getVehicleData = [[SDLGetVehicleData alloc] init];
getVehicleData.prndl = @YES;
[self.sdlManager sendRequest:getVehicleData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error) {
        NSLog(@"Encountered Error sending GetVehicleData: %@", error);
        return;
    }
    
    if (![response isKindOfClass:SDLGetVehicleDataResponse.class]) {
        return;
    }
    
    SDLGetVehicleDataResponse* getVehicleDataResponse = (SDLGetVehicleDataResponse *)response;
    SDLResult *resultCode = getVehicleDataResponse.resultCode;
    if (![resultCode isEqualToEnum:SDLResult.SUCCESS]) {
        if ([resultCode isEqualToEnum:SDLResult.REJECTED]) {
            NSLog(@"GetVehicleData was rejected. Are you in an appropriate HMI?");
        } else if ([resultCode isEqualToEnum:SDLResult.DISALLOWED]) {
            NSLog(@"Your app is not allowed to use GetVehicleData");
        } else {
            NSLog(@"Some unknown error has occured!");
        }
        return;
    }
    
    SDLPRNDL *prndl = getVehicleDataResponse.prndl;
}];
```

#### Swift
```swift
let getVehicleData = SDLGetVehicleData()!
getVehicleData.prndl = true
sdlManager.send(getVehicleData) { (request, response, error) in
    guard let response = response as? SDLGetVehicleDataResponse,
        let resultCode = response.resultCode else {
            return
    }
    
    if let error = error {
        print("Encountered Error sending GetVehicleData: \(error)")
        return
    }
    
    if !resultCode.isEqual(to: SDLResult.success()) {
        if resultCode.isEqual(to: SDLResult.rejected()) {
            print("GetVehicleData was rejected. Are you in an appropriate HMI?")
        } else if resultCode.isEqual(to: SDLResult.disallowed()) {
            print("Your app is not allowed to use GetVehicleData")
        } else {
            print("Some unknown error has occured!")
        }
        return
    }
    
    let prndl = response.prndl
}
```

### Subscribing to Vehicle Data
Subscribing to vehicle data allows you to get notified whenever we have new data available. This data should not be relied upon being received in a consistent manner. New vehicle data is available roughly every second.

**First**, Register to observe the `SDLDidReceiveVehicleDataNotification` notification: 

#### Objective-C
```objc
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(vehicleDataAvailable:) name:SDLDidReceiveVehicleDataNotification object:nil];
```

#### Swift
```swift
NotificationCenter.default.addObserver(self, selector: #selector(vehicleDataAvailable(_:)), name: .SDLDidReceiveVehicleData, object: nil)
```

**Then**. Send the Subscribe Vehicle Data Request

#### Objective-C
```objc
SDLSubscribeVehicleData *subscribeVehicleData = [[SDLSubscribeVehicleData alloc] init];
subscribeVehicleData.prndl = @YES;

[self.sdlManager sendRequest:subscribeVehicleData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (![response isKindOfClass:[SDLSubscribeVehicleDataResponse class]]) {
        return;
    }
    
    
    if (!response.success.boolValue) {
        SDLSubscribeVehicleDataResponse *subscribeVehicleDataResponse = (SDLSubscribeVehicleDataResponse*)response;
        SDLVehicleDataResult *prndlData = subscribeVehicleDataResponse.prndl;
        
        if ([response.resultCode isEqualToEnum:SDLResult.DISALLOWED]) {
            // Not allowed to register for this vehicle data.
        } else if ([response.resultCode isEqualToEnum:SDLResult.USER_DISALLOWED]) {
            // User disabled the ability to give you this vehicle data
        } else if ([response.resultCode isEqualToEnum:SDLResult.IGNORED]) {
            if ([prndlData.resultCode isEqualToEnum:SDLVehicleDataResultCode.DATA_ALREADY_SUBSCRIBED]) {
                // You have access to this data item, and you are already subscribed to this item so we are ignoring.
            } else if ([prndlData.resultCode isEqualToEnum:SDLVehicleDataResultCode.VEHICLE_DATA_NOT_AVAILABLE]) {
                // You have access to this data item, but the vehicle you are connected to does not provide it.
            } else {
            	NSLog(@"Unknown reason for being ignored: %@", prndlData.resultCode.value);
            }
        } else if (error) {
            NSLog(@"Encountered Error sending SubscribeVehicleData: %@", error);
        }
        
        return;
    }
    
    if (![response isKindOfClass:SDLSubscribeVehicleDataResponse.class]) {
        return;
    }
    
    // Successfully subscribed
}];
```

#### Swift
```swift
let subscribeVehicleData = SDLSubscribeVehicleData()!
subscribeVehicleData.prndl = true

sdlManager.send(subscribeVehicleData) { (request, response, error) in
    guard let response = response as? SDLSubscribeVehicleDataResponse else { return }

    guard response.success.boolValue == true else {
        if response.resultCode.isEqual(to: SDLResult.disallowed()) {
            // Not allowed to register for this vehicle data.
        } else if response.resultCode.isEqual(to: SDLResult.user_DISALLOWED()) {
            // User disabled the ability to give you this vehicle data
        } else if response.resultCode.isEqual(to: SDLResult.ignored()) {
            if let prndlData = response.prndl {
                if prndlData.resultCode.isEqual(to: SDLVehicleDataResultCode.data_ALREADY_SUBSCRIBED()) {
                    // You have access to this data item, and you are already subscribed to this item so we are ignoring.
                } else if prndlData.resultCode.isEqual(to: SDLVehicleDataResultCode.vehicle_DATA_NOT_AVAILABLE()) {
                    // You have access to this data item, but the vehicle you are connected to does not provide it.
                } else {
                    print("Unknown reason for being ignored: \(prndlData.resultCode.value)")
                }
            } else {
                print("Unknown reason for being ignored: \(response.info)")
            }
        } else if let error = error {
            print("Encountered Error sending SubscribeVehicleData: \(error)")
        }
        return
    }
    
    // Successfully subscribed
}
```

**Finally**, react to notification when Vehicle Data is received:

#### Objective-C
``` objc
- (void)vehicleDataAvailable:(SDLRPCNotificationNotification *)notification {
    if (![notification.notification isKindOfClass:SDLOnVehicleData.class]) {
        return;
    }
    
    SDLOnVehicleData *onVehicleData = (SDLOnVehicleData *)notification.notification;
    
    SDLPRNDL *prndl = onVehicleData.prndl;
}
```

#### Swift
```swift
func vehicleDataAvailable(_ notification: SDLRPCNotificationNotification) {
    guard let onVehicleData = notification.notification as? SDLOnVehicleData else {
        return
    }
    
    let prndl = onVehicleData.prndl
}
```

### Unsubscribing from Vehicle Data
Sometimes you may not always need all of the vehicle data you are listening to. We suggest that you only are subscribing when the vehicle data is needed. To stop listening to specific vehicle data items, utilize `SDLUnsubscribeVehicleData`.

#### Objective-C
```objc
SDLUnsubscribeVehicleData *unsubscribeVehicleData = [[SDLUnsubscribeVehicleData alloc] init];
unsubscribeVehicleData.prndl = @YES;

[self.sdlManager sendRequest:unsubscribeVehicleData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (![response isKindOfClass:[SDLUnsubscribeVehicleDataResponse class]]) {
        return;
    }

    if (!response.success.boolValue) {
        SDLUnsubscribeVehicleDataResponse *unsubscribeVehicleDataResponse = (SDLUnsubscribeVehicleDataResponse*)response;
        SDLVehicleDataResult *prndlData = unsubscribeVehicleDataResponse.prndl;

        if ([response.resultCode isEqualToEnum:SDLResult.DISALLOWED]) {
            // Not allowed to register for this vehicle data, so unsubscribe also will not work.
        } else if ([response.resultCode isEqualToEnum:SDLResult.USER_DISALLOWED]) {
            // User disabled the ability to give you this vehicle data, so unsubscribe also will not work.
        } else if ([response.resultCode isEqualToEnum:SDLResult.IGNORED]) {
            if ([prndlData.resultCode isEqualToEnum:SDLVehicleDataResultCode.DATA_NOT_SUBSCRIBED]) {
                // You have access to this data item, but it was never subscribed to so we ignored it.
            } else {
                NSLog(@"Unknown reason for being ignored: %@", prndlData.resultCode.value);
            }
        } else if (error) {
            NSLog(@"Encountered Error sending UnsubscribeVehicleData: %@", error);
        }
        return;
    }
    
    // Successfully unsubscribed
}];
```

#### Swift
```swift
let unsubscribeVehicleData = SDLUnsubscribeVehicleData()!
unsubscribeVehicleData.prndl = true

sdlManager.send(unsubscribeVehicleData) { (request, response, error) in
    guard let response = response as? SDLUnsubscribeVehicleDataResponse else { return }
    
    guard response.success == true else {
        if response.resultCode.isEqual(to: SDLResult.disallowed()) {
            
        } else if response.resultCode.isEqual(to: SDLResult.user_DISALLOWED()) {
            
        } else if response.resultCode.isEqual(to: SDLResult.ignored()) {
            if let prndlData = response.prndl {
                if prndlData.resultCode.isEqual(to: SDLVehicleDataResultCode.data_ALREADY_SUBSCRIBED()) {
                    // You have access to this data item, and you are already unsubscribed to this item so we are ignoring.
                } else if prndlData.resultCode.isEqual(to: SDLVehicleDataResultCode.vehicle_DATA_NOT_AVAILABLE()) {
                    // You have access to this data item, but the vehicle you are connected to does not provide it.
                } else {
                    print("Unknown reason for being ignored: \(prndlData.resultCode.value)")
                }
            } else {
                print("Unknown reason for being ignored: \(response.info)")
            }                } else if let error = error {
            print("Encountered Error sending UnsubscribeVehicleData: \(error)")
        }
        return
    }
    
    // Successfully unsubscribed
}
```
