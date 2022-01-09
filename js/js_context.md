## The this keyword

The `this` keyword is a very unique and powerful feature of the javascript langauge

Every program in almost every language has different contexts of execution, such as the `global` context or the `object` context.

`this` is a special keyword that always binds/points to the active context during the execution of the program.

## Default Binding

The default context is always the `global` context. All declarations begin from the `global` context and the same applies for the `this` keyword. It originally binds to the `global` context. The following example should give you a good picture of this idea:

```js
foo = function () {
  console.log(this.x); // 100
  this.y = 500;
  bar(); // context = global > foo
};

bar = function () {
  console.log(this.y);
};

x = 100;
foo(); // context = global
```

In the above example, the entire execution is using an implicit binding to the default context, the `global` context.

We are invoking `foo()` using the `global` context. `this` binds/points to the `global` context throughout the entire execution of the program.

Here is a simplified pseudo-code to visualize the mechanism:

```js
ACTIVATE GLOBAL CONTEXT                 // CONTEXT = GLOBAL
 + BIND `THIS` TO CONTEXT               // THIS = CONTEXT = GLOBAL
 + SET X TO 100                         // GLOBAL.X = 100

 + INVOKE FOO()                         // CALL GLOBAL.FOO
  + PRINT X                             // PRINT GLOBAL.X (100)
  + SET THIS.X TO 500                   // SET GLOBAL.X = 500
  + INVOKE GLOBAL.BAR()                 // CALL GLOBAL.BAR
    + PRINT X                           // PRINT GLOBAL.X
```

As you can clearly see, the context has never changed during the execution of our program.
The `this` keyword has never been modified during the execution and thus always pointed/bind to the implicitly activated global context.

## Implicit Binding

Consider another example:

```js
function foo() {
  console.log(this.x);
  this.x = 444;
  bar();
}

function bar() {
  console.log(this.x);
}

obj = {
  x: 250,
  foo: foo,
};

x = 102030;
obj.foo(); // 250
```

Here is a simplified pseudo-code to visualize the mechanism:

```js
ACTIVATE `GLOBAL` CONTEXT                 // CONTEXT = GLOBAL
 + BIND `THIS` TO CONTEXT                 // THIS = CONTEXT = GLOBAL
 + DECLARE AND DEFINE OBJ                 // GLOBAL.OBJ = { ... }

 + INVOKE OBJ.FOO()                       // CALL GLOBAL.FOO
  + ACTIVATE `OBJ` CONTEXT                // CONTEXT = OBJ
  + BIND `THIS` TO CONTEXT                // THIS = CONTEXT = OBJ
  + PRINT X                               // PRINT OBJ.X (250)
  + SET THIS.X TO 444                     // SET OBJ.X = 444

  + INVOKE GLOBAL.BAR()                   // CALL GLOBAL.BAR
    + ACTIVATE `GLOBAL` CONTEXT           // CONTEXT = GLOBAL
      + BIND `THIS` TO CONTEXT            // THIS = CONTEXT = GLOBAL
      + PRINT X                           // PRINT GLOBAL.X (102030)
```

The execution of our program is yielding the expected results. The context is switched to the OBJ context as soon as the call to `obj.foo()` is executed. Javascript is implicitly switching the active context to `obj` and updating the `this` to bind / point to the activated context `obj`. Within `obj` we are invoking bar using the `global` context, thus switching back to the global context for new function scope.

Here's another contrived example that should give you a better understanding:

```js
function foo() {
  console.log(this.x);
}

var obj1 = { x: 100, foo: foo };
var obj2 = { x: 200, foo: foo, obj1: obj1 };
var obj3 = { x: 123, obj2: obj2 };

obj2.foo();
obj3.obj2.foo();
obj3.obj2.obj1.foo();
```

And here is the pseudo-code:

```js
ACTIVATE `GLOBAL` CONTEXT                 // CONTEXT = GLOBAL
 + BIND `THIS` TO CONTEXT                 // THIS = CONTEXT = GLOBAL
 + DECLARE AND DEFINE OBJ1                // GLOBAL.OBJ1 = { ... }
 + DECLARE AND DEFINE OBJ2                // GLOBAL.OBJ2 = { ... }
 + DECLARE AND DEFINE OBJ3                // GLOBAL.OBJ3 = { ... }

 + INVOKE OBJ2.FOO()                      // CALL OBJ2.FOO
  + ACTIVATE `OBJ2` CONTEXT               // CONTEXT = OBJ2
  + BIND `THIS` TO CONTEXT                // THIS = CONTEXT = OBJ2
  + PRINT X                               // PRINT OBJ2.X (200)

 + INVOKE OBJ2.FOO()                      // CALL OBJ3.OBJ2.FOO
  + ACTIVATE `OBJ2` CONTEXT               // CONTEXT = OBJ2
  + BIND `THIS` TO CONTEXT                // THIS = CONTEXT = OBJ2
  + PRINT X                               // PRINT OBJ2.X (200)

 + INVOKE OBJ1.FOO()                      // CALL OBJ3.OBJ2.OBJ1.FOO
  + ACTIVATE `OBJ1` CONTEXT               // CONTEXT = OBJ1
  + BIND `THIS` TO CONTEXT                // THIS = CONTEXT = OBJ1
  + PRINT X                               // PRINT OBJ1.X (100)
```

