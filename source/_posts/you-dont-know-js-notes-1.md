---
title: You don't know JS - Notes 1 (Types & Coercion)
date:  2021-10-24 12:00:00
tags:
- javascript
---

## Types

#### Typeof operator
Type has some surprising cases that could lead to bugs if carelessly handled

```javascript
type of doesnotExist;   // undefined

var v = null;
type of v;   // "object" OOPs!!!

var v = [1,2,3];
type of v;   // "object" hmmm...  use Array.isarray() instead
```

#### Temporary Dead Zone (TDZ)
> undeclared -> uninitialized -> undefined

let variables cannot be read/written until they have been fully initialized, which happens when they are declared

```javascript
{ // TDZ starts at beginning of scope
  console.log(bar); // undefined
  console.log(foo); // ReferenceError
  var bar = 1;
  let foo = 2; // End of TDZ (for foo)
}
```

This phenomenon can be confusing in a situation like the following. The instruction let n of n.a is already inside the private scope of the for loop's block. So, **the identifier n.a is resolved to the property 'a' of the 'n' object located in the first part of the instruction itself (let n)**.
```
function go(n) {
  // n here is defined!
  console.log(n); // Object {a: [1,2,3]}

  for (let n of n.a) { // ReferenceError
    console.log(n);
  }
}

go({a: [1, 2, 3]});
```

#### NaN & isNaN

NaN is the only value that is not equal to itself

NaN: an invalid number (a number but not valid)

isNaN coerces values to numbers before it checks for them to be NaN
```javascript
isNaN("8") // false
isNaN("not a number")   // true
```

Number.isNaN() does not coerce 
```javascript
Number.isNaN(Number("test")) // true
Number.isNaN("not a number")   // false
```

#### Negative 0 
```
-0 === 0;         // true
Object.is(-0, 0); // false
Object.is(0, 0);  // true
```

#### Fundamental Objects (aka: buit-in objects aka Native Functions)
> Don't use them but understand them



## Coercion Corner Cases
Be aware about the corner cases and you need to handle it in your code, especially from DOM elements.

String to Number Example
```javascript
function addAStudent(numStudents) {
	// here we need to do coercion 
	// as plus sign will concatenate string with number
	// corner cases of coercion is that
	// empty string could be coerced to 0
	return numStudents + 1;
}

// + sign will coerce the variable into number
addAStudent (+studentInputElem.value)

```

A possible validation function for numStudents could be
```javascript
// make sure we coerce by excluding corner cases
if(
    typeof numStudents == "string" &&
    numStudents.trim() != ""
) {
    numStudents = Number(numStudents)
}

if(
    typeof numStudents == "number" &&
    numStudents >= 0 &&
    Number.isInteger(numStudents)
) {
    return true;
}

return false
```


String Example

```
// make sure name string 
// non-empty
// must contain non-whitespace of at least 3 chars
if(
	typeof name == "string" &&
	name.trim().length >= 3
){
	return true
}
```

#### Double equal ==

Double equal will work like the following in order:
1. If the types are the same: ===
2. If null or undefined: equal
3. If non-primitives: ToPrimitive
4. Prefer: ToNumber

To avoid:
* == with 0 or "" (or even " ")
* == with non-primitives
* == true or == false: allow ToBoolean or use ===

