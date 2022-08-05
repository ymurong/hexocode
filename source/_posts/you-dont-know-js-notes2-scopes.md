---
title: You don't know JS - Notes 2 (Scopes)
date:  2021-10-28 12:00:00

tags:
- javascript
---
## Lexical Scopes
#### Javascript as a two pass processing

* First pass: we could call it compilation or parsing which will go through the entire code and establish the execution plan. 

* Second pass: we could call it execution of the plan that the javascript engine created. 

#### Compilation process with bucket && marble
The scope is like a bucket and variable reference is like a marble. 

During the parsing process, the engine will prepare required buckets and marbles according to the target or source references

* Target reference  => a formal declaration of variables or functions => for variable if not exists then make the marbles and put into the right bucket, if it is also a function then we need also create a child bucket

* Source reference => check the existence and give the marbles if founded (recursively searching from current level to outer level), otherwise Reference Error in strict mode


#### Strict mode 
if strict mode, the topic will be Reference Error
if not, auto-global variable will be declared (really bad!!!)
```
function other(){
	topic = "React"
}
```

#### Undefined vs Undeclared
* undeclared does not exist
* undefined exists but does not have a value

#### Function Expressions
function expression put their identifiers into their own scope
```
var myTeacher = function anotherTeacher(){
     console.log(anotherTeacher);
}
```

#### Naming Function Expressions 
> Named Function Declaration > Named Function Expression > Anonymous Function Expression (only in arrow function)

Named Function Expressions: Benefits
* Reliable function self-reference (recursion,etc)
* More debuggable stack traces
* More self-documenting code 

#### Dynamic Scope 
Dynamic scopes are determined at Runtime, ex: bash

#### IIFE Pattern
By wrapping a function expression around some statement expressions in order to make ephemeral and 
```
var teacher = (function getTeacher(){
	try {
	    return featchTeacher(1);
	}
	catch (err){
	    return "kyle"
	}

})()
```

## Hoisting 

As a matter of fact, hoisting is not changing the order of execution. It is actually the process of parsing that allows the JS engine to know the identifiers in advance. For example, var declarations and function declarations could be considered at put at the top of the scope when execution starts. 

* const, let don't hoist, TDZ error
* function expression don't hoist (assignment part)


## Closures

The closure is when a function "remembers" its lexical scope even when the function is executed outside that lexical scope. It preserves the link to a variable. 





