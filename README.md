# javascript_and_dom_notes

Just some notes of stuff I forgot, never knew, are important to be reminded of or never fully understood about javascript.

Notable source with very good and terse explanations, examples and exercises: https://javascript.info/

Most examples here are from that site.

## Basics

### use strict

"use strict" is enabled implicitly / automatically in any code using classes or modules.

### 1/0 doesn't throw an error

1/0 = Infinity, not an error or NaN

### NaN is (almost) always sticky

Every mathematical operation involving a NaN value results in NaN EXCEPT NaN**0 which is 1

### `typeof` null is object

Just a mistake in javascript that never got fixed for backward compatibility

### Operators

#### `=` is an operator and thus returns a value

```js
let a = 1;
let b = 2;

let c = 3 - (a = b + 1);

alert( a ); // 3
alert( c ); // 0
```

#### prefix / postfix operators

- `+x` is the same as `Number(x)`
- `b = ++a` increments a and then assigns its value to b
- `b = a++` assigns a to b and THEN increments a's value

### Boolean doesn't do implicit type conversion

```js
let a = 0;
alert( Boolean(a) ); // false

let b = "0";
alert( Boolean(b) ); // true

alert(a == b); // true!
```

### "a" > "p" is false, "a" > "A" is true

javascript compares string values by their index in the unicode dictionary

### Any non-empty string is evaluated as true in the logical context (no conversion of "0" to 0 to false)

```js
if ("0") {
  alert( 'Yes, this alert is shown' );
}
```

The same goes for the `switch` statement.

### `??` and `||`

- `||` checks the truthiness of the value
- `??` checks if the value is defined (only `undefined` returns false)

```js
let height = 0;

alert(height || 100); // 100
alert(height ?? 100); // 0
```

