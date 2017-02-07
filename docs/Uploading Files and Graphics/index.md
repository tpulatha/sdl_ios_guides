## Uploading Files and Graphics
Graphics allows for you to better customize what you would like to have your users see, and provide a better User Interface.

To learn how to use these graphics once they are uploaded, please see [Displaying Information > Text, Images, and Buttons](Displaying Information/Text, Images, and Buttons).

When developing an application using SmartDeviceLink, two things must always be remembered when using graphics:

1. You may be connected to a head unit that does not display graphics.
2. You must upload them from your mobile device to Core before using them.

### Detecting if Graphics are Supported
Being able to know if graphics are supported is a very important feature of your application, as this avoids you uploading unneccessary images to the head unit. In order to see if graphics are supported, take a look at `SDLManager`'s `registerResponse` property once in the completion handler for `startWithReadyHandler`.

!!! note 
If you need to know how to create and setup `SDLManager`, please see [Getting Started > Integration Basics](Getting Started/Integration Basics).
!!!

#### Objective-C
```objc
__weak typeof (self) weakSelf = self;
[self.sdlManager startWithReadyHandler:^(BOOL success, NSError * _Nullable error) {
    if (!success) {
        NSLog(@"SDL errored starting up: %@", error);
        return;
    } 

    SDLDisplayCapabilities *displayCapabilities = weakSelf.sdlManager.registerResponse.displayCapabilities;
    BOOL areGraphicsSupported = NO;
    if (displayCapabilities != nil) {
        areGraphicsSupported = displayCapabilities.graphicSupported.boolValue;
    } 
}];
```

#### Swift
```swift
sdlManager.start { (success, error) in
    if success == false {
        print("SDL errored starting up: \(error)")
        return
    }
    
    var areGraphicsSupported = false
    if let displayCapabilities = self.sdlManager.registerResponse?.displayCapabilities {
        areGraphicsSupported = displayCapabilities.graphicSupported.boolValue
    }
}
```

### Uploading a File using SDLFileManager
As of SDL 4.3, we have provided a class that makes managing files easier. `SDLFileManager` handles uploading files, like images, and keeping track of what already exists and what does not. To assist, there are two classes: `SDLFile` and `SDLArtwork`. `SDLFile` is the base class for file uploads and handles pulling data from local `NSURL`s and `NSData`. `SDLArtwork` is subclass of this, and provides additional functionality such as creating a file from a `UIImage`.

#### Objective-C
```objc
UIImage* image = [UIImage imageNamed:@"<#Image Name#>"];
if (!image) {
    NSLog(@"Error reading from Assets");
    return;    
}

SDLArtwork* file = [SDLArtwork artworkWithImage:image name:@"<#Name to Upload As#>" asImageFormat:SDLArtworkImageFormatJPG /* or SDLArtworkImageFormatPNG */];

[self.sdlManager.fileManager uploadFile:file completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError * _Nullable error) {
	if (error) {
		if (error.code == SDLFileManagerErrorCannotOverwrite) {
	        // Attempting to replace a file, but failed due to settings in SDLArtwork.
	    } else {
	        // Error uploading
	    }
	    return;
	}

    // Successfully uploaded.
}];
```

#### Swift
```swift
guard let image = UIImage(named: "<#Image Name#>") else {
	print("Error reading from Assets")
	return
}
let file = SDLArtwork.artwork(with: image, name: "<#Name to Upload As#>", as: .JPG /* or .PNG */)

sdlManager.fileManager.uploadFile(file) { (success, bytesAvailable, error) in
    if let error = error as? NSError {
        if error.code == SDLFileManagerError.cannotOverwrite.rawValue {
            // Attempting to replace a file, but failed due to settings in SDLArtwork.
        } else {
            // Error uploading
        }
        return;
    }
    
    // Successfully uploaded
}
```

### File Naming
The file name can only consist of letters (a-Z) and numbers (0-9), otherwise the SDL Core may fail to find the uploaded file (even if it was uploaded successfully).

