## Designing a User Interface
### Designing for Different User Interfaces
Each car manufacturer has different user interface style guidelines, so the number of lines of text, buttons, and images supported will vary between different type of head units. When the app first connects to the SDL Core, a `RegisterAppInterface` RPC will be sent by the SDL Core containing the `displayCapability` properties. You can use this information to determine how to layout the user interface.

The `RegisterAppInterface` response contains information about the display type, the type of images supported, the number of text fields supported, the HMI display language, and a lot of other useful properties. For a full list of `displayCapability` properties returned by the `RegisterAppInterface` response, please consult the *SDLRegisterAppInterfaceResponse.h* file.

### Templates
Each car manufacturer supports a set of templates for the user interface. These templates determine the position and size of the text, images, and buttons on the screen. A list of supported templates is sent with RegisterAppInterface response.  

To change a template at any time, send a `SDLSetDisplayLayout` RPC to the SDL Core. If you want to ensure that the new template is used, wait for a response from the SDL Core before sending any more user interface RPCs.
```swift
let layout = SDLPredefinedLayout.GRAPHIC_WITH_TEXT() // Set template type here
let display = SDLSetDisplayLayout()
display.displayLayout = display.value
sdlManager?.sendRequest(display, withResponseHandler: { (request, response, error) in
    if response?.resultCode == SDLResult.SUCCESS() {
      // The template has been set successfully
    }
})
```
#### Available Templates
The following examples show how templates will appear on the generic head unit
##### 1. MEDIA - with and without progress bar
![MEDIA - with progress bar](assets/MediaWithProgressBar.png)

![MEDIA - without progress bar](assets/MediaWithoutProgressBar.png)
##### 2. NON-MEDIA - with and without soft buttons
![NON-MEDIA - with soft buttons](assets/NonMediaWithSoftButtons.png)

![NON-MEDIA - without soft buttons](assets/NonMediaWithoutSoftButtons.png)
##### 3. GRAPHIC_WITH_TEXT
![GRAPHIC_WITH_TEXT](assets/GraphicWithText.png)
##### 4. TEXT_WITH_GRAPHIC
![TEXT_WITH_GRAPHIC](assets/TextWithGraphic.png)
##### 5. TILES_ONLY
![TILES_ONLY](assets/TilesOnly.png)
##### 6. GRAPHIC_WITH_TILES
![GRAPHIC_WITH_TILES](assets/GraphicWithTiles.png)
##### 7. TILES_WITH_GRAPHIC
![TILES_WITH_GRAPHIC](assets/TilesWithGraphic.png)
##### 8. GRAPHIC_WITH_TEXT_AND_SOFTBUTTONS
![GRAPHIC_WITH_TEXT_AND_SOFTBUTTONS](assets/GraphicWithTextAndSoftButtons.png)
##### 9. TEXT_AND_SOFTBUTTONS_WITH_GRAPHIC
![TEXT_AND_SOFTBUTTONS_WITH_GRAPHIC](assets/TextAndSoftButtonsWithGraphic.png)
##### 10. GRAPHIC_WITH_TEXTBUTTONS
![GRAPHIC_WITH_TEXTBUTTONS](assets/GraphicWithTextButtons.png)
##### 11. DOUBLE_GRAPHIC_SOFTBUTTONS
![DOUBLE_GRAPHIC_SOFTBUTTONS](assets/DoubleGraphicSoftButtons.png)
##### 12. TEXTBUTTONS_WITH_GRAPHIC
![TEXTBUTTONS_WITH_GRAPHIC](assets/TextButtonsWithGraphic.png)
##### 13. TEXTBUTTONS_ONLY
![TEXTBUTTONS_ONLY](assets/TextButtonsOnly.png)
##### 14. LARGE_GRAPHIC_WITH_SOFTBUTTONS
![LARGE_GRAPHIC_WITH_SOFTBUTTONS](assets/LargeGraphicWithSoftButtons.png)
##### 15. LARGE_GRAPHIC_ONLY
![LARGE_GRAPHIC_ONLY](assets/LargeGraphicOnly.png)
