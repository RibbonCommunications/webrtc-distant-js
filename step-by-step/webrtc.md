## Using the Kandy.js WebRTC SDK

The Kandy.js SDK supports a proxy mode in which the media engine of the SDK can be separated from the signaling engine. This is a great fit for Distant applications running in VDI where we want the media to run on the user device, but the signaling to take place on the host device.

This "proxy mode" requires a special setup for the SDK which is not normally needed for a regular WebRTC scenario (with local media & signaling).

These steps need to happen on both the controller application and the remote application:
1. creating an "SDK Channel" and
1. preparing the SDK.

## Creating the SDK Channel

As seen in the previous section, after the controller application and the remote application have used Distant to connect to each other, they will have a session that allows them to send and receive messages with each other.

```javascript {highlight: [5, 9]}
function sendMessage(distantSession, message) {
  const encoded = jsonCodec.encode(message)

  // Send a message to the session.
  distantSession.sendMessage(encoded)
}

// Receive a message from the sesion.
distantSession.on('message', encoded => { ... })
```

Both applications need to make use of this session to create a "SDK channel" within it for their SDK to use. This will allow the two SDKs to communicate with each other by piggy-backing on the same channel that the applications use to communicate with each other. Both applications follow the same procedure for setting up the channel.

The SDKs expect to be provided a channel with a simple interface:

``` javascript
// The SDK Channel.
const sdkChannel = {
  // The function that the SDKs will use internally
  //    to send messages to the remote SDK.
  // This is defined by the application.
  send: function (message) { ... },
  // The function the application will call when it
  //    receives a message from the session meant for the SDKs.
  // This is set internally by the SDKs.
  receive: undefined
}
```

#### Sending SDK Messages

The sdkChannel's `send` function is defined by the application. It receives one parameter, the SDK-generated message meant for the remote SDK, which the application needs to send to the session.

``` javascript {highlight: ['5-15']}
// The SDK Channel.
const sdkChannel = {
  // The function that the SDKs will use internally
  //    to send messages to the remote SDK.
  send: function (message) {
    // Wrap the SDK message in a format the remote
    //    application will understand.
    const sdkMessage = {
      type: SDK_MESSAGE,
      payload: message
    }

    const encoded = jsonCodec.encode(sdkMessage)
    // Send a message to the session.
    distantSession.sendMessage(encoded)
  },
  receive: undefined
}
```
This will allow the Kandy SDK to send messages that the remote application will receive.

It is up to the application to determine how it wants to define an "SDK message" to be sent. This will also define how the remote application will need to interpret an SDK message.

#### Receiving SDK Messages

Each application should already be setup to receive messages from their session.

``` javascript
distantSession.on('message', encoded => {
  // Message received from the controller.

  const message = jsonCodec.decode(encoded)

  // Then let's handle it.
  switch (message.type) {
    case messageTypes.MOVE_WINDOW:
      // Handle the message ...
      break
    default:
      console.log(`Unknown message type received "${message.type}"`)
      break
  }
})
```

The application will then receive the SDK messages from the remote SDK in the same way. It should handle the SDK messages by forwarding them to the SDK through the channel.

``` javascript {highlight: ['11-14']}
distantSession.on('message', encoded => {
  // Message received from the controller.

  const message = jsonCodec.decode(encoded)

  // Then let's handle it.
  switch (message.type) {
    case messageTypes.MOVE_WINDOW:
      // Handle the message ...
      break
    case messageTypes.SDK_MESSAGE:
      if (channel.receive) {
        channel.receive(message.payload)
      }
      break
    default:
      console.log(`Unknown message type received "${message.type}"`)
      break
  }
})
```
This completes the SDK Channel, allowing the two Kandy SDKs to communicate with each other. In this way, the application can optionally access the messages that the SDKs are sending, but does not need to know anything about the messages.

It is important to note that the `channel.receive` function will not be defined until the application provides the SDK Channel to the SDK. This will be explained in the next section.

## Preparing the SDK

