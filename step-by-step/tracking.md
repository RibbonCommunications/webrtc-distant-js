# Tracking

Having successfully created a remote session in VDI mode in a Citrix environment, you may now want to render media inside your remote application, and you'll want any visible media elements to appear as though they are part of your application (remembering that media (i.e. video) is actually rendered on the client, and not within your app or in the VDA).
To accomplish this, we'll want to monitor the size, location and visibility of your application window and its video elements and report any changes to the remote application so that it can resize, move, and show, hide or clip accordingly.

## tracker

The `@rbbn/distant-tracker` package provides 3 tracking creator functions: `createElementTracker`, `createWindowTracker` and `createScreenTracker`. Element trackers and window trackers enable us to report bounds and visibility for Electron windows and for DOM elements. A screen tracker can be used to normalize window bounds. This is needed because the remote and controller operating systems may use a different point as the origin for screen coordinates and also may be using different DPI scaling.

```javascript {highlight: [7, 8]}
import { createElementTracker, createWindowTracker, createScreenTracker } from '@rbbn/distant-tracker'

const { remote } = require('electron')
const window = remote.getCurrentWindow()
const element = document.getElementById('target-element')

const elementTracker = createElementTracker(element, 1000)
const windowTracker = createWindowTracker(window.id, 1000)
const screenTracker = await createScreenTracker()
```

As shown above, createElementTracker and createWindowTracker accept 2 arguments:

- a target
- and an optional throttle wait time

createScreenTracker does not require any arguments.

All functions return an object with `on` and `off` methods to subscribe or unsubscribe from event types with a handler. The object returned by `createElementTracker` and `createWindowTracker` contains `getBounds` and `getVisibility` methods to get the state of the tracked object. The object returned by `createScreenTracker`  contains a function that performs window bounds normalization that will be discussed later.

```javascript
function createElementTracker(element, throttle) => ({
  on: function (eventType, handler) { ... },
  off: function (eventType, handler) { ... },
  getBounds: function () { ... },
  getVisibility: function () { ... }
})

function createWindowTracker(window, throttle) => ({
  on: function (eventType, handler) { ... },
  off: function (eventType, handler) { ... },
  getBounds: function () { ... },
  getVisibility: function () { ... }
})

function createScreenTracker() => ({
  on: function (eventType, handler) { ... },
  off: function (eventType, handler) { ... },
  normalizeBounds: function (bounds) { ... }
})
```

There are 2 event types used by both window tracker and element tracker objects:

- `boundsUpdate` which emits an object: `{ x, y, width, height }`
- `visibilityUpdate` which emits an object: `{ visibility }` (visibility is a boolean)

A screen tracker uses a different event type:

- `displaysUpdated` which is emitted when the screen arrangement changes (i.e. new monitors are added or removed, Citrix moves into or out of fullscreen), or scaling or resolution change wihin the VDA

## Normalizing Window Bounds with Screen Tracker
The screen tracker's `normalizeBounds` function normalizes window bounds between different operating systems, display arrangements and scaling values within the VDA. For example, Windows sets the bounds origin as the top left of the primary monitor within the VDA, while eLux sets the origin as the top left point of its aggregate screen space. As well, when Windows 10 is used as the VDA, each monitor may be assigned a different scaling value (100%, 125%, 150%, 175%) within the VDA, which the eLux client has no knowledge of. The screen tracker normalizes bounds to account for these differences and returns adjusted bounds to your application, which your app should then forward to the remote application.

## Tracking in the Controller Application

We will assume that the application has a simple layout, in which the video elements for a call are rendered when the application first starts up.

To monitor the position of the video elements, we will need to:

- Create messages for bounds and visibility updates
- Create trackers for Electron window and elements that send updates for:
  - bounds
  - visibility
- Create screen tracker for normalizing window bounds
- Track the main window and the video elements

### Messages for Bounds and Visibility

Previously, we created a message for moving the window. In our message library (`./applicationDefinedMessageCreators`), we will add messages for:

- window visibility update
- element bounds update
- element visibility update

```javascript {highlight:['2-4', 6, 9, '13-18', '21-29', '31-38']}
export const MOVE_WINDOW = '@@APPLICATION/MOVE_WINDOW'
export const SHOW_HIDE_WINDOW = '@@APPLICATION/SHOW_HIDE_WINDOW'
export const MOVE_ELEMENT = '@@ELEMENT/MOVE_WINDOW'
export const SHOW_HIDE_ELEMENT = '@@ELEMENT/SHOW_HIDE_WINDOW'
// Creates a move window message
export function createMoveWindowMessage(bounds)
  return {
    type: MOVE_WINDOW,
    payload: { bounds }
  }
}
// Creates a show/hide window message
export function createVisibilityWindowMessage{visibility) {
  return {
    type: SHOW_HIDE_WINDOW,
    payload: { visibility }
  }
}

// Creates a move element message
export function createMoveElementMessage(id, bounds)
  return {
    type: MOVE_ELEMENT,
    payload: {
      id, bounds
    }
  }
}
// Creates a show/hide element message
export function createVisibilityElementMessage{id, visibility) {
  return {
    type: SHOW_HIDE_ELEMENT,
    payload: {
      id, visibility
    }
  }
}
```

### Tracking Functions

In the controller application, let's create functions in a separate file to track bounds and visibility updates for a window and for elements in a Windows enviroment.