The code could be simplified even further:

```pseudo
(obj2)           [OBJ2].foo();
(obj3.obj2)      [OBJ2].foo();
(obj3.obj2.obj1) [OBJ1].foo();
```

In the first line we are calling foo() with obj2 context
In the second line we are getting obj2 from obj3, thus calling foo with obj2 context
In the last line we are getting obj1 from obj2 and obj3, thus calling foo with obj1 context

In the last line we are simply accessing obj2 using obj3 context, and then obj1 using obj2 context and finally calling foo with obj1 context, which implicitly binds the `this` keyword to the newly activate context of obj1.

Here's another example that might be tricky at first but easily explained:

```js
function foo() {
  console.log(this.x);
}

bar = {
  x: 300,
  foo: foo,
};

baz = bar.foo;

x = 5000;
baz();
```

And here is the pseudo-code:

```js
ACTIVATE `GLOBAL` CONTEXT                 // CONTEXT              = GLOBAL
 + BIND `THIS` TO CONTEXT                 // THIS                 = GLOBAL
 + DECLARE AND DEFINE BAR                 // GLOBAL.BAR           = { ... }
 + DECLARE AND DEFINE BAZ                 // GLOBAL.BAZ           = BAR.FOO
 + DECLARE AND DEFINE X                   // GLOBAL.X             = 5000

 + INVOKE GLOBAL.BAZ()                    // CALL OBJ2.FOO
  + BIND `THIS` TO CONTEXT                // THIS                 = GLOBAL
  + PRINT THIS.X                          // PRINT OBJ2.X         = 5000
```

As you can see, the context never changes during the execution of the program.
We are simply assigning the function `foo` which is accessed using `bar` context to our global variable `baz` using the global context. Invoking `baz` would be using the global context in this case and binding `this` to the global context, thus never switching execution context.

Here is another example passing callback functions:

```js
function foo() {
  console.log(this.x);
}

function bar(cb) {
  cb();
}

baz = {
  x: 100,
  foo: foo,
};

x = 12345;
bar(baz.foo);
```

Pseudo code as follows:

```js
ACTIVATE `GLOBAL` CONTEXT                 // CONTEXT          = GLOBAL
 + BIND `THIS` TO CONTEXT                 // THIS = CONTEXT   = GLOBAL
 + DECLARE AND DEFINE FOO                 // GLOBAL.FOO       = Function
 + DECLARE AND DEFINE BAR                 // GLOBAL.BAR       = Function
 + DECLARE AND DEFINE BAZ                 // GLOBAL.BAZ       = { ... }
 + DECLARE AND DEFINE X                   // GLOBAL.X         = 12345

 + INVOKE GLOBAL.BAR()                    // CALL OBJ2.FOO
 + SET PARAM TO BAZ.FOO                   // CB               = FOO FUNCTION
  + BIND `THIS` TO CONTEXT                // THIS             = GLOBAL
  + PRINT THIS.X                          // PRINT GLOBAL.X   = 12345
```

We're passing a reference to the `foo` function. The `foo` function is accessed using the `baz` context during an implicit reference assignment. There is no actual context switch outside of the assigning scope and thus the active context remains the `global` context.

## Explicit Binding

Javascript allows our program to excplicitly bind the `this` keyword. In order to do that, javascript has built-in functions such as `call`, `apply`, `bind` as well as others, which explicitly bind the `this` keyword to a chosen context.

Here's a contrived example along with its pseudo-code counterpart:

```js
function foo() {
  console.log(this.x);
}

bar = {
  x: 50,
};

function baz() {
  foo.call(bar);
  console.log(this.x);
}

foo.call(bar); // 50
baz.call({ x: 2000 }); // 50, 2000
```

Pseudo code:

