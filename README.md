# JavaScript Records & Tuples Proposal

**Authors:**

- Robin Ricard (Bloomberg)
- Rick Button (Bloomberg)
- Nicolò Ribaudo (Babel)

**Champions:**

- Robin Ricard (Bloomberg)
- Rick Button (Bloomberg)

**Advisors**

- Philipp Dunkel (Bloomberg)
- Dan Ehrenberg (Igalia)
- Maxwell Heiber (Bloomberg)

**Stage:** 1

### [Try out Record and Tuple in the playground!](https://rickbutton.github.io/record-tuple-playground)

### [Spec Text](https://tc39.es/proposal-record-tuple/)

### [Tutorial](https://tc39.es/proposal-record-tuple/tutorial/)

### [Cookbook](https://tc39.es/proposal-record-tuple/cookbook/)

# Overview

This proposal introduces two new deeply immutable data structures to JavaScript:

- `Record`, a deeply immutable Object-like structure `#{ x: 1, y: 2 }`
- `Tuple`, a deeply immutable Array-like structure `#[1, 2, 3, 4]`

Records and Tuples can only contain primitives and other Records and Tuples. You could think of Records and Tuples as "compound primitives". By being thoroughly based on primitives, not objects, Records and Tuples are deeply immutable.

Records and Tuples support comfortable idioms for construction, manipulation and use, similar working with objects and Arrays. They are compared deeply by their contents, rather than by their shallow identity.

JavaScript engines may perform certain optimizations on construction, manipulation and comparison of Records and Tuples, analogous to the way Strings are often implemented in JS engines. (It should be understood, these optimizations are not guaranteed.)

Records and Tuples aim to be usable and understood with external typesystem supersets such as TypeScript or Flow.

## Prior work on immutable data structures in JavaScript