For the Kandy SDK to know whether WebRTC operations will be performed locally or remotely, the application needs to prepare the SDK.

By default, the SDK performs operations locally. To perform them remotely, the application needs to:

1. Create the SDK with allowProxy option
1. Set the Channel
1. Enable Proxy Mode
1. Initialize the Remote SDK

### Create the SDK with allowProxy option
``` javascript
 const kandy = Kandy.create({
   common: {
     allowProxy: true
   }
 })
 ```
### Set the Channel

After the application has created the SDK channel as described above, it can provide that channel to the SDK by using the `proxy.setChannel` API.

``` javascript {highlight: [5]}
// The Kandy Channel.
const sdkChannel = { ... }

// Provide the channel to the SDK.
kandy.proxy.setChannel(sdkChannel)
```

This will trigger an event, so the application knows the channel has been set.

``` javascript
// Listen for changes to proxy state.
kandy.on('proxy:change', () => {
  const proxyInfo = kandy.proxy.getInfo()

  if (proxyInfo.hasChannel) {
    console.log('SDK has a valid channel for proxy mode.')
  }
})
```

Alternatively, a `proxy:error` event will be emitted if the operation failed.

### Enable Proxy Mode

Enabling proxy mode is done using the `kandy.proxy.setProxyMode` API. This will also trigger an event.

``` javascript {highlight: [2]}
// Enable proxy mode.
kandy.proxy.setProxyMode(true)

kandy.on('proxy:change', () => {
  const proxyInfo = kandy.proxy.getInfo()

  console.log(`SDK Proxy Mode set to: ${proxyInfo.proxyMode}.`)
})
```

### Remote Initialization

The last step is initializing the Remote SDK to ensure it is prepared for WebRTC operations. This will send a message across the SDK channel and wait for a response, verifying that the two SDKs can communicate with each other.

This is done with the `kandy.proxy.initializeRemote` API. This can only be done when the SDK has a channel and is in proxy mode.

``` javascript {highlight: [4]}
const proxyInfo = kandy.proxy.getInfo()
if (proxyInfo.hasChannel && proxyInfo.proxyMode) {
  // Attempt to initialize the remote SDK.
  kandy.proxy.initializeRemote()
} else {
  console.log('Cannot initialize remote SDK: Requirements not met.')
}

kandy.on('proxy:change', () => {
  const proxyInfo = kandy.proxy.getInfo()

  if (proxyInfo.remoteInitialized) {
    console.log('The SDK is ready for remote WebRTC operations.')
  }
})
```

## Preparing the Remote SDK

The Remote Kandy SDK also needs to be prepared before it can be used. It requires a channel in the same way that the Kandy SDK needs one, but is always in proxy mode.

### Create the SDK
 ``` javascript
 const client = Kandy.create()
 ```

### Set the Channel

The channel is set the same way in both Kandy SDKs:

``` javascript {highlight: [5]}
// The SDK Channel.
const sdkChannel = { ... }

// Provide the channel to the Remote SDK.
remoteKandy.proxy.setChannel(sdkChannel)
```

### Wait for Initialization

The Remote Kandy SDK will also emit an event when it has been initialized by the Kandy SDK. This lets the remote application know that it may begin to receive SDK messages.

``` javascript
remoteKandy.on('initialize', () => {
  console.log('Received initialization message from the Kandy SDK.')
})
```

## WebRTC Operations in Proxy Mode

After the Kandy SDK has been prepared for proxy mode, all WebRTC related operations will be sent over the channel instead of being performed locally. This will be done by the SDK automatically, without changes needing to be made to the application.

The only caveat is about rendering media. When in proxy mode, the SDK will know about the media being used by the Remote SDK. This can cause odd scenarios if the application tries to use the media locally, but it doesn't exist. It shouldn't create issues, but you may see error messages in the logs depending on how the application handles these scenarios.

Have the application use the Kandy.js SDK "render" APIs as you would normally use them, but use the html selector for an element that is located in the remote application.