```javascript
// ./utils/trackers.js
import { createElementTracker, createWindowTracker, createScreenTracker } from '@rbbn/distant-tracker'

export async function startTrackingWindow(window, session, sendMessage, throttle) {
  const windowTracker = createWindowTracker(window.id, throttle)
  const screenTracker = await createScreenTracker()

  // This function returns the given bounds object normalized to 1 DPI and positioned relative to the top left point of the aggregate screen space
  const normalizeWindowBounds = bounds => screenTracker.normalizeBounds(bounds)

  windowTracker.on('boundsUpdate', ({ bounds }) => {
    const message = messages.createMoveWindowMessage(normalizeWindowBounds(bounds))
    sendMessage(session, message)
  })

  screenTracker.on('displaysUpdated', () => {
    const message = messages.createMoveWindowMessage(normalizeWindowBounds(windowTracker.getBounds()))
    sendMessage(session, message)
  })

  windowTracker.on('visibilityUpdate', ({ visibility }) => {
    const message = messages.createVisibilityWindowMessage(visibility)
    sendMessage(session, message)
  })
}

export function startTrackingElement(element, session, sendMessage, throttle) {
  const elementTracker = createElementTracker(element, throttle)

  elementTracker.on('boundsUpdate', ({ bounds }) => {
    const message = messages.createMoveElementMessage(element.id, bounds)
    sendMessage(session, message)
  })

  elementTracker.on('visibilityUpdate', ({ visibility }) => {
    const message = messages.createVisibilityElementMessage(element.id, visibility)
    sendMessage(session, message)
  })
}
```

### Tracking Window and Elements

Now that we have functions to create our trackers, we will track the main window and video elements when starting a call. We'll assume that the video elements for calls are rendered when the application first starts.

```javascript {highlight:[3, '6-16']}
function startCall(number) {
  WebRTC.call.make(number)
  startTracking()
}

function startTracking() {
  const { remote } = require('electron')
  const window = remote.getCurrentWindow()
  const localVideo = document.getElementById('local-video')
  const remoteVideo = document.getElementById('remote-video')
  const throttle = 100

  startTrackingWindow(window, session, sendMessage, throttle)
  startTrackingElement(localVideo, session, sendMessage, throttle)
  startTrackingElement(remoteVideo, session, sendMessage, throttle)
}
```

## Updating the Remote Application

Now that the controller application sends bounds and visibility updates, the remote application must react to them. We'll build on what we did previously in [Sending a response](http://localhost:3000/#/step-by-step/connections#sending-a-response).

First, we'll handle the new visibility updates from the window.

```javascript {highlight: ['24-28']}
import * as messageTypes from './applicationDefinedMessageTypes'
import * as messages from './applicationDefinedMessageCreators'
import remoteController from '@rbbn/distant-remote'
import { createDistantJSONCodec } from '@rbbn/distant-codecs'

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
    case messageTypes.SHOW_HIDE_WINDOW:
      const { visibility } = message.payload
      remoteController.setWindowVisible(visibility)
      const message = messages.createWindowBoundsStateMessage(visibility)
      sendMessage(remoteController, message)
    default:
      console.log(`Unknown message type received "${message.type}"`)
      break
  }
})
```

Now, we'll add cases for our element updates for bounds and visibility with handler functions that we will define in the next step.

```javascript {highlight: ['29-35']}
import * as messageTypes from './applicationDefinedMessageTypes'
import * as messages from './applicationDefinedMessageCreators'
import remoteController from '@rbbn/distant-remote'
import { createDistantJSONCodec } from '@rbbn/distant-codecs'

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
    case messageTypes.SHOW_HIDE_WINDOW:
      const { visibility } = message.payload
      remoteController.setWindowVisible(visibility)
      const message = messages.createWindowVisibilityStateMessage(visibility)
      sendMessage(remoteController, message)
    case messageTypes.MOVE_ELEMENT:
      const { id, bounds } = message.payload
      moveElement(id, bounds)
      break
    case messageTypes.SHOW_HIDE_ELEMENT:
      const { id, visibility } = message.payload
      showHideElement(id, visibility)
    default:
      console.log(`Unknown message type received "${message.type}"`)
      break
  }
})
```

Finally, we'll tell the remote application what to do with the bounds and visibility updates for elements.

```javascript
const trackedElements = {}

function moveElement (id, bounds) {
  if (!trackedElements[id]) {
    trackedElements[id] = document.getElementById(id)
  }

  const element = trackedElements[id]
  const { x, y, width, height } = bounds
  element.style.left = `${x}px`
  element.style.top = `${x}px`
  element.style.width = `${width}px`
  element.style.height = `${height}px`

  const message = messages.createElementBoundsStateMessage(id, bounds)
  sendMessage(remoteController, message)
}

function showHideElement (id, visibility) {
  if (!trackedElements[id]) {
    trackedElements[id] = document.getElementById(id)
  }

  const element = trackedElements[id]

  if (visibility) {
    element.classList.remove('hidden')
  } else {
    element.classList.add('hidden')
  }

  const message = messages.createElementVisibilityStateMessage(id, bounds)
  sendMessage(remoteController, message)
```

And that's it! Your main application now informs your remote application of bounds and visibility updates for your electron window and your call's video elements, and your remote application reacts to those updates.