- `||` and `??` have a low [precedence](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence#table) of 3, meaning they are applied after mathematical and other operations

```js
let height = null;
let width = null;

// important: use parentheses
let area = (height ?? 100) * (width ?? 50);
```

### `break` and `continue` are syntax constructs and can't be used as expressions

`(i > 5) ? alert(i) : continue; // continue isn't allowed here`

Gets caught as a syntax error though.

### Loops can have labels for use with `break`/`continue`

```js
outer: for (let i = 0; i < 3; i++) {

  for (let j = 0; j < 3; j++) {

    let input = prompt(`Value at coords (${i},${j})`, '');

    // if an empty string or canceled, then break out of both loops
    if (!input) break outer; 

    // ...
  }
}
```

### Optional chaining using `?.`

- Optional chaining (e.g. `user?.name?.firstName`) can be used with functions and properties, too


  - `obj?.prop` – returns obj.prop if obj exists, otherwise undefined.
  - `obj?.[prop]` – returns obj[prop] if obj exists, otherwise undefined.
  - `obj.method?.()` – calls obj.method() if obj.method exists, otherwise returns undefined.


```js
let userAdmin = {
  admin() {
    alert("I am admin");
  }
};

let userGuest = {};

userAdmin.admin?.(); // I am admin

userGuest.admin?.(); // nothing happens (no such method)
```

```js
let key = "firstName";

let user1 = {
  firstName: "John"
};

let user2 = null;

alert( user1?.[key] ); // John
alert( user2?.[key] ); // undefined
```

```js
delete user?.name; // delete user.name if user exists
```

## Debugging

- `step` continues execution with the next command and steps into the function code if the command is a function call, but ignores async actions
- `step into` goes also into async functions and waits for them if necessary (https://developer.chrome.com/blog/new-in-devtools-65/#async)

## Functions

### General

- `functionName.toString()` returns the function code
- as functions are just values, everything else is a logical consequence
    - block scoped (except in in non-strict mode)
    - function expressions (`blah = function() {...}`) must be defined before they are called, unlike function declarations which are defined before the execution of the script `function blah() {...}`

### Functions as constructors

- a function can check if it was called normally or with `new`:

```js
function User() {
  alert(new.target);
}

// without "new":
User(); // undefined

// with "new":
new User(); // function User { ... }
```

### Other

- max. reliable recursion depth is 10.000, in practice most engines support up to 100.000
- `arguments` is an array-like variable that contains all arguments given to the function
- arrow functions don't have an `arguments` variable
- arguments collected with the spread syntax (`function sum(...nums)`) IS an array
- ever function is an object and has a `.name` property that even gets smartly assigned when using e.g. `const myFunc = function(){}` (`myFunc.name === 'sayHi`); also works with methods. Doesn't work on e.g. `let arr = [function() {}];` (`arr[0].name === ""`)
  - this can get automatically re-assigned when assigning to a different variable!

```js
let sayHi = function(who) {
  if (who) {
    alert(`Hello, ${who}`);
  } else {
    sayHi("Guest"); // Error: sayHi is not a function
  }
};

let welcome = sayHi;
sayHi = null;

welcome(); // Error, the nested sayHi call doesn't work any more!
```
- function expressions can be assigned a name to work around the problem above

```js
let sayHi = function func(who) {
  if (who) {
    alert(`Hello, ${who}`);
  } else {
    func("Guest"); // use func to re-call itself
  }
};

sayHi(); // Hello, Guest

// But this won't work:
func(); // Error, func is not defined (not visible outside of the function)
```

- functions can be declarated using `new`, e.g. for receiving function code from a server:

```js
let sum = new Function('a', 'b', 'return a + b');
```

- the lexical environment of a function declared using `new` is always the global scope

- `.length` contains the number of function parameters; it is 0 for spread arguments (`function s(...args) {}; s.length === 0`)

Use case: callbacks with optional parameter:

```js
function ask(question, ...handlers) {
  let isYes = confirm(question);

  for(let handler of handlers) {
    if (handler.length == 0) {
      if (isYes) handler();
    } else {
      handler(isYes);
    }
  }

}
```
- arbitrary properties can be defined for functions because they are just (somewhat special) objects; unlike arrays, this doesn't have detrimental effects

- arrow functions recap:
  - don't create a context (`this`) but take it from the surrounding scope
  - can't be used with `new`
  - have no automatic `arguments` variable


## Objects

### Creating objects

- Literal (`const o = {}`) creates an object with `Object` as its prototype
- `Object.create(null)` creates an object without any prototype (and thus no toString), but most static methods of `Object` still work, e.g. `Object.keys(obj)`; These objects w/o a prototype are useful for using them as associative arrays without having to check if a key would overwrite a property of the prototype


### Properties & property order

- numbers and keywords can be used as property names (but imo shouldn't); `{ 0: "somevalue"}` -> 0 will be automatically converted to string
- `"test" in obj` returns true if the property exists in the object, even if it's value is set to "undefined"; it doesn't return symbol properties (see below)
- when iterating using `for(property in object)`, integer properties are sorted ascending, otherwise, otherwise in creation order

```js
let codesAndOtherStuff={"49": "germany", "41": "switzerland". 1: "USA", "none": "nada"}
for(prop in codesAndOtherStuff) { console.log(prop)} // 1, 41, 49, none

// avoid integer sorting:

let codes = {
  "+49": "Germany",
  "+41": "Switzerland",
  "+44": "Great Britain",
  // ..,
  "+1": "USA"
};

for (let code in codes) {
  alert( +code ); // 49, 41, 44, 1
}
```

### Cloning

- deep cloning can be done with `const targetObj = structuredClone(srcObj)`, but *doesn't support function properties!*

### Symbols

#### Unique symbols

Only strings and symbols can be used as object property keys (numbers get converted to string automatically).

- A symbol represents a (guaranteedd) unique identifier
- The label used when creating a symbol doesn't affect anything and is just a label, meaning there can be multiple unique symbols that have the same ID
- symbols don't auto-convert to strings

```js
let id = Symbol("id");
alert(id); // TypeError: Cannot convert a Symbol value to a string

let id = Symbol("id");
alert(id.toString()); // Symbol(id), now it works

let id = Symbol("id");
alert(id.description); // id
```

Because symbols are guaranteed to be unique, it is safe to add them to existing objects, ensuring that no existing property is overwritten

```js
let id = Symbol("id");

user[id] = "Their id value";
```

In object literals, square brackets must be added (obviously):

```js
let id = Symbol("id");

let user = {
  name: "John",
  [id]: 123 // not "id": 123
};
```

- Symbols are not returned in `for...in` loops or `Object.keys(obj)`.
- `Object.assign` copies symbol properties as well.

#### creating global symbols in the symbol registry

There is a global symbol registry so we can create and access global symbols by a label:

```js
// read from the global registry
let id = Symbol.for("id"); // if the symbol did not exist, it is created

// read it again (maybe from another part of the code)
let idAgain = Symbol.for("id");

// the same symbol
alert( id === idAgain ); // true

alert(Symbol.keyFor(id)); // "id"
```

**This only works for symbols explicitely created in the global registry** using `Symbol.for`, not normally created symbols using e.g. `Symbol("id")`

#### In-built symbols and using them for object conversion

[There are a number of in-build system symbols.](https://tc39.github.io/ecma262/#sec-well-known-symbols)


```js
let user = {
  name: "John",
  money: 1000,

  [Symbol.toPrimitive](hint) {
    alert(`hint: ${hint}`);
    return hint == "string" ? `{name: "${this.name}"}` : this.money;
  }
};

// conversions demo:
alert(user); // hint: string -> {name: "John"}
alert(+user); // hint: number -> 1000
alert(user + 500); // hint: default -> 1500
```

If there’s no `Symbol.toPrimitive` then JavaScript tries to find methods `toString` and `valueOf`:


```js
let user = {
  name: "John",
  money: 1000,

  // for hint="string"
  toString() {
    return `{name: "${this.name}"}`;
  },

  // for hint="number" or "default"
  valueOf() {
    return this.money;
  }

};

alert(user); // toString -> {name: "John"}
alert(+user); // valueOf -> 1000
alert(user + 500); // valueOf -> 1500
```

### Property flags and descriptors

All object properties have configuration flags:


- writable – if true, the value can be changed, otherwise it’s read-only.
- enumerable – if true, then listed in loops, otherwise not listed.
- configurable – if true, the property can be deleted and these attributes can be modified, otherwise not.

All default to `true` for created properties.

The descriptor is an object with 4 properties.

Read:

```js
let user = {
  name: "John"
};

let descriptor = Object.getOwnPropertyDescriptor(user, 'name');

alert( JSON.stringify(descriptor, null, 2 ) );
/* property descriptor:
{
  "value": "John",
  "writable": true,
  "enumerable": true,
  "configurable": true
}
*/
```

Write:

```js
// all property descriptors not defined in the descriptor object are set to false
Object.defineProperty(user, "name", {
  value: "John",
  writable: false,
});

// multiple:
Object.defineProperties(user, {
  name: { value: "John", writable: false },
  surname: { value: "Smith", writable: false },
  // ...
});
```

`Object.defineProperty` can be used to create a new property on an object or update the descriptor object on an existing one.

Related methods:

`Object.preventExtensions(obj)`, `Object.seal(obj)`, `Object.freeze(obj)`, `Object.isExtensible(obj)`, `Object.isSealed(obj)`, `Object.isFrozen(obj)`

### Getters and Setters

In classes and objects, getters and setters can be defined using `get` and `set`:

```js
let user = {
  name: "John",
  surname: "Smith",

  get fullName() {
    return `${this.name} ${this.surname}`;
  },

  set fullName(value) {
    [this.name, this.surname] = value.split(" ");
  }
};

// set fullName is executed with the given value.
user.fullName = "Alice Cooper";

alert(user.name); // Alice
alert(user.surname); // Cooper
```

Getter and setter properties have their own property descriptor object that is different from normal properties:

```js
let user = {
  name: "John",
  surname: "Smith"
};

Object.defineProperty(user, 'fullName', {
  get() {
    return `${this.name} ${this.surname}`;
  },

  set(value) {
    // custom logic can be added here to e.g. throw an Error
    [this.name, this.surname] = value.split(" ");
  },
  //enumerable – same as for data properties,
  //configurable – same as for data properties.
});
```

## Data types

### Primitives as objects

- To allow methods to be called on (some) primitives (string, number, bigint, boolean, symbol, null and undefined), javascript creates a temporary object that provides fitting methods for the given type and destroys the object afterwards (if it really does this in all cases or uses a different way  depends on the internals of the JS engine)
- there exist no wrapper objects or methods for `null` and `undefined`
- Never use constructors for primitives (e.g. `new String("abc")`)
- Using the functions without `new`, e.g. `let n = Number("123")` is fine of course as it just creates a primitive value

### Numbers

#### Defining numbers

```js
let billion = 1000000000;
let billion = 1_000_000_000;
// e multiplies the number by 1 with the given zeroes count.
1e3 === 1 * 1000; // e3 means *1000
1.23e6 === 1.23 * 1000000; // e6 means *1000000
// a negative number after "e" means a division by 1 with the given number of zeroes:
// -3 divides by 1 with 3 zeroes
1e-3 === 1 / 1000; // 0.001

// -6 divides by 1 with 6 zeroes
1.23e-6 === 1.23 / 1000000; // 0.00000123

// an example with a bigger number
1234e-2 === 1234 / 100; // 12.34, decimal point moves 2 times

// hex, bin, octal notation (they are still just numbers)
alert( 0xff ); // 255
alert( 0xFF ); // 255 (the same, case doesn't matter)
let a = 0b11111111; // binary form of 255
let b = 0o377; // octal form of 255

// converting to string with a given base
let num = 255;

num.toString(16);  // ff
num.toString(2);   // 11111111

// using literal numbers
// 2 dots because JS implies that the first dot is the start of the decimal part. 
// With the following second dot, it interprets the decimal part as empty
123456..toString(36) // 2n9c
// same as 
(123456).toString(36)
```

- `.toFixed` also rounds!
```js
let num = 12.36;
num.toFixed(1); // "12.4"
```

#### Number() vs `+` vs parseInt / parseFloat

- `Number("123")` and `+"123"` does the same thing; trims spaces automatically, so `Number(" 123 ")` is ok
- `parseInt` and `parseFloat` reads a number from a string until it can't be parsed as a number anymore; trims whitespace; base (radix) can be added as a second argument

```js
parseInt('a123'); // NaN, the first symbol stops the process

parseInt('100px'); // 100
parseFloat('12.5em'); // 12.5

parseInt('12.3'); // 12, only the integer part is returned
parseFloat('12.3.4'); // 12.3, the second point stops the reading

parseInt('0xff', 16); // 255
parseInt('ff', 16); // 255, without 0x also works

```

### Strings

- I always forget that `.at` exists, which allows negative positions when accessing string indices like in python, e.g. `"abc".at(-1) === 'c'`
- same goes for `slice` and `splice`
- `includes`, `startsWith` and `endsWith` are a thing now
- `substr` and `substring` are not the same!
  - slice(start, end) 	from start to end (not including end) 	allows negatives
  - substring(start, end) 	between start and end (not including end) 	negative values mean 0
  - substr(start, length) 	from start get length characters 	allows negative start
- `localCompare` takes language rules into account (e.g. Ö and O are in the correct order when in a german context)

Noteworthy new methods:

- `repeat(numOfRepetitions)`

### Arrays

#### General

- `someArray.push()` accepts multiple values and thus can also be used with the spread operator (`someArray.push(...someOtherArray)`)
- `someArray.length = 0` as opposed to `someArray = []` is a good practice when you want to clear an array but maintain its reference across the application. It is also beneficial in terms of performance and memory efficiency.
- recently supports `at` method which allows negative indices
- `splice` works in-place 
- `concat` allows mixed arguments: ` [1,2].concat([3, 4], 5, 6); // 1,2,3,4,5,6`
- we can make non-array objects behave like arrays when passed to concat:

```js
let arr = [1, 2];

let arrayLike = {
  0: "something",
  1: "else",
  [Symbol.isConcatSpreadable]: true,
  length: 2
};

alert( arr.concat(arrayLike) ); // 1,2,something,else
```

- `sort` sorts an array in-place **in alphabetical order, even if the values are numbers**
- `.toSorted` returns a new, sorted array (ES2023)
  - sort numerically using `arr.sort( (a, b) => a - b )`
  - use `localCompare` when needed: `countries.sort( (a, b) => a.localeCompare(b)); `
- most methods that accept callbacks also accept `thisArg`:

```js
let army = {
  minAge: 18,
  maxAge: 27,
  canJoin(user) {
    return user.age >= this.minAge && user.age < this.maxAge;
  }
};

let users = [
  {age: 16},
  {age: 20},
  {age: 23},
  {age: 30}
];

// find users, for who army.canJoin returns true
let soldiers = users.filter(army.canJoin, army);
```
- using `delete` on an array element sets the element to undefined but doesn't remove it (use `splice` to remove and re-index)
- `.join()` defaults to `,` as the separator, not ""!

#### Noteworthy new (ES2023) methods of arrays

- `toReversed`
- `toSorted`
- `toSpliced`
- `add`
- `with(index, value)` (updating an index and returning a new array without modifying the original array); only [typedArray](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray), 

#### Performance tips 

- **Arrays are objects, but if we "misuse" them, javascript turns off optimizations meant for arrays!**
  - Misuse: Add a non-numeric property like arr.test = 5.
  - Misuse: Make holes, like: add arr[0] and then arr[1000] (and nothing between them).
  - Misuse: Fill the array in the reverse order, like arr[1000], arr[999] and so on.
- As in all languages, `push` and `pop` are faster than `shift` and `undshift` because elements don't need to be renumbered
- Don't iterate over keys using `for...in` as it's slower and returns ALL properties of the array object! Always use `for...of` with arrays to iterate over values (or use a normal loop with indices)

### Iterables

- Iterable objects can be traversed using `for...of`
- To make an object iterable, implement a method called `Symbol.iterator`
- The **result** of `obj[Symbol.iterator]` is called an iterator
- The iterator is a separate object from the object iterated over
- The iterator must have a method called `next()` that returns an object with the properties `done` (boolean) and `value` (any=)


```js
let range = {
  from: 1,
  to: 5
};

// 1. call to for..of initially calls this
range[Symbol.iterator] = function() {

  // ...it returns the iterator object:
  // 2. Onward, for..of works only with the iterator object below, asking it for next values
  return {
    current: this.from,
    last: this.to,

    // 3. next() is called on each iteration by the for..of loop
    next() {
      // 4. it should return the value as an object {done:.., value :...}
      if (this.current <= this.last) {
        return { done: false, value: this.current++ };
      } else {
        return { done: true };
      }
    }
  };
};

// now it works!
for (let num of range) {
  alert(num); // 1, then 2, 3, 4, 5
}
```

Using a generator function:

```js
let range = {
  from: 1,
  to: 5,

  *[Symbol.iterator]() { // a shorthand for [Symbol.iterator]: function*()
    for(let value = this.from; value <= this.to; value++) {
      yield value;
    }
  }
};
```

Iterators can be asynchronous by using `[Symbol.asyncIterator]` and `for await(... of ...)`:

```js
let range = {
  from: 1,
  to: 5,

  [Symbol.asyncIterator]() { // (1)
    return {
      current: this.from,
      last: this.to,

      async next() { // (2)

        // note: we can use "await" inside the async next:
        await new Promise(resolve => setTimeout(resolve, 1000)); // (3)

        if (this.current <= this.last) {
          return { done: false, value: this.current++ };
        } else {
          return { done: true };
        }
      }
    };
  }
};

(async () => {

  for await (let value of range) { // (4)
    alert(value); // 1,2,3,4,5
  }

})()
```

Note: The spread syntax does not work with async iterators!

Async iteration can also be achieved using generators:

```js
async function* fetchCommits(repo) {
  let url = `https://api.github.com/repos/${repo}/commits`;

  while (url) {
    const response = await fetch(url, { // (1)
      headers: {'User-Agent': 'Our script'}, // github needs any user-agent header
    });

    const body = await response.json(); // (2) response is JSON (array of commits)

    // (3) the URL of the next page is in the headers, extract it
    const link = response.headers.get('Link');
    if(link) {
    	let nextPage = link.match(/<(.*?)>; rel="next"/);
    	nextPage = nextPage?.[1];
      url = nextPage;
    } else {
      url = null;
    }
    

    for(let commit of body) { // (4) yield commits one by one, until the page ends
      yield commit;
    }
  }
}

(async () => {

  for await (let commit of fetchCommits("pkro/javascript_and_dom_notes")) {
   console.log(commit);
  }

})();

```

### Map and Set

- `Map` allows objects as keys
- `Set` returns the map and thus allows method chaining
- `Object.entries(obj)` returns a map with key / value pairs
- `Object.fromEntries(map)` returns an object from a map

### WeakMap and WeakSet

- keys must be objects
- key objects that don't have any reference anymore are garbage collected
- no `keys`, `values` or `entries` methods or  `size` property, only set / get / delete / has

```js
const m = new WeakMap();
m.set({name: "Paul"}, "some value"); // is "immediately" garbage collected, no reference to the inline object
let person = {name: "Paul"};
m.set(person, "another value");
m.get(person); // "another value"
person = null;
m.get(person); // undefined
```

WeakMap Example usage:

Resource tracking

```js
let resourceUsageTracker = new WeakMap();

function trackResourceUsage(obj, usageData) {
    resourceUsageTracker.set(obj, usageData);
}

function getResourceUsage(obj) {
    return resourceUsageTracker.get(obj) || "No data";
}

let resource = { name: "Resource1" };
trackResourceUsage(resource, { memoryUsage: "100MB", cpuUsage: "10%" });

// ... later in your code
console.log(getResourceUsage(resource)); // Outputs the usage data

// Once 'resource' is no longer referenced elsewhere, it and its associated data in resourceUsageTracker can be garbage collected.

```

WeakMap example usage:

```js
let activeConnections = new WeakSet();

function addConnection(connection) {
    activeConnections.add(connection);
}

function isConnectionActive(connection) {
    return activeConnections.has(connection);
}

let connection1 = { id: 1, info: "Connection 1" };
addConnection(connection1);

// ... later in your code
console.log(isConnectionActive(connection1)); // Outputs true

// Once 'connection1' is no longer referenced elsewhere, it can be garbage collected, and it will automatically be removed from activeConnections.
```

### Date and time

- Dates can have negative timestamps

```js
// 31 Dec 1969
let Dec31_1969 = new Date(-24 * 3600 * 1000);
alert( Dec31_1969 );
```


### JSON

- `JSON.stringify` can encode also booleans and strings (adds `"` to a string)
- can't do circular references obviously
- we can pass an array or function as a second argument (replacer) to specify which properties should be serialized
- if an object has a `toJSON()` method defined, it is used by `JSON.stringify`
- `JSON.parse` has a second argument "reviver" that is a `function(key, value)` that can be used to transform values:

```js
let str = '{"title":"Conference","date":"2017-11-30T12:00:00.000Z"}';

let meetup = JSON.parse(str, function(key, value) {
  if (key == 'date') return new Date(value);
  return value;
});
```

## Scope and closures

Just an illustration of Javascript's (theoretical) internal Environment object:

```js
class MyClass {
    private memberVariable: string;

    constructor() {
        this.memberVariable = "Hello";
    }

    public outerFunction() {
        let outerVariable = "World";

        const innerFunction = () => {
            let innerVariable = "!";
            console.log(this.memberVariable + " " + outerVariable + innerVariable);
        }

        innerFunction();
    }
}

const myObject = new MyClass();
myObject.outerFunction();

/*
Global Environment
|
|-- MyClass (class)
|   |
|   |-- constructor
|   |   |-- this.memberVariable (Hello)
|   |   |-- [Reference to Global Environment]
|   |
|   |-- outerFunction (method)
|       |
|       |-- outerVariable (World)
|       |-- [Reference to MyClass Environment]
|       |
|       |-- innerFunction (function)
|           |
|           |-- innerVariable (!)
|           |-- this (reference to MyClass instance)
|           |-- [Reference to outerFunction Environment]

*/
```

This also explains how closures work: if a function is returned that references a variable of an outer scope, the outer scope isn't garbage collected and thus the variable still exists (the engines can do other optimizations such as gc'ing unneeded variables in the scope)

## setInterval / setTimeout

- both accept a string as the function argument and creates a function from it automatically (`setTimeout("alert('Hello')", 1000);`); not recommended
- neste setTimeouts are more flexible than setInterval

```js
let delay = 5000;

let timerId = setTimeout(function request() {
  ...send request...

  if (request failed due to server overload) {
    // increase the interval to the next run
    delay *= 2;
  }

  timerId = setTimeout(request, delay);

}, delay);
```
- Always do `clearInterval` when not needed anymore because otherwise it (and possibly the outer scope(s) in case of referenced variables) will otherwise be kept in memory

## call / apply / memoization / method borrowing / decorators

### call and apply

`context` can be thought of `thisArg`

- `someFunction.call(context, arg1, arg2, ...rest)`; using spread syntax, we can pass all iterables as arguments, e.g. `someFunc.call(this, ...someIterable)`
- `someFunction.apply(context, argArray)` - mnemonic: **a**pply starts with **a** like **a**rray; second argument must be array-like (not just iterable)

### memoization

To be able to use memoization decorators for object methods that rely on the correct `this`, e.g. to call other methods, we can use `someFunction.call` and explicitely pass `this`, which points to the function that `call` is called on:

```js
let worker = {
  someMethod() {
    return 1;
  },

  slow(x) {
    alert("Called with " + x);
    return x * this.someMethod(); // (*)
  }
};

function cachingDecorator(func) {
  let cache = new Map();
  return function(x) {
    if (cache.has(x)) {
      return cache.get(x);
    }
    let result = func.call(this, x); // "this" is passed correctly now - "this" is "func"
    cache.set(x, result);
    return result;
  };
}
```

### bind

- `func.bind(context, ...args)` can be used to create partials

```js
function mul(a, b) {
  return a * b;
}

let double = mul.bind(null, 2);

alert( double(3) ); // = mul(2, 3) = 6
alert( double(4) ); // = mul(2, 4) = 8
alert( double(5) ); // = mul(2, 5) = 10
```

- simple partials for functions not relying on `this` can be just done with arrow functions:

```js
let double = (b) => mul(2, b);
```

- a function can't be re-bound

```js
f = f.bind( {name: "John"} ).bind( {name: "Pete"} );

f(); // John
```
- the result of `bind` is a new function object; if there were additional properties added to the original function, they won't exist in the newly bound function


### Method borrowing

We can use call and apply to "borrow" methods

```js
function hash() {
  // remember, the automatic arguments variable is not a real array
  alert( arguments.join() ); // Error: arguments.join is not a function
}

hash(1, 2);

// solution: borrow "join" from an array created on the fly and use call to pass it the arguments array-like (run it in the context of "arguments" as "this")
function hash() {
  alert( [].join.call(arguments) ); // 1,2
}

hash(1, 2);

```

### Some useful decorators (from exercises)

```js
function debounce(func, ms) {
  let time = new Date();
  let to;
  return function() {
    if((new Date()) - time < ms) {
        clearTimeout(to);
        time = new Date();
      }
    to = setTimeout(()=>{
        func.call(this, ...arguments);
    }, ms);
  }
}
```

## Prototypes

### `[[Prototype]]` and `__proto__`

- `[[Prototype]]` is a hidden object property bein either `null` or referencing another object
- `__proto__` is a getter / setter property to set `[[Prototype]]`
  - `Object.getPrototypeOf` / `Object.setPrototypeOf` can (and should?) also be used for that
- `this` references always the current object (not affected by prototype chain):


```js
let animal = {
  walk() {
    if (!this.isSleeping) {
      alert(`I walk`);
    }
  },
  sleep() {
    this.isSleeping = true; // creates a new property isSleeping
  }
};

let rabbit = {
  name: "White Rabbit",
  __proto__: animal
};

// modifies rabbit.isSleeping
rabbit.sleep(); // creates a new property on THE CURRENT OBJECT (rabbit), not the prototype

alert(rabbit.isSleeping); // true
alert(animal.isSleeping); // undefined (no such property in the prototype)
```
- `for ... in` iterates over inherited properties, `Object.keys` (and related methods such as `.entries()` or `.values` only over own properties:

```js
let animal = {
  eats: true
};

let rabbit = {
  jumps: true,
  __proto__: animal
};

// Object.keys only returns own keys
alert(Object.keys(rabbit)); // jumps

// for..in loops over both own and inherited keys
for(let prop in rabbit) alert(prop); // jumps, then eats
```
- use `obj.hasOwnProperty(prop)` to filter out prototype properties in for...in loops

### `.prototype`

To create some confusion, there's a "normal" `.prototype` property that can be created and is used to specify the prototype on an object created with `new`.

Functions have a default prototype that points to an object with the only property `constructor` which points to itself:

```js
function Rabbit() {}
// by default:
// Rabbit.prototype = { constructor: Rabbit }

alert( Rabbit.prototype.constructor == Rabbit ); // true
```

If we want to use the same constructor as an existing object (e.g. from a library where we don't know the constructor):

```js
function Rabbit(name) {
  this.name = name;
  alert(name);
}

let rabbit = new Rabbit("White Rabbit");

let rabbit2 = new rabbit.constructor("Black Rabbit");
```

### Borrowing from prototypes

```js
let obj = {
  0: "Hello",
  1: "world!",
  length: 2,
};

// works because the built-in join method only cares about the correct indexes and the length property
obj.join = Array.prototype.join;

alert( obj.join(',') ); // Hello,world!
```

## Classes

### Basics

Basic class syntax

```js
class MyClass {
  prop = value; // property

  constructor(...) { // constructor
    // ...
  }

  method(...) {} // method

  get something(...) {} // getter method
  set something(...) {} // setter method

  [Symbol.iterator]() {} // method with computed name (symbol here)
  // ...
}```

- `MyClass` is technically a function (the one that we provide as constructor), while methods, getters and setters are written to MyClass.prototype.
- can be passed around and created on the fly (e.g. `return class Blah {...}`)

```js
class User {
  name = "Joe"; // class fields - they exist only on the instantiated object!
  constructor(name) { 
    this._name = name; 
    this.somethingelse = "class fields can be created on the fly (NOT in typescript though)";
  } // NO comma between methods, like in literal objects

  sayHi() { 
    alert(this._name); 
  }

  // if an object method is passed around, e.g. setTimeout(Joe.sayHi, 1000),
  // it loses its "this" context!
  // avoid with arrow functions (beside binding or using an arrow function in setTimeout)
  // this function is created on a per-objet basis, meaning a new function is created for each
  // new object
  sayHello = () => alert(this._name); 

  get name() { // allow accessor method notation like in objects
    return this._name;
  }

  ["some"+"Method"]() {
    return "Computed method names are a-ok";
  }
}

```

### Inheritance

- `extends` can be used with expressions:

```js
function f(phrase) {
  return class {
    sayHi() { alert(phrase); }
  };
}

class User extends f("Hello") {}

new User().sayHi(); // Hello
```

- Base class and its methods is availabe with `super` except for arrow functions which have no `super`
- if no constructor is defined in the subclass, a constructor is "generated": `constructor(...args) { super(...args); }`
- if the constructor is overridden, `super()` must be called before referencing `this`
- if an overridden **field** variable is used in the **parent constructor**, it uses the value defined in the parent class:

```js
class Animal {
  name = 'animal';

  constructor() {
    alert(this.name); // (*)
  }
}

class Rabbit extends Animal {
  name = 'rabbit';
}

new Animal(); // animal
new Rabbit(); // animal
```

### static methods and properties

- not availabe on individual objects

factory methods:

```js
class Article {
  static someStaticProperty = "hey";

  static createArticle() {
    return new this();
  }
}
```

Static declaration is the same as doing

```js
MyClass.property = ...
MyClass.method = ...
```

### Private fields

- no language level way to enforce protected (available in the parent and subclasses) fields and methods
  - workaround: specify getter function but no setter; prefix private variables with "_" as a convention (not enforced)
- not supported everywhere yet, but will be enforced on language level: prefix **private** fields an methods with `#`
  - these will not be inherited / accessible in derived classes
  - private fields are not available as `this[fieldName]`, even in the base class where it's defined
- public / private / protected keywords are supported **in typescript**

Javascript example:

```js
class ExampleClass {
    #privateField;
    _protectedField;

    constructor() {
        this.#privateField = 'Private';
        this._protectedField = 'Protected';
    }

    #privateMethod() {
        console.log('This is a private method');
    }

    _protectedMethod() {
        console.log('This is a protected method');
    }
}

class DerivedClass extends ExampleClass {
    callProtectedMethod() {
        this._protectedMethod(); // Allowed
    }
}

const example = new ExampleClass();
// example.#privateMethod(); // Syntax Error: Private method is not accessible
// example._protectedMethod(); // Error: Method is intended to be protected and should not be called outside class hierarchy

const derived = new DerivedClass();
derived.callProtectedMethod(); // Allowed
```

Typescript example:

```typescript
class ExampleClass {
    private privateField: string;
    protected protectedField: string;

    constructor() {
        this.privateField = 'Private';
        this.protectedField = 'Protected';
    }

    private privateMethod(): void {
        console.log('This is a private method');
    }

    protected protectedMethod(): void {
        console.log('This is a protected method');
    }
}

class DerivedClass extends ExampleClass {
    public callProtectedMethod(): void {
        this.protectedMethod(); // Allowed
    }
}

const example = new ExampleClass();
// example.privateMethod(); // Error: Method is private
// example.protectedMethod(); // Error: Method is protected and only accessible within class and its subclasses

const derived = new DerivedClass();
derived.callProtectedMethod(); // Allowed
```

### Extending built-in classes

- no inheritance of static properties and methods in classes derived from built-in classes, e.g. `Array.isArray`
- if no constructor is specified in the derived class, methods will return results of the type of the derived class, not the base class (which is a good thing)
- if another type should be returned, a special getter can be defined:

```js
class PowerArray extends Array {
  isEmpty() {
    return this.length === 0;
  }

  // built-in methods will use this as the constructor
  static get [Symbol.species]() {
    return Array;
  }
}

let arr = new PowerArray(1, 2, 5, 10, 50);
alert(arr.isEmpty()); // false

// filter creates new array using arr.constructor[Symbol.species] as constructor
let filteredArr = arr.filter(item => item >= 10);

// filteredArr is not PowerArray, but Array
alert(filteredArr.isEmpty()); // Error: filteredArr.isEmpty is not a function
```

### Checking types

- `typeof` 	primitives 	string
- `{}.toString` 	for primitives, built-in objects and objects with `Symbol.toStringTag`, returns `string`
- `instanceof` for	objects, returns 	true/false
  - `instanceof SomeClass` returns `true` if an object is of type `SomeClass` or any class that `SomeClass` is inheriting from (so e.g. anything `instanceof Array` is also `instanceof Object`)
  - `instanceof` uses the `.prototype` of the object
  - `Object.prototype.toString` can be used in the context (this) of any object!

```js
let s = Object.prototype.toString;

alert( s.call(123) ); // [object Number]
alert( s.call(null) ); // [object Null]
alert( s.call(alert) ); // [object Function]
```

### Mixins

A mixin is an objet containing methods that can be used by other classes without a need to inherit from it.

Simple event handling mixins:

```js
const EventHandlerMixin = {
  _eventHandlers = {}

  on(eventName, handler) {
    if( ! this._eventHandlers[eventName]) {
      this._eventHandlers[eventName] = [];
    }
    this._eventHandlers[eventName].push(handler);
  }

  // unsusbscribe - important: handler must be referencing the same object as when
  // it was subscribed to, as always
  off(eventName, handler) {
    if( ! this._eventHandlers[eventName]) {
      return;
    }
    for(let i=0; i<this.eventHandlers[eventName].length; i++) {
      if(this._eventHandlers[eventName][i] === handler) {
        this._eventHandlers[eventName].splice(i, 1); 
        i--;
      }
    }
    const idx = this._eventHandlers.findIndex(e=>e === handler);
    this._eventHandlers.splice(idx);
  }

  trigger(eventName, ...args) {
    if(this._eventHandlers[eventName]) {
      for(handler of this._eventHandlers[eventName]) {
        handler.apply(this, args);
      }
    }
  }
}
```

Usage:

```js
// Make a class
class Menu {
  choose(value) {
    this.trigger("select", value);
  }
}
// Add the mixin with event-related methods
Object.assign(Menu.prototype, eventMixin);

let menu = new Menu();

// add a handler, to be called on selection:
menu.on("select", value => alert(`Value selected: ${value}`));

// triggers the event => the handler above runs and shows:
// Value selected: 123
menu.choose("123");
```

## Try / catch / finally

- `try / catch / finally` works only on synchronous code and won't catch errors happening in promises without `await`, errors in functions passed to `setTimeout` etc.
- `Error` objects can be of different types (e.g. ) and usually contain `message`, `name` (error type / error constructor name), `stack` properties
- global handlers can be assigned to catch all errors:
  - browser: `window.onerror = function(message, url, line, col, error) {...}`
  - node: `process.on("uncaughtException", err=>{...})`

Example browser:

```js
<script>
  window.onerror = function(message, url, line, col, error) {
    alert(`${message}\n At ${line}:${col} of ${url}`);
  };

  function readData() {
    badFunc(); // Whoops, something went wrong!
  }

  readData();
</script>
```

- we can create own error classes by inheriting from `Error` or one of its subclasses;

Explicit:

```js
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = "ValidationError";
  }
}

class PropertyRequiredError extends ValidationError {
  constructor(property) {
    super("No property: " + property);
    this.name = "PropertyRequiredError";
    this.property = property;
  }
}
```

Simplified:

```js
class MyError extends Error {
  constructor(message) {
    super(message);
    this.name = this.constructor.name;
  }
}

class ValidationError extends MyError { }

class PropertyRequiredError extends ValidationError {
  constructor(property) {
    super("No property: " + property);
    this.property = property;
  }
}
```

- We can wrap exceptions so we don't have to check for each individual exceptions if they are variants of the same parent exception (not a language feature, but a design pattern):

Verbatim from javascript.info:

```js
class ReadError extends Error {
  constructor(message, cause) {
    super(message);
    this.cause = cause;
    this.name = 'ReadError';
  }
}

class ValidationError extends Error { /*...*/ }
class PropertyRequiredError extends ValidationError { /* ... */ }

function validateUser(user) {
  if (!user.age) {
    throw new PropertyRequiredError("age");
  }

  if (!user.name) {
    throw new PropertyRequiredError("name");
  }
}

function readUser(json) {
  let user;

  try {
    user = JSON.parse(json);
  } catch (err) {
    if (err instanceof SyntaxError) {
      throw new ReadError("Syntax Error", err);
    } else {
      throw err;
    }
  }

  try {
    validateUser(user);
  } catch (err) {
    if (err instanceof ValidationError) {
      throw new ReadError("Validation Error", err);
    } else {
      throw err;
    }
  }

}

try {
  readUser('{bad json}');
} catch (e) {
  if (e instanceof ReadError) {
    alert(e);
    // Original error: SyntaxError: Unexpected token b in JSON at position 1
    alert("Original error: " + e.cause);
  } else {
    throw e;
  }
}
```

## Callbacks, promises

### Callbacks

Simple script loader that executes a function once the script is loaded (or not):

```js
loadScript('/my/script.js', function(error, script) {
  if (error) {
    // handle error
  } else {
    // script loaded successfully
    // possibly load another script the same way (callback hell)
  }
});
```

### Promises

#### Basics

Good analogy: you subscribe to a waiting list for a new song (value) from an artist (executor) and get notified once the song is finished (value) or if the song project is abandoned (error)

```js
let promise = new Promise(function(resolve, reject) {
  // executor (the producing code, "singer")
  // runs when the promise is created
  if(thereWasSomeError) {
    reject(new Error("something went wrong"));
  }
  resolve(someValue)
});
```

`resolve` and `reject` are functions provided by javascript itself. They can be called only once by the executor (all further calls to resolve or reject are ignored once one of them is called, so there can't be multiple resolves and resolve and reject are mutually exclusive).

#### `.then`

The object returned by `new Promise` has the following **internal** (can't be directly accessed) properties:

- `.state`: initially "pending", then changes to "fulfilled" when `resolve` is called or "rejected" when `reject` is called.
- `.result`: `undefined`, then changes to `value` when `resolve(value)` is called or `error` when `reject(error)`` is called.

As these properties are internal, we can indirectly access them using `.then`, `.catch` and `finally`:

```js
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => reject(new Error("Whoops!")), 1000);
});

// reject runs the second function in .then
promise.then(
  result => alert(result), // doesn't run; can be null if we're not interested in the success value
  error => alert(error) // shows "Error: Whoops!" after 1 second; optional; can be null
);
```

- we can attach multiple `.then` handlers to a promise (NOT promise chaining, they run individually!); this is rarely used.

```js
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => resolve(1), 1000);
});

