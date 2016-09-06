## Installation
In order for your app to communicate with a SmartDeviceLink (SDL) Core, the SDL software development kit (SDK) must be installed in your app. The following steps will guide you through adding the SDL SDK to your workspace and configuring the environment.
### Install SDL SDK
We have provided three different ways to install the SDL SDK in your project: CocoaPods, Carthage, or manually.
#### CocoaPods Installation
1. Xcode should be closed for the following steps.
2. Open the terminal app on your Mac.
3. Make sure you have the latest version of [CocoaPods](https://cocoapods.org) installed. For more information on installing CocoaPods on your system please consult:  https://cocoapods.org.

        ```
        sudo gem install cocoapods
        ```
4. Navigate to the root directory of your app. Make sure your current folder contains the **.xcodeproj** file
5. Create a new Podfile.

        ```
        pod init
        ```
6. In the Podfile, add the following text. This tells CocoaPods to install the SDL for iOS framework. This will install the latest SDL SDK version up to minor version 4.3.0

        ```
        target ‘YourAppName’ do
          pod ‘SmartDeviceLink-iOS’ ‘~> 4.3.0’
        end
        ```
7. Install SDK for iOS:

        ```
        pod install
        ```
8. There will be a newly created **.xcworkspace** file in the directory in addition to the **.xcodeproj** file. Always use the **.xcworkspace** file from now on.
9. Open the **.xcworkspace** file. To open from the  terminal, type:

        ```
        open YourAppName.xcworkspace
        ```
#### Carthage
SDL iOS supports Carthage! Install using Carthage by following [this guide](https://github.com/Carthage/Carthage#adding-frameworks-to-an-application). Carthage supports iOS 8+.
