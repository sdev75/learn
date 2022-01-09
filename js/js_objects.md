## Javascript Objects

Javascript is a very unique and dynamic language using objects. Unlike other Object-Oriented languages such as C++, Javascript might seem to resemble class class-oriented design, similar to other object-oriented languages, especially with the introduction of the new keyword `class` in ES6. However, Javascript works using delegation, rather than copying a class, it merely creates a link between objects, this link is often referred to as prototype chain.

Not everything in Javascript is an object. In fact, Javascript has several primitive types such as `string`, `boolean`, `number`, `null` and `undefined`. Most of the time these primitives are automatically coerced into their object equivalent when necessary.

```js
foo = "javascript";
typeof foo; // 'string'

bar = true;
typeof bar; // boolean
bar instanceof Boolean; // false

baz = new Boolean();
baz instanceof Boolean; // true
```

Objects come in two forms: declarative / literal and constructed.

The declarative syntax is one the most common way to construct and object in javascript, allowing better flexibility during definition.

```js
/*
 * Both declarative/literal and constructed form yield the exact same result
 */

// literal form (most common)
foo = {
  bar: 100,
};

// constructed form (less common)
foo = new Object();
foo.bar = 100;
```

##

## Object subtypes

Javascript has several built-in object subtypes:

- String
- Boolean
- Number
- Function
- Array
- Date
- RegExp
- Error

Although these built-in objects are classified as being object types, they are actually built-in functions which can be used as a constructor using the `new` operator.

Javascript automatically coerces primitive types to their object form when necessary. For example, a string literal is not an object, but an actual primitive immutable string type:

##

## Object properties

Properties of an object can be accessed using either the property access `.` operator or the key access `[]` operator. The latter allows any Unicode-compatible string key index as input.

Javascript object supports computed property names using the key access operator.
One of the most common usage of computed property names is the ES6 `Symbol`.

##

## Object property descriptors

Javascript allows you to fine tune the descriptor characteristics of an object's property.

#### Writable

```js
foo = {};
Object.defineProperty(foo, "x", {
  value: 1000,
  writable: false,
  configurable: true,
  enumerable: true,
});
foo.x = 2000;
console.log(foo.x); // 1000
```

Using `"use strict"` would cause Javascript to fail with a TypeError.

#### Configurable

When a property is flagged as non configurable, javascript will prevent further
modifications, regardless of strict mode. This is cannot be undone. An attempt to modify the property would fail with error `Uncaught TypeError: Cannot redefine property`.

```js
foo = { x: 100 };
Object.defineProperty(foo, "x", {
  value: 1000,
  writable: true,
  configurable: false,
  enumerable: true,
});
foo.x = 2000;

Object.defineProperty(foo, "x", {
  value: 5000,
  writable: true,
  configurable: true,
  enumerable: true,
});
```

The non configurable descriptor also prevent the functionality of the `delete` operator. Calling delete on a non configurable object would be ignored.

```js
foo = { x: 100 };
Object.defineProperty(foo, "x", {
  value: 1000,
  writable: true,
  configurable: false,
  enumerable: true,
});

delete foo.x;
console.log(foo.x); // 100
```

Delete is used to remove properties directly from the object allowing any references to be cleaned up and garbage collected.

#### Enumerable

This descriptor allows a property to be hidden from enumerations such as in for loops.

##

## Objects immutability

In order to achieve immutability of an object javascript offers several solutions. Setting both configurable with writable to false would prevent the object from being modified, deleted or redefined.

JavaScript has several other methods such as `preventExtensions` for preventing creation of new properties, `Seal` and `Freeze`.

#### Getters and Setters

When a property has a setter or getter or both defined, its definition becomes an accessor descriptor (instead of data descriptor). When a property is an accessor descriptor, the value and writable characteristic of the descriptor are ignored, thus giving precedence to property access.

When a property access (get) is performed, the the object performs the get operation on itself first, and will trasversale examine the prototype chain if it does not find the requested property.

When an assignment to a property (put/assignment) is performed, JavaScript will check whether the `property accessor descriptor` has an existing setter, if so, call it. It will then check whether the property is a data descriptor with the writable set to false, otherwise set the value to the existing property as normal.

#### Object Enumeration

The `in` operator checks the existence of a property including its prototype chain. By contrast, `hasOwnProperty` checks the existence of the property only for the object itself without consulting the prototype chain.

```js
foo = { x: true };
"x" in foo; // true
```

The enumerable property descriptor characteristic makes a property hidden to `for` loops and other functions.

```js
foo = { bar: 500 };
Object.defineProperty(foo, "baz", {
  value: 100,
  enumerable: false,
});
foo.baz; // 100
"baz" in foo; // true
foo.hasOwnProperty("baz"); // true;
for (let p in foo) {
  console.log(p); // bar
}
```