promise.then(function(result) {
  alert(result); // 1
  return result * 2;
});

promise.then(function(result) {
  alert(result); // independent of the other handler, result is still 1
  return result * 2;
});
```

- `.then` **returns a new promise** (or, more precise, a “thenable” object to allow for creation of Promise-compatible objects without inheriting from `Promise`)

We can chain `.then`s to allow for async operations on the value.

```js
fetch('/article/promise-chaining/user.json')
  // .then below runs when the remote server responds
  .then(function(response) {
    // response.text() returns a new promise that resolves with the full response text
    // when it loads
    return response.text();
  })
  .then(function(text) {
    // ...and here's the content of the remote file
    alert(text); // {"name": "iliakan", "isAdmin": true}
  });
```

Errors thrown in the executor (explicitely by the programmer or javascript errors) are automatically handled like `reject(error)`:


```js
new Promise((resolve, reject) => {
  throw new Error("Whoops!");
}).catch(alert); // Error: Whoops!
```

The same happens in the promises created by the `.then` handlers.

```js
new Promise((resolve, reject) => {
  resolve("success!")
})
.then(val => noSuchFunction(val))
.catch(alert); // // noSuchFunction: blabla is not defined
```

#### `.catch`

`.catch(f)` is a complete analog / shorthand of `.then(null, f)`

#### `.finally`

- used to clean up (e.g. disable loading state)
- runs when the promise is resolved or rejected
- finally handler has no arguments
- passes through result and error

Example

```js
new Promise((resolve, reject) => {
  setTimeout(() => resolve("value"), 2000);
})
  .finally(() => alert("Promise ready")) // triggers first
  .then(result => alert(result)); // <-- .then shows "value"
