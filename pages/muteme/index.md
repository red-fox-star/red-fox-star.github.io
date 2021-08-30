---
layout: home
title: MuteMe USB-HID Protocol
---

The [MuteMe](https://muteme.com/) is a nice little usb-button device. It combines a capacitive sensor on top and an RGB LED. It enumerates as a USB-HID. I suspect it's running an ESP of some sort internally.

As of 2021-08-31 no github account is public and no source code is published. The MuteMe-Client application is a React javascript application packaged as a cross platform Electron App. The webpack-compressed source code is included in the MacOS application by navigating to `/Applications/MuteMe-Client.app/Contents/Resources/app/dist/index.js`

## Resources

[HIDPyToy](https://github.com/todbot/hidpytoy) - a gui HID device browser.
[HIDAPI](https://github.com/signal11/hidapi) - a cross platform low level HID API wrapper.
[Node-HID](https://github.com/node-hid/node-hid) - Nodejs bindings for HIDAPI. Also provides a CLI utility `hid-showdevices` if installed globally.

## MuteMe DIY Library

Though the source code isn't published in a readable form, it is distributed very readable albeit lightly compressed form with the MacOS binary. The file at `/Applications/MuteMe-Client.app/Contents/Resources/app/dist/index.js` formats well enough with any relatively modern javascript formatter and contains a dozen lines of code (amidst 2300!) which form the device driver. There is also a lot of mumbojumbo about interacting with various web-confrence-call software stuffs, which I don't have any interest in.

[Here is a source file](https://github.com/red-fox-star/muteme-diy/blob/master/src/mute_me_button.js) which provides the basic functionality for finding and interacting with the USB-HID MuteMe. Hopefully they'll let the code live out here so people can hack on the device!

This is not a robust library! It contains none of these safeguards:

- Preventing you from setting color or mode bits which don't make sense and reboot the device.
- Reasonably timing out when connecting to the device fails.
- Preventing you from setting the color or mode too fast, too early, or to late.

## Code

The [demo code](https://github.com/red-fox-star/muteme-diy/blob/master/demo.js) is rather straightforward. In this configuration it will cycle colors when tapped.

{% highlight javascript linenos %}
let { MuteMeButton } = require("./src/mute_me_button.js")

let muteme = new MuteMeButton(MuteMeButton.vendorId, MuteMeButton.productId)
muteme.connect()

let colors = MuteMeButton.colors
let modes = MuteMeButton.modes
let color = 0
let mode = 0

muteme.onTouchStop(function() {
  colorChange()
  // modeChange()
})

function colorChange(){
  muteme.setColor(colors[color])
  if (++ color >= colors.length) color = 0
}

function modeChange(){
  muteme.setMode(modes[mode])
  if (++ mode >= modes.length) mode = 0
}
{% endhighlight %}
