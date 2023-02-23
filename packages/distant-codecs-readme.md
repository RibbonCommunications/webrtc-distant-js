## Distant Codecs

Holds various codecs that can be used with Distant.

## Usage

### JavaScript

Install using npm:

```
npm install @rbbn/distant-codecs
```

or yarn:

```
yarn add @rbbn/distant-codecs
```

Use the package like so:

```javascript
import { createDistantProtobufCodec } from '@rbbn/distant-codecs'

const codec = createDistantProtobufCodec()

const message = { foo: 55 }

const invalidMessageError = codec.verify(message)

if (invalidMessageError) {
  console.error("The message is invalid: " + invalidMessageError)
  return
}

const encodedMessage = codec.encode(message)

const decodedMessage = codec.decode(encodedMessage)

expect(decodedMessage).toEqual(message)
```
