# Getting Started

## Installation

### JavaScript packages

All of the JavaScript libraries are provided as Node modules that can also run in the browser using a package bundler like WebPack, Rollup, Parcel.

Using npm:

```shell
$ npm install @rbbn/distant
```

Using yarn:

```shell
$ yarn add @rbbn/distant
```

See the list of available modules [here](./modules.md)

### Distant Driver for VDI

Distant Driver for VDI comes in the form of a dynamic library that gets loaded into the Citrix Receiver process at runtime. We support Windows, Mac and eLux.

See the [Distant Driver for VDI](https://github.com/RibbonCommunications/webrtc-distant-vdi) page for more information.

#### eLux EPM package

Distant Driver for VDI is distributed as a Scout Enterprise EPM package for use with eLux RP thin-clients.

#### Windows / Mac

 On Windows and Mac we provide executables files, libraries and other resources that need to be packaged into your own installer.