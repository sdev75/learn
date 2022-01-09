## Streams

Streams are an important feature of Node.js. Streams allow for efficient I/O handling, making them more efficient in memory consumption and speed, allowing data to flow and to be processed without waiting for an entire buffer to be read all at once. This allows data to flow as in a stream and to be processed in real time.

All stream classes extend the EventEmitter class and implement one of the built-in base abstract classes:

- stream.Readable (for example: fs.CreateReadStream())
- stream.Writable (for example: fs.createWriteStream())
- stream.Duplex (for example: net.Socket)
- stream.Transform (for example: zlib.createDeflate())

Extending the `events.EventEmitter` class, allows the stream classes to emit events using the observer pattern. All streams are created in "pause" mode, meaning the data must be requested explicitly, this is also known as pulling mode, or non-flowing mode. A stream can also be set to operate in pushing or flowing-mode, with the use of the EventEmitter mechanism.

Streams use internal buffers to keep data in memory. The `highWaterMark` option passed into the stream's constructor, determines the total number of bytes of the data potentially buffered internally. Streams can also work with objects, in that case the `highWaterMark` specifies a total number of objects.

### Backpressure

A stream is like a bucket, it has a capacity (highWaterMark), and once the bucket is full, it will overflow or backpressure. The data overflowing is not lost, but rather stored in an internal buffer and flushed out as soon as it is consumed.

A bucket having a highWaterMark of 4 bytes, will be considered full if the data being pushed down the queue is equal or greater than 4 bytes. In that case, consuming the internal buffer will flush out (drain) the entire bucket (internal buffer). At that point, the bucket has space for more data. More data will automatically fill the bucket, pushing it to the queue until full. As soon as the bucket is full, the filling process will interrupt until consumed. This will continue to repeat in order to avoid high memory consumptions, especially when working on huge amount of data.

#### Readable Stream

Readable streams are producers of data. They make use of an internal buffer which is filled by calling the `push()` method. The `push()` method pushes a chunk of data down to the internal queue. The data produced will sit in the internal queue until it is consumed.

The `_read()` method fills the internal buffer with new chunk of data. If the total size of the internal buffer reaches the threshold specified by the `highWaterMark` option, then the Readable stream will stop pushing chunk of data down to the internal buffer queue, until the internal buffer is consumed.

Readable stream can be pushed or pulled. Pulling makes use of the event emitter to automatically send events when new data is ready to be read. The pull method requires explicit pulling to consume the internal buffer.

**Pulling method:**

```js
process.stdin
  // pushes data as soon as it is available to be consumed
  .on("readable", function () {
    let chunk;
    // flushing data from the internal buffer
    while ((chunk = process.stdin.read()) !== null) {
      process.stdout.write(`Flushed out ${chunk.length} bytes\n`);
    }
  })
  // fires as soon as the end of data is reached
  .on("end", function () {
    process.stdout.write("End of data");
  });
```

**Pulling method**

```js
readable.read(); // flushes out the internal buffer
readable.read(2); // flushes out 2 bytes from the internal buffer
```

Readable streams can be implemented by extending the stream.Readable class:

```js
class Foo extends stream.Readable {
  constructor(opts) {
    super(opts);
    this.data = ["foo", "bar", "baz"];
  }
  _read(size) {
    if (!this.data.length) {
      this.push(null);
      return;
    }

    this.push(this.data.shift());
  }
}
let foo = new Foo();
console.log(foo.read()); // <Buffer 66 6f 6f>
console.log(foo.read()); // <Buffer 62 61 72>
console.log(foo.read()); // <Buffer 62 61 7a>
console.log(foo.read()); // null

foo = new Foo();
foo.on("readable", function () {
  const chunk = foo.read();
  chunk && process.stdout.write(chunk);
});
foo.on("end", function () {
  console.log(" - End of data");
});

// Output: foobarbaz - End of data
```

Here's another implementation of the stream Readable class:

```js
producer = new stream.Readable({ highWaterMark: 3 });
producer.data = ["abcd", "12", "12345", "a"];
producer._read = function (size) {
  // * the `size` param is useful to determine how much data to fetch
  // pulling more data from our data source (producer.data)
  let chunk = this.data.shift();
  if (!chunk) {
    // pushing end of data to queue
    this.push(null);
    // _read() will stop filling the bucket with new data
    return false;
  }

  // _read() will continue to push data down onto the queue
  // _read() will stop pushing data if overflow occurs or the data ends
  const canfill = this.push(chunk);
  console.log(`Fill with ${chunk.length} bytes | Full:`, !canfill);
  return canfill;
};

// proper null catching is advised
console.log("Flushed:", producer.read()); // 1
console.log("Flushed:", producer.read()); // 2

// Output:
// read() - 1
//  Fill with 4 bytes | Full: true   (bucket capacity: 4/3   | overflow: 1)
//  Flushed: abcd                    (bucket drained)
//  Fill with 2 bytes | Full: false  (bucket capacity: 2/3   | overflow: 0)
//  Fill with 5 bytes | Full: true   (bucket capacity: 7/3   | overflow: 4)

// read() - 2
//  Fill with 1 bytes | Full: true   (bucket capacity: 8/3   | overflow: 5)
//  Flushed: 1212345a                (bucket drained)
```

A stream can also be set to operate on objects rather than a sequence of bytes:

```js
class Foo extends stream.Readable {
  constructor() {
    super({ objectMode: true });
    this.data = [
      { id: 100, data: "foo" },
      { id: 200, data: "bar" },
      { id: 300, data: "baz" },
    ];
  }
  _read(size) {
    if (!this.data.length) {
      this.push(null);
      return;
    }

    this.push(this.data.shift());
  }
}
let foo = new Foo();
console.log(foo.read()); // { id: 100, data: 'foo' }
console.log(foo.read(5)); // { id: 200, data: 'bar' } (size arg, ignored)
console.log(foo.read()); // { id: 300, data: 'baz' }
console.log(foo.read()); // null
```

