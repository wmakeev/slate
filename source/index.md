---
title: scaleApp

toc_footers:
  - <a href='https://github.com/flosse/scaleApp'>GitHub page</a>

includes:
  - download
  - install
  - quick-start
  - api
  - build
  - testing
  - demo
  - changelog
  - licence

search: true
---

# What is scaleApp?

**scaleApp** is a tiny JavaScript framework for scalable and maintainable
[One-Page-Applications / Single-Page-Applications](http://en.wikipedia.org/wiki/Single-page_application).
The framework allows you to easily create complex web applications.

[![Build Status](https://img.shields.io/travis/flosse/scaleApp.svg?style=flat-square)](http://travis-ci.org/flosse/scaleApp)
[![Dependency Status](https://img.shields.io/gemnasium/flosse/scaleApp.svg?style=flat-square)](https://gemnasium.com/flosse/scaleApp)
[![NPM version](https://img.shields.io/npm/v/scaleapp.svg?style=flat-square)](http://badge.fury.io/js/scaleapp)
[![Coverage Status](https://img.shields.io/coveralls/flosse/scaleApp/master.svg?style=flat-square)](https://coveralls.io/r/flosse/scaleApp?branch=master)

You can dynamically start and stop/destroy modules that acts as small parts of
your whole application.

## Architecture overview

scaleApp is based on a decoupled, event-driven architecture that is inspired by
the talk of Nicholas C. Zakas -
["Scalable JavaScript Application Architecture"](https://www.youtube.com/watch?v=vXjVFPosQHw)
([Slides](http://www.slideshare.net/nzakas/scalable-javascript-application-architecture)).
There also is a little [Article](http://www.ubelly.com/2011/11/scalablejs/) that
describes the basic ideas.

![scaleApp architecture](https://raw.github.com/flosse/scaleApp/master/architecture.png)

### Module

A module is a completely independent part of your application.
It has absolutely no reference to another piece of the app.
The only thing the module knows is your sandbox.
The sandbox is used to communicate with other parts of the application.

### Sandbox

The main purpose of the sandbox is to use the
[facade pattern](https://en.wikipedia.org/wiki/Facade_pattern).
In that way you can hide the features provided by the core and only show
a well defined custom static long term API to your modules.
This is actually one of the most important concept for creating
mainainable apps. Change plugins, implementations etc.
but keep your API stable for your modules.
For each module a separate sandbox will be created.

### Core

The core is responsible for starting and stopping your modules.
It also handles the messages by using the
[Publish/Subscribe (Mediator) pattern](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern)

### Plugin

Plugins can extend the core or the sandbox with additional features.
For example you could extend the core with basic functionalities
(like DOM manipulation) or just aliases the features of a base library (e.g. jQuery).

## Features

+ loose coupling of modules
+ small (about 300 sloc / 8,7k min / 3.3k gz)
+ no dependencies
+ modules can be tested separately
+ replacing any module without affecting other modules
+ extendable with plugins
+ browser and [Node.js](http://nodejs.org/) support
+ flow control
+ [AMD](https://en.wikipedia.org/wiki/Asynchronous_module_definition) & [CommonJS](http://www.commonjs.org/) support
+ framework-agnostic

### Extendable

scaleApp itself is very small but it can be extended with plugins. There already
are some plugins available:

- `mvc` - simple MVC
- `i18n` - multi language UIs
- `permission` - take care of method access
- `state` - Finite State Machine
- `submodule` - cascade modules
- `dom` - DOM manipulation
- `strophe` - XMPP communication
- `modulestate` - event emitter for `init` and `destroy`
- `util` - helper methods like `mixin`, `uniqueId` etc.
- `ls` - list modules, instances & plugins

You can easily define your own plugin (see plugin section).