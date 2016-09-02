## Connecting to SDL

### SDL Core
If you are connecting to the SDL Core emulator, make sure you are implementing a TCP connection, and that the emulator and app are connected to the same network (i.e. remember to set the correct IP address and port number in the `SDLLifecycleConfiguration`). The IP will most likely be the IP address of the operating system running the SDL Core emulator. The port will most likely be `12345`.

!!! IMPORTANT
Known issues:
* When app is in the background mode, the app will be unable to communicate with SDL Core. This will work on IAP connections.
* Audio will not play on the SDL Core. Only IAP connections are currently able to play audio.
!!!

### Vehicle Head Unit or Technical Development Kit (TDK)
#### Production
If you want to connect your iOS device directly to a head unit or TDK, make sure you are implementing an iAP (default) connection in the `SDLLifecycleConfiguration`. Then build the app on your device and connect the iOS device to the head unit or TDK using a USB cord.

#### Debugging
If you are using a head unit or TDK  and want to see debug logs in Xcode while the app is running, you must use another app called the [relay app](https://github.com/smartdevicelink/relay_app_ios) to help you connect to the device. When using the relay app, make sure to implement a TCP connection. Please see the readme for the relay app to learn how to setup the connection between the device, the relay app and your app.

!!! NOTE
If you wish to test your app using a TDK or vehicle head unit, you may use the relay app to continue to get log messages and debugging capabilities. Connect the device with the relay app to the vehicle head unit, then use TCP debugging (see below) to connect your app to the relay app's server. The same issues apply to using the Relay app with a production unit as do connecting to SDL Core, above.
!!!
