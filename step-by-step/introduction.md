# About

In the next few sections we will walk through the steps necessary to build a successful Distant application. We will try to adhere to certain principles that will allow this application to be easily maintained and extended in the future.

## Assumptions

1. We're developing a desktop application using Electron.
1. The application will use WebRTC.JS SDK to enable WebRTC calls.
1. The application can run in both VDI and non-VDI mode.
1. This will require some advanced knowledge of JavaScript but only limited knowledge of VDI.
1. We will be using modern language features (ES2018+) and modern methodologies and tools.

## About Electron process model, and our toolkit.

 As you may know, Electron has a multi process architecture, where there is a main process and possibly multiple rendering processes (one per window). Our Distant JS libraries will be used inside the main Electron process, where the WebRTC.js SDK will be part of the renderer process. To simplify the example here we will consider that both the WebRTC.js SDK and the Distant libraries are executed together.

Let's get [started small](./starting-small.md).
