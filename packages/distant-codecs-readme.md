## Distant Codecs

Holds various codecs that can be used with Distant.

## Usage

### JavaScript

Install using npm:

```
npm install @distant/distant-codecs
```

or yarn:

```
yarn add @distant/distant-codecs
```

Use the package like so:

```javascript
import { createDistantProtobufCodec } from '@distant/distant-codecs'

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
