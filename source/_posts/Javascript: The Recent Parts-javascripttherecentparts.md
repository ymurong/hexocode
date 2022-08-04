---
title: Javascript:The Recent Parts
date:  2021-11-21 12:00:00
tags:
- javascript
---

## Array Destructuring 

assignment expression get the whole thing that was subject to assignment 
```
function data (){
    return [1,2,3]
}

var tmp;
var o = [];
tmp = [
    o[0],
] = data()

console.log(tmp). // [1,2,3]
console.log(o)   //. [1]
```

### comma separation


empty positions using comma 
```
var [
   first,
   ,
   third,
   ...others
] = data()
```

swap variables
```
[a,b]=[b,a]
```

### parameter arrays

recommended way to have a default deconstructing fallback to prevent type error			
```
function data([
    first = 10,
    second = 20,
    third = 30,
  ] = []){
    console.log(first) // 10
  }
data(undefined)
```

### nested array restructuring

recommended way to have a default deconstructing fallback to prevent type error	
```
function data(){
   return [1, undefined, 4]
}


var [
   first,
   [ 
     second,
     third,
   ] = [],
   forth
] = data()
```


## Iterators

an iterator is an object which defines a sequence and potentially a return value upon its termination.

Specifically, an iterator is any object which implements the Iterator protocol by having a next() method that returns an object with two properties:

value
The next value in the iteration sequence.

done
This is true if the last value in the sequence has already been consumed. If value is present alongside done, it is the iterator's return value.

```
var it1 = "Hello"
var it1 = str[Symbol.iterator]()

it1.next() // { value: "H", done: false}
...
it1.next() // { value: "o", done: true}
```

```
var obj = {
   a: 1,
   b: 2,
   c: 3,
   [Symbol.interator]: function(){
	var keys = Object.keys(this);
	var index = 0;
	return {
	  next: ()=>{
		(index< keys.length)?
		  {done: false, value: this[keys[index++]]}:
    		  {done: true, value: undefined}
	  }
	}
   }
}

[..obj] // [1,2,3]
```

## Generators
The Generator object is returned by a generator function and it conforms to both the iterable protocol and the iterator protocol.
```
var obj = {
   a: 1,
   b: 2,
   c: 3,
   *[Symbol.interator](){
	for(let key of Object.keys(this)){
		yield this[key];
	}
   }
}

[..obj] // [1,2,3]
```

## Regular Expressions
### Look Ahead

```javascript
var msg = "Hello Message"
// positive look ahead
msg.match(/(l.)(?=o)/g)
// ["l"]

// negative look ahead
msg.match(/(l.)(?!=o)/g)
// ["lo","ld"]
```


### Look Behind

```
// positive look behind
msg.match(/(?<=e)(l.)/g)
// ["ll"]

// negative look behind
msg.match(/(?<!e)(l.)/g)
// ["lo", "ld"]
```

### Regex Excercise

A regex generator example 

```javascript
var poem = `
The power of a gun can kill
and the power of fire can burn
the power of wind can chill
and the power of a mind can learn
the power of anger can rage
inside until it tears u apart
but the power of a smile
expecially yours can heal a frozen heart
`

for (let power of powers(poem)) {
    console.log(power)
}

function* powers(poem) {
    // ?<= positive look behind
    // ?<thing> ?<verb> named capture groups
    // (?: ) non-capturing group (not necessary here)
    // *? lazy mode
    // /g don't stop after first match -> global mode
    // /s dotall mode -> dot matches newline
    var re = /(?<=power of )(?<thing>(?:a )?\w+).*?(?<=can )(?<verb>\w+)/gs
    while (match = re.exec(poem)) {
        let {
            groups: {
                thing,
                verb
            }
        } = match

        yield `${thing}: ${verb}`
    }
}
```
output 
```
a gun: kill ​​​​​at ​​​​​​​​power​​​ ​quokka.js:13:4​

fire: burn ​​​​​at ​​​​​​​​power​​​ ​quokka.js:13:4​

wind: chill ​​​​​at ​​​​​​​​power​​​ ​quokka.js:13:4​

a mind: learn ​​​​​at ​​​​​​​​power​​​ ​quokka.js:13:4​

anger: rage ​​​​​at ​​​​​​​​power​​​ ​quokka.js:13:4​

a smile: heal ​​​​​at ​​​​​​​​power​​​ ​quokka.js:13:4​
```