```

#### `unhandledrejection`

Only errors we can actually handle should be caught.

We can catch errors in promises using the `unhandledrejection` event.


```js
window.addEventListener('unhandledrejection', function(event) {
  // the event object has two special properties:
  alert(event.promise); // [object Promise] - the promise that generated the error
  alert(event.reason); // Error: Whoops! - the unhandled error object
});

new Promise(function() {
  throw new Error("Whoops!");
}); // no catch to handle the error
```

#### Promise API

- Used to return a value as a promise:
  - `Promise.resolve(value)` creates an immediately resolved promise and has the same effect as `new Promise((resolve, reject)=>resolve(value))`
  - `Promise.reject(error)` creates an immediately rejected promise and has the same effect as `new Promise((resolve, reject)=>reject(error))`
- Multiple promises:
  - `Promise.all(iterableOfPromisesOrValues)` waits until all passed promises are resolved OR one of the promises is rejected; if all are resolved, the results are returned in an array in order of the promises of the iterable; if rejected, it returns the error of the rejected promise

```js
  Promise.all([
  new Promise((resolve, reject) => {
    setTimeout(() => resolve(1), 1000)
  }),
  2, // normal values are allowed too for convenience

]).then(alert); // 1, 2, 3
```

  - `Promise.allSettled(iterable)` returns the values or errors in order when all promises passed are either resolved or rejected
  - `Promise.race(iterable)` returns the value or error of the first settled promise
  - `Promise.any(iterable)` returns the value of the first resolved promise; if all promises are rejected, it returns an `AggregateError` with all promise errors in its `.error` array property (e.g. `errors.error[0]`)

#### Promisification

- Pattern used to wrap a function that uses callbacks in a function that returns a promise

https://javascript.info/promisify


#### Gotchas

##### errors not handled by executor

There’s an "implicit try..catch" around the function code. So all synchronous errors are handled.

But here the error is generated not while the executor is running, but later. So the promise can’t handle it.

```js
new Promise(function(resolve, reject) {
  setTimeout(() => {
    throw new Error("Whoops!"); // will NOT be handled by catch!
  }, 1000);
}).catch(alert);
```

##### Microtask queue / Immediately resolved promises are styll asynchronous!


>>Promise handling is always asynchronous, as all promise actions pass through the internal “promise jobs” queue, also called “microtask queue” (V8 term).
>>So .then/catch/finally handlers are always called after the current code is finished.

```js
let promise = Promise.resolve();

