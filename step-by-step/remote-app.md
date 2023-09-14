# Remote application

The remote application in a distant application is an HTML and JavaScript application running in a Distant Session.

## First steps

Our simple remote application will have the following folder structure:

```
/remote-app
  /package.json
  /index.js
  /index.html
```

## index.html

A simple HTML file that simply has a `div` element that will be used host a video element.

```html
<!DOCTYPE html>
<html>
  <head>
    <!-- Your headers here -->
  </head>
  <body>
    <div id="video" style="height:480px;width:640px;background-color:blue;"></div>

    <script src="./index.js"></script>
  </body>
</html>
```

## index.js

```javascript {highlight:[1,8,11]}
import remoteController from '@rbbn/distant-remote'

// If this application is not running in a Distant Session, the remoteController will be undefined.
if (remoteController) {
  // Let's define some initial conditions for our remote application

  // Show the window
  remoteController.setWindowVisible(true)

  // Give the window a location and size
  remoteController.moveWindow({ x: 0, y: 0, width: 640, height: 480 })
}
```

Now our application, if running inside a distant session, will show itself and position itself in the top left corner with a size of 480p.

## Hosting

In order for our application to be able to run in the Distant Session, it has to be hosted somewhere. For development purposes and in order to use ES6 Modules imports with proper bundling, we will be using Parcel ([parceljs.org](https://parceljs.org)) to host a development server.

We do this by installing it:

```shell
$ yarn add --dev parcel
```

And then adding the following entry to our package.json

```json {highlight:[3]}
{
  "scripts": {
    "serve": "parcel index.html"
  }
}
```

Then from the command line you can start this application using the following command:

```shell
$ cd remoteApplication
$ npm run serve
```

This will launch your application and it will be available at the default Parcel url: http://localhost:1234/

---

Next we will look into connecting the application and the remote application in order for them to perform some tasks.
