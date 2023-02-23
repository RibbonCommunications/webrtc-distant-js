## Distant Remote

Library for remote applications to interact with Distant

## Usage

### JavaScript

Install using npm:

```
npm install @rbbn/distant-remote
```

or yarn:

```
yarn add @rbbn/distant-remote
```

Use the package like so:

```javascript
import distantRemote from '@rbbn/distant-remote'

//// OR
// import { createRemoteController } from '@rbbn/distant-remote'
// const distantRemote = createRemoteController(window.distant, window.cefQuery)

const textEncoder = new TextEncoder()
const textDecoder = new TextDecoder()

distantRemote.on('message', (message) => console.log(textDecoder.decode(message)))

distantRemote.sendMessage(textEncoder.encode("hello"))

distantRemote.moveWindow(44, 55, 666, 777)

distantRemote.setWindowVisibile(true)

distantRemote.close()
```
