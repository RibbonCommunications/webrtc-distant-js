# Distant

The Distant library is used to control a remote browser-type application via an arbitrary channel.

## Features

- Create remote sessions
- Monitor remote sessions
- Send and receive custom user defined messages from the remote session.

# Installation

Install using npm:

```
npm install @rbbn/distant
```

or yarn:

```
yarn add @rbbn/distant
```

# Basic Usage

To get started, you need to create a controller that is connected to a remote agent.

```javascript
import { createController } from '@rbbn/distant'
import { createConnection } from '@rbbn/distant-web'

const distantWebConnection = createConnection()

const controller = createController(distantWebConnection)

// ... Use the controller
```

Note that the connection and the controller are maintained separately, it is up to the application to monitor the connection and re-establish the connection if it goes down or fails.

Once we have created the controller we can start by creating a session.

```javascript
async function startSession(sessionId) {
  try {
    const session = await controller.createSession({
      id: sessionId, // Session id must be an integer
      url: 'https://remoteapp.url',
      timeout: 2000
    })

    // Session is created.
    return session
  } catch (err) {
    console.error(err)
  }
}
```

With the session we can then start sending and receiving messages. First though we need to register some event listeners in order to get notified of the status of the session as well as messages.

```javascript
function registerSession(session) {
  session.on('changed', event => {
    // Handle session changed events
  })

  session.on('closed', event => {
    // Handle session closed events
  })

  session.on('message', event => {
    // Handle session message events
  })
}
```

Now to send a message to the session we simply call `session.sendMessage`.

```javascript
function sendMessage(session, message) {
  // In order to send a message we must first encode into a byte array.
  const encoder = new TextEncoder()
  const encodedMessage = encoder.encode(JSON.stringify(message))

  session.sendMessage(encodedMessage)
}
```

> Note: Encoding and decoding is application specific. Distant only deals with Uint8Array (Byte arrays) for messages. Here we use JSON messages encoded with the TextEncoder for example.

Receiving messages works much the same way.

```javascript
session.on('message', payload => {
  const decoder = new TextDecoder()
  const message = JSON.parse(decoder.decode(payload))

  // Application specific message handling goes here.
})
```
