# javascript_and_dom_notes

Just some notes of stuff I forgot, never knew or never fully understood about javascript.

Notable source: https://javascript.info/

## this and that

### Clearing arrays

`someArray.length = 0` as opposed to `someArray = []` is a good practice when you want to clear an array but maintain its reference across the application. It is also beneficial in terms of performance and memory efficiency.

### pushing multiple values to an array

- `someArray.push()` accepts multiple values and thus can also be used with the spread operator (`someArray.push(...someOtherArray)`)

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