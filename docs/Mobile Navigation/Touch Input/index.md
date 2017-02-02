## Touch Input

Navigation applications have support for touch events, including both single and multitouch events. This includes interactions such as panning and pinch. A developer may use the included `SDLTouchManager` class, or yourself by listening to the `SDLDidReceiveTouchEventNotification` notification.

### 1. Using `SDLTouchManager`

`SDLTouchManager` has multiple callbacks that will ease the implementation of touch events. The following callbacks are provided:

#### Swift
```swift
optional public func touchManager(_ manager: SDLTouchManager, didReceiveSingleTapAt point: CGPoint)
optional public func touchManager(_ manager: SDLTouchManager, didReceiveDoubleTapAt point: CGPoint)

optional public func touchManager(_ manager: SDLTouchManager, panningDidStartAt point: CGPoint)
optional public func touchManager(_ manager: SDLTouchManager, didReceivePanningFrom fromPoint: CGPoint, to toPoint: CGPoint)
optional public func touchManager(_ manager: SDLTouchManager, panningDidEndAt point: CGPoint)

optional public func touchManager(_ manager: SDLTouchManager, pinchDidStartAtCenter point: CGPoint)
optional public func touchManager(_ manager: SDLTouchManager, didReceivePinchAtCenter point: CGPoint, withScale scale: CGFloat)
optional public func touchManager(_ manager: SDLTouchManager, pinchDidEndAtCenter point: CGPoint)
```

#### Objective-C
```objc
- (void)touchManager:(SDLTouchManager*)manager didReceiveSingleTapAtPoint:(CGPoint)point;
- (void)touchManager:(SDLTouchManager*)manager didReceiveDoubleTapAtPoint:(CGPoint)point;

- (void)touchManager:(SDLTouchManager *)manager panningDidStartAtPoint:(CGPoint)point;
- (void)touchManager:(SDLTouchManager*)manager didReceivePanningFromPoint:(CGPoint)fromPoint
                                                                  toPoint:(CGPoint)toPoint;
- (void)touchManager:(SDLTouchManager*)manager panningDidEndAtPoint:(CGPoint)point;

- (void)touchManager:(SDLTouchManager *)manager pinchDidStartAtCenterPoint:(CGPoint)point;
- (void)touchManager:(SDLTouchManager*)manager didReceivePinchAtCenterPoint:(CGPoint)point
                                                                  withScale:(CGFloat)scale;
- (void)touchManager:(SDLTouchManager *)manager pinchDidEndAtCenterPoint:(CGPoint)point;
```

!!! note

  * Points that are provided via these callbacks are in the head unit's coordinate space.

!!!

### 2. Self Implementation of `onTouchEvent`

If apps want to have access to the raw touch data, the `SDLDidReceiveTouchEventNotification` notification can be evaluated. This callback will be fired for every touch of the user and contains the following data:

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

#### Example

#### Swift
```swift
// To Register
NotificationCenter.default.addObserver(self, selector: #selector(touchEventAvailable(_:)), name: .SDLDidReceiveTouchEvent, object: nil)

// On Receive
@objc private func touchEventAvailable(_ notification: SDLRPCNotificationNotification) {
    guard let touchEvent = notification.notification as? SDLOnTouchEvent else {
        print("Error retrieving onTouchEvent object")
        return
    }

    // Grab something like type
    let type = touchEvent.type
}
```

#### Objective-C
```objc
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(touchEventAvailable:) name:SDLDidReceiveTouchEventNotification object:nil];

- (void)touchEventAvailable:(SDLRPCNotificationNotification *)notification {
    if (![notification.notification isKindOfClass:SDLOnTouchEvent.class]) {
      return;
    }
    SDLOnTouchEvent* touchEvent = (SDLOnTouchEvent *)notification.notification;

    // Grab something like type
    SDLTouchType* type = touchEvent.type;
}

```