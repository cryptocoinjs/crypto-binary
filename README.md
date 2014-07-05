Crypto-Binary
=============
Assemble and disassemble binary messages used in cryptocoin applications.

MessageBuilder
--------------
Assemble a binary message out of chunks of data. All the methods return the builder itself, so are chainable. General usage is to use the various `put*` methods and then call `raw()` to get the output as a Buffer.

```js
var b = new MessageBuilder();
b.putInt8(255).putInt16(0xbeef);
b.put(new Buffer([1,2,3,4,5]));
console.log(b.raw().toString('hex')); // 'ffbeef0102030405'
```

* **putInt[8,16,32]**: Append an integer of the specified bit length to the message
* **put**: Append raw data to the message. If a number less than 255 is passed, acts like `putInt8`, otherwise input is expected to be a Buffer, which is appended to the message
* **putString**: Convert a string into ASCII-representation of the characters and append that to the message
* **putVarInt**: Convert a number into the Bitcoin [variable length integer](https://en.bitcoin.it/wiki/Protocol_specification#Variable_length_integer) notation and append it to the message
* **putVarString**: Convert a string into the Bitcoin [variable length string](https://en.bitcoin.it/wiki/Protocol_specification#Variable_length_string) notation and append it to the message


MessageParser
-------------
Given a string of binary data saved in a buffer, walk through it looking for structured data. Each of the methods returns the specified data from the current point in the data, and increments the pointer to the end of the data extracted. The point of this class is to allow you to not have to constantly check "is there enough data in the buffer to complete my next action?" or "did the last action fail?" and defer those checks to the end of parsing, making for cleaner code.

```js
var p = new MessageParser(rawData);
var itemNum = p.readVarInt();
for (var i = 0; i < itemNum; i++) {
  items.push(s.readUInt32LE());
}
if (p.hasFailed) {
  throw new Error('Ran out of data before this point!');
}
```

* **readUInt8**, **readUInt[16,32]LE**: Match the NodeJS Buffer methods of the same name; read an integer from the current pointer location of of the specified bit-size
* **readVarInt**: Grab a number encoded in the Bitcoin [variable length integer](https://en.bitcoin.it/wiki/Protocol_specification#Variable_length_integer) notation from the data stream
* **readVarString**: Grab a string encoded in the Bitcoin [variable length string](https://en.bitcoin.it/wiki/Protocol_specification#Variable_length_string) notation from the data stream
* **raw**: Grab a number of raw bytes from the data stream and return it as a Buffer.
* **incrPointer**, **setPointer**: Manipulate the current cursor location, either by incrementing it by a certain amount, or setting it to a specific absolute value

The MessageParser object has a `hasFailed` property, which is set to `true` if the actions request cannot be completed (pointer goes beyond the length of the buffer, usually). There is also a `failedStack` property that holds a stack trace from when the first failure occurred (in case you want to diagnose when in the process it failed)
