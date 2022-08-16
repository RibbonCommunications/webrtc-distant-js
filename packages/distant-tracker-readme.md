# Distant Tracker

This package is intended to track and report the bounds and visibility of an Electron window as well as DOM elements. In a VDI environment this package is essential for the server to report the positioning, size and visibility of the window and elements that need to be painted.

The target environment for this library is Windows 7 or 10.

## Features
- Track an Electron window for bounds and visibility updates
- Track a DOM element for bounds and visibility updates
- Track updates to the user's monitor and display setup

### Electron Security
This SDK can be used in an Electron environment where security features such as contextIsolation, enableRemoteModule, nodeIntegration are enabled or disabled.

### Supported Display Configurations
- Up to six monitors can be used on an eLux Thin Client.
- Up to four monitors can be used in Mac or Windows VDI.
- When Windows 10 is used as the server-side machine, each display can be assigned one of Windows' predefined scaling values: 100%, 125%, 150% or 175%.
  - Custom scaling values are not supported.
  - All displays must have the same resolution at the **client level**. See Limitations / Assumptions below.
- Monitors can be arranged horizontally (side by side), vertically (top / bottom) or in a square (2x2, 2x3). Other monitor arrangements are not supported.

## Installation
To install using npm:

```
npm install @distant/tracker
```

To install using yarn:

```
yarn add @distant/tracker
```
## Usage
This SDK communicates between Main and Renderer Electron processes. The installation package includes everything required for independent operation, however your app must import / require certain files.

In the Electron main process, the preload/main file must be `require`'d, even if the app does not use context isolation or a preload script.

```javascript
// Main process
require('@distant/tracker/preload/main);
```

In the Electron renderer process, the preload/renderer file must be required in your preload script if contextIsolation is being used. Otherwise this `require` is not necessary.

```javascript
// Renderer process, preload.js
require('@distant/tracker/preload/renderer);
```

## Window Tracker
A window tracker is created by calling `createWindowTracker()` with the ID of an Electron `BrowserWindow` and an optional throttle time in milliseconds.

The throttle is an option to control the frequency at which the window tracker emits bounds or visibility updates.

```javascript
import { createWindowTracker } from `@distant/tracker`;

const electronWindow = new BrowserWindow();

const windowTracker = createWindowTracker(electronWindow.id, 100);

const boundsHandler = event => {
  console.log(`The bounds for window with id ${event.target} have changed to`, event.bounds)
}

const visibilityHandler = event => {
  console.log(`The visibility for window with id ${event.target} has changed to`, event.visibility)
}

console.log('Initial bounds are', windowTracker.getBounds())
console.log('Initial visibility is', windowTracker.getVisibility())

windowTracker.on('boundsUpdate', boundsHandler)
windowTracker.on('visibilityUpdate', visibilityHandler)

...

windowTracker.off('boundsUpdate', boundsHandler)
windowTracker.off('visibilityUpdate', visibilityHandler)
```

### Events

#### boundsUpdate
Fired whenever the position or size of the window changes. 

The event object passed to the handler contains the following information:

```typescript
{
  // the window's current bounds
  bounds: {
    borders: {
      bottom: number,
      left: number,
      right: number,
      top: number
    },
    innerHeight: number,
    innerWidth: number,
    x: number,
    y: number,
    width: number,
    height: number
  },

  // id of the tracked window
  target: BrowserWindowID
}
```

#### visibilityUpdate
Fired whenever the window changes between being shown or hidden. See `getVisibility()` for more details.

The event object passed to the handler contains the following information:

```typescript
{
  // the window's current visibility
  visibility: boolean,

  // id of the the tracked window
  target: BrowserWindowID
}
```

### Methods

#### getBounds()
Returns the tracked window's current bounds.

```typescript
{
  x: number,
  y: number,
  width: number,
  height: number
}
```

#### getVisibility()
Returns a boolean indicating whether the tracked window is currently visible.

