## Text, Images, and Buttons
All text, images, and buttons on the HMI screen must be sent as part of a `SDLShow` RPC. Subscribe buttons are sent as part of a `SDLSubscribeButton` RPC.

#### Text
A maximum of four lines of text can be set in `SDLShow` RPC, however, some templates may only support 1, 2, or 3 lines of text. If all four lines of text are set in the `SDLShow` RPC, but the template only supports three lines of text, then the fourth line will simply be ignored.

**Objective-C**
```objc
SDLShow* show = [[SDLShow alloc] initWithMainField1:@"<Line 1 of Text#>" mainField2:@"<Line 2 of Text#>" mainField3:@"<Line 3 of Text#>" mainField4:@"<Line 4 of Text#>" alignment:SDLTextAlignment.CENTERED()];
self.manager sendRequest:show withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if ([response.resultCode isEqualToEnum:SDLResult.SUCCESS]) {
      // The text has been set successfully
    }
}];
```

**Swift**
```swift
let show = SDLShow(mainField1: "<#Line 1 of Text#>", mainField2: "<#Line 2 of Text#>", mainField3: "<#Line 3 of Text#>", mainField4: "<#Line 4 of Text#>", alignment: .centered())
manager?.send(show) { (request, response, error) in
    guard let response = response else { return }
    if response.resultCode.isEqual(to: SDLResult.success()) {
        // The text has been set successfully
    }
}
```

#### Images
The position and size of images on the screen is determined by the currently set template. All images must first be uploaded the SDL Core using the `SDLManager`’s file manager before being used in a `SDLShow` RPC. Once the image has been successfully uploaded, the app will be notified by the SDL Core.

!!! NOTE
Some head units you may be connected to may not support images at all. Please consult the `graphicsSupported` property in the `display capabilities` property of the `RegisterAppInterface` response.
!!!

##### 1. Image File Type
Images may be formatted as PNG, JPEG, or BMP. Check the `displayCapability` properties to find out what image formats the head unit supports.

##### 2. Upload an Image
Use the SDLArtwork class to easily upload images to the head unit. The image name is a unique id for the image.

!!! NOTE
The image name can only consist of letters (a-Z) and numbers (0-9), otherwise the SDL Core may fail to find the uploaded image (even if it was uploaded successfully).
!!!

**Objective-C**
```objc
UIImage* image = <#Create Image#>
SDLArtwork* artwork = [SDLArtwork artworkWithImage:image name:@"<#Save As Name#>" asImageFormat:SDLArtworkImageFormatPNG /* or SDLArtworkImageFormatJPG */];
[self.manager.fileManager uploadFile:artwork completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError * _Nullable error) {
      if (success) {
          // Image has been uploaded
      } else {
          // Image was not uploaded
      }
}];
```

**Swift**
```swift
// SDLArtwork
let image = <#Create UIImage#>
let artwork = SDLArtwork(image: image, name: "<#Save As Name#>", as: .PNG /* or .JPG */)

// Upload the image data to the head unit
manager?.fileManager.uploadFile(artwork) { (success, bytesAvailable, error) in
    if success {
        // Image has been uploaded
    } else {
        // Image was not uploaded
    }
}
```

##### 3. Storing Images
An image can be stored forever (persistent) on the head unit, or for only as long as the app is running (ephemeral). Set the `persistent` parameter in `SDLArtwork` to false if the image should only be stored as long as the app is open. Set it to true if the image should stay on the head unit forever. Remember that your app has a finite amount of storage, so it may be best to use ephemeral images in image intensive apps. To delete a persistent image, you will either need to manually delete the image or do a factory reset on the head unit.

**Objective-C**
```objc
UIImage* image = <#Create Image#>
SDLArtwork* artwork = [SDLArtwork alloc] initWithImage:image name:@"<#Save As Name#>" persistent:NO asImageFormat:SDLArtworkImageFormatPNG /* or SDLArtworkImageFormatJPG */];
```

**Swift**
```swift
// SDLArtwork
let image = <#Create UIImage#>
let artwork = SDLArtwork(image: image, name: "<#Save As Name#>", persistent: false, as: .PNG /* or .JPG */)
```

###### Check the Amount of File Storage
To find the amount of file storage left on the head unit, use the `SDLManager`’s file manager, `bytesAvailable` method.

**Objective-C**
```objc
NSUInteger bytesAvailable = self.manager.fileManager.bytesAvailable;
```

**Swift**
```swift
let bytesAvailable = manager?.fileManager.bytesAvailable
```

##### 4. Check if an Image Has Already Been Uploaded
Use the file manager to check if an image name has already been used to upload an image  to the head unit.

**Objective-C**
```objc
BOOL isFileOnHeadUnit = [self.manager.fileManager.remoteFileNames containsObject:@"<#Save As Name#>"];
```

**Swift**
```swift
let isFileOnHeadUnit = manager?.fileManager.remoteFileNames.contains("<#Save As Name#>")
```