promise.then(() => alert("promise done!"));

alert("code finished"); // this alert shows first
```

### async / await

#### Async

- `async` ensures that a function return a promise instead of a value:

```js
async function myFunc() {
  return 1;
}

// same as 

async function myOtherFunc() {
  return new Promise(resolve=>resolve(1));
}

myFunc().then(...)
```

- It allows us to use `await` inside the function body


#### Await

- Waits (suspends execution) until a promise is settled and returns its result. It doesn't block the javascript engine itself
- errors can be caught using try / catch or with `.catch`

```js
async function f() {
  let response = await fetch('http://no-such-url');
}

// f() becomes a rejected promise
f().catch(alert); // TypeError: failed to fetch // (*)

// same as 

try {
  f()
} catch(error) {
  alert(error)
}
```

## Generators / yield

Similar to python generators. Returns a generator that returns values as an iterable using `yield` and pauses function execution.

```js
function* generateSequence(start, end) { // * denotes a generator function
  for (let i = start; i <= end; i++) {
    yield i;
  }
}

const oneToFive = generateSequence(1,5);
for(const num of oneToFive) {
  console.log(num); // 1,2,3,4,5
}

// the generator can't be re-used!
const myAr = [...oneToFive]; // myAr is []
```

`yield*` delegates to another generator:

```js
function* generateAnotherSequence() {
  yield* generateSequence(1,5);
  yield* generateSequence(6,10);
}

