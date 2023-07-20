# Distant Remote

Library for remote applications to interact with Distant

## Installation

Install using npm:

```
npm install @rbbn/distant-remote
```

or yarn:

```
yarn add @rbbn/distant-remote
```

## Usage
Use the package like so:

### Creating the remote instance
```javascript
import distantRemote from '@rbbn/distant-remote'

//// OR
// import { createRemoteController } from '@rbbn/distant-remote'
// const distantRemote = createRemoteController(window.distant, window.cefQuery)

const textEncoder = new TextEncoder()
const textDecoder = new TextDecoder()

distantRemote.on('message', (message) => console.log(textDecoder.decode(message)))
```

### Sending messages
```javascript
distantRemote.sendMessage(textEncoder.encode("hello"))
```

### Window controlling
distant-remote API includes methods for controlling the Distant Window.

`moveWindow()` takes a **bounds** object with four **number** parameters representing values in pixels:
- *x*, *y*, *width* and *height*.

`setWindowVisible()` takes a boolean value.

```javascript
distantRemote.moveWindow({ x: 44, y: 55, width: 666, height: 777 })

distantRemote.setWindowVisible(true)
```

### Device authorization
```javascript
distantRemote.on('deviceAuth', (auth) => {
  const { cameraAccess, microphoneAccess } = auth
})

distantRemote.requestDeviceAuth() // result will be emitted as `deviceAuth` event
```

### Opening web pages
distant-remote is capable of opening webpages directly on the system where it runs. Pages will be opened using the client's **default browser**. This can be achieved by doing the following:

```javascript
distantRemote.openBrowser('https://google.com')
```


### Closing the session
The session can be controlled from the remote instance.

```javascript
distantRemote.close() // Calling this method will close the Distant session
```