##### 5. Overwrite Stored Images
If an image being uploaded has the same name as an already uploaded image, the new image will be ignored. To override this setting, set the `SDLArtwork`’s `overwrite` property to true.

**Objective-C**
```objc
UIImage* image = <#Create Image#>
SDLArtwork* artwork = [SDLArtwork alloc] initWithImage:image name:@"<#Save As Name#>" persistent:NO asImageFormat:SDLArtworkImageFormatPNG /* or SDLArtworkImageFormatJPG */];
artwork.overwrite = YES;
```

**Swift**
```swift
// SDLArtwork
let image = <#Create UIImage#>
let artwork = SDLArtwork(image: image, name: "<#Save As Name#>", persistent: false, as: .PNG /* or .JPG */)
artwork.overwrite = true
```

##### 6. Delete Stored Images
Use the file manager’s delete request to delete an image associated with an image name.

**Objective-C**
```objc
[self.manager.fileManager deleteRemoteFileWithName:@"<#Save As Name#>" completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError *error) {
    if (success) {
        // Image was deleted successfully
    }
}];
```

**Swift**
```swift
manager?.fileManager.deleteRemoteFileWithName("<#Save As Name#>") { (success, length, error) in
    if success {
        // Image was deleted successfully
    }
}
```

##### 7. Show the Image on a Head Unit
Once the image has been uploaded to the head unit, you can show the image on the user interface. To send the image, create a `SDLImage`, and send it using the `SDLShow` RPC. The `value` property should be set to the same value used in the `name` property of the uploaded `SDLArtwork`. Since you are uploading your own image, the `imageType` property should be set to `dynamic`.

**Objective-C**
```objc
SDLImage* image = [[SDLImage alloc] initWithName:@"<#Save As Name#>" ofType:SDLImageType.DYNAMIC];

SDLShow* show = [[SDLShow alloc] init];
show.graphic = image;

self.manager sendRequest:show withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if ([response.resultCode isEqualToEnum:SDLResult.SUCCESS]) {
      // The text has been set successfully
    }
}];
```

**Swift**
```swift
let sdlImage = SDLImage(name: "<#Save As Name", of: .dynamic())

let show = SDLShow()
show.graphic = image

manager?.send(show) { (request, response, error) in
    guard let response = response else { return }
    if response.resultCode.isEqual(to: SDLResult.success()) {
      // Success
    }
}
```

##### 8. Upload Image Example
**Objective-C**
```objc
- (void)uploadAnImageExample {
    // SDLArtwork
    UIImage* image = <#Create Image#>
    SDLArtwork* artwork = [SDLArtwork artworkWithImage:image name:@"<#Save As Name#>" asImageFormat:SDLArtworkImageFormatPNG /* or SDLArtworkImageFormatJPG */];

    [self uploadArtwork:artwork completionHandler:^(SDLImage *__nullable sdlImage, NSError *__nullable error) {
        if (image != nil) {
            // Image was uploaded successfully
            // You can now show the image on the head unit
            SDLShow* show = [[SDLShow alloc] init];
            show.graphic = sdlImage;
            [self.manager sendRequest:show withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
                if ([response.resultCode isEqualToEnum:SDLResult.SUCCESS]) {
                    // Successful Show update.
                }
            }];
        } else if (error) {
            NSLog(@"Error: %@", error.localizedDescription);
        }
    }];
}

typedef void (^ImageUploadedCompletionHandler)(SDLImage *__nullable sdlImage, NSError *__nullable error);
- (void)uploadArtwork:(SDLArtwork *)artwork completionHandler:(ImageUploadedCompletionHandler)completionHandler {
    // SDLImage
    __block SDLImage *image = [[SDLImage alloc] initWithName:artwork.name ofType:SDLImageType.DYNAMIC];

    BOOL isFileOnHeadUnit = [self.manager.fileManager.remoteFileNames containsObject:artwork.name];
    if (isFileOnHeadUnit) {
        // The image has already been uploaded to the head unit, simply return the image data
        completionHandler(image, nil);
    } else {
      [self.manager.fileManager uploadFile:artwork completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError * _Nullable error) {
            if (success) {
                completionHandler(image, nil);
            } else {
                completionHandler(nil, error);
            }
      }];
    }
}
```

