---
title: Functional-Light Javascript
date:  2021-11-23 12:00:00
tags:
- javascript
---
## Function Purity

### Side effects
* I/O (console, file ,etc)
* Database Storage
* Network Calls
* DOM
* TimeStamp
* Random Numbers


### Pure Function

* The function return values are identical for identical arguments
* The function application has no side effects

### Impurity Solutions
* Wrappers 
* Adapters 

```javascript
"use strict";

var students = [
	{ id: 260, name: "Kyle" },
	{ id: 729, name: "Susan" },
	{ id: 42, name: "Frank" },
	{ id: 74, name: "Jessica" },
	{ id: 491, name: "Ally" }
];



function sortStudentsByID() {
	// Don't modify this function
	students.sort(function byID(s1,s2){
		return s1.id - s2.id;
	});
	return students;
}

// *************************************

// Wrappers
function getStudentsByName(students) { 
    students = students.slice() // students belongs to function scope
    return sortStudentsByName()

    function sortStudentsByName() {
        // Don't modify this function
        students.sort(function byName(s1,s2){
            if (s1.name < s2.name) return -1;
            else if (s1.name > s2.name) return 1;
            else return 0;
        });
        return students;
    }
}

// Adapters
// modify/move this function
function getStudentsByID(curStudents) { 
    var originStudents = students.slice()
    students = curStudents.slice()
    var newStudents = sortStudentsByID()
    students = originStudents
    return newStudents; 
}

// *************************************

var studentsTest1 = getStudentsByName(students);
console.log(studentsTest1[0].name === "Ally");
console.log(studentsTest1[1].name === "Frank");
console.log(studentsTest1[2].name === "Jessica");
console.log(studentsTest1[3].name === "Kyle");
console.log(studentsTest1[4].name === "Susan");

var studentsTest2 = getStudentsByID(students);
console.log(studentsTest2[0].id === 42);
console.log(studentsTest2[1].id === 74);
console.log(studentsTest2[2].id === 260);
console.log(studentsTest2[3].id === 491);
console.log(studentsTest2[4].id === 729);

var studentsTest3 = students;
console.log(studentsTest3[0].id === 260 && studentsTest3[0].name === "Kyle");
console.log(studentsTest3[1].id === 729 && studentsTest3[1].name === "Susan");
console.log(studentsTest3[2].id === 42 && studentsTest3[2].name === "Frank");
console.log(studentsTest3[3].id === 74 && studentsTest3[3].name === "Jessica");
console.log(studentsTest3[4].id === 491 && studentsTest3[4].name === "Ally");

```

## Arguments Adapters

#### Arguments Shape Adapters

```
function unary(fn){
  return function one(arg){
     return fn(arg)
  }
}

function binary(fn){
  return function two(arg1, arg2){
     return fn(arg1, arg2)
  }
}

var g = unary(f);
var h = binary(f);

g(1,2,3,4)  // [1]
g(1,2,3,4)  // [1,2]
```

#### Filp adapter, reverse adapter

```
function flip(fn) {
   return function flipped(arg1, arg2, ...args) {
      return fn(arg2, arg1, ...args)
   }
}

function reverseArgs(fn) {
   return function reversed(arg1, arg2, ...args) {
      return fn(...args.reverse())
   }
}

var g = reverseArgs(f)

g(1,2,3,4)
```

#### Spread adapter


```
function spreadArgs(fn){
  return function spread(args){
    return fn(...args)
  }
}

function f(x,y,z,w){
  return x + y + z + w;
}

var g = spreadArgs(f);

g([1,2,3,4])
```

## Point Free

Tacit programming (point-free programming) is a programming paradigm in which a function definition does not include information regarding its arguments, using combinators and function composition [...] instead of variables.

### Equational Reasoning

```
getPerson(function onPerson(person){
   return renderPerson(person)
})

getPerson(renderPerson)
```

### point free refactor

not 
```
function not(fn) {
  return function negated(...args) {
     return !fn(...args)
  }
}

function isOdd(v){
   return v % 2 == 1
}

var isEven = not(isOdd)
```

when
```
function when(fn) {
  return function(predicate){
     return function(...args){
        if (predicate(...args)) {
	   return fn(...args)
        }
     }
  }
}
```

### advanced point free

compose
```
function (fn2, fn1) {
   return function composed(v){
      return fn2(fn1(v))
   }
}
```

## Closure 

Closure is not necessarily functional pure. But closure could be used in functional theory.

```
// this is a unpure function 
function makeCounter(){
   var counter = 0;
   return function increment(){
      return ++counter;
   }
}

// this is functional pure 
function addAnother(z) {
   return function addTwo(x,y){
      return x + y + z
   }
}
```

