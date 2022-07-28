# Connections

Now that we have our application and our remote application, it's time to make them communicate so they can cooperate.

## Messages

The messages used by the 2 parts of the application are application defined, meaning it's up to the user of Distant to define the format of the message.

We suggest using a standard encoding such as JSON and a standard structure for your messages such as [Flux Standard Actions](https://github.com/redux-utilities/flux-standard-action)

Then we can start defining messages that our remote application will respond to and perform operation.

Let's define a simple message to move the window, it will look like this:

```javascript
export const MOVE_WINDOW = '@@APPLICATION/MOVE_WINDOW'

// Creates a move window message
export function createMoveWindowMessage(bounds)
  return {
    type: MOVE_WINDOW,
    payload: {
      bounds
    }
  }
}

```

Then it's simple to create a shared a library that can be used both in the application and the remote application in order to create these messages and parse them.

## Sending Messages

Let's send a message from the Distant Controller to a previously created session.

```javascript {highlight:[1, 3, 9, 11]}
import { createDistantJSONCodec } from '@distant/distant-codecs'

const jsonCodec = createDistantJSONCodec()

function sendMessage(distantSession, message) {
  // At this point, message is a simple JavaScript object.

  // Let's encode it to send over the channel.
  const encoded = jsonCodec.encode(message)

  distantSession.sendMessage(encoded)
}
```

In order to send a message to the remote side, it must be encoded into a UInt8Array (an array of bytes). This is due to the nature of the possible channels that Distant operates on, such as Citrix Virtual Channels.

The `@distant/distant-codecs` package has some tools that can help with this. Namely an encoder that generates a JSON payload and then encodes it into an array of bytes.

Effectively, the `jsonCodec.encode` line is equivalent to the following encode function:

```javascript
function encode(message) {
  const encoder = new TextEncoder()
  return encoder.encode(JSON.stringify(message))
}
```

> Future: In the future we expect to make this easier by allowing simple JavaScript objects to be passed directly by default.

## Receiving from the remote

From the remote application, receiving the message is quite simple.

```javascript
import * as messageTypes from './applicationDefinedMessageTypes'
import remoteController from '@distant/distant-remote'
import { createDistantJSONCodec } from '@distant/distant-codecs'

const jsonCodec = createDistantJSONCodec()

remoteContoller.on('message', encoded => {
  // Message received from the controller.

  // First let's decode it.
  const message = jsonCodec.decode(encoded)

  // Then let's handle it.
  switch (message.type) {
    case messageTypes.MOVE_WINDOW:
      const { bounds } = message.payload
      remoteController.moveWindow(bounds)
      break
    default:
      console.log(`Unknown message type received "${message.type}"`)
      break
  }
})
```

There we go, we received the message, decoded it, and then performed an operation.

But what if the remote wants to communicate back to the application it's new state?

## Sending a response

The remote application can simply use the exact same mechanism to send a message to the controller side of the application.

Let's revise our last example to see how the remote application could do this.

```javascript {highlight: [2,'8-11','25-27']}
import * as messageTypes from './applicationDefinedMessageTypes'
import * as messages from './applicationDefinedMessageCreators'
import remoteController from '@distant/distant-remote'
import { createDistantJSONCodec } from '@distant/distant-codecs'

const jsonCodec = createDistantJSONCodec()

function sendMessage(controller, message) {
  const encoded = jsonCodec.encode(message)
  controller.sendMessage(encoded)
}

remoteContoller.on('message', encoded => {
  // Message received from the controller.

  // First let's decode it.
  const message = jsonCodec.decode(encoded)

  // Then let's handle it.
  switch (message.type) {
    case messageTypes.MOVE_WINDOW:
      const { bounds } = message.payload
      remoteController.moveWindow(bounds)

      // Now let's report our new state
      const message = messages.createWindowBoundsStateMessage(remoteController.getWindowBounds)
      sendMessage(remoteController, message)

      break
    default:
      console.log(`Unknown message type received "${message.type}"`)
      break
  }
})
```

Notice the new lines we added? They are highlighted in the diagram. Sending a message from the controller or the session is handled very much in the same way.

We omit some implementation details of `messages.createWindowBoundsStateMessage` but you can imagine that it looks very much like the `createMoveWindowMessage` we saw a few sections above.

Now, we're almost there. Let's complete the picture.

## Receiving messages in the controller application

In the controller application to receive a message from the session is quite simple and looks very similar to the remote side receiving a message.

```javascript
import { createDistantJSONCodec } from '@distant/distant-codecs'
import * as messageTypes from './applicationDefinedMessageTypes'

const jsonCodec = createDistantJSONCodec()

function registerMessageHandler(distantSession) {
  distantSession.on('message', encoded => {
    // Message received from the session.

    // First let's decode it.
    const message = jsonCodec.decode(encoded)

    // Then handle it:
    switch (message.type) {
      case messageTypes.WINDOW_BOUNDS_STATE_CHANGED:
        doSomethingWithNewState(message.payload)
        break
      default:
        break
    }
  })
}
```

There you have it. The 2 sides of the application are communicating to each other.
