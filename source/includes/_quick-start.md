# Quick Start

## Create your own Sandbox

First of all create your own sandbox.
By doing that you're able to guarantee a
stable maintainable API for your modules.

```javascript
var MySandbox = function(core, instanceId, options, moduleId) {

  // define your API
  this.myFooProperty = "bar";

  // e.g. provide the Mediator methods 'on', 'emit', etc.
  core._mediator.installTo(this);

  // ... or define your custom communication methods
  this.myEmit = function(channel, data){
    core.emit(channel + '/' + instanceId, data);
  };

  // maybe you'd like to expose the instance ID
  this.id = instanceId;

  return this;
};

// ... and of course you can define shared methods etc.
MySandbox.prototype.foo = function() { /*...*/ };
```

## Create a core

Now create a new core instance with your sandbox:

```javascript
var core = new scaleApp.Core(MySandbox);
```

## Register modules

```javascript
core.register( "myModuleId", function( sandbox ){
  return {
    init:    function(){ /*...*/ },
    destroy: function(){ /*...*/ }
  };
});
```

As you can see the module is a function that takes the sandbox as a parameter
and returns an object that has two functions `init` and `destroy` (the latter is
optional).
Of course your module can be any usual class with those two functions.

```javascript
var MyGreatModule = function(sandbox){
  return {
    init:    function(){ alert("Hello world!"); }
    destroy: function(){ alert("Bye bye!");     }
  };
};

core.register("myGreatModule", MyGreatModule);
```

The `init` function is called by the framework when the module is supposed to
start. The `destroy` function is called when the module has to shut down.


## Asynchronous initialization

You can also init or destroy you module in a asynchronous way:

```javascript
var MyAsyncModule = function(sandbox){
  return {
    init: function(options, done){
      doSomethingAsync(function(err){
        // ...
        done(err);
      });
    },
    destroy: function(done){
      doSomethingElseAsync(done);
    }
  };
};

core.register("myGreatModule", MyGreatModule);
core.start("myGreatModule", { done:function(){
  alert("now the initialization is done");
}});
```

## Start modules

> After your modules are registered, start your modules:

```javascript
core
  .start( "myModuleId" )
  .start( "anOtherModule", function(err){
    // 'anOtherModule' is running now
  });
```

### Start options

> You may also want to start several instances of a module:

```javascript
core.start( "myModuleId", {instanceId: "myInstanceId" } );
core.start( "myModuleId", {instanceId: "anOtherInstanceId" });
```

> All you attach to `options` is accessible within your module:

```javascript
core.register( "mod", function(sandbox){
  return {
    init: function(opt){
      (opt.myProperty === "myValue")  // true
    },
    destroy: function(){ /*...*/ }
  };
});

core.start("mod", {
  instanceId: "test",
  options: { myProperty: "myValue" }
});
```

> If all your modules just needs to be instanciated once, you can simply starting them all:

```javascript
core.start();
```

> To start some special modules at once you can pass an array with the module names:

```javascript
core.start(["moduleA","moduleB"]);
```

> You can also pass a callback function:

```javascript
core.start(function(){
  // do something when all modules were initialized
});
```

> Moreover you can use a separate sandbox for each instance:

### second

```javascript
var MySandbox = function(){/*...*/};
core.start("module", { sandbox: MySandbox });
```

## Stopping

It's obvious:

```javascript
core.stop("moduleB");
core.stop(); // stops all running instances
```

## Publish/Subscribe

If the module needs to communicate with others, you can use the `emit` and
`on` methods.

### emit

The `emit` function takes three parameters whereas the last one is optional:
- `topic` : the channel name you want to emit to
- `data`  : the data itself
- `cb`    : callback method

The emit function is accessible through the sandbox
(as long as you exposed the Mediator methods of course):

```javascript
sandbox.emit( "myEventTopic", myData );
```

### on

A message handler could look like this:

```javascript
var messageHandler = function( data, topic ){
  switch( topic ){
    case "somethingHappend":
      sandbox.emit( "myEventTopic", processData(data) );
      break;
    case "aNiceTopic":
      justProcess( data );
      break;
  }
};
```

... and it can listen to one or more channels:

