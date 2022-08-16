## JavaScript Packages

### [@distant/distant](./../packages/distant-readme.md)

This is the main Distant library that users interact with. It allows you to create a Distant Controller that you can connect to a Distant Remote Agent via a Distant channel and start creating and controlling Distant Sessions.

### @distant/distant-controller

 This is the library that creates a session and has functionality to send and receive messages via a supplied channel.

### [@distant/distant-remote](../packages/distant-remote-readme.md)

This is the library used within a session to manipulate certain aspects of the session such as its position on screen. It is also used to allow the remote application running in the Distant Session to communicate back and forth with the Distant Controller.

### [@distant/distant-web](../packages/distant-web-readme.md)

This is a simple implementation of a Distant Remote Agent that works in the browser or in Electron. It can manage sessions either through the [window.open](https://developer.mozilla.org/en-US/docs/Web/API/Window/open) API or through Electron's [BrowserWindow](https://electronjs.org/docs/api/browser-window). It can be very useful as a tool for testing Distant applications in a more developer-friendly environment.

### [@distant/vchannel](../packages/distant-vchannel-readme.md)

This a low level JavaScript library that can be used to connect to a Citrix Receiver driver such as `distant-vdi`. `vchannel` is a thin wrapper on top of the Citrix WinFrame SDK that can only serve as a communication channel and doesn't have any of the business logic of Distant.

### [@distant/pull-vchannel](../packages/pull-vchannel-readme.md)

This is a library used to convert a `vchannel` instance into a pull-stream in order to be able to stream data back and forth from the channel.

### [@distant/distant-codecs](../packages/distant-codecs-readme.md)

This library contains some of the codecs used in the Distant communication stack.

### [@distant/distant-protocol](../packages/distant-protocol-readme.md)

This package is where the Distant protocol buffer messages are defined. It is imported by other packages in order to have the protobuf message definitions required for encoding/decoding.

### [@distant/tracker](../packages/distant-tracker-readme.md)
#### [Change Log](../packages/distant-tracker-changelog.md)
This package provides Distant Tracker creator functions for DOM elements and Electron windows. Tracked elements and Electron windows can report bounds and visibility updates. Monitoring elements and windows is essential in a VDI environment that require a thin client's media elements to be overlayed over a controller application and to follow its position and size.

### [@distant/kandy-hid](https://github.com/Kandy-IO/kandy-hid-sdk)
#### [Change Log](https://github.com/Kandy-IO/kandy-hid-sdk/blob/master/CHANGELOG.md)
This package enables application developers to handle HID device call operations.