if we are using closure in functional programming, we have to make sure that we are closing over non-changing, non-mutating variables.

```
// here each time, next function close over a different variable that did not change its state.
function strBuilder(str){
  return function next(v) {
     if(typeof v == "string") {
        return strBuilder(str + v)
     }
     return str;
  }
}
```

### Memoization

```
function repeater(count) {
   var str
   return function allTheAs() {
      if (str == undefined) {
  	  str = "".padStart(count,'A')
      }
      return str;
   }
}

function repeater(count) {
    return memorize(function allTheAs(){
	 return "".padStart(count, "A")
    }) 
}
```

### Referential Transparency 

A function call is pure if it has referential transparency. Referential Transparency means a function call can be replaced with its return value and not affect any of the rest of the program


### Partial Application & Currying
Both are specialization techniques. Currying is a special form of partial application. 
Partial Application presets some arguments now, receives the rest on the next call
Currying doesn't preset any arguments, received each argument one at a time. 

### Specialization Adapts Shape

```
function add(x,y) {
	return x + y 
}

[0,2,4,6,8].map(function addOne(v){
	return add(1,v)
})

add = curry(add);

[0,2,4,6,8].map(add (1));
```

### Composition with Currying

 
```
var mod2 = mod(2)
var eq1 = eq(1)

function isOdd(x){
  return eq1(md2(x))
}

function composeTwo(fn2,fn1){
  return function composed(v){
     return fn2(fn1(v))
  }
}

var isOdd = composeTwo(eq1, mod2)
var isOdd = composeTwo(eq(1), mod(2))
```

## Immutability

### Value Immuatability
* when calling a function with a data structure, do your reader of the benefit of annotating it that you want it to not to be changed (Object.freeze).
* when writing a function that receives a data structure, treat it as if it's read-only no matter what. Make a copy of it.

```
"use strict";

function lotteryNum() {
	return (Math.round(Math.random() * 100) % 58) + 1;
}

// make sure this function is pure
function pickNumber(num,nums) {
	if (!nums.includes(num)) {
		nums = [num,...nums];
		nums.sort(function asc(a,b){ return a - b; });
	}
	return nums;
}

var luckyLotteryNumbers = [];
const howMany = 6;

while (luckyLotteryNumbers.length < howMany) {
	luckyLotteryNumbers = pickNumber(
		lotteryNum(),
		Object.freeze(luckyLotteryNumbers)
	);
}

console.log(luckyLotteryNumbers);

```

## Recursion 

### Example isPalindrome

```
function isPalindrome(str) {
    if (str.length <= 1) return true
    const first = str[0]
    const last = str[str.length-1]
    if(first === last){
        return isPalindrome(str.slice(1, str.length -1))
    }
    return false
}

console.log( isPalindrome("") === true );
console.log( isPalindrome("a") === true );
console.log( isPalindrome("aa") === true );
console.log( isPalindrome("aba") === true );
console.log( isPalindrome("abba") === true );
console.log( isPalindrome("abccba") === true );

console.log( isPalindrome("ab") === false );
console.log( isPalindrome("abc") === false );
console.log( isPalindrome("abca") === false );
console.log( isPalindrome("abcdba") === false );

```

### Proper Tail Calls

* strict mode
* return keyword
* single function call, nothing else in the expression that needs to be computed afterwards

```
function countVowels(count, str) {
  count += (isVowel(str[0]? 1: 0))
  if (str.length <= 1) return count;
  return countVowels(count, str.slice(1))
}

var countVowels = curry(2, countVowels)(0)
```

### Trampolines

With the regular recursion function, we literally stack up the work. In the trampoline, we never stack up the work, we do some work and return work back to this helper(iterative).

we could write a tail call for function then we could wrap the tail call function into a function to make it trampolinable function.

```javascript
function trampoline(fn) {
   return function trampilined(...args){
      var result = fn(...args);
      while(typeof result == "function"){
	  result = result()
      }
      return result;
   }
}
```

trampoline version countVowels
```
function countVowels(count, str) {
  count += (isVowel(str[0]? 1: 0))
  if (str.length <= 1) return count;

  return function f(){
      countVowels(count, str.slice(1))
  }
}

var countVowels = trampoline(countVowels);

// optionally 
countVowels = curry(2, countVowels)(0);
```

## Lists Operation

### Composition with reduce

```
function add1(v) { return v + 1}
function mul2(v) { return v * 2}
function div3(v) { return v / 3}

function compose(...fns){
    return function composed(v){
       return fns.reduceRight(function invoke(bigFn,fn){
          return fn(bigFn)
       },v)
    }
 }

 var f = compose(div3, mul2, add1)(8)
 console.log(f) // 6
```

























   