```javascript
sub1 = sandbox.on( "somthingHappend", messageHandler );
sub2 = sandbox.on( "aNiceTopic", messageHandler );
```
Or just do it at once:

```javascript
sandbox.on({
  topicA: cbA,
  topicB: cbB,
  topicC: cbC
});
```

You can also subscribe to several channels at once:

```javascript
sandbox.on(["a", "b"], cb);
```

If you prefer a shorter method name you can use the alias `on`.

#### attache and detache

A subscription can be detached and attached again:

```javascript
sub.detach(); // don't listen any more
sub.attach(); // receive upcoming messages
```

#### Unsubscribe

You can unsubscribe a function from a channel

```javascript
sandbox.off("a-channel", callback);
```

And you can remove a callback function from all channels

```javascript
sandbox.off(callback);
```

Or remove all subscriptions from a channel:

```javascript
sandbox.off("channelName");
```

## Flow control

### Series

```javascript
var task1 = function(next){
  setTimeout(function(){
    console.log("task1");
    next(null, "one");
  },0);
};

var task2 = function(next){
  console.log("task2");
  next(null, "two");
};

scaleApp.util.runSeries([task1, task2], function(err, result){
  // result is ["one", "two"]
});

// console output is:
// "task1"
// "task2"
```

### Parallel

```javascript
var task1 = function(next){
  setTimeout(function(){
    console.log("task1");
    next(null, "a");
  },0);
};

var task2 = function(next){
  console.log("task2");
  next(null, "b");
};

scaleApp.util.runParallel([task1, task2],function(err,result){
  // result is ["a", "b"]
});

// console output is:
// "task2"
// "task1"
```

There is also a little helper tool to run the same async task
again and again in parallel for different values:

```javascript
var vals = ["a","b", "c"];
var worker = function(val, next){
  console.log(val);
  doSomeAsyncValueProcessing(val,function(err,result){
    next(err, result);
  });
};

scaleApp.util.doForAll(args, worker, function(err, res){
  // fini
});
```

### Waterfall

```javascript
var task1 = function(next){
  setTimeout(function(){
    next(null, "one", "two");
  },0);
};

var task2 = function(res1, res2, next){
  // res1 is "one"
  // res2 is "two"
  next(null, "yeah!");
};

scaleApp.util.runWaterfall([task1, task2], function(err, result){
  // result is "yeah!"
});
```

## Plugins

There are some plugins available within the `plugins` folder.
For more information look at the
[plugin README](https://github.com/flosse/scaleApp/blob/master/plugins/README.md).

### Register plugins

A single plugin can be registered with it option object in that way:

```javascript
core.use(plugin,options);
```
If you want to register multiple plugins at once:

```javascript
core.use([
  plugin1,
  plugin2,
  { plugin: plugin3, options: options3 }
]);
```

### Write your own plugin

It's easy:

```javascript
core.use(function(core){
  core.helloWorld = function(){ alert("helloWorld"); };
};
```

Here a more complex example:

```javascript
core.use(function(core, options, done){

  // extend the core
  core.myCoreFunction = function(){ alert("Hello core plugin") };
  core.myBoringProperty = "boring";

  // extend the sandbox class
  core.Sandbox.prototype.myMethod = function(){/*...*/};

  // define a method that gets called when a module starts
  var onModuleInit = function(instanceSandbox, options, done){

    // e.g. define sandbox methods dynamically
    if (options.mySwitch){
      instanceSandbox.appendFoo = function(){
       core.getContainer.append("foo");
      };
    }

    // or load a something asynchronously
    core.myAsyncMethod(function(data){

      // do something...
      // now tell scaleApp that you're done
      done();
    });
  };

  // define a method that gets called when a module stops
  var onModuleDestroy = function(done){
    myCleanUpMethod(function(){
      done()
    });
  };

  // don't forget to return your methods
  return {
    init: onModuleInit,
    destroy: onModuleDestroy
  };

});
```

Usage:

```javascript
core.myCoreFunction() // alerts "Hello core plugin"

var MyModule = function(sandbox){
  init: function(){ sandbox.appendFoo(); },  // appends "foo" to the container
};
```