JavaScript makes iteration over the values in data structure using the `for of` loop. A custom iterator could be implemented using the internal property of the object using the Symbol.iterator.

```js
foo = { x: 100, y: 200 };
Object.defineProperty(foo, Symbol.iterator, {
  configurable: true,
  writable: false,
  enumerable: false,
  value: function () {
    const o = this;
    const k = Object.keys(o);
    let i = 0;
    return {
      next: function () {
        return {
          value: o[k[i++]],
          done: i > k.length,
        };
      },
    };
  },
});
const it = foo[Symbol.iterator]();
it.next(); // {value: 100, done: false}
it.next(); // {value: 200, done: false}
it.next(); // {value: undefined, done: true}
```

## JavaScript classes

Although JavaScript syntactic like `new`, `instanceof` and `class` exist in JavaScript, the class design pattern implementation is completely different from languages like C++, as a matter of fact does not actually have classes.

#### Mixins

Javascript does not have any mechanism to automatically perform a copy of the object during instantiation or inheritance, that's primarily because Javascript does not have classes like other languages. The `mixin pattern` is often used to emulate copy-behaviour feature.

Below is a copy of an `explicit mixin`, sometimes referred to as `extend`:

```js
function mixin(src, dst) {
  for (let k in src) {
    if (!(k in dst)) {
      dst[k] = src[k];
    }
  }
  return dst;
}

let foo = {
  x: 100,
  y: 200,
  printPoints: function () {
    console.log(this.x, this.y);
  },
};
let bar = mixin(foo, {
  x: 500,
  printPoints: function () {
    foo.printPoints.call(this);
    console.log("bar.printPoints()");
  },
});

bar.printPoints(); // 500 200 bar.printPoints()
```

In the example above, `mixin` is just a simple way to copy all the properties from an object to another without overwriting existing ones. Definitely this has nothing to do with classes, because Javascript does not have any such thing as Classes.

Another example showing the usage of `implicit mixins` below:

```js
foo = {
  bar() {
    this.x = 5000;
    this.y = this.x + 1;
  },
};

baz = {
  bar() {
    foo.bar.call(this);
    this.y += 1;
  },
};

foo.bar();
foo.x; // 5000
foo.y; // 5001

baz.bar();
baz.x; // 5000
baz.y; // 5002 (as expected)
```

By using an `implicit mixin`, we are borrowing the function from `foo` to construct `baz` without sharing any state.

## Javascript Prototypes

Every object in Javascript has an internal property `prototype`. This allows an object to be linked/chained to another object, thus allowing an object to share state between newly constructed object.l

```js
let foo = { x: 100 };
bar = Object.create(foo);

foo.x = 200;
bar.__proto__.x; // 200
bar.x; // 200 chain trasversal (this property exists in bar.__proto__.x)
```

Using `Object.create` method creates a new object from an existing object's prototype. In this case `bar` would have `foo`'s chain prototype.

Setting an access property value involes several steps, as follows:

- if the data accessor property `bar` already exists directly in `foo`, the assignment would be executed normally.
- if `foo` object does not have an access property `bar`, the prototype chain is trversed until a `bar` access property is found.
- if no access property `bar` is found anywhere in the chain, a new value is directly assigned to object `foo`.

```js
let foo = { bar: 500 };
foo.__proto__.baz = 100;
foo.bar = 500; // direct assignment
foo.baz = 200; // chain traversal and assignment
foo.baz; // 200
foo.baz = 1000;
foo.__proto__.baz; // 100
foo.baz; // 1000
delete foo.baz;
foo.baz; // 100
```

As you can see, the prototype chain is a simple way to allow an object from extending properties, similar to inheritance for classes.

If an `x` property is found by traversing the prototype chain and its marked as unwritable, then any direct assignment will be ignored or trigger an error if strict mode is enabled, as shown in the example below:

```js
let foo = {};
Object.defineProperty(foo, "x", {
  writable: false,
  value: 50,
});
let bar = Object.create(foo);
bar.x = 100;
bar.x; // 50
```

It's very important to understand that Javascript uses an object-linking mechanism. It doesn't copy object properties, it created a link between two object and delegates access to these properties by trasversing the prototype chain.

## JavaScript prototypes

All functions in JavaScript get a public, non-enumerable property called `prototype`. Whenever the new operator is invoked to create a new object using this function, it will automatically link the new object with the function prototype object. You could think of it like a parent-child relationship, where using the `new` operator on the parent would result in a new object (child) with a reference to the parent’s `prototype` property.