**Swift**
```swift
func uploadAnImageExample() {
    // SDLArtwork
    let image = <#Create UIImage#>
    let artwork = SDLArtwork(image: image, name: "<#Save As Name#>", persistent: false, as: .PNG /* or .JPG */)

    upload(artwork: artwork) { (sdlImage, error) in
        if let sdlImage = sdlImage {
            // Image was uploaded successfully
            // You can now show the image on the head unit
            let show: SDLShow = SDLShow()
            show.graphic = sdlImage
            self.manager?.send(show) { (request, response, error) in
                guard let response = response else { return }
                if response.resultCode.isEqual(to: SDLResult.success()) {
                    // Successful Show update.
                }
            }
        } else if let error = error {
            print("Error: \(error.localizedDescription)")
        }
    }
}

typealias ImageUploadedCompletionHandler = (_ sdlImage: SDLImage?, _ error: Error?) -> Void
func upload(artwork: SDLArtwork, completionHandler: @escaping ImageUploadedCompletionHandler) {

    // SDLImage
    let image = SDLImage(name: artwork.name of:.dynamic())

    // Check if the image has already been uploaded to the head unit
    let fileIsOnHeadUnit = manager?.fileManager.remoteFileNames.contains(artwork.name)
    if fileIsOnHeadUnit == true {
        // The image has already been uploaded to the head unit, simply return the image data
        completionHandler(image, nil)
    } else {
        // Upload the image data to the head unit
        manager?.fileManager.uploadFile(artwork, completionHandler: { (successful, length, error) in
            if successful {
                completionHandler(image, nil)
            }
            else {
                completionHandler(image, nil)
            }
        })
    }
}
```

#### Soft & Subscribe Buttons
Buttons on the HMI screen are referred to as soft buttons to distinguish them from hard buttons, which are physical buttons on the head unit. Don’t confuse soft buttons with subscribe buttons, which are buttons that can detect user selection on hard buttons (or built-in soft buttons).
##### Soft Buttons
Soft buttons can be created with text, images or both text and images. The location, size, and number of soft buttons visible on the screen depends on the template. If the button has an image, remember to upload the image first to the head unit before setting the image in the `SDLSoftButton` instance.

!!! NOTE
If a button type is set to `image` or `both` but no image is stored on the head unit, the button might not show up on the HMI screen.
!!!

**Objective-C**
```objc
SDLSoftButton* softButton = [[SDLSoftButton alloc] init];

softButton.handler = ^(SDLRPCNotification *notification) {
    if (![notification isKindOfClass:SDLOnButtonEvent.class]) {
        return;
    }
    SDLOnButtonEvent* onButtonEvent = (SDLOnButtonEvent*)notification;
    if ([onButtonEvent.buttonEventMode isEqualToEnum:SDLButtonEventMode.BUTTONDOWN]) {
      // The user has pressed down on the button
    } else if ([onButtonEvent.buttonEventMode isEqualToEnum:SDLButtonEventMode.BUTTONUP]) {
      // The user has released the button
    }
};

// Button type can be text, image, or both text and image
softButton.type = SDLSoftButtonType.BOTH;

// Button text
softButton.text = @"<#Button Text#>";

// Button image
softButton.image = [[SDLImage alloc] initWithName:@"<#Save As Name#>" ofType: SDLImageType.DYNAMIC];

SDLShow *show = [[SDLShow alloc] init];

// The buttons are set as part of an array
show.softButtons = [NSMutableArray arrayWithObject:softButton]

// Send the request
[self.manager sendRequest:show withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if ([response.resultCode isEqualToEnum:SDLResult.SUCCESS]) {
      // The button was created successfully
    }
}];
```

**Swift**
```swift
let softButton = SDLSoftButton()!

// Button handler - This is called when user presses the button
softButton.handler = { (notification) in
    guard let onButtonEvent = notification as? SDLOnButtonEvent else { return }
    if onButtonEvent.buttonEventMode.isEqual(to: SDLButtonEventMode.buttondown()) {
        // The user has pressed down on the button
    } else if onButtonEvent.buttonEventMode.isEqual(to: SDLButtonEventMode.buttonup()) {
        // The user has released the button
    }
}

// Button type can be text, image, or both text and image
softButton.type = .both()

// Button text
softButton.text = "<#Button Text#>"

// Button image
softButton.image = SDLImage(name: "<#Save As Name#>", of: .dynamic())

let show = SDLShow()!

// The buttons are set as part of an array
show.softButtons = [softButton]

// Send the request
manager?.send(show) { (request, response, error) in
    guard let response = response else { return }
    if response.resultCode.isEqual(to: SDLResult.success()) {
        // The button was created successfully
    }
}
```
##### Subscribe Buttons
Subscribe buttons are used to detect changes to hard buttons. You can subscribe to the following hard buttons:

| Button  | Template | Button Type |
| ------------- | ------------- | ------------- |
| Ok (play/pause) | media template only | soft button and hard button |
| Seek left | media template only | soft button and hard button |
| Seek right | media template only | soft button and hard button |
| Tune up | media template only | hard button |
| Tune down | media template only | hard button |
| Preset 0-9 | any template | hard button |
| Search | any template |hard button |
| Custom | any template | hard button |

Audio buttons like the OK (i.e. the `play/pause` button), seek left, seek right, tune up, and tune down buttons can only be used with a media template. The OK, seek left, and seek right buttons will also show up on the screen in a predefined location dictated by the media template on touchscreens. The app will be notified when the user selects the subscribe button on the screen or when the user manipulates the corresponding hard button.

!!! NOTE
You will automatically be assigned the media template if you set your configuration app type as `MEDIA`.
!!!
