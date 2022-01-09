## The observer pattern

The observer pattern plays a fundamental role in Node.js, considering the reactive nature of Node.js, most core API is built around asyncronous event-driven architecture.
The observer pattern, unlike the traditional continuation-passing style (callback), propagates and distributes its results to multiple observers, kind of like broadcasting.

The observer pattern is a built-in feature of Node.js and available through the EventEmitter built-in class.

The EventEmitter class allows to register one or more observers (also known as listeners), and distribute or notify them once an event is emitted.

```js
EventEmitter = require("events").EventEmitter;
emitter = new EventEmitter();
emitter.on("custom_event", function (obj1, obj2) {
  console.log("Event 'custom_event' triggered: ", obj1, obj2);
});
emitter.emit("custom_event", { foo: "bar" }, { bar: "baz" });
```

The preceding example would trigger `custom_event` and display the following:

```
Event 'custom_event' triggered:  { foo: 'bar' } { bar: 'baz' }
```

Node.js facilitates turning existing objects into observable with the `util` built-in module. The util module supports many useful utilities for application and module developers.

```js
const EventEmitter = require("events").EventEmitter;
const util = require("util");

function Foo() {}
util.inherits(Foo, EventEmitter);
console.log(Foo.prototype.emit); // [Function: emit]
```

Another and recommended way to do it in ES6 would be to extend the class directly:

```js
const EventEmitter = require("events");
class Foo extends EventEmitter {}
const foo = new Foo();
console.log(Foo.prototype.emit); // [Function: emit]
```