Note that this does not take into account other applications that may be on top and obstruct the view to the window. At this point visibility is essentially whether the window is minimized or not.

### Known Issues
- Does not take into account whether the window might be in a workspace or virtual desktop that is out of view.

## Element Tracker
A element tracker is created by calling `createElementTracker()` with a DOM element and an optional throttle time in milliseconds. Note that the throttle has a minimum value of 20 milliseconds.

```javascript
import { createElementTracker } from `@distant/tracker`;

const element = document.getElementById('tracked-element');

const elementTracker = createElementTracker(element, 100);

const boundsHandler = event => {
  console.log(`The bounds for element ${event.target} have changed to`, event.bounds)
}

const visibilityHandler = event => {
  console.log(`The visibility for element ${event.target} has changed to`, event.visibility)
}

console.log('Initial bounds are', elementTracker.getBounds())
console.log('Initial visibility is', elementTracker.getVisibility())

elementTracker.on('boundsUpdate', boundsHandler)
elementTracker.on('visibilityUpdate', visibilityHandler)

...

elementTracker.off('boundsUpdate', boundsHandler)
elementTracker.off('visibilityUpdate', visibilityHandler)
```

### Events

#### boundsUpdate

Fired whenever the position (relative to the body) or size of the element changes. 

The event object passed to the handler contains the following information:

```typescript
{

  // the window's current bounds
  bounds: {
    x: number,
    y: number,
    width: number,
    height: number
  },

  // the tracked element
  target: HTMLElement

}
```

#### visibilityUpdate

Fired whenever the window changes between being shown or hidden. See `getVisibility()` for more details.

The event object passed to the handler contains the following information:

```typescript
{

  // the element's current visibility
  visibility: boolean,

  // the tracked element
  target: HTMLElement

}
```

### Methods

#### getBounds()

Returns the tracked window's current bounds.

```typescript
{
  x: number,
  y: number,
  width: number,
  height: number
}
```

#### getVisibility()

Returns a boolean indicating whether the tracked element is currently visible.

Note that this does not take into account other elements that may be on top and obstruct the view neither properties like opacity or visibility. At this point visibility only takes into account whether the element is part of the DOM tree and is not `display: none`.

## Screen Tracker

A screen tracker is created by calling `createScreenTracker()` which returns a Promise that is resolved with the screen tracker object.

The `createScreenTracker` function takes an optional object with a single key `rendererLogs`, which can be set to `true` to enable screen tracker debug logs in the Renderer process. Default is `false`.

```
import { createScreenTracker } from `@distant/tracker`;

const screenTracker = await createScreenTracker();

// or, to enable debug logs in the renderer process
const screenTracker = await createScreenTracker({ rendererLogs: true });
```

### Methods

#### normalizeBounds(boundsObject)

Accepts a bounds object (see getBounds()).

Returns an object containing values of x, y, width and height normalized to account for differences in display arrangement and scaling.

```typescript
{
  x: number,
  y: number,
  width: number,
  height: number
}
```

No further "normalization" is required in the controlling app.

### Events

#### displaysUpdated

Fired when the server-side display settings change, for example when a new display is connected or the display layout or scaling changes. No data is provided to the event callback; the app should call `getBounds()`, `normalizeBounds()` etc. and invoke a remote window move to relocate the window to its correct new location (if required) in the new display arrangement.

## Limitations / Assumptions
- This SDK is only intended for use in Electron's Renderer process
- This SDK normalizes electron window movement (resizing, etc) in the server-side machine. It does not have access to resolution information on the host client. Therefore, for correct operation, all host client monitors should have the same resolution and scaling.
- If using Citrix in fullscreen mode with multiple monitors on Mac VDI, all displays must remain at 100% scaling within Windows (KAJ-1336)
- If using Citrix in fullscreen mode on a single monitor when multiple monitors are present on Mac VDI, video may appear on the wrong monitor or may not be visible at all (KAJ-1314)