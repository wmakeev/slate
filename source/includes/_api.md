# API

* `scaleApp.VERSION` - the current version of scaleApp
* `scaleApp.Mediator` - the Mediator class
* `scaleApp.Sandbox` - the Sandbox class
* `scaleApp.Core` - the Core class

<div></div>

## Module

**scaleApp** module is a function that takes the sandbox as a parameter and returns an object that has two functions `init` and `destroy` (the latter is optional).

The `init` function is called by the framework when the module is supposed to start. The `destroy` function is called when the module has to shut down.

```javascript
var myModule = function (sandbox) {
  return {
    init    : function (options) { /* ... */ },
    destroy : function () { /* ... */ }
  }
}
```

<div></div>

### Asynchronous module initialization

You can also init or destroy you module in a asynchronous way:

```javascript
var MyAsyncModule = function (sandbox) {
  return {
    init: function (options, done) {
      doSomethingAsync(function (err) {
        // ...
        done(err);
      });
    },
    destroy: function (done) {
      doSomethingElseAsync(done);
    }
  };
};
```

## Core

Create core instance.

```javascript
// use default sandbox
var core = new scaleApp.Core();

// use your own sandbox
var core = new scaleApp.Core(yourSandboxClass);
```

<div></div>

#### `core.register(moduleName, myModule, [options])`

Register a module.

```javascript
core.register("myModuleId", myModule, options);
```

#### Arguments

`moduleName` - name of module

`myModule` - module factory

`options` - object that will be passed to the module init method

<div></div>

### `core.use(plugin, [options])`

Register a plugin

```javascript
core.use(plugin, options);
```

<div></div>

#### Arguments

`plugin` - plugin factory

> Plugin factory receives an instance of sandbox and returns a module instance.

```javascript
var plugin = function (core, options, done) {
  return {
    init: function (options) { /* ... */ }
  }
}
```

`options` - object that will be passed to the module init method

<div></div>




- `core.use(pluginArray)` - registers an array of plugins
- `core.boot(callback)` - initialize plugins
   (will be executed automatically on ´start´)
- `core.start(moduleId, options, callback)` - start a module
- `core.stop(instanceId, callback)` - stop a module

## Mediator

```javascript
// create a mediator
var mediator = new scaleApp.Mediator();

// create a mediator with a custom context object
var mediator = new scaleApp.Mediator(context);

// create a mediator with cascaded channels
var mediator = new scaleApp.Mediator(null, true);
```

- `mediator.emit(channel, data, callback)`
- `mediator.on(channel, callback, context)`
- `mediator.off(channel, callback)`
- `mediator.installTo(context, force)`

```javascript
// subscribe
var subscription = mediator.on(channel, callback, context);
```
- `subscription.detach` - stop listening
- `subscription.attach` - resume listening

```javascript
var fn  = function(){ /*...*/ };
var obj = { emit: fn };

// the installTo method prevents existing properties by default
mediator.installTo(obj);
obj.emit === fn // true

// set the second paramater to 'true'
// to force the mediator to override existing propeties
mediator.installTo(obj, true);
obj.emit === mediator.emit // true
```

## Sandbox

This is the default sandbox of scaleApp.
It's a better idea to use your own one.

```javascript
var sandbox =  new scaleApp.Sandbox(core, instanceId, options, moduleId) - create a Sandbox
```
- `sandbox.emit` is `mediator.emit`
- `sandbox.on` is `mediator.on`
- `sandbox.off` is `mediator.off`
