# pull-vchannel

Create a pull-stream from a vchannel.

## Usage

Install using npm:

```
npm install @distant/pull-vchannel
```

or yarn:

```
yarn add @distant/pull-vchannel
```

Use the package like so:

```javascript
import { pull, drain, values } from 'pull-stream'
import { createPullChannel } from '@distant/pull-vchannel'

const channel = getChannel()

const done = (err) => {
  if (err) {
    return console.error("There was an error that closed the channel", err)
  }

  console.log("The channel closed normally")
}

const pullChannel = createPullChannel(channel, done)

pull(
  pullChannel,
  drain(
    (message) => console.log("We got a message", message),
    (err) => console.log("The stream closed", err)
  )
)

pull(
  values(["foo", "bar", "xyzzy"]),
  pullChannel
)

```