There is no copying involved into a concrete object, but merely a chain link reference. This mechanism is called `prototypal inheritance`. Instead of copying / instantiating an object, two object get linked with each other without any copying involved. The `prototype` property provides a fallback lookup in case if a property reference isn't found directly on an object, as part of the default `get` algorithm.

```js
// Foo gets a public, non-enumerable property `prototype`
function Foo() {}
Foo.prototype; // Foo {}
bar = new Foo();
Object.getPrototypeOf(bar) === Foo.prototype; // true
```

All declared functions get a public, non-enumerable `constructor` property, which it references back the original function. This `constructor` property is also added to the object when a new object is created using the `new` operator. Both of these `constructor` properties point to the same original function. The constructor reference is delegated up to function prototype.

```js
function Foo() {}
Foo.prototype.constructor === Foo; // true
bar = new Foo();
bar.constructor === Foo; // true
bar.hasOwnProperty("constructor"); // false
Object.getPrototypeOf(bar).hasOwnProperty("constructor"); // true
```

the `constructor` property on `Foo.prototype` is only there by default on the object created when Foo the function is declared. If you create a new object, and replace a function’s default `prototype` property object reference, the new object will not by default magically get a `constructor` property on it.

```js
function Foo() {}
Foo.prototype = {};
bar = new Foo();
bar.constructor === Foo; // false
bar.constructor === Object; // true
bar.hasOwnProperty("constructor"); // false
Object.getPrototypeOf(bar).hasOwnProperty("constructor"); // false
```

The `constructor` is mutable and non-enumerable property. It can be overwritten, thus making it unreliable reference to rely on.

It’s important to note that functions themselves are not constructors. The `new` operator makes a special `constructor call` sets up an object with additional properties, in addition to executing the function itself.

Here is an example of prototypal inheritance:

```js
function Foo() {
  this.x = 10;
  this.y = 20;
}
function Bar() {
  // inherit from Foo
  Foo.call(this);
  this.z = 30;
}
Foo.prototype.printXY = function () {
  console.log(this.x, this.y);
};

// This will remove the original Bar constructor
Bar.prototype = Object.create(Foo.prototype);
Bar.prototype.constructor; // Foo

Bar.prototype.printXYZ = function () {
  console.log(this.x, this.y, this.z);
};
```

Assigning the prototype of `Foo` to the prototype of `Bar` would create an internal reference to the `Foo` prototype. Therefore, any changes to `Bar` prototype would propagate to `Foo` prototype, because of this internal prototype chain link.

```js
function Foo() {}
function Bar() {}
Bar.prototype = Foo.prototype;
Bar.prototype.x = 100;
Foo.prototype.x; // 100
Foo.prototype.x = 200;
Foo.prototype.x; // 200
```

There are side effects of using the `new` operator, as shown below:

```js
function Foo() {
  this.x = 10;
}
function Bar() {}
Bar.prototype = new Foo();
baz = new Bar();
baz.constructor; // Foo
baz.x; // 10
```

Using the `new` operator will cause side effects, such as setting properties and changing the `constructor` reference, including making introspection / debugging a bit more difficult. An alternative way of doing this, would be to use the new ES6 `Object.setPrototypeOf` method, which does work in a more predictable way.

```js
function Foo() {}
function Bar() {}
Object.setPrototypeOf(Bar.prototype, Foo.prototype);
baz = new Bar();
baz.constructor; // Bar
baz instanceof Bar; // true
baz instanceof Foo; // true
baz instanceof Object; // true
```

## Javascript ES6 class syntax

ES6 introduced the `class` keyword in order to make prototypal inheritance easier to write and maintain. Consider the following example below:

```js
class Foo {
  constructor() {
    this.x = 100;
    this.y = 200;
  }
  print() {
    console.log(this);
  }
}
class Bar extends Foo {
  constructor() {
    super();
    this.z = 300;
  }
  print() {
    console.log(this);
  }
}
Foo; // Function: Foo
Foo.prototype; // Foo {}
Bar; // Function: Bar
Bar.prototype; // Bar {}

foo = new Foo();
foo.print(); // Foo { x: 100, y: 200 }

Foo.prototype.print = function () {
  console.log("Foo print called");
};

foo.print(); // Foo print called
```

As expected, the `class` keyword is simplifying prototypal inheritance. The `class` keyword does not create a static copy of the declaration, rather linking the two objects together via prototype link.

Another example:

```js
class Foo {}
foo = new Foo();
Foo.prototype.x = 100;
foo.x; // 100
foo.x = 1000;
foo.__proto__.x; // 100
Foo.prototype.x; // 100
delete Foo.prototype.x;
foo.__proto__.x; // undefined
foo.x; // 1000
```

The example above shows that despite the new `class` keyword, Javascript is using dynamic delegation using the `prototype` chain link.
