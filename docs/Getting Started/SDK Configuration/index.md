## SDK Configuration
### 1. Get  a SDL Core
If you do not have a SDL enabled head unit for testing, you should build the [sdl_core project](https://github.com/smartdevicelink/sdl_core). The sdl_core project is an emulator that lets you simulate sending and receiving remote procedure calls between a smartphone app and a SDL Core.

### 2. Enable Background Capabilities
Your application must be able to maintain a connection to the SDL Core even when it is in the background. This capability must be explicitly enabled for your application (available for iOS 5+). To enable the feature, select your application's build target, go to *Capabilities*, *Background Modes*, and select *External accessory communication mode*.

### 3. Add SDL Protocol Strings
Your application must support a set of SDL protocol strings in order to be connected to SDL enabled head units. Go to your application's **.plist** file and add the following code under the top level dictionary.

!!! NOTE
This is only required for USB and Bluetooth enabled head units. It is not necessary during development using SDL Core.
!!!  

```
<key>UISupportedExternalAccessoryProtocols</key>
<array>
<string>com.smartdevicelink.prot29</string>
<string>com.smartdevicelink.prot28</string>
<string>com.smartdevicelink.prot27</string>
<string>com.smartdevicelink.prot26</string>
<string>com.smartdevicelink.prot25</string>
<string>com.smartdevicelink.prot24</string>
<string>com.smartdevicelink.prot23</string>
<string>com.smartdevicelink.prot22</string>
<string>com.smartdevicelink.prot21</string>
<string>com.smartdevicelink.prot20</string>
<string>com.smartdevicelink.prot19</string>
<string>com.smartdevicelink.prot18</string>
<string>com.smartdevicelink.prot17</string>
<string>com.smartdevicelink.prot16</string>
<string>com.smartdevicelink.prot15</string>
<string>com.smartdevicelink.prot14</string>
<string>com.smartdevicelink.prot13</string>
<string>com.smartdevicelink.prot12</string>
<string>com.smartdevicelink.prot11</string>
<string>com.smartdevicelink.prot10</string>
<string>com.smartdevicelink.prot9</string>
<string>com.smartdevicelink.prot8</string>
<string>com.smartdevicelink.prot7</string>
<string>com.smartdevicelink.prot6</string>
<string>com.smartdevicelink.prot5</string>
<string>com.smartdevicelink.prot4</string>
<string>com.smartdevicelink.prot3</string>
<string>com.smartdevicelink.prot2</string>
<string>com.smartdevicelink.prot1</string>
<string>com.smartdevicelink.prot0</string>
<string>com.ford.sync.prot0</string>
</array>
```  

### 4. Access the Documentation
You can find the latest reference documentation on [CocoaDocs](http://cocoadocs.org/docsets/SmartDeviceLink-iOS/).  
##### Download the Documentation    
Install this documentation to [Dash](https://kapeli.com/dash) or to Xcode by using [Docs for Xcode](https://documancer.com/xcode). On the [SDL Docs page](http://cocoadocs.org/docsets/SmartDeviceLink-iOS/), click the **upload** icon located in the upper right hand corner of the screen and add the documentation to Dash or Xcode.

### 5. Get an App Id
An app id is required for production level apps. The app id gives your app special permissions to access vehicle data. If your app does not need to access vehicle data, a dummy app id (i.e. create a fake id like "1234") is sufficient during the development stage. However, you must get an app id before releasing the app to the public.

To obtain an app id, sign up at [smartdevicelink.com](www.smartdevicelink.com).
