# Mobile Navigation Technical Introduction

#### v 0.1.0

SmartDeviceLink (SDL) enables smartphone navigation applications to stream video and audio to the head unit. This enables a brought in navigation experience that is on par with the embedded navigation.

!!! note

In order to use SDL's Mobile Navigation feature, the app must have a minimum requirement of iOS 8.0. This is due to using iOS's provided video encoder.

!!!

## SDL Basics

The mobile navigation feature uses the standard SDL library, but has a different behavior on the head unit. The main differences are:

* Navigation Apps don't use base screen templates. Their main view is the video stream sent from the device
* Navigation Apps can send audio via a binary stream. This will attenuate the current audio source and should be used for navigation commands
* Navigation Apps can receive touch events from the video stream

### Connecting an app

The basic connection is the similar for all apps. Please follow the basic instructions in the "Preparation" section of the [mobile weather tutorial](https://github.com/smartdevicelink/sdl_mobileweather_tutorial_ios/wiki/Section%201%20Preparation).

The first difference for a navigation app is the `appHMIType` of Navigation that has to be set after the Register App Interface has been created. Navigation apps are also non-media apps.

!!! note

For more information relating to app registration, please reference the "App Registration" section of the [mobile weather tutorial](https://github.com/smartdevicelink/sdl_mobileweather_tutorial_ios/wiki/Section%201%20Preparation#app-registration).

!!!

```objective-c
[request setAppHMIType:[@[SDLAppHMIType.NAVIGATION] mutableCopy];
```

After Register App Interface, apps should upload and set their app icon as shown in this part of the [mobile weather tutorial](https://github.com/smartdevicelink/sdl_mobileweather_tutorial_ios/wiki/Section-7-Work-with-files-and-images)

After being registered, the app will start receiving callbacks. One important callback is `onHMIStatus:`, which informs the app about the currently visible application on the head unit. `onHMIStatus:` has three parameters of which `hmiLevel` is the most important for a navigation app. Right after registering, the `hmiLevel` will be `NONE`. Streaming should commence once the `hmiLevel` has been set to `FULL` by the head unit. More information on `hmiStatus` can be found [here](https://github.com/smartdevicelink/sdl_mobileweather_tutorial_ios/wiki/Section-2-Basic-communication)

To interact with the customer, the basic SDL design principles shall be used. You can find out how to use Add Command to add menu items to the head unit in Step 3 [here](https://github.com/smartdevicelink/sdl_mobileweather_tutorial_ios/wiki/Section-2-Basic-communication)

## Sending a video stream
In order to stream video from a SDL app, we focus on the `SDLStreamingMediaManager` class. A reference to this class is available from `SDLProxy`.

### Starting the Stream
In order to start a video stream, an app must have an HMI state of at least `HMI_LIMITED`. It is also notable that you should only start one video stream per session. You may observe HMI state changes from `SDLProxyListener`'s protocol callback `onOnHMIStatus:`. An example of starting a video stream can be seen below:

```objective-c
- (void)onHMIStatus:(SDLOnHMIStatus *)notification {
  if ([notification.hmiLevel isEqualToEnum:SDLHMILevel.LIMITED]
      || [notification.hmiLevel isEqualToEnum:SDLHMILevel.FULL]) {
      // Start Audio Streaming
      ...
      if (self.proxy.streamingMediaManager.videoSessionConnected == NO
          && self.isAttemptingToStartVideoSession == NO) {
        self.isAttemptingToStartVideoSession = YES;
        [self.proxy.streamingMediaManager startVideoSessionWithStartBlock:^(BOOL success,
                                                                            NSError* error) {
            self.isAttemptingToStartVideoSession = NO;
            if (success) {
              // Video Session Connected
            } else if (error) {
                NSLog(@"Error starting video stream: %@", error.localizedDescription);
            } else {
                NSLog(@"Unknown Error starting video session.");
            }
        }];
      }
  }
}
```

### Sending Data to the Stream
Sending video data to the head unit must be provided to `SDLStreamingMediaManager` as a `CVImageBufferRef` (Apple documentation [here](https://developer.apple.com/library/mac/documentation/QuartzCore/Reference/CVImageBufferRef/)). Once the video stream has started, you will not see video appear until a few frames have been received. To send a frame, refer to the snippet below:

```objective-c
...
// Insert code to obtain Video Frame as CVImageBufferRef
...
if (self.proxy.streamingMediaManager.videoSessionConnected) {
  if ([self.proxy.streamingMediaManager sendVideoData:imageBuffer] == NO) {
    NSLog(@"Could not send Video Data");
  }
} else {
  // Video session has not started.
}
```

### Best Practices
* A constant stream of map frames is not necessary to maintain an image on the screen. Because of this, we advise that a batch of frames are only sent on map movement or location movement. This will keep the application's memory consumption lower.
* For an ideal user experience, we recommend sending 30 frames per second.

## Sending an Audio Stream
Navigation apps can use two different options to give the user audible feedback.

### 1. Using the App's Audio (Audio Stream)

Navigation apps are allowed to stream raw audio to be played by the head unit. The audio received this way is played immediately, and the current audio source will be attenuated. The raw audio has to be played with the following parameters:

* **Format**: PCM
* **Sample Rate**: 16k
* **Number of Channels**: 1
* **Bits Per Second (BPS)**: 16 bits per sample / 2 bytes per sample

In order to stream audio from a SDL app, we focus on the `SDLStreamingMediaManager` class. A reference to this class is available from an `SDLProxy` property `streamingMediaManager`.

### Starting the Stream
In order to start an audio stream, an app must have an HMI state of at least `HMI_LIMITED`. It is also notable that you should only start one audio stream per session. You may observe HMI state changes from `SDLProxyListener`'s protocol callback `onOnHMIStatus:`. An example of starting an audio stream can be seen below:

```objective-c
- (void)onHMIStatus:(SDLOnHMIStatus *)notification {
    if ([notification.hmiLevel isEqualToEnum:SDLHMILevel.LIMITED]
        || [notification.hmiLevel isEqualToEnum:SDLHMILevel.FULL]) {
        if (self.proxy.streamingMediaManager.audioSessionConnected == NO
            && self.isAttemptingToStartAudioSession == NO) {
          self.isAttemptingToStartAudioSession = YES;
          [self.proxy.streamingMediaManager startAudioSessionWithStartBlock:^(BOOL success,
                                                                              NSError* error) {
              self.isAttemptingToStartAudioSession = NO;
              if (success) {
                // Audio Session Connected
              } else if (error) {
                  NSLog(@"Error starting audio stream: %@", error.localizedDescription);
              } else {
                  NSLog(@"Unknown Error starting audio session.");
              }
          }];
        }
        ...
        // Start Video Streaming
    }
}
```

### Sending Data to the Stream
Once the audio stream is connected, data may be easily passed to the Head Unit. The function `sendAudioData:` provides us with whether or not the PCM Audio Data was successfully transferred to the Head Unit.

```objective-c
...
// Insert code to obtain PCM Audio Data as NSData
...
if (self.proxy.streamingMediaManager.audioSessionConnected) {
  if ([self.proxy.streamingMediaManager sendAudioData:audioData] == NO) {
    NSLog(@"Could not send Audio Data");
  }
} else {
  // Audio session has not started.
}
```

### 2. Embedded Text To Speech (TTS)

If the navigation app does not have their own TTS engine, the TTS engine that is embedded in the head unit can be used. To do this, use Alert Maneuver RPC parameter `ttsChunks` with the appropriate array of TTS Chunks before sending the Alert Maneuver request (see chapter "Using SDL's TurnByTurn message interface").

!!! note

Alert Maneuver allows speaking when presenting a Navigation Turn by Turn Advice. If you wish to provide speech outside of this, refer to using the Speak RPC in the [mobile weather tutorial](https://github.com/smartdevicelink/sdl_mobileweather_tutorial_ios/wiki/Section-2-Basic-communication#step-1-send-a-welcome-message).

!!!

## Receiving Touch Events

Navigation applications have support for touch events, including both single and multitouch events. This includes interactions such as panning and pinch. A developer may use the included `SDLTouchManager` class, or yourself by listening to the `SDLProxyListener` protocol callback.

### 1. Using `SDLTouchManager`

`SDLTouchManager` has multiple callbacks that will ease the implementation of touch events. The following callbacks are provided:

```objective-c
- (void)touchManager:(SDLTouchManager*)manager didReceiveSingleTapAtPoint:(CGPoint)point;
- (void)touchManager:(SDLTouchManager*)manager didReceiveDoubleTapAtPoint:(CGPoint)point;

- (void)touchManager:(SDLTouchManager *)manager panningDidStartAtPoint:(CGPoint)point;
- (void)touchManager:(SDLTouchManager*)manager didReceivePanningFromPoint:(CGPoint)fromPoint
                                                                  toPoint:(CGPoint)toPoint;
- (void)touchManager:(SDLTouchManager*)manager panningDidEndAtPoint:(CGPoint)point;

- (void)touchManager:(SDLTouchManager *)manager pinchDidStartAtPoint:(CGPoint)point;
- (void)touchManager:(SDLTouchManager*)manager didReceivePinchAtCenterPoint:(CGPoint)point
                                                                  withScale:(CGFloat)scale;
- (void)touchManager:(SDLTouchManager *)manager pinchDidEndAtPoint:(CGPoint)point;
```

!!! note

  * Points that are provided via these callbacks are in the device's coordinate space, not the head unit's.
  * `SDLTouchManager` is currently a work in progress, and will be available soon in the SmartDeviceLink Library.

!!!

### 2. Self Implementation of `onTouchEvent`

If apps want to have access to the raw touch data, the `onTouchEvent:` callback can be evaluated. This callback will be fired for every touch of the user and contains the following data:

#### Type

BEGIN
: Sent for the first touch event

MOVE
: Sent if the touch moved

END
: Sent when the touch is lifted

#### Event

touchEventId
: Unique ID of the touch. Increases for multiple touches (0, 1, 2, ...)

timeStamp
: Timestamp of the head unit time. Can be used to compare time passed between touches

coord
: X and Y coordinates in the head unit coordinate system. (0, 0) is the top left

## Using SDL's TurnByTurn message interface

Currently, to display a Turn by Turn advice, a combination of the `SDLShowConstantTBT` and `SDLAlertManeuver` RPCs must be used. The `SDLShowConstantTBT` RPC involves the data that will be shown on the head unit. The main properties of this object to set are `navigationText1`, `navigationText2`, and `turnIcon`. A best practice for navigation applications is to use the `navigationText1` as the advice to give (Turn Right) and `navigationText2` to provide the distance to that advice (3 mi). When an `SDLAlertManeuver` is sent, you may also include accompanying text that you would like the head unit to speak when an advice is displayed on screen (In 3 miles turn right.).

```objective-c
...
// Create SDLImage object for turnIcon.
...
SDLShowConstantTBT* turnByTurn = [[SDLShowConstantTBT alloc] init];
turnByTurn.navigationText1 = @"Turn Right";
turnByTurn.navigationText2 = @"3 mi";
turnByTurn.turnIcon = turnIcon;
// Send SDLShowConstantTBT

SDLAlertManeuver* alertManeuver = [[SDLAlertManeuver alloc] init];
alertManeuver.ttsChunks = [SDLTTSChunkFactory buildTTSChunksFromSimple:@"In 3 miles turn right"];
// Send SDLAlertManeuver
```

> Remember when sending a SDLImage, that the image must first be uploaded to the head unit via a PutFile RPC.

To clear a navigation advice from the screen, we send an `SDLShowConstantTBT` with the `maneuverComplete` property as `YES`. This specific RPC does not require an accompanying `SDLAlertManeuver`.  

```objective-c
SDLShowConstantTBT* turnByTurn = [[SDLShowConstantTBT alloc] init];
turnByTurn.maneuverComplete = @(YES);
// Send SDLShowConstantTBT
```

## Receiving Keyboard Input
Keyboard input is available via the `SDLPerformInteraction` RPC. For a general understanding of how this RPC works, reference the [mobile weather tutorial](https://github.com/smartdevicelink/sdl_mobileweather_tutorial_ios/wiki/Section-3-Change-units-with-interactions). As opposed to the normal process for using `SDLPerformInteraction` with a required `SDLCreateInteractionChoiceSet`, using the keyboard requires no interaction choice sets to be created beforehand. It does, however, require an empty array to be passed in. To show the perform interaction as a keyboard, we modify the `interactionLayout` property to be `KEYBOARD`. Note that while the vehicle is in motion, keyboard input will be unavailable (resulting in a grayed out keyboard).

```objective-c
SDLPerformInteraction* performInteraction = [[SDLPerformInteraction alloc] init];
performInteraction.initialText = @"Find Location";
performInteraction.interactionChoiceSetIDList = [@[] mutableCopy];
performInteraction.timeout = @(100000);
performInteraction.interactionMode = SDLInteractionMode.MANUAL_ONLY;
performInteraction.interactionLayout = SDLLayoutMode.KEYBOARD;
// Send RPC.
```

> In Ford's current SYNC 3 implementation of SmartDeviceLink, there is a bug resulting in the need for a valid interaction choice to be created beforehand and used in the RPC call.

# Changelog

Vers. | Date     | Changes
------|----------|-------------------
0.1.0 | 20160525 | Initial creation