```js
ACTIVATE `GLOBAL` CONTEXT                 // CONTEXT          = GLOBAL
 + BIND `THIS` TO CONTEXT                 // THIS             = GLOBAL
 + DECLARE AND DEFINE FOO                 // GLOBAL.FOO       = Function
 + DECLARE AND DEFINE BAR                 // GLOBAL.BAR       = { ... }
 + DECLARE AND DEFINE BAZ                 // GLOBAL.BAZ       = Function

 + INVOKE FOO.CALL(CONTEXT=BAR)           // CALL FOO.CALL
  + ACTIVATE `BAR` CONTEXT                // CONTEXT          = BAR
  + BIND `THIS` TO CONTEXT                // THIS             = BAR
  + PRINT THIS.X                          // BAR.X            = 50

 + INVOKE BAZ.CALL(CONTEXT={...})         // CALL BAZ.CALL
  + ACTIVATE `BAZ` CONTEXT                // CONTEXT          = BAZ
  + BIND `THIS` TO CONTEXT                // THIS             = BAZ

  + INVOKE FOO.CALL(CONTEXT=BAR)          // CALL FOO.CALL
    + ACTIVATE `BAR` CONTEXT              // CONTEXT          = BAR
    + BIND `THIS` TO CONTEXT              // THIS             = BAR
    + PRINT THIS.X                        // BAR.X            = 50

  *** CONTEXT IS STILL `BAZ` ***
  + PRINT THIS.X                          // BAZ.X            = 2000
```

As you can see the function call is simply a way to switch context between call, thus switching the context for the duration of the function's scope. The call to baz.call is also referred to as `hard binding`. Hard binding is basically a code pattern to explicitly forcing the binding of the function call to a context.

Hard binding can also be achieved using `apply` function, which only differs in paremeters passing. With function `apply`, arguments are provided as an `array`, whereas with function `call`, arguments are provided individually.

There is another way to perform hard binding using the function `bind` as shown below:

```js
function foo(y) {
  return this.x + y;
}

bar = {
  x: 4000,
};

baz = foo.bind(bar);
res = baz(1);
console.log(res); // 4001
```

Here `foo.bind` would create a closure using `bar` as active context. Thus, any subsequent calls to `baz` would be invoking `foo` function with `bar` context. This is another common type of hard-binding that forces the program to use a specified context.

## New Binding

Unlike other programming languages, javascript `new` operator is another fancy way to create a brand new object with a context bound to itself.

```js
function foo(x) {
  this.x = x;
}
bar = new foo(100);
console.log(bar.x); // 100
```

Another interesting aspect of the `new` operator cannot be used together with `call` or `apply`. It also takes precendence over implicit binding:

```js
function foo(y) {
  this.x = y;
}
bar = { foo: foo };
foo.call(bar, 1000); // bar.x = 1000
baz = new bar.foo(2000); // baz.x = 2000, bar.x is unchanged = 1000
console.log(bar.x, baz.x); // 1000, 2000
```

As you can see, `new` has precendence over implicit binding leaving the value of `bar` untouched. It creates a new copy of the object, binds the context and return the newly constructed/initialized object as a result.

The same would apply to `bind` as follows:

```js
function foo(y) {
  this.x = y;
}
bar = {};
baz = foo.bind(bar);
baz(1000); // modify bar using foo(with ctx=bar)
console.log(bar.x); // 1000
qux = new baz(2000);
console.log(bar.x, qux.x); // 1000, 2000
```

The code above is creating a variation of the `foo` function having as default implicit context the `bar` object, basically a closure with default context set to `bar`. Calling `baz()` would actually be calling `foo(ctx=bar)`, thus explicitly setting the `this` keyword to `bar` context (hard-binding).

However, using the `new` operator over `baz` will override the explicit binding, including hard-binding.

## Indirect references

```js
function foo() {
  console.log(this.x);
}

x = 100;
bar = { foo: foo };
baz = { x: 300 };
qux = baz.foo = bar.foo;
qux(); // 100
```

Pseudo code:

```js
ACTIVATE `GLOBAL` CONTEXT              // CONTEXT          = GLOBAL
BIND `THIS` TO CONTEXT                 // THIS             = GLOBAL
SET BAR.FOO TO BAZ.FOO                 // RESULT           = FOO REF
INVOKE RESULT                          // CALL GLOBAL.FOO()
```

## Lexical context

Javascript ES6 introduced a new function keyword called `arrow-function`. It is a special kind of function that lexically captures the context at call-time. An `arrow-function` cannot be overriden by the `new` operator.

```js
function foo(y) {
  this.y = y;
  return () => {
    console.log(this.x, this.y);
  };
}

bar = { x: 250 };
baz = { x: 500 };

qux = foo.call(bar, "foobar");
qux.call(baz); // 250, foobar
```

In this case `qux` will call `foo` with `bar` as context. Foo will return a reference to a new function having `bar` context binded to it. Therefore, any subsequent calls to `qux` will always bind to the `bar` context.

Another common example would be the usage of setTimeout function:

```js
function foo() {
  setTimeout(() => {
    console.log(this);
  }, 50);
}

bar = { x: 1000 };
baz = { x: 2000 };
foo.call(bar); // 1000
foo.call(baz); // 2000
```
