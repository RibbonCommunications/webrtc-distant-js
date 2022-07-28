# Starting small

To start with, we will highlight the steps to create an application that is not running on VDI but is instead using Distant Web. This will come in handy even once we have a VDI setup to quickly iterate our application and our remote application.

## Initialization

Let's begin by initializing Distant, Distant Web and creating a controller.

```javascript {highlight:['1-2',7,12]}
import { createController } from '@distant/distant'
import { createConnection } from '@distant/distant-web'

let distantController

function createDistantConnection() {
  return createConnection()
}

function initializeDistant() {
  const distantConnection = createDistantConnection()
  distantController = createController(distantConnection)
}
```

Here we create 2 things. First we create a connection to Distant Web. Since Distant Web simply runs in the browser, there is no need to provide an address or anything else. It is simply an in process Distant Remote Agent. Then once the connection is successful (💡 It always is for Distant Web running in a browser) we create a Distant Controller with this connection.

<blockquote>

  __Distant Web in Electron__

  If you are creating a Distant Web connection in the *main* process of an Electron application you will need to use the electron window factory provided by Distant Web.

  ```javascript {highlight:[2,8]}
  import { createController } from '@distant/distant'
  import { createConnection, createElectronWindow } from '@distant/distant-web'

  let distantController

  function createDistantConnection() {
    return createConnection({
      windowFactory: createElectronWindow
    })
  }

  function initializeDistant() {
    const distantConnection = createDistantConnection()
    distantController = createController(distantConnection)
  }
  ```

</blockquote>

## Let's show something

Now we're ready to start interacting with the Distant Remote Agent and create a session.

```javascript {highlight:[7,19]}
const sessionId = 42
const sessions = {}
const remoteApplicationUrl = 'http://www.google.com'

async function createSession(controller, sessionId, url) {
  try {
    // Let's create a session that shows a google search window.
    const session = await controller.createSession({id: sessionId, targetUrl: url, timeout: 5000 })
    return session
  } catch(err) {

    // We had an error while creating the session. It's possible that the creation timed-out.
    console.log(err.message)
  }
}

async function initializeSession(controller) {
  const session = await createSession(controller, sessionId, remoteApplicationUrl)
  if (session) {
    sessions[sessionId] = session
    console.log(`Session information: ${JSON.stringify(session.getInfo())}`)
  }
}

// Our imaginary application has a showSessionButton that when clicked should show the
// distant session.
showSessionButton.addEventListener('click', event => {
  initializeSession(distantController)
})

```

Hmmm... why don't we see anything?

This is because Distant doesn't define a protocol for controlling a window's size, or showing/hiding it. It leaves all control of the remote application to the controller application. So if the application wants to show the Distant session, it needs to ask the session to show itself via a custom, application-defined message.

Let's move on to the next step and introduce the Distant Remote Application in order to show how this can be done.
