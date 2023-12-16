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
alert(user + 500); // valueOf -> 1500```

## Arrays

### Clearing arrays

`someArray.length = 0` as opposed to `someArray = []` is a good practice when you want to clear an array but maintain its reference across the application. It is also beneficial in terms of performance and memory efficiency.

### pushing multiple values to an array

- `someArray.push()` accepts multiple values and thus can also be used with the spread operator (`someArray.push(...someOtherArray)`)

## Debugging

- `step` continues execution with the next command and steps into the function code if the command is a function call, but ignores async actions
- `step into` goes also into async functions and waits for them if necessary (https://developer.chrome.com/blog/new-in-devtools-65/#async)