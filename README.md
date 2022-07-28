# Introduction to the Distant Toolkit

## Description

The Distant Toolkit is a set of JavaScript libraries and tools that allows a web or Electron application to control remote web applications. The application creates a Distant Controller which is used to connect to a Distant Agent in order to create and manage Distant Sessions containing the remote web applications.

The main use case for Distant is to separate in two an application running in a VDI environment: one part on the VDI host, and the other on the client device.

## Getting started

See the [Getting Started](./guide/getting-started.md) section of the guide.

## How it works

At it's core, Distant is driven as a well defined protocol. This protocol uses Protocol Buffer (proto3) messages that are exchanged between the Distant Controller and the Distant Session in order to perform it's tasks.

You can learn more about the protocol [here](./how-it-works/distant-protocol.md).