It's possible to implement custom Readable streams by inheriting the prototype of stream.Readable and implementing the `_read()` method.

```js
const stream = require("stream");
const util = require("util");

class Foo extends stream.Readable {
  constructor(options) {
    super(options);
    this.count = 10;
  }
  _read(size) {
    if (this.count < 0) {
      this.push(null);
      return;
    }
    let buf = (this.count--).toString(10);
    this.push(buf);
  }
}

foo = new Foo();
let buf;
while ((buf = foo.read()) !== null) {
  console.log("foo received: ", buf.toString());
}
```

The preceding example will print a countdown from 10 to 0 using stream.Readable implementation. Not the best example, but it helps to get an idea.

#### Writable streams

Writable streams are responsible for writing data to a destination. The method `write(chunk)` pushes data down onto the internal buffer queue. When the total size of the internal buffer is above the threshold set by `highWaterMark`, calls to `write(chunk)` will return false. In that case the `drain` event will be emitted when it is appropriate to resume writing data to the stream.

Pushing data to a Writable stream is very similar to using the Readable stream interface.

```js
class Foo extends stream.Writable {
  constructor(options) {
    super(options);
    this.count = 10;
  }
  _write(chunk, encoding, callback) {
    console.log("Foo.write(): ", chunk.toString());
    callback();
  }
}

foo = new Foo();
foo.write("foo");
foo.write("bar");
foo.end(function () {
  console.log("end fired");
});

// Output:
// Foo.write():  foo
// Foo.write():  bar
// end fired
```

#### Duplex streams

A duplex stream is both Readable and Writable. In order to implement a Duplex stream, both \_read() and \_write() must be implemented.

#### Transform streams

Transform streams are Duplex stream specifically designed for data transformation. Every read and write call is going through a filtering / processing / transformation, some sort of middleware processing.

```js
class Foo extends stream.Transform {
  constructor(options) {
    super(options);
  }
  _write(chunk, encoding, callback) {
    if (chunk == "foo") {
      console.log("Foo word is not allowed");
      callback();
      return;
    }
    console.log(chunk.toString());
    callback();
  }
}

foo = new Foo();
foo.write("foo");
foo.write("bar");
foo.end(function () {
  console.log("end fired");
});

// Output:
// Foo word is not allowed
// bar
// end fired
```

#### Piping streams

Every stream has a `pipe` method that allows a Readable stream (source), to be combined with a Writable stream (dest). The pipe mechanism will ensure that the destination threshould is respected along with setting the appropriate event emitting mechanisms to pipe the stream together properly.

```js
const stream = require("stream");
reader = new stream.Readable();
reader.count = 0;
reader._read = function () {
  if (this.count++ > 5) {
    this.push(null);
    return false;
  }
  return this.push("foo");
};
writer = new stream.Writable();
writer._write = function (chunk, encoding, callback) {
  console.log(chunk);
  callback();
};
reader.pipe(writer);
```

## Multiplexing

Multiplexing is when multiple streams are merged together into a single shared channel.

Here's an example of a multiplexer or mux:

```js
// mux.js
const child_process = require("child_process");
const net = require("net");
const path = require("path");
console.log(process.argv);
const socket = net.connect(3000, function () {
  // multiplex channels
  const child = child_process.fork("child", null, { silent: true });
  const sources = [child.stdout, child.stderr];
  let channels = sources.length;

  for (let i = 0; i < sources.length; i++) {
    sources[i]
      .on("readable", function () {
        let chunk;
        while ((chunk = this.read()) !== null) {
          let buf = Buffer.alloc(5 + chunk.length);
          buf.writeUInt8(i, 0);
          buf.writeUInt32BE(chunk.length, 1);
          chunk.copy(buf, 5);
          socket.write(buf);
        }
        switch (i) {
          case 0:
            console.log("on readable for (stdout)", i);
            break;
          case 1:
            console.log("on readable for (stderr)", i);
            break;
        }
      })
      .on("end", function () {
        console.log("source channel closed");
        if (--channels === 0) {
          console.log("all source channels are closed. Closing socket...");
          socket.end();
        }
      });
  }
});
```

#### Demultiplexing the data received

```js
const net = require("net");

let channel = null;
let length = null;

net
  .createServer(function (socket) {
    socket
      .on("readable", function () {
        let chunk;

        if (channel === null) {
          chunk = socket.read(1);
          channel = chunk && chunk.readUInt8(0);
        }

        if (length === null) {
          chunk = this.read(4);
          length = chunk && chunk.readUInt32BE(0);
          if (length === null) {
            return;
          }
        }

        chunk = this.read(length);
        if (chunk === null) {
          return;
        }

        console.log(`Received data for channel: ${channel}, length: ${length}`);
        console.log(`chunk: '${chunk}'`);
        channel = null;
        length = null;
      })
      .on("end", function () {
        console.log("socket closed");
        process.exit();
      });
  })
  .listen(3000, function () {
    console.log("Server listening on port 3000...");
  });
```

The source of the data (child process):

```js
console.log("this is for the stdout");
console.error("this is for the stderr");
console.log("Another text for the standard output");
console.log("yet another chunk for stdout");
console.error("this is the last error for std error");
```

The output would look like this:

```
Server listening on port 3000...
Received data for channel: 0, length: 23
chunk: 'this is for the stdout
'
Received data for channel: 0, length: 66
chunk: 'Another text for the standard output
yet another chunk for stdout
'
Received data for channel: 1, length: 60
chunk: 'this is for the stderr
this is the last error for std error
'
socket closed
```