Today, userland libraries implement similar concepts, such as [Immutable.js](https://immutable-js.github.io/immutable-js/). Also [a previous proposal attempt](https://github.com/sebmarkbage/ecmascript-immutable-data-structures) has been done previously but abandoned because of the complexity of the proposal and lack of sufficient use cases.

This new proposal is still inspired by this previous proposal but introduces some significant changes: Record and Tuples are now deeply immutable. This property is fundamentally based on the observation that, in large projects, the risk of mixing immutable and mutable data structures grows as the amount of data being stored and passed around grows as well so you'll be more likely handling large record & tuple structures. This can introduce hard-to-find bugs.

As a built-in, deeply immutable data structure, this proposal also offers a few usability advantages compared to userland libraries:
- Records and Tuples are easily introspectable in a debugger, while library provided immutable types are often hard to inspect as you have to inspect through data structure details.
- Because they're accessed through typical object and Array idioms, no additional branching is needed in order to write a generic library that consumes both immutable and JS objects; with user libraries, method calls may be needed just in the immutable case.
- We avoid cases where developers may expensively convert between regular JS objects and immutable structures, by making it easier to just always use the immutable ones.

[Immer](https://github.com/mweststrate/immer) is a notable approach to immutable data structures, and prescribes a pattern for manipulation through producers and reducers. It is not providing immutable data types however, as it generates frozen objects. This same pattern can be adapted to the structures defined in this proposal in addition to frozen objects.

Deep equality as defined in user libraries can vary significantly, in part due to possible references to mutable objects. By drawing a hard line about only deeply containing primitives, Records and Tuples, and recursing through the entire structure, this proposal defines simple, unified semantics for comparisons.

# Examples

#### `Record`

```js
const document = #{
  id: 1234,
  title: "Record & Tuple proposal",
  contents: `...`,
  // tuples are primitive types so you can put them in records:
  keywords: #["ecma", "tc39", "proposal", "record", "tuple"],
};

// Accessing keys like you would with objects!
console.log(document.title); // Record & Tuple proposal
console.log(document.keywords[1]); // tc39

// Spread like objects!
const document2 = #{
  ...document,
  title: "Stage 1: Record & Tuple",
};
console.log(document.title); // Stage 1: Record & Tuple
console.log(document.keywords[1]); // tc39

// Object work functions on Records:
console.log(Object.keys(document)); // ["contents", "id", "keywords", "title"]
```

> [Open in playground](https://rickbutton.github.io/record-tuple-playground/#eyJjb250ZW50IjoiXG5jb25zdCBkb2N1bWVudCA9ICN7XG4gIGlkOiAxMjM0LFxuICB0aXRsZTogXCJSZWNvcmQgJiBUdXBsZSBwcm9wb3NhbFwiLFxuICBjb250ZW50czogYC4uLmAsXG4gIGtleXdvcmRzOiAjW1wiZWNtYVwiLCBcInRjMzlcIiwgXCJwcm9wb3NhbFwiLCBcInJlY29yZFwiLCBcInR1cGxlXCJdLFxufTtcblxuLy8gQWNjZXNzaW5nIGtleXMgbGlrZSB5b3Ugd291bGQgd2l0aCBvYmplY3RzIVxuY29uc29sZS5sb2coZG9jdW1lbnQudGl0bGUpOyAvLyBSZWNvcmQgJiBUdXBsZSBwcm9wb3NhbFxuY29uc29sZS5sb2coZG9jdW1lbnQua2V5d29yZHNbMV0pOyAvLyB0YzM5XG5cbi8vIFNwcmVhZCBsaWtlIG9iamVjdHMhXG5jb25zdCBkb2N1bWVudDIgPSAje1xuICAuLi5kb2N1bWVudCxcbiAgdGl0bGU6IFwiUmVjb3JkICYgVHVwbGUgRUNNQVNjcmlwdCBwcm9wb3NhbFwiLFxufTtcbmNvbnNvbGUubG9nKGRvY3VtZW50LnRpdGxlKTsgLy8gUmVjb3JkICYgVHVwbGUgRUNNQVNjcmlwdCBwcm9wb3NhbFxuY29uc29sZS5sb2coZG9jdW1lbnQua2V5d29yZHNbMV0pOyAvLyB0YzM5XG5cbi8vIEZpbmFsbHkgeW91IGNhbiBhbHNvIHVzZSBPYmplY3QgZnVuY3Rpb25zIG9uIFJlY29yZHM6XG5jb25zb2xlLmxvZyhPYmplY3Qua2V5cyhkb2N1bWVudCkpOyAvLyBbXCJjb250ZW50c1wiLCBcImlkXCIsIFwia2V5d29yZHNcIiwgXCJ0aXRsZVwiXVxuIiwic3ludGF4IjoiaGFzaCJ9)

Functions can handle Records and Objects in generally the same way:

```js
const ship1 = #{ x: 1, y: 2 };
// ship2 is an ordinary object:
const ship2 = { x: -1, y: 3 };

function move(start, deltaX, deltaY) {
  // we always return a record after moving
  return #{
    x: start.x + deltaX,
    y: start.y + deltaY,
  };
}

const ship1Moved = move(ship1, 1, 0);
// passing an ordinary object to move() still works:
const ship2Moved = move(ship2, 3, -1);

console.log(ship1Moved === ship2Moved); // true
// ship1 and ship2 have the same coordinates after moving
```

> [Open in playground](https://rickbutton.github.io/record-tuple-playground/#eyJjb250ZW50IjoiXG5jb25zdCBzaGlwMSA9ICN7IHg6IDEsIHk6IDIgfTtcbi8vIHNoaXAyIGlzIGFuIG9yZGluYXJ5IG9iamVjdDpcbmNvbnN0IHNoaXAyID0geyB4OiAtMSwgeTogMyB9O1xuXG5mdW5jdGlvbiBtb3ZlKHN0YXJ0LCBkZWx0YVgsIGRlbHRhWSkge1xuICAvLyB3ZSBhbHdheXMgcmV0dXJuIGEgcmVjb3JkIGFmdGVyIG1vdmluZ1xuICByZXR1cm4gI3tcbiAgICB4OiBzdGFydC54ICsgZGVsdGFYLFxuICAgIHk6IHN0YXJ0LnkgKyBkZWx0YVksXG4gIH07XG59XG5cbmNvbnN0IHNoaXAxTW92ZWQgPSBtb3ZlKHNoaXAxLCAxLCAwKTtcbi8vIHBhc3NpbmcgYW4gb3JkaW5hcnkgb2JqZWN0IHRvIG1vdmUoKSBzdGlsbCB3b3JrczpcbmNvbnN0IHNoaXAyTW92ZWQgPSBtb3ZlKHNoaXAyLCAzLCAtMSk7XG5cbmNvbnNvbGUubG9nKHNoaXAxTW92ZWQgPT09IHNoaXAyTW92ZWQpOyAvLyB0cnVlXG4vLyBzaGlwMSBhbmQgc2hpcDIgaGF2ZSB0aGUgc2FtZSBjb29yZGluYXRlcyBhZnRlciBtb3ZpbmdcbiIsInN5bnRheCI6Imhhc2gifQ==)

See [more examples here](./details.md#records).

#### `Tuple`


```js
const measures = #[42, 12, 67, "measure error: foo happened"];

// Accessing indices like you would with arrays!
console.log(document[0]); // 42
console.log(document[3]); // measure error: foo happened

// Slice and spread like arrays!
const correctedMeasures = #[
  ...measures.sliced(0, measures.length - 1),
  -1
];
console.log(document[0]); // 42
console.log(document[3]); // -1

// or use the .with() shorthand for the same result:
const correctedMeasures2 = document.with(3, -1);
console.log(document[0]); // 42
console.log(document[3]); // -1

// Tuples support methods similar to Arrays
console.log(correctedMeasures2.map(x => x + 1)); // #[43, 13, 68, 0]
```

> [Open in playground](https://rickbutton.github.io/record-tuple-playground/#eyJjb250ZW50IjoiXG5jb25zdCBtZWFzdXJlcyA9ICNbNDIsIDEyLCA2NywgXCJtZWFzdXJlIGVycm9yOiBmb28gaGFwcGVuZWRcIl07XG5cbi8vIEFjY2Vzc2luZyBpbmRpY2VzIGxpa2UgeW91IHdvdWxkIHdpdGggYXJyYXlzIVxuY29uc29sZS5sb2coZG9jdW1lbnRbMF0pOyAvLyA0MlxuY29uc29sZS5sb2coZG9jdW1lbnRbM10pOyAvLyBtZWFzdXJlIGVycm9yOiBmb28gaGFwcGVuZWRcblxuLy8gU2xpY2UgYW5kIHNwcmVhZCBsaWtlIGFycmF5cyFcbmNvbnN0IGNvcnJlY3RlZE1lYXN1cmVzID0gI1tcbiAgLi4ubWVhc3VyZXMuc2xpY2VkKDAsIG1lYXN1cmVzLmxlbmd0aCAtIDEpLFxuICAtMVxuXTtcbmNvbnNvbGUubG9nKGRvY3VtZW50WzBdKTsgLy8gNDJcbmNvbnNvbGUubG9nKGRvY3VtZW50WzNdKTsgLy8gLTFcblxuLy8gb3IgdXNlIHRoZSAud2l0aCgpIHNob3J0aGFuZCBmb3IgdGhlIHNhbWUgcmVzdWx0OlxuY29uc3QgY29ycmVjdGVkTWVhc3VyZXMyID0gZG9jdW1lbnQud2l0aCgzLCAtMSk7XG5jb25zb2xlLmxvZyhkb2N1bWVudFswXSk7IC8vIDQyXG5jb25zb2xlLmxvZyhkb2N1bWVudFszXSk7IC8vIC0xXG5cbi8vIEZpbmFsbHksIHlvdSBjYW4gYWxzbyB1c2UgdGhlIGZ1bmN0aW9ucyBhdmFpbGFibGUgb24gdGhlIFR1cGxlIHByb3RvdHlwZVxuLy8gdGhhdCBpcyBzaW1pbGFyIHRvIHRoZSBBcnJheSBwcm90b3R5cGU6XG5jb25zb2xlLmxvZyhjb3JyZWN0ZWRNZWFzdXJlczIubWFwKHggPT4geCArIDEpKTsgLy8gI1s0MywgMTMsIDY4LCAwXVxuIiwic3ludGF4IjoiaGFzaCJ9)

Similarly than with records, we can treat tuples as array-like:

```js
const ship1 = #[1, 2];
// ship2 is an array:
const ship2 = [-1, 3];

function move(start, deltaX, deltaY) {
  // we always return a tuple after moving
  return #[
    start[0] + deltaX,
    start[1] + deltaY,
  ];
}

const ship1Moved = move(ship1, 1, 0);
// passing an array to move() still works:
const ship2Moved = move(ship2, 3, -1);

console.log(ship1Moved === ship2Moved); // true
// ship1 and ship2 have the same coordinates after moving
```

> [Open in playground](https://rickbutton.github.io/record-tuple-playground/#eyJjb250ZW50IjoiXG5jb25zdCBzaGlwMSA9ICNbMSwgMl07XG4vLyBzaGlwMiBpcyBhbiBhcnJheTpcbmNvbnN0IHNoaXAyID0gWy0xLCAzXTtcblxuZnVuY3Rpb24gbW92ZShzdGFydCwgZGVsdGFYLCBkZWx0YVkpIHtcbiAgLy8gd2UgYWx3YXlzIHJldHVybiBhIHR1cGxlIGFmdGVyIG1vdmluZ1xuICByZXR1cm4gI1tcbiAgICBzdGFydC54ICsgZGVsdGFYLFxuICAgIHN0YXJ0LnkgKyBkZWx0YVksXG4gIF07XG59XG5cbmNvbnN0IHNoaXAxTW92ZWQgPSBtb3ZlKHNoaXAxLCAxLCAwKTtcbi8vIHBhc3NpbmcgYW4gb3JkaW5hcnkgb2JqZWN0IHRvIG1vdmUoKSBzdGlsbCB3b3JrczpcbmNvbnN0IHNoaXAyTW92ZWQgPSBtb3ZlKHNoaXAyLCAzLCAtMSk7XG5cbmNvbnNvbGUubG9nKHNoaXAxTW92ZWQgPT09IHNoaXAyTW92ZWQpOyAvLyB0cnVlXG4vLyBzaGlwMSBhbmQgc2hpcDIgaGF2ZSB0aGUgc2FtZSBjb29yZGluYXRlcyBhZnRlciBtb3ZpbmdcbiIsInN5bnRheCI6Imhhc2gifQ==)

See [more examples here](./details.md#tuples).

#### Forbidden cases

As stated before Record & Tuple are deeply immutable: attempting to insert an object in them will result in a TypeError:

```js
const instance = new MyClass();
const constContainer = #{
    instance: instance
};
// TypeError: Record literals may only contain primitives, Records and Tuples

const tuple = #[1, 2, 3];

tuple.map(x => new MyClass(x));
// TypeError: Callback to Tuple.prototype.map may only return primitives, Records or Tuples

// The following should work:
Array.from(tuple).map(x => new MyClass(x))
```

# Syntax

This defines the new pieces of syntax being added to the language with this proposal.

We define a record or tuple expression by using the `#` modifier in front of otherwise normal object or array expressions.

#### Examples

```js
#{}
#{ a: 1, b: 2 }
#{ a: 1, b: #[2, 3, #{ c: 4 }] }
#[]
#[1, 2]
#[1, 2, #{ a: 3 }]
```

#### Syntax errors

Holes are prevented in syntax, unlike Arrays, which allow holes. See [issue #84](https://github.com/tc39/proposal-record-tuple/issues/84) for more discussion.

```js
const x = #[,]; // SyntaxError, holes are disallowed by syntax
```

Using the `__proto__` identifier as a property is prevented in syntax. See [issue #46](https://github.com/tc39/proposal-record-tuple/issues/46) for more discussion.

```js
const x = #{ __proto__: foo }; // SyntaxError, __proto__ identifier prevented by syntax

const y = #{ "__proto__": foo }; // valid, creates a record with a "__proto__" property.
```

Concise methods are disallowed in Record syntax.

```js
#{ method() { } }  // SyntaxError
```

#### Runtime errors

Records may only have String keys, not Symbol keys, due to the issues described in [#15](https://github.com/tc39/proposal-record-tuple/issues/15). Creating a Record with a Symbol key is a `TypeError`.

```js
const record = #{ [Symbol()]: #{} };
// TypeError: Record may only have string as keys
```

Records and Tuples may only contain primitives and other Records and Tuples. Attempting to create a `Record` or `Tuple` that contains any type except the following: `Record`, `Tuple`, `String`, `Number`, `Symbol`, `Boolean`, `Bigint`, `undefined`, or `null` is a `TypeError`.

# Equality

Equality of Records and Tuples works like that of other JS primitive types like Boolean and String values, comparing by contents, not identity:

```js
assert(#{ a: 1 } === #{ a: 1 });
assert(#[1, 2] === #[1, 2]);
```

This is distinct from how equality works for JS objects: comparison of objects will observe that each object is distinct:

```js
assert({ a: 1 } !== { a: 1 });
assert(Object(#{ a: 1 }) !== Object(#{ a: 1 }));
assert(Object(#[1, 2]) !== Object(#[1, 2]));
```

Insertion order of record keys does not affect equality of records, because there's no way to observe the original ordering of the keys, as they're implicitly sorted:

```js
assert(#{ a: 1, b: 2 } === #{ b: 2, a: 1 });

Object.keys(#{ a: 1, b: 2 })  // ["a", "b"]
Object.keys(#{ b: 2, a: 1 })  // ["a", "b"]
```

If their structure and contents are deeply identical, then `Record` and `Tuple` values considered equal according to all of the equality operations: `Object.is`, `==`, `===`, and the internal SameValueZero algorithm (used for comparing keys of Maps and Sets). They differ in terms of how `-0` is treated:
- `Object.is` treats `-0` and `0` as unequal
- `==`, `===` and SameValueZero treat `-0` with `0` as equal

Note that `==` and `===` are more direct about other kinds of values nested in Records and Tuples--returning `true` if and only if the contents are identical (with the exception of `0`/`-0`). This directness has implications for `NaN` as well as comparisons across types. See examples below.

See further discussion in [#65](https://github.com/rricard/proposal-const-value-types/issues/65).

```js
assert(#{ a:  1 } === #{ a: 1 });
assert(#[1] === #[1]);

assert(#{ a: -0 } === #{ a: +0 });
assert(#[-0] === #[+0]);
assert(#{ a: NaN } === #{ a: NaN });
assert(#[NaN] === #[NaN]);

assert(#{ a: -0 } == #{ a: +0 });
assert(#[-0] == #[+0]);
assert(#{ a: NaN } == #{ a: NaN });
assert(#[NaN] == #[NaN]);
assert(#[1] != #["1"]);

assert(!Object.is(#{ a: -0 }, #{ a: +0 }));
assert(!Object.is(#[-0], #[+0]));
assert(Object.is(#{ a: NaN }, #{ a: NaN }));
assert(Object.is(#[NaN], #[NaN]));

// Map keys are compared with the SameValueZero algorithm
assert(new Map().set(#{ a: 1 }, true).get(#{ a: 1 }));
assert(new Map().set(#[1], true).get(#[1]));
assert(new Map().set(#[-0], true).get(#[0]));
```

# The object model of `Record` and `Tuple`

In general, you can treat Records like objects. For example, the `Object` namespace and the `in` operator work with `Records`.

```js
const keysArr = Object.keys(#{ a: 1, b: 2 }); // returns the array ["a", "b"]
assert(keysArr[0] === "a");
assert(keysArr[1] === "b");
assert(keysArr !== #["a", "b"]);
assert("a" in #{ a: 1, b: 2 });
```

## Advanced internal details: `Record` and `Tuple` wrapper objects

JS developers will typically not have to think about `Record` and `Tuple` wrapper objects, but they're a key part to how Records and Tuples work "under the hood" in the JavaScript specification.

Accessing of a Record or Tuple via `.` or `[]` follows the typical [`GetValue`](https://tc39.es/ecma262/#sec-getvalue) semantics, which implicitly converts to an instance of the corresponding wrapper type. You can also do the conversion explicitly through `Object()`:

- `Object(record)` creates a Record wrapper object
- `Object(tuple)` creates a Tuple wrapper object

(One could imagine that `new Record` or `new Tuple` could create these wrappers, like `new Number` and `new String` do, but Records and Tuples follow the newer convention set by Symbol and BigInt, making these cases throw, as it's not the path we want to encourage programmers to take.)

Record and Tuple wrapper objects have all of their own properties with the attributes `writable: false, enumerable: true, configurable: false`. The wrapper object is not extensible. All put together, they behave as frozen objects. This is different from existing wrapper objects in JavaScript, but is necessary to give the kinds of errors you'd expect from ordinary manipulations on Records and Tuples.

An instance of `Record` has the same keys and values as the underlying `record` value.  The `__proto__` of each of these Record wrapper objects is `null` (discussion: [#71](https://github.com/tc39/proposal-record-tuple/issues/71)).

An instance of `Tuple` has keys that are `${index}` for each index in the underlying `tuple` value. The value for each of these keys is the corresponding value in the original `tuple`. In addition, there is a non-enumerable `length` key. Overall, these properties match those of the `String` wrapper object. That is, `Object.getOwnPropertyDescriptors(Object(#["a", "b"]))` and `Object.getOwnPropertyDescriptors(new String("ab"))` each return an object that looks like this:

```json
{
  "0": {
    "value": "a",
    "writable": false,
    "enumerable": true,
    "configurable": false
  },
  "1": {
    "value": "b",
    "writable": false,
    "enumerable": true,
    "configurable": false
  },
  "length": {
    "value": 2,
    "writable": false,
    "enumerable": false,
    "configurable": false
  }
}
```

The `__proto__` of Tuple wrapper objects is `Tuple.prototype`. Note that, if you're working across different JavaScript global objects ("Realms"), the `Tuple.prototype` is selected based on the current Realm when the Object conversion is performed--it's not attached to the Tuple value itself. `Tuple.prototype` has various methods on it, analogous to Arrays.

For integrity, out-of-bounds numerical indexing on Tuples returns `undefined`, rather than forwarding up through the prototype chain, as with TypedArrays. Lookup of non-numerical property keys forwards up to `Tuple.prototype`, which is important to find their Array-like methods.

# `Record` and `Tuple` standard library support

`Tuple` values have functionality broadly analogous to `Array`. Similarly, `Record` values are supported by different `Object` static methods.

```js
assert(Object.keys(#{ a: 1, b: 2 }) === #["a", "b"]);
assert(#[1, 2, 3].map(x => x * 2), #[2, 4, 6]);
```

See the [appendix](./NS-Proto-Appendix.md) to learn more about the `Record` & `Tuple` namespaces.

## Converting from Objects and Arrays

You can convert structures using `Record()` or `Tuple.from()`:

```js
const record = Record({ a: 1, b: 2, c: 3 });
const record2 = Record.fromEntries([#["a", 1], #["b", 2], #["c": 3]]); // note that an iterable will also work
const tuple = Tuple.from([1, 2, 3]); // note that an iterable will also work
assert(record === #{ a: 1, b: 2, c: 3 });
assert(tuple === #[1, 2, 3]);
Record.from({ a: {} }); // TypeError: Can't convert Object with a non-const value to Record
Tuple.from([{}, {} , {}]); // TypeError: Can't convert Iterable with a non-const value to Tuple
```

Note that `Record()` and `Tuple.from()` expect collections consisting of Records, Tuples or other primitives (such as Numbers, Strings, etc). Nested object references would cause a TypeError. It's up to the caller to convert inner structures in whatever way is appropriate for the application.

> _Note_: The current draft proposal does not contain recursive conversion routines, only shallow ones. See discussion in [#122](https://github.com/tc39/proposal-record-tuple/issues/122)

`Tuple`, when called, simply throws an error--the Array-like semantics aren't very meaningful/useful when Tuples are immutable.

## Iteration protocol

Like Arrays, Tuples are iterable.

```js
const tuple = #[1, 2];

// output is:
// 1
// 2
for (const o of tuple) { console.log(o); }
```

Similarly to Objects, Records are only iterable in conjunction with APIs like `Object.entries`.

```js
const record = #{ a: 1, b: 2 };

// TypeError: record is not iterable
for (const o of record) { console.log(o); }

// Object.entries can be used to iterate over Records, just like for Objects
// output is:
// a
// b
for (const [key, value] of Object.entries(record)) { console.log(key) }

```

## JSON.stringify

- The behavior of `JSON.stringify(record)` is equivalent to calling `JSON.stringify` on the object resulting from recursively converting the record to an object that contains no records or tuples.
- The behavior of `JSON.stringify(tuple)` is equivalent to calling `JSON.stringify` on the array resulting from recursively converting the record to an object that contains no records or tuples.

```js
JSON.stringify(#{ a: #[1, 2, 3] }); // '{"a":[1,2,3]}'
JSON.stringify(#[true, #{ a: #[1, 2, 3] }]); // '[true,{"a":[1,2,3]}]'
```

## JSON.parseImmutable

We propose to add `JSON.parseImmutable` so we can extract a Record/Tuple type out of a JSON string instead of an Object/Array.

The signature of `JSON.parseImmutable` is identical to `JSON.parse` with the only change being in the return type that is now a Record or a Tuple.

## `Tuple.prototype`

`Tuple` supports instance methods similar to Array with a few changes:

- The mechanics of Tuple and Array methods are a bit different; Array methods generally depend on being able to incrementally modify the Array, and are built for subclassing, neither of which would apply for Tuples.
- Operations which mutate the Array are replaced by operations which return a new, modified Array. Because it has a different signature, there's a different name, e.g., `Tuple.prototype.pushed` in parallel to `Array.prototype.push`.
- We added `Tuple.prototype.with()` that returns a new tuple with a value changed at a given index

The [appendix](./NS-Proto-Appendix.md#tuple-prototype) contains a full description of `Tuple`'s prototype.

## `typeof`

`typeof` identifies Records and Tuples as distinct types:

```js
assert(typeof #{ a: 1 } === "record");
assert(typeof #[1, 2]   === "tuple");
```

## Usage in {`Map`|`Set`|`WeakMap`|`WeakSet`}

It is possible to use a `Record` or `Tuple` as a key in a `Map`, and as a value in a `Set`. When using a `Record` or `Tuple` here, they are compared by value.

It is not possible to use a `Record` or `Tuple` as a key in a `WeakMap` or as a value in a `WeakSet`, because `Records` and `Tuple`s are not `Objects`, and their lifetime is not observable.

### Examples

#### Map

```js
const record1 = #{ a: 1, b: 2 };
const record2 = #{ a: 1, b: 2 };

const map = new Map();
map.set(record1, true);
assert(map.get(record2));
```

#### Set

```js
const record1 = #{ a: 1, b: 2 };
const record2 = #{ a: 1, b: 2 };

const set = new Set();
set.add(record1);
set.add(record2);
assert(set.size === 1);
```

#### WeakMap and WeakSet

```js
const record = #{ a: 1, b: 2 };
const weakMap = new WeakMap();

// TypeError: Can't use a Record as the key in a WeakMap
weakMap.set(record, true);
```

#### WeakSet

```js
const record = #{ a: 1, b: 2 };
const weakSet = new WeakSet();

// TypeError: Can't add a Record to a WeakSet
weakSet.add(record);
```

# Rationale

## Why introduce new primitive types? Why not just use objects in an immutable data structure library?

One core benefit of the Records and Tuples proposal is that they are compared by their contents, not their identity. At the same time, `===` in JavaScript on objects has very clear, consistent semantics: to compare the objects by identity. Making Records and Tuples primitives enables comparison based on their values.

At a high level, the object/primitive distinction helps form a hard line between the deeply immutable, context-free, identity-free world and the world of mutable objects above it. This category split makes the design and mental model clearer.

An alternative to implementing Record and Tuple as primitives would be to use [operator overloading](https://github.com/tc39/proposal-operator-overloading) to achieve a similar result, by implementing an overloaded abstract equality (`==`) operator that deeply compares objects. While this is possible, it doesn't satisfy the full use case, because operator overloading doesn't provide an override for the `===` operator. We want the strict equality (`===`) operator to be a reliable check of "identity" for objects and "observable value" (modulo -0/+0/NaN) for primitive types.

Another option is to perform what is called _interning_: we track globally Record or Tuple objects and if we attempt to create a new one that happens to be identical to an existing Record object, we now reference this existing Record instead of creating a new one. This is essentially what the [polyfill](https://github.com/bloomberg/record-tuple-polyfill) does. We're now equating value and identity. This approach creates problems once we extend that behavior across multiple JavaScript contexts and wouldn't give deep immutability by nature and **it is particularly slow** which would make using Record & Tuple a performance-negative choice.

## Will developers be familiar with this new concept?

Record & Tuple is built to interoperate with objects and arrays well: you can read them exactly the same way as you would do with objects and arrays. The main change lies in the deep immutability and the comparison by value instead of identity.

Developers used to manipulating objects in an immutable manner (such as transforming pieces of Redux state) will be able to continue to do the same manipulations they used to do on objects and arrays, this time, with more guarantees.

We are going to do empirical research through interviews and surveys to figure out if this is working out as we think it does.

## Why are Record & Tuple not based on `.get()`/`.set()` methods like [Immutable.js](https://immutable-js.github.io/immutable-js/)?

If we want to keep access to Record & Tuple similar to Objects and Arrays as described in the previous section, we can't rely on methods to perform that access. Doing so would require us to branch code when trying to create a "generic" function able to take Objects/Arrays/Records/Tuples.

Here is an example function that has support for [Immutable.js](https://immutable-js.github.io/immutable-js/) Records and ordinary objects:

```js
const profileObject = {
    name: "Rick Button",
    githubHandle: "rickbutton",
};
const profileRecord = Immutable.Record({
    name: "Robin Ricard",
    githubHandle: "rricard",
});

function getGithubUrl(profile) {
    if (Immutable.Record.isRecord(profile)) {
        return `https://github.com/${
            profile.get("githubHandle")
        }`;
    }
    return `https://github.com/${
        profile.githubHandle
    }`;
}

console.log(getGithubUrl(profileObject)) // https://github.com/rickbutton
console.log(getGithubUrl(profileRecord)) // https://github.com/rricard
```

This is error-prone as both branches could easily get out of sync over time... We also want to avoid an ecoystem split where libraries choose which structures they want to operate on.

Here is how we would write that function taking Records from this proposal and ordinary objects:

```js
const profileObject = {
  name: "Rick Button",
  githubHandle: "rickbutton",
};
const profileRecord = #{
  name: "Robin Ricard",
  githubHandle: "rricard",
};

function getGithubUrl(profile) {
  return `https://github.com/${
    profile.githubHandle
  }`;
}

console.log(getGithubUrl(profileObject)) // https://github.com/rickbutton
console.log(getGithubUrl(profileRecord)) // https://github.com/rricard
```

This function support both Objects and Records in a single code-path as well as not forcing the consumer to choose which data structures to use.

## Why introduce new syntax? Why not just introduce the Record and Tuple globals?

The proposed syntax significantly improves the ergonomics of using `Record` and `Tuple` in code. For example:

```js
// with the proposed syntax
const record = #{
  a: #{
    foo: "string",
  },
  b: #{
    bar: 123,
  },
  c: #{
    baz: #{
      hello: #[
        1,
        2,
        3,
      ],
    },
  },
};

// with only the Record/Tuple globals
const record = Record({
  a: Record({
    foo: "string",
  }),
  b: Record({
    bar: 123,
  }),
  c: Record({
    baz: Record({
      hello: Tuple(
        1,
        2,
        3,
      ),
    }),
  }),
});
```

The proposed syntax is intended to be simpler and easier to understand, because it is intentionally similar to syntax for object and array literals. This takes advantage of the user's existing familiarity with objects and arrays. Additionally, the second example introduces additional temporary object literals, which adds to complexity of the expression.


## Why **specifically** the #{}/#[] syntax? What about an existing or new keyword?

Using a keyword as a prefix to the standard object/array literal syntax presents issues around
backwards compatibility. Additionally, re-using existing keywords can introduce ambiguity.

ECMAScript defines a set of [_reserved keywords_](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#Keywords) that can be used for future extensions to the language.
Defining a new keyword that is not already reserved is theoretically possible, but requires significant effort to validate
that the new keyword will not likely break backwards compatibility.

Using a reserved keyword makes this process easier, but it is not a perfect solution because there are no reserved keywords
that match the "intent" of the feature, other than `const`. The `const` keyword is also tricky, because it describes
a similar concept (variable reference immutability) while this proposal intends to add new immutable data structures.
While immutability is the common thread between these two features, there has been significant community feedback that
indicates that using `const` in both contexts is undesirable.

Instead of using a keyword, `{| |}` and `[||]` have been suggested as possible alternatives. Currently, the champion group is leaning towards `#[]`/`#{}`, but discussion is ongoing in [#10](https://github.com/tc39/proposal-record-tuple/issues/10).

# FAQ

## What are the performance expectations of those Data Structures?

This proposal in itself does not put any performance guarantees and does not require specific optimizations on the implementers. It is however built in a way that some performance optimizations can be done in most cases if implementers choose to do so.

This proposal is designed to enable classical optimizations for purely functional data structures, including but not limited to:

- Optimizations for making deep equality checks fast:
  - For returning true quickly, intern ("hash-cons") some data structures
  - For returning false quickly, maintain a hash up the tree of the contents of some structures
- Optimizations for manipulating data structures
  - In some cases, reuse existing data structures (e.g., when manipulated with object spread), similar to ropes or typical implementations of functional data structures
  - In other cases, as determined by the engine, use a flat representation like existing JavaScript object implementations

These optimizations are analogous to the way that modern JavaScript engines handle string concatenation, with various different internal types of strings. The validity of these optimizations rests on the unobservability of the identity of records and tuples. It's not expected that all engines will act identically with respect to these optimizations, but rather, they will each make decisions about particular heuristics to use. Before Stage 4 of this proposal, we plan to publish a guide for best practices for cross-engine optimizable use of Records and Tuples, based on the implementation experience that we will have at that point.

## How can I make a Record or Tuple which is based on an existing one, but with one part changed or added?

In general, the spread operator works well for this:

```js
// Add a Record field
let rec = #{ a: 1, x: 5 }
#{ ...rec, b: 2 }  // #{ a: 1, b: 2, x: 5 }

// Change a Record field
#{ ...rec, x: 6 }  // #{ a: 1, x: 6 }

// Append to a Tuple
let tup = #[1, 2, 3];
#[...tup, 4]  // #[1, 2, 3, 4]

// Prepend to a Tuple
#[0, ...tup]  // #[0, 1, 2, 3]
```

And if you're changing something in the middle of a Tuple, the `Tuple.prototype.with` method works:

```js
// Change a Tuple index
let tup = #[1, 2, 3];
tup.with(1, 500)  // #[1, 500, 3]
```

Some manipulations of "deep paths" can be a bit awkward. For that, the [Deep Path Properties for Records](https://github.com/rickbutton/proposal-deep-path-properties-for-record) proposal adds additional shorthand syntax to Record literals.

We are developing the deep path properties proposal as a separate follow-on proposal because we don't see it as core to using Records, which work well independently. It's the kind of syntactic addition which would work well to prototype over time in transpilers, and where we have many decision points which don't have to do with Records and Tuples (e.g., how it works with objects).

## Could I "box" a pointer to an object, and put that in a Record or Tuple?

Yes! Because you can store primitives in a Record or Tuple, you are free to use any primitive value to represent a "reference" for an object, and store said object in a side table. For example, integers or Symbols could be used as these sorts of references.

Additionally, the [Symbols as WeakMap keys](https://github.com/rricard/proposal-symbols-as-weakmap-keys) proposal provides a way of accomplishing this via Symbols and WeakMaps. Using Symbols as WeakMap keys, you will be able to reference objects in your code in the following way without leaking memory:

```js
import { ref, deref } from "ref-bookkeeper.js";

const server = #{
    port: 8080,
    handler: ref(function handler(req) { /* ... */ }),
};
deref(server.handler)({ /* ... */ });
```
<details>
<summary><code>ref-bookkeeper.js</code></summary>

```js
const references = new WeakMap();

export function ref(obj) {
  // (Simplified; we may want to return an existing symbol if it's already there)
  const sym = Symbol();
  references.set(sym, obj);
  return sym;
}

export function deref(sym) { return references.get(sym); }
```

</details>

## How does this relate to the [Readonly Collections](https://github.com/tc39/proposal-readonly-collections) proposal?

We've talked with the Readonly Collections champions, and both groups agree that these are complements:
- Readonly collections are shallowly immutable and may point to objects; they may be mutated during construction, and read-only views of mutating objects are supported.
- Records and Tuples are deeply immutable and consist just of primitives.

Neither one is a subset of the other in terms of functionality. At best, they are parallel, just like each proposal is parallel to other collection types in the language.

So, the two champion groups have resolved to ensure that the proposals are in parallel *with respect to each other*. For example, this proposal adds a new `Tuple.prototype.pushed` method. The idea would be to check, during the design process, if this signature would also make sense for read-only Arrays (if those exist), so we're designing an API which builds a consistent, shared mental model.

In the current proposal drafts, there aren't any overlapping types for the same kind of data, but both proposals could grow in these directions in the future, and we're trying to think these things through ahead of time. Who knows, some day TC39 could decide to add primitive RecordMap and RecordSet types, as the deeply immutable versions of [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set) and [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)! And these would be in parallel with Readonly Collections types.

## Could we have classes whose instances are Records?

TC39 has been long discussing "value types", which would be some kind of class declaration for a primitive type, for several years on and off. An [earlier version of this proposal](./history/with-classes.md) even made an attempt. This proposal tries to start off simple and minimal, providing just the core structures. The hope is that it could provide the data model for a future proposal for classes.

This proposal is loosely related to a broader set of proposals, including [operator overloading](https://github.com/littledan/proposal-operator-overloading/) and [extended numeric literals](https://github.com/tc39/proposal-extended-numeric-literals): These all conspire to provide a way for user-defined types to do the same as [BigInt](https://github.com/tc39/proposal-bigint). However, the idea is to add these features if we determine they're independently motivated.

If we had user-defined primtiive/value types, then it could make sense to use them in built-in features, such as [CSS Typed OM](https://developer.mozilla.org/en-US/docs/Web/API/CSS_Typed_OM_API/Guide) or the [Temporal Proposal](https://github.com/tc39/proposal-temporal). However, this is far in the future, if it ever happens; for now, it works well to use objects for these sorts of features.

## What's the relationship between this proposal's Record & Tuple and TypeScript's [Record](https://www.typescriptlang.org/docs/handbook/utility-types.html#recordkt) & [Tuple](https://www.typescriptlang.org/docs/handbook/basic-types.html#tuple)?

Although both kinds of Records relate to Objects, and both kinds of Tuples relate to Arrays, that's about where the similarity ends.

Records in Typescript are a generic utility type to represent an object taking a key type matching with a value type. They still represent objects.

Likewise Tuples in Typescript are a notation to express types in an array of a limited size (starting with TypeScript 4.0 they have a [variadic form](https://github.com/microsoft/TypeScript/pull/39094)). Tuples in TypeScript are a way to express arrays with heterogeneous types. ECMAScript tuples can correspond to TS arrays or TS tuples easily as they can either contain an indefinite number of values of the same type or contain a limited number of values with different types.

TS Records or Tuples are orthogonal features to ECMAScript Records and Tuples and both could be expressed at the same time:

```ts
const record: readonly Record<string, number> = #{
  foo: 1,
  bar: 2,
};
const tuple:  readonly [number, string] = #[1, "foo"];
```

# Glossary

#### Record

A new, deeply immutable, compound primitive type data structure, proposed in this document, that is analogous to Objects. `#{ a: 1, b: 2 }`

#### Tuple

A new, deeply immutable, compound primitive type data structure, proposed in this document, that is analogous to Arrays. `#[1, 2, 3, 4]`

#### Compound primitive types

Values which act like other JavaScript primitives, but are composed of other constituent values. This document proposes the first two compound primitive types: `Record` and `Tuple`.

#### Simple primitive types

`String`, `Number`, `Boolean`, `undefined`, `null`, `Symbol` and `BigInt`

### Primitive types

Things which are either compound or simple primitive types. All primitives in JavaScript share certain properties:
- They are deeply immutable
- Comparison is by value, not by identity
- They are not objects in the object model--object operations lead to exceptions or implicit wrapper creation.

#### Immutable Data Structure

A Data Structure that doesn't accept operations that change it internally, it has operations that return a new value that is the result of applying that operation on it.

In this proposal `Record` and `Tuple` are deeply immutable data structures.

#### Strict Equality

The operator `===` is defined with the [Strict Equality Comparison](https://tc39.es/ecma262/#sec-strict-equality-comparison) algorithm. Strict Equality refers to this particular notion of equality.

#### Structural Sharing

Structural sharing is a technique used to limit the memory footprint of immutable data structures. In a nutshell, when applying an operation to derive a new version of an immutable structure, structural sharing will attempt to keep most of the internal structure intact and used by both the old and derived versions of that structure. This greatly limits the amount to copy to derive the new structure.
