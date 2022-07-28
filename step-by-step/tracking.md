# Tracking

Having successfully created a remote session in VDI mode in a Citrix environment, you may want to render your media elements inside your remote application. But you'll want to give the impression that the video elements are part of your controller application. To do so, we will monitor the bounds and visibility of the controller application's window and its video elements using the tracker package, and report these updates to the remote application so that it can position the video elements correctly.

## tracker

The `@distant/tracker` package provides 3 tracking creator functions: `createElementTracker`, `createWindowTracker` and `createScreenTracker`. Element trackers and window trackers enable us to report bounds and visibility for Electron windows and for DOM elements. A screen tracker can be used to normalize window bounds. This is needed because the remote and controller operating systems may use a different point of origin for screen coordinates and may also use different DPI scaling.

```javascript {highlight: [7, 8]}
import { createElementTracker, createWindowTracker, createScreenTracker } from '@distant/tracker'

const { remote } = require('electron')
const window = remote.getCurrentWindow()
const element = document.getElementById('target-element')

const elementTracker = createElementTracker(element, 1000)
const windowTracker = createWindowTracker(window, 1000)
const screenTracker = await createScreenTracker()
```

As shown above, createElementTracker and createWindowTracker accept 2 arguments:

- a target
- and an optional throttle wait time

createScreenTracker does not need any arguments.

All functions return an object with the 4 methods, `on` and `off` to subscribe or unsubscribe from event types with a handler as well as `getBounds` and `getVisibility` to get the state of the tracked object. Window trackers and screen trackers have additional functions to aid with window bound normalization that will be discussed later.

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
  getTopLeftPoint: function () { ... },
  getDPIScale: function () { ... },
  on: function (eventType, handler) { ... },
  off: function (eventType, handler) { ... }
})
```

There are 2 event types used by both window tracker and element tracker objects:

- `boundsUpdate` which emits an object: `{ x, y, width, height }`
- `visibilityUpdate` which emits an object: `{ visibility }` (visibility is a boolean)

A screen tracker uses a different event type:

- `topLeftPointUpdate` which emits an object: `{ x, y }` (location of the top left of the screen space relative to origin)

## Normalizing Window Bounds with Screen Tracker
The screen tracker provides functions that can help create logic to normalize window bounds between different operating systems.

- getTopLeftPoint() which returns the coordinates of the top-leftmost point of the aggregate screen space. This can be used as an offset for when the screen origin differs between the two sides of a Distant session. For example, Windows sets the origin as the top left of the primary monitor, while eLux sets the origin as the top left of the screen space.
- getDPIScale() which returns the display's DPI scale. This can be used to transform a bounds object when the DPI scale differs between both sides of a Distant session.

When a window tracker emits a boundsUpdate an object containing the bounds is emitted. In the case that a screen tracker emits a topLeftPointUpdate you may not have the window bounds stored. For this reason a window tracker contains a function to retrive this information

- getBounds() which returns the current window bounds

## Tracking in the Controller Application

We will assume that the application has a simple layout, in which the video elements for a call are rendered when the application first starts up.

To monitor the position of the video elements, we will need to:

- Create messages for bounds and visibility updates
- Create trackers for Electron window and elements that send updates for:
  - bounds
  - visibility
- Create a screen tracker for normalizing window bounds
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

In the controller application, let's create functions in a separate file to track bounds and visibility updates for a window and for elements in a win32 enviroment.

```javascript
// ./utils/trackers.js
import { createElementTracker, createWindowTracker, createScreenTracker } from '@distant/tracker'

export async function startTrackingWindow(window, session, sendMessage, throttle) {
  const windowTracker = createWindowTracker(window, throttle)
  const screenTracker = await createScreenTracker()

  // This function returns the given bounds object normalized to 1 DPI and positioned relative to the top left point of the aggregate screen space.
  // note: you may need to use different logic to normalize window bounds depending on the operating systems used in your setup
  const normalizeWin32Bounds (bounds) => {
    const topLeftPoint = screenTracker.getTopLeftPoint()
    const DPIScale = screenTracker.getDPIScale()

    return {
      width: Math.round(bounds.width * DPIScale),
      height: Math.round(bounds.height * DPIScale),
      x: Math.round((bounds.x - topLeftPoint.x) * DPIScale),
      y: Math.round((bounds.y - topLeftPoint.y) * DPIScale)
    }
  }


  windowTracker.on('boundsUpdate', ({ bounds }) => {
    const message = messages.createMoveWindowMessage(normalizeWin32Bounds(bounds))
    sendMessage(session, message)
  })

  screenTracker.on('topLeftPointUpdate', ({ point }) => {
    const message = messages.createMoveWindowMessage(normalizeBounds(windowTracker.getBounds()))
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
  Kandy.call.make(number)
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

Last but not least, we'll tell the remote application what to do with the bounds and visibility updates for elements.

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
