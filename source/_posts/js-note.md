---
title: js-note
date: 2020-07-02 18:23:21
tags: javascript
toc: true
---

# Template literals (String literals)

add variables at runtime

```js
let msg = `some ${variable1}, 
    ${"$" + variable2}`;
```

use tag function to add bold style to variables in template literals

```js
function highlighText(strings, ...values) {
  let str = "";
  for (var i = 0; i < strings.raw.lengths; i++) {
    if (i > 0) {
      str += `<b>${values[i - 1]}</b>`;
    }
    str += strings.raw[i];
  }
}

let msg = highlighText`some ${variable1}, ${"$" + variable2}`;
```

# var, let and const

var:
- no block scope
- can be redeclared anywhere
- can be used and reassigned anywhere

let:
- block scope
- can't be redeclared within scope
- can be reassigned within scope

const:
- block scope
- can't be reassigned or redeclared
- the value **can** be changed

const is more used as readability purpose

```js
const arr = [3, 4, 5];
arr = 3; // error
arr[0] = 22; // okay!
var arr = Object.freeze([3, 4, 5]); // instead
```

# Destructing an array or object

array:

- don't have to catch all values in the array
- variable is undefined if arr is not enough to unpack
- can set default value

```js
var [a, b = true, c, ...moreArgs] = arr;
```

object:

```js
var { Id, ApplicantName = "Barry" } = obj;
```

# String

```js
str.trim(); // trim white space
str.toLowerCase();
str.startWith("dr");
str.endWith("md", 4);
str.search("house");
str.includes("house");
```

# Number

```js
Number.isInteger(num); // 25.0 is integer
Number.MAX_SAFE_INTEGER; // Number.MIN_SAFE_INTEGER
Number.isSafeInteger(num);
```

# Symbol

```js
var id = Symbol("My Id");
var id2 = Symbol("My Id");
console.log(id === id2); // false

var id = Symbol.for("My Id");
var id2 = Symbol.for("My Id");
console.log(id === id2); // true

// create secret property
var loan = {
    name: "Barry",
    [Symbol("income")]: 15000
};
console.log(load[Symbol.for("income")]); // 15000
console.log(Object.getOwnPropertyNames(loan)); // ["name"]
console.log(Object.getOwnPropertySymbol(loan)); // [Symbol(income)]
```