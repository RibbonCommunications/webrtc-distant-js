# Heartbeat

The Distant Toolkit provides complete, bi-directional, end-to-end heartbeating between your client application and the remote application.

**Note**: These instructions assume you are capable of creating a Distant Controller using the procedures described in the other `step-by-step` documentation. If you need to review these instructions, please [click here](/step-by-step/introduction.md).

## Setup

### 1. Default behaviour
Heartbeating is disabled by default, and must be explicitly enabled on a per-session basis by calling `startHeartbeat` on your session object (the object returned by `createSession`). Heartbeating will only begin if your session status is in a `READY` state.

```javascript
import { createController } from '@rbbn/distant'
... // Other imports

let controller
... // Create controller

const session = await controller.createSession({
  id: 1,
  targetUrl: 'https://your-remote-application.com'
})

// Start heartbeating
session.startHeartbeat()
```

### 2. Receiving Events
The `heartbeatLost` event will be emitted if the maximum number of timeouts is reached. You can register an observer for this event by calling the `on` method on your session object.

```javascript
session.on('heartbeatLost', event => {
  console.log(`Heartbeat has stopped: ${event.message}`)
})
```

### 3. Custom Values
By default, heartbeating is started with an interval of 2 seconds (2000ms) and a maximum of 3 timeouts before the `heartbeatLost` event is emitted. 
If you wish to enable heartbeating with different values, you can pass them as arguments to the `startHeartbeat` method. Note that interval units are expressed in milliseconds (ms) and the interval value **must be at least** `200` in order for it to be active.

```javascript
const session = await controller.createSession({
  id: 1,
  targetUrl: 'https://your-remote-application.com'
})

const heartbeatInterval = 3000
const maxTimeouts = 5
session.startHeartbeat(heartbeatInterval, maxTimeouts)
```

### 4. Return value
The `startHeartbeat` method returns a boolean value indicating whether heartbeating was successfully started. If heartbeating was not started, the method will return `false`.

```javascript
const result = session.startHeartbeat()
console.log(`Heartbeating started: ${result}`)
```
