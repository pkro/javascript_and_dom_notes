# javascript_and_dom_notes

Just some notes of stuff I forgot, never knew, are important to be reminded of or never fully understood about javascript.

Notable source: https://javascript.info/

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

## Objects

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

## Other stuff

- `globalThis` references `window` in the browser and `global` in node and is now supported pretty much everywhere
- setInterval