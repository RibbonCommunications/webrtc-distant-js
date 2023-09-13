# Enabling VDI

So far we have been using Distant Web only as a distant agent. Let's see what it takes to enable VDI mode.

## vchannel

The `@rbbn/distant-vchannel` package is used to communicate over the Citrix Virtual Channel to the distant-vdi remote agent.

This package contains native bindings to a C++ library called the Citrix WinFrame Library in order to connect to the channel. This library currently only runs in Windows; therefore, this mode is for Windows exclusively.

Additionally, vchannel can only run in a Node.js process as it is also a Node Add-On.

## Electron processes

The process model of Electron will inform us on how we will be using the vchannel package. Namely, electron consists of a main Node.js based process that manages multiple BrowserWindow processes.

So far we have not made clear the location of the code that is responsible for talking to Distant. Now we see that some parts (at least the `vchannel`) must run inside the Node.js main electron process.

## Connecting everything

Let's first re-state our Distant initialization code that we had [previously defined](./starting-small).

```javascript
import { createController } from '@rbbn/distant'
import { createWebConnection } from '@rbbn/distant-web'

let distantController

function createDistantConnection() {
  return createWebConnection()
}

function initializeDistant() {
  const distantConnection = createDistantConnection()
  distantController = createController(distantConnection)
}
```

Now let's add a VDI mode by adding the necessary packages first.

```javascript {highlight: ['3-5']}
import { createController } from '@rbbn/distant'
import { createWebConnection } from '@rbbn/distant-web'
import { openVirtualChannel } from '@rbbn/distant-vchannel'
import { createPullChannel } from '@rbbn/distant-pull-vchannel'
import { createPacketizationCodec } from '@rbbn/distant-codecs'
```

We need 3 new packages. Once you've done these next steps, you won't need to repeat them.

> Future: We aim to make this part easier.

Now let's add a mode to our `createDistantConnection` function in order to choose VDI or Web mode.

```javascript {highlight:['1-9','11-15']}
async function createDistantConnection(mode) {
  try {
    if (mode === 'vdi') {
      // First we open the channel
      const channel = await openVirtualChannel('RIBBON')

      // TODO: Return the appropriate connection
      // return connection
    } else {
      return createWebConnection()
    }
  } catch (err) {
    console.error('An error occurred while opening the channel.', err)
  }
}
```

Notice that the function now needs to be asynchronous because opening the channel is a blocking action that returns a promise.

We still have to complete the connection as a vchannel is not in an appropriate format to use with Distant directly.

```javascript {highlight:['7-10']}
async function createDistantConnection(mode) {
  try {
    if (mode === 'vdi') {
      // First we open the channel
      const channel = await openVirtualChannel('RIBBON')

      const pullChannel = createPullChannel(channel, err => {
        // This handler gets called if there was an error in the channel
        console.error(err)
      })

      return pullChannel
    } else {
      return createWebConnection()
    }
  } catch (err) {
    console.error('An error occurred while opening the channel.', err)
  }
}
```

So now we have an appropriate connection type for Distant. Are we done? Not quite.

Currently, VDI requires packetization. This will eventually be built into a package that aggregates vchannel, pull-vchannel and the packetization codec, but in the meantime we have to handle it manually.

```javascript {highlight:['1-3', '16-20']}
import { pull } from 'pull-stream'

const packetization = createPacketizationCodec()

async function createDistantConnection(mode) {
  try {
    if (mode === 'vdi') {
      // First we open the channel. Note that RIBBON is the name of the channel for Distant VDI.
      const channel = await openVirtualChannel('RIBBON')

      const pullChannel = createPullChannel(channel, err => {
        // This handler gets called if there was an error in the channel
        console.error(err)
      })

      // Add packetization to our duplex stream.
      const connection = {
        source: pull(pullChannel.source, packetization.encode),
        sink: pull(packetization.decode, pullChannel.sink)
      }

      return connection
    } else {
      return createWebConnection()
    }
  } catch (err) {
    console.error('An error occurred while opening the channel.', err)
  }
}
```

Finally we have arrived. We've now established a connection to the Distant VDI Remote Agent.

Note that we could improve on the above design in a few ways:

1. We could detect the mode and always choose VDI if we're able to create the channel.
2. We could handle errors better. Errors will both be propagated through Distant as well as through the pull-vchannel's `done` callback handler.
3. We could create some recovery mechanisms for re-establishing the connection if it drops.