const myAr = [...generateAnotherSequence()] // 1,2,...,10
```

Values can be passed TO generators!

```js
function* questionGenerator(initialValue) {
  let myVal = initialValue;
  while(myVal) {
    myVal = yield `${myVal} + ${myVal} = ?`;
  }
}

const gen = questionGenerator(1);

const questions = [
  gen.next().value,
  gen.next(2).value,
  gen.next(3).value
]

console.log(questions) // Array(3) [ "1 + 1 = ?", "2 + 2 = ?", "3 + 3 = ?" ]
```

- `generator.throw(error)` throws an error *into* the generator
- `generator.return(value)` finishes the generator "prematurely" ) e.g. `g.return('foo'); // { value: "foo", done: true }`


## Modules

Core takeaways in the following sections are taken verbatim from javascript.info.

### General

- A module is a file. To make import/export work, browsers need `<script type="module">`. Modules have several differences:
- Deferred by default.
- Async works on inline scripts.
- To load external scripts from another origin (domain/protocol/port), CORS headers are needed.
- Duplicate external scripts are ignored.
- Modules have their own, local top-level scope and interchange functionality via import/export.
- Modules always use strict.
- Module code is executed only once. Exports are created once and shared between importers.

### Export

- Before declaration of a class/function/…: `export [default] class/function/variable ...`
- Standalone export: `export {x [as y], ...}`
- Re-export:
  - `export {x [as y], ...} from "module"`
  - `export * from "module"` (doesn’t re-export default).</li>
  - `export {default [as y]} from "module"` (re-export default).</li>


### Import

- Importing named exports: `import {x [as y], ...} from "module"`
- Importing the default export:
  - `import x from "module"`
  - `import {default as x} from "module"`
- Import all: `import * as obj from "module"`
- Import the module (its code runs), but do not assign any of its exports to variables: `import "module"`

### Import expression

`import("/somefile.js)` loads the code and returns a promise that resolves into the module. They work also in scripts that aren't loaded with `type="module"`.

- named exports: `let {hi, bye} = await import('./say.js');`
- default exports: 

```js
let obj = await import('./say.js');
let say = obj.default;
```

## Proxy

Syntax: `let proxy = new Proxy(target, handler)`

A proxy 
- wraps an object and intercepts its operations
- is a special object with no own properties
- forwards all operations for which there are no traps set to the target:

```js
let target = {};
let proxy = new Proxy(target, {}); // empty handler

proxy.test = 5; // writing to proxy (1)
alert(target.test); // 5, the property appeared in target!

alert(proxy.test); // 5, we can read it from proxy too (2)

for(let key in proxy) alert(key); // test, iteration works (3)
```

## Other stuff

- `globalThis` references `window` in the browser and `global` in node and is now supported pretty much everywhere
