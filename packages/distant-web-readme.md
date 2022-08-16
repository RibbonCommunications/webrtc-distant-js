## Distant Web

A library for testing your application without the need for a Citrix channel by
providing APIs compatible with Distant and Distant VDI and providing an
implementation based on the browser native `window.open()` method.

## Usage

### JavaScript

Install using npm:

```
npm install @distant/distant-web
```

or yarn:

```
yarn add @distant/distant-web
```


Here's some example code from an application. Distant Web is effectively a
drop-in replacement for Pull VChannel, as you can see below the only difference
between this and using a Citrix channel is that we are passing a distant-web
connection rather than a pull-vchannel connection to distant's
`createController()`. This code does however need to be run from within a
browser due to the fact that Distant Web uses `window.open()`.

```javascript
import { createController } from '@distant/distant'

import { createConnection } from '@distant/distant-web'

const textEncoder = new TextEncoder()

const distantWebConnection = createConnection()

// here we pass a distant-web connection rather one from pull-vchannel
const distantController = createController(distantWebConnection)

// creating a session will now open a window with window.open()
const session = await distantController.createSession({
  id: 1,
  url: "http://example.com/remote.html"
})

// we use the session as we would normally
session.sendMessage(textEncoder.encode('hello'))
```


On the remote app side of this example we don't need to do anything
differently. The remote app will be opened in a new browser window and
distant-web will provided all that is needed for distant-remote to work.

```javascript
import distantRemote from '@distant/distant-remote'

const textDecoder = new TextDecoder()

distantRemote.on('message', (message) => {
  if (textDecoder.decode(message) === 'hello') {
    distantRemote.setWindowVisible(true)
  }
})
```

### Platform support

Support for different platforms is handled by passing in a `windowFactory` to
Distant Web's `createConnection()` function.  Distant Web provides
window factories for both browsers and Electron. If no `windowFactory` is
specified the browser windowFactory is used.

```javascript
import {
  createConnection,
  createWebWindow,
  createElectronWindow
} from '@distant/distant-web'

// if we're running in a browser (this is the default if none specified)
const distantWebConnection = createConnection({
  windowFactory: createWebWindow
})

// if we're running in the Electron main process
const distantWebConnection = createConnection({
  windowFactory: createElectronWindow
})
```

## Known Issues

### Web Window Factory

  - When trying to resize a window to very small dimensions the window will
    fall back to using larger dimensions. The minimum size of a window is
    usually between 100x100 and 280x225 depending on the platform

  - When using Electron or Chrome on Windows 10 the window's title bar will
    still be visible when it should be hidden

  - When using Chrome on Mac the window will be small when it should be hidden.
    The window might change size while it should still be hidden.
