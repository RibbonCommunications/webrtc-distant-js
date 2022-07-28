## JavaScript Packages

### @distant/distant

This is the main distant library that a user will interact with. It allows you to create a Distant Controller that you can connect to a Distant Remote Agent via a distant channel and start creating and controlling Distant Sessions.

### @distant/distant-controller

 This is the library that creates a session and has functionality to send and receive messages via a supplied channel.

### @distant/distant-remote

This is the library used within a session to manipulate certain aspects of the session such as it's position on screen. It is also used to allow the remote application running in the Distant Session to communicate back and forth with the Distant Controller.

### @distant/distant-web

This is a simple implementation of a Distant Remote Agent that works in the browser or in Electron. It can manage sessions either through the [window.open](https://developer.mozilla.org/en-US/docs/Web/API/Window/open) API or through Electron's [BrowserWindow](https://electronjs.org/docs/api/browser-window). It can be very useful as a tool for testing Distant applications in a more developer-friendly environment.

### @distant/vchannel

This a low level javascript library that can be used to connect to a Citrix Receiver driver such as `distant-vdi`. `vchannel` by itself can only serve as a communication channel and doesn't have any of the business logic of distant. It is a thin wrapper on top of the Citrix WinFrame SDK.

### @distant/pull-vchannel

This is a library used to convert a `vchannel` instance into a pull-stream in order to be able to stream data back and forth from the channel.

### @distant/distant-codecs

This library contains some of the codecs used in the Distant communication stack.

### @distant/distant-protocol

This package is where the distant protocol buffer messages are defined. It is imported by other packages in order to have the protobuf message definitions required for encoding/decoding.

### @distant/tracker

This package provides tracker creator functions for DOM elements and Electron windows. Tracked elements and Electron windows can report bounds and visibility updates.  Monitoring elements and windows is essential in a VDI environment that require a thin client's media elements to be overlayed over controller application and to follow its position and size.

## Native Packages

### distant-vdi

This is a VDI Distant Remote Agent implemented as a Citrix Receiver driver. This runs in-process with your Citrix Receiver and allows distant connections to manage Distant Session directly in the Citrix Receiver window.
