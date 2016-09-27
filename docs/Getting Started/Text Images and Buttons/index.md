## Text, Images, and Buttons
All text, images, and buttons on the HMI screen must be sent as part of a `SDLShow` RPC. Subscribe buttons are sent as part of a `SDLSubscribeButton` RPC.

#### Text
A maximum of four lines of text can be set in `SDLShow` RPC, however, some templates may only support 1, 2, or 3 lines of text. If all four lines of text are set in the `SDLShow` RPC, but the template only supports three lines of text, then the fourth line will simply be ignored.
```swift
let show: SDLShow = SDLShow()
show.mainField1 = "line 1 of text"
show.mainField2 = "line 2 of text"
show.mainField3 = "line 3 of text"
show.mainField4 = "line 4 of text"
sdlManager?.sendRequest(show, withResponseHandler: { (request, response, error) in
    if response?.resultCode == SDLResult.SUCCESS() {
        // The text has been set successfully
    }
})
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

```swift
// SDLArtwork
let imageName: String = "unique name for image"
let image: UIImage = UIImage(named: imageName)
let artwork: SDLArtwork = SDLArtwork(image: image, name: imageName, as: SDLArtworkImageFormat.JPG)

// Upload the image data to the head unit
sdlManager?.fileManager.uploadFile(artwork, completionHandler: { (successful, length, error) in
    if successful {
        // Image has been uploaded
    }
    else {
        // Image was not uploaded
    }
})
```

##### 3. Storing Images
An image can be stored forever (persistent) on the head unit, or for only as long as the app is running (ephemeral). Set the `persistent` parameter in `SDLArtwork` to false if the image should only be stored as long as the app is open. Set it to true if the image should stay on the head unit forever. Remember that your app has a finite amount of storage, so it may be best to use ephemeral images in image intensive apps. To delete a persistent image, you will either need to manually delete the image or do a factory reset on the head unit.

```swift
// SDLArtwork
let imageName: String = "unique name for image"
let image: UIImage = UIImage(named: imageName)
let artwork: SDLArtwork = SDLArtwork(image: image, name: imageName, persistent: false, as: SDLArtworkImageFormat.JPG)
```

###### Check the Amount of File Storage
To find the amount of file storage left on the head unit, use the `SDLManager`’s file manager, `bytesAvailable` method.
```swift
let bytesAvailable = sdlManager?.fileManager.bytesAvailable
```

##### 4. Check if an Image Has Already Been Uploaded
Use the file manager to check if an image name has already been used to upload an image  to the head unit.
```swift
let fileIsOnHeadUnit: Bool = sdlManager?.fileManager.remoteFileNames.contains(“name for image”)
```

##### 5. Overwrite Stored Images
If an image being uploaded has the same name as an already uploaded image, the new image will be ignored. To override this setting, set the `SDLArtwork`’s `overwrite` property to true.
```swift
// SDLArtwork
let imageName: String = "unique name for image"
let image: UIImage = UIImage(named: imageName)
let artwork: SDLArtwork = SDLArtwork(image: image, name: imageName, persistent: false, as: SDLArtworkImageFormat.JPG)
artwork.overwrite = true
```

##### 6. Delete Stored Images
Use the file manager’s delete request to delete an image associated with an image name.
```swift
sdlManager.fileManager.deleteRemoteFileWithName("name for image", completionHandler: { (successful, length, error) in
    if successful {
        // Image was deleted successfully
    }
})
```

##### 7. Show the Image on a Head Unit
Once the image has been uploaded to the head unit, you can show the image on the user interface. To send the image, create a `SDLImage`, and send it using the `SDLShow` RPC. The `value` property should be set to the same value used in the `name` property of the uploaded `SDLArtwork`. Since you are uploading your own image, the `imageType` property should be set to `dynamic`.

```swift
// SDLImage
let sdlImage: SDLImage = SDLImage()
sdlImage.imageType = SDLImageType.dynamic()
sdlImage.value = artwork.name

let show: SDLShow = SDLShow()
show.graphic = sdlImage
sdlManager?.send(show)
```

##### 8. Upload Image Example
```swift
func uploadAnImageExample() {
    // SDLArtwork
    let imageName: String = "unique name for image"
    let image: UIImage = UIImage(named: imageName) ?? UIImage()
    let artwork: SDLArtwork = SDLArtwork(image: image, name: imageName, persistent: false, as: SDLArtworkImageFormat.JPG)
    artwork.overwrite = false

    uploadImage(artwork: artwork) { (sdlImage) in
        if let sdlImage = sdlImage {
            // Image was uploaded successfully
            // You can now show the image on the head unit
            let show: SDLShow = SDLShow()
            show.graphic = sdlImage
            sdlManager?.send(show)
        }
    }
}

typealias ImageUploadedCompletionHandler = (_ sdlImage: SDLImage?) -> Void
func uploadImage(artwork: SDLArtwork, completionHandler: @escaping ImageUploadedCompletionHandler) {

    // SDLImage
    let sdlImage: SDLImage = SDLImage()
    sdlImage.imageType = SDLImageType.dynamic()
    sdlImage.value = artwork.name

    // Check if the image has already been uploaded to the head unit
    let fileIsOnHeadUnit = sdlManager?.fileManager.remoteFileNames.contains(artwork.name)
    if fileIsOnHeadUnit == true {
        // The image has already been uploaded to the head unit, simply return the image data
        completionHandler(sdlImage)
    }
    else {
        // Upload the image data to the head unit
        sdlManager?.fileManager.uploadFile(artwork, completionHandler: { (successful, length, error) in
            if successful {
                completionHandler(sdlImage)
            }
            else {
                completionHandler(nil)
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

```swift
let show = SDLShow()
let softButton = SDLSoftButton()

// Button handler - This is called when user presses the button
softButton.handler = { (notification) in
    if let notification: SDLOnButtonEvent = notification as? SDLOnButtonEvent {
        if notification.buttonEventMode == SDLButtonEventMode.BUTTONDOWN() {
            // The user has pressed down on the button
        }
        if notification.buttonEventMode == SDLButtonEventMode.BUTTONUP() {
            // The user has release the button
        }
    }
}

// Button type can be text, image, or both text and image
softButton.type = SDLSoftButtonType.BOTH()

// Button text
softButton.text = "title for button"

// Button image
let image: SDLImage = SDLImage()
image.imageType = SDLImageType.DYNAMIC()
image.value = "button identifier that was sent with SDLPutfile"
softButton.image = image

// The buttons are set as part of an array
show.softButtons = [softButton]

// Send the request
sdlManager?.sendRequest(show, withResponseHandler: { (request, response, error) in
    if response?.resultCode == SDLResult.SUCCESS() {
        // The button was created successfully
    }
})
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
