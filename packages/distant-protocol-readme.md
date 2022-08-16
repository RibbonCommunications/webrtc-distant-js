---
title: Distant Protocol
notes: To view this markdown file properly, it is suggested you view it in VSCode using the markdown-preview-enhanced add-on.
---

# Distant Protocol

This package holds the distant protobuf protocol definition. It's meant as a central repository that dependents can link to as a single place for the protocol.

The protocol file is here: [distant.proto](./distant.proto)

# Usage

## JavaScript

Install using npm:

```
npm install @distant/distant-protocol
```

or yarn:

```
yarn add @distant/distant-protocol
```

Use the package like so:

```javascript
import {distant as distantProtocol} from '@distant/distant-protocol

function useProtocol() {
  // Use the protocol ...
  distantProtocol.SessionStart.create({start: { id: 1, targetUrl: 'http://example.org'}})

}
```

# Development

After modifying the protocol, you will need to run `yarn build` in order to update code-generated source code.
