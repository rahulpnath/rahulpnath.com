---
title: "How To Take iOS App Store Screenshots Using Google Chrome For Cordova Applications"
description: Using Chrome Browser to take screenshots in all resolutions as required by the App Stores.
date: 2019-05-01
cover: /images/ios_screenshots_devices.jpg
---

When submitting an app to the iOS App Store, it now mandates to upload screenshots for iPhone XS Max or the 12.9-inch iPad Pro. I had just moved to a new client and was to push an update for their existing mobile application. I had none of the iOS app development related applications setup and had to get this done as soon as possible. The mobile app is the website packaged as a [Cordova application](https://cordova.apache.org/).

In this post, we will see how we can use Google Chrome Browser to take screenshots in the different resolution that the App Store requires for app submission. 

### Simulate Mobile Devices in Chrome

Use the [Chrome DevTools to simulate mobile devices](https://developers.google.com/web/tools/chrome-devtools/device-mode/) and load the website in mobile view. In this mode, the devices drop-down list shows all the predefined list of mobile device lists. You can switch between them to see how the site will render on various devices. 

![](/images/ios_screenshots_chrome_devices.jpg)

Only a subset of available devices is shown in the drop-down. All available devices are listed on selecting the Edit button. Only the devices that are ticked in the list shown are visible in the drop-down. One can edit this to add/remove devices from the drop-down list.

![](/images/ios_screenshots_chrome_all_devices.jpg)

### Add Custom Device

The iPhone XS Max is a relatively new device and the device setting is not yet available in the predefined list of devices. This could very well be available at the time of reading; however, there can be another device that you are looking for that that is not in the list. In this case, you can add the device to the list using the 'Add custom device' button in the Edit screen that lists all the devices (shown above).

iPhone XS Max has a screen size of [1242px x 2688px](https://developer.apple.com/design/human-interface-guidelines/ios/icons-and-images/launch-screen#static-launch-screen-images). Using the actual pixels might render the page size large for your laptop/monitor. You can reduce the size by a factor, Device Pixel Ratio (DPR), and enter that along with the device details. In the example below I have used a DPR of 3 which makes the width the height smaller - 414px x 896px (*1242/3 = 414px; 2688/3 = 896px*)

![](/images/ios_screenshots_iphone_xs_max_chrome.jpg)

### Capture Screenshot in Native Resolution

To upload the screenshots to the App Store you need them to be in the native resolution, as you would have taken a screenshot in the actual devices. Since the page rendered on the Mobile Device layout is of a different size you cannot simply take a screen capture of the rendered page, since that will be in a different resolution. To capture a screenshot in the native resolution there are two options

From the options menu drop down (that can be accessed from the three vertical dots button) as shown below you can use the 'Capture Screenshot' menu item.

![](/images/ios_screenshots_options_chrome.jpg)

Screenshot option is also available in the [Command Menu](https://developers.google.com/web/tools/chrome-devtools/ui#command-menu), that is accessible via using the menu option in the DevTools or Control+Shift+P [keyboard shortcut](https://developers.google.com/web/tools/chrome-devtools/shortcuts). The list of available commands can be filtered and choose the 'Capture Screenshot' command to take a screenshot in native resolution.

![](/images/ios_screenshots_chrome.jpg)

The generated screenshots using any of the above methods will be in the actual device resolution, in this case, 1242px x 2688px. The screenshots can be uploaded as is to the App Store and submitted for review. 

Hope this helps you to generate screenshots for your mobile applications built using Cordova.