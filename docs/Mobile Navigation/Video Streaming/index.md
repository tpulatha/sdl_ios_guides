## Video Streaming
In order to stream video from a SDL app, we focus on the `SDLStreamingMediaManager` class. A reference to this class is available from `SDLManager`.

### Video Stream Lifecycle
Currently, the lifecycle of the video stream must be maintained by the developer. Below is a set of guidelines for when a device should stream frames, and when it should not. The main players in whether or not we should be streaming are HMI State, and your app's state. Due to an iOS limitation of `VideoToolbox`'s encoder and `openGL`, we must stop streaming when the device moves to the background state. 

HMI State   | App State         | Can Open Video Stream | Should Close Video Stream
------------|-------------------|-----------------------|--------------------------
NONE        | Background        | No                    | Yes
NONE        | Regaining Active  | No                    | Yes
NONE        | Foreground        | No                    | Yes
NONE        | Resigning Active  | No                    | Yes
BACKGROUND  | Background        | No                    | Yes
BACKGROUND  | Regaining Active  | No                    | Yes
BACKGROUND  | Foreground        | No                    | Yes
BACKGROUND  | Resigning Active  | No                    | Yes
LIMITED     | Background        | No                    | No
LIMITED     | Regaining Active  | Yes                   | Yes
LIMITED     | Foreground        | Yes                   | No
LIMITED     | Resigning Active  | No                    | No
FULL        | Background        | No                    | No
FULL        | Regaining Active  | Yes                   | Yes
FULL        | Foreground        | Yes                   | No
FULL        | Resigning Active  | No                    | No

!!! note

For occurences of "Can Open Video Stream" and "Should Close Video Stream" both being **Yes**, this means that the stream should be closed, and then opened (restarted).

!!!

### Starting the Stream
In order to start a video stream, an app must have an HMI state of at least `LIMITED`, and the app must be in the foreground. It is also notable that you should only start one video stream per session. You may observe HMI state changes from `SDLManager`'s protocol callback `hmiLevel:didChangeToLevel:`. The flags `SDLEncryptionFlagAuthenticateOnly` or `SDLEncryptionFlagAuthenticateAndEncrypt` should be used if a security manager was provided when setting up the `SDLLifecycleConfiguration`. An example of starting a video stream can be seen below:


#### Swift
```swift
func hmiLevel(_ oldLevel: SDLHMILevel, didChangeTo newLevel: SDLHMILevel) {
    // Code for starting video stream
    if newLevel.isEqual(to: SDLHMILevel.limited()) || newLevel.isEqual(to: SDLHMILevel.full()) {
        startVideoSession()
    } else {
        stopVideoSession()
    }
}

private func stopVideoSession() {
    guard let streamManager = self.manager.streamManager, streamManager.videoSessionConnected else {
      return
    }

    streamManager.stopVideoSession()
}

private func startVideoSession() {
    guard let streamManager = self.manager.streamManager,
        streamManager.videoSessionConnected,
        UIApplication.shared.applicationState != .active else {
        return
    }
    streamManager.startVideoSession(withTLS: .authenticateAndEncrypt) { (success, encryption, error) in
        if !success {
            if let error = error {
                NSLog("Error starting video session. \(error.localizedDescription)")
            }
        } else {
            if encryption {
                // Video will be encrypted
            } else {
                // Video will not be encrypted
            }
        }
    }
}
```

#### Objective-C
```objc
- (void)hmiLevel:(SDLHMILevel*)oldLevel didChangeToLevel:(SDLHMILevel*)newLevel {
    // Code for starting video stream
    if ([newLevel isEqualToEnum:SDLHMILevel.FULL] || [newLevel isEqualToEnum:SDLHMILevel.LIMITED]) {
        [self startVideoSession];
    } else {
        [self stopVideoSession];
    }
}

- (void)stopVideoSession {
    if (!self.manager.streamManager.videoSessionConnected) {
        return;
    }
    [self.manager.streamManager stopVideoSession];
}

- (void)startVideoSession {
    if (!self.manager.streamManager.videoSessionConnected
        || [UIApplication sharedApplication].applicationState != UIApplicationStateActive) {
        return;
    }
    [self.manager.streamManager startVideoSessionWithTLS:SDLEncryptionFlagAuthenticateAndEncrypt startBlock:^(BOOL success, BOOL encryption, NSError * _Nullable error) {
        if (!success) {
            if (error) {
                NSLog(@"Error starting video session. %@", error.localizedDescription);
              }
        } else {
            if (encryption) {
                // Video will be encrypted
            } else {
                // Video will not be encrypted
            }
        }
    }];
}
```

!!! note

You should also cache the HMI Level, and when the device state changes, be sure to handle closing/opening the session.

!!!

### Sending Data to the Stream
Sending video data to the head unit must be provided to `SDLStreamingMediaManager` as a `CVImageBufferRef` (Apple documentation [here](https://developer.apple.com/library/mac/documentation/QuartzCore/Reference/CVImageBufferRef/)). Once the video stream has started, you will not see video appear until a few frames have been received. To send a frame, refer to the snippet below:

#### Objective-C
```objective-c
CVPixelBufferRef imageBuffer = <#Acquire Image Buffer#>;

if ([self.manager.streamManager sendVideoData:imageBuffer] == NO) {
  NSLog(@"Could not send Video Data");
}
```

#### Swift
```swift
let imageBuffer = <#Acquire Image Buffer#>;

guard let streamManager = self.manager.streamManager, streamManager.videoSessionConnected else {
  return
}

if streamManager.sendVideoData(imageBuffer) == false {
  print("Could not send Video Data")
}
```

### Best Practices
* A constant stream of map frames is not necessary to maintain an image on the screen. Because of this, we advise that a batch of frames are only sent on map movement or location movement. This will keep the application's memory consumption lower.
* For an ideal user experience, we recommend sending 30 frames per second.
