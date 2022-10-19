# Heartbeat

The Distant Toolkit provides complete, bi-directional, end-to-end heartbeating between your client application and the remote application.

**Note**: These instructions assume you are capable of creating a Distant Controller using the procedures described in the other `step-by-step` documentation. If you need to review these instructions, please [click here](/step-by-step/introduction.md).

## Setup

### 1. Default behaviour
Heartbeating is enabled by default, and is configured on a per-session basis by calling `createSession` on your controller object. The heartbeat begins as soon as your session status is in a `READY` state.

```javascript
import { createController } from '@distant/distant'
... // Other imports, based on your use case (Web or VDI)

let controller
... // Create controller

const session = await controller.createSession({
  id: 1,
  targetUrl: 'https://your-remote-application.com'
})
```

### 2. Receiving Events
The `heartbeatLost` event will be emitted if the maximum number of timeouts is reached. You can register an observer for this event by calling the `on` method on your session object.

```javascript
session.on('heartbeatLost', event => {
  console.log(`Heartbeat has stopped: ${event.message}`)
})
```

### 3. Disabling
You can disable Heartbeating by setting the `heartbeatInterval` property in the object passed to `createController` and setting its value to `0`.

```javascript
const session = await controller.createSession({
  id: 1,
  targetUrl: 'https://your-remote-application.com',
  heartbeatInterval: 0
})
```

### 4. Custom Values
If you wish for Heartbeating to be enabled with your own custom values, you can pass values for `heartbeatInterval` and `maxTimeouts`, which default to `200` and `3` respectively. Note that the `heartbeatInterval` value **must be set to at least** `20` in order for it to be active.

```javascript
const session = await controller.createSession({
  id: 1,
  targetUrl: 'https://your-remote-application.com',
  heartbeatInterval: 300,
  maxTimeouts: 5
})
```

### 5. Custom Behaviour
If you wish to have complete control over *when* heartbeating begins, you can choose to disable heartbeat initially, and then call `startHeartbeat` on your `session` object at an arbitrary timepoint. Please remember that heartbeating will **not** begin unless the interval value is above 20 and the session status is in `READY` state.

```javascript
const session = await controller.createSession({
  id: 1,
  targetUrl: 'https://your-remote-application.com',
  heartbeatInterval: 0
})

... // Session creation is successful

session.startHeartbeat()
```

Alternatively, `session::startHeartbeat` takes two optional parameters which set the interval value and the maximum number of timeouts before the remote application is considered lost.

```javascript
const optionalHeartbeatInterval = 200
const optionalMaxTimeouts = 3

session.startHeartbeat(optionalHeartbeatInterval, optionalMaxTimeouts)
```
