## Audio Streaming
Navigation apps are allowed to stream raw audio to be played by the head unit. The audio received this way is played immediately, and the current audio source will be attenuated. The raw audio has to be played with the following parameters:

* **Format**: PCM
* **Sample Rate**: 16k
* **Number of Channels**: 1
* **Bits Per Second (BPS)**: 16 bits per sample / 2 bytes per sample

In order to stream audio from a SDL app, we focus on the `SDLStreamingMediaManager` class. A reference to this class is available from an `SDLProxy` property `streamingMediaManager`.

#### Audio Stream Lifecycle
Currently, the lifecycle of the audio stream must be maintained by the developer. Below is a set of guidelines for when a device should stream frames, and when it should not. The main player in whether or not we should be streaming are HMI State. The audio stream may be opened in any application state.

HMI State   | Can Open Audio Stream | Should Close Audio Stream
------------|-----------------------|--------------------------
NONE        | No                    | Yes
BACKGROUND  | No                    | Yes
LIMITED     | Yes                   | No
FULL        | Yes                   | No

#### Starting the Stream
In order to start an audio stream, an app must have an HMI state of at least `HMI_LIMITED`. It is also notable that you should only start one audio stream per session. You may observe HMI state changes from `SDLProxyListener`'s protocol callback `onOnHMIStatus:`. An example of starting an audio stream can be seen below:

#### Swift
```swift
func hmiLevel(_ oldLevel: SDLHMILevel, didChangeTo newLevel: SDLHMILevel) {
    // Code for starting video stream
    if newLevel.isEqual(to: SDLHMILevel.limited()) || newLevel.isEqual(to: SDLHMILevel.full()) {
        startAudioSession()
    } else {
        stopAudioSession()
    }
}

private func stopAudioSession() {
    guard let streamManager = self.sdlManager.streamManager, streamManager.audioSessionConnected else {
      return
    }

    streamManager.stopAudioSession()
}

private func startAudioSession() {
    guard let streamManager = self.sdlManager.streamManager,
        streamManager.audioSessionConnected else {
        return
    }

    streamManager.startAudioSession(withTLS: .authenticateAndEncrypt) { (success, encryption, error) in
        if !success {
            if let error = error {
                NSLog("Error starting audio session. \(error.localizedDescription)")
            }
        } else {
            if encryption {
                // Audio will be encrypted
            } else {
                // Audio will not be encrypted
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
        [self startAudioSession];
    } else {
        [self stopAudioSession];
    }
}

- (void)stopAudioSession {
    if (!self.sdlManager.streamManager.audioSessionConnected) {
        return;
    }
    [self.sdlManager.streamManager stopAudioSession];
}

- (void)startAudioSession {
    if (!self.sdlManager.streamManager.audioSessionConnected) {
        return;
    }
    [self.sdlManager.streamManager startAudioSessionWithTLS:SDLEncryptionFlagAuthenticateAndEncrypt startBlock:^(BOOL success, BOOL encryption, NSError * _Nullable error) {
        if (!success) {
            if (error) {
                NSLog(@"Error starting video session. %@", error.localizedDescription);
              }
        } else {
            if (encryption) {
                // Audio will be encrypted
            } else {
                // Audio will not be encrypted
            }
        }
    }];
}
```

#### Sending Data to the Stream
Once the audio stream is connected, data may be easily passed to the Head Unit. The function `sendAudioData:` provides us with whether or not the PCM Audio Data was successfully transferred to the Head Unit.

#### Objective-C
```objective-c

NSData* audioData = <#Acquire Audio Data#>;

if ([self.sdlManager.streamManager sendAudioData:imageBuffer] == NO) {
  NSLog(@"Could not send Audio Data");
}
```

#### Swift
```swift
let audioData = <#Acquire Audio Data#>;

guard let streamManager = self.sdlManager.streamManager, streamManager.audioSessionConnected else {
  return
}

if streamManager.sendAudioData(imageBuffer) == false {
  print("Could not send Audio Data")
}
```