### File Persistance
`SDLFile`, and it's subclass `SDLArtwork`, support uploading persistant images, i.e. images that do not become deleted when your application disconnects. Persistance should be used for images relating to your UI, and not for dynamic aspects, such as Album Artwork.

#### Objective-C
```objc
file.persistent = YES;
```

#### Swift
```swift
file.persistent = true
```

!!! note
Be aware that persistance will not work if space on the head unit is limited. `SDLFileManager` will always handle uploading images if they are non-existant. 
!!!

### Overwrite Stored Files
If a file being uploaded has the same name as an already uploaded image, the new image will be ignored. To override this setting, set the `SDLFile`’s `overwrite` property to true.

#### Objective-C
```objc
file.overwrite = YES;
```

#### Swift
```swift
file.overwrite = true
```

### Check the Amount of File Storage
To find the amount of file storage left on the head unit, use the `SDLFileManager`’s `bytesAvailable` property.

#### Objective-C
```objc
NSUInteger bytesAvailable = self.sdlManager.fileManager.bytesAvailable;
```

#### Swift
```swift
let bytesAvailable = sdlManager.fileManager.bytesAvailable
```

### Check if a File Has Already Been Uploaded
Although the file manager will return with an error if you attempt to upload a file of the same name that already exists, you may still be able to find out the currently uploaded images via `SDLFileManager`'s `remoteFileNames` property.

#### Objective-C
```objc
BOOL isFileOnHeadUnit = [self.sdlManager.fileManager.remoteFileNames containsObject:@"<#Name Uploaded As#>"];
```

#### Swift
```swift
let isFileOnHeadUnit = sdlManager.fileManager.remoteFileNames.contains("<#Name Uploaded As#>")
```

### Delete Stored Files
Use the file manager’s delete request to delete a file associated with a file name.

#### Objective-C
```objc
[self.sdlManager.fileManager deleteRemoteFileWithName:@"<#Save As Name#>" completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError *error) {
    if (success) {
        // Image was deleted successfully
    }
}];
```

#### Swift
```swift
sdlManager.fileManager.deleteRemoteFileWithName("<#Save As Name#>") { (success, bytesAvailable, error) in
    if success {
        // Image was deleted successfully
    }
}
```

## Image Specifics
### Image File Type
Images may be formatted as PNG, JPEG, or BMP. Check the `displayCapability` properties to find out what image formats the head unit supports.

### Image Sizes
If an image is uploaded that is larger than the supported size, that image will be scaled down to accomodate. All image sizes are available from the `SDLManager`'s `registerResponse` property once in the completion handler for `startWithReadyHandler`.

#### Image Specifications
ImageName 		  	 | Used in RPC				  |	Details 																							  |	Height 		 | Width  | Type
---------------------|----------------------------|-------------------------------------------------------------------------------------------------------|--------------|--------|-------
softButtonImage		 | Show 					  | Will be shown on softbuttons on the base screen														  | 70px         | 70px   | png, jpg, bmp
choiceImage 		 | CreateInteractionChoiceSet | Will be shown in the manual part of an performInteraction either big (ICON_ONLY) or small (LIST_ONLY) | 70px         | 70px   | png, jpg, bmp
choiceSecondaryImage | CreateInteractionChoiceSet | Will be shown on the right side of an entry in (LIST_ONLY) performInteraction						  | 35px 		 | 35px   | png, jpg, bmp
vrHelpItem			 | SetGlobalProperties		  | Will be shown during voice interaction 																  | 35px 		 | 35px   | png, jpg, bmp
menuIcon			 | SetGlobalProperties		  | Will be shown on the “More…” button 																  | 35px 		 | 35px   | png, jpg, bmp
cmdIcon				 | AddCommand				  | Will be shown for commands in the "More…" menu 														  | 35px 		 | 35px   | png, jpg, bmp
appIcon 			 | SetAppIcon				  | Will be shown as Icon in the "Mobile Apps" menu 													  | 70px 		 | 70px   | png, jpg, bmp
graphic 			 | Show 					  | Will be shown on the basescreen as cover art 														  | 185px 		 | 185px  | png, jpg, bmp
