---
title: You don't know JS - Notes 3 (Objects & Prototypes)
date:  2021-11-05 12:00:00
tags:
- javascript
---
## Objects 

#### This keyword
A function's this references the execution context for that call, determined entirely by how the function was called

#### Implicit & Explicit Binding

* Implicit Binding
```
function ask(question) {
    console.log(this.teacher, question);
}

var workshop1 = {
    teacher: "Kyle",
    ask: ask,
}

var workshop2 = {
    teacher: "Suzy",
    ask: ask,
}

workshop1.ask("How do I share a method")
workshop2.ask("How do I share a method")
```

* Explicit Binding
  * call: call a function by specifying the context 
  * apply: call a function by specifying the context (arguments in an array)

* Hard Binding
  * bind: recreate the function by specifying the context 

#### When to use this | closure
* if we want flexible dynamism, use this keyword
* if we want predictability, use closure

#### Binding rule precedence
1. Is the function called by new ?
2. Is the function called by call() or apply()?
3. Is the function called on a context object?
4. Default: global object (except strict mode)

#### Arrow Functions & Lexical this

An ArrowFunction does not define local bindings for arguments, super, this, or new.target. Any reference to arguments, super, this, or new.target within an ArrowFunction must resolve to a binding in a lexically enclosing environment. 

> JS spec: https://262.ecma-international.org/9.0/#sec-arrow-function-definitions-runtime-semantics-evaluation
```
var workshop = {
	teacher: "Kyle",
	ask(question) {
		setTimeout(()=>{
			console.log(this.teacher, question);
		},100)
	}
}

workshop.ask("Is this lexical 'this' ?");
// Kyle Is this lexical 'this' ?
```

Workshop object does not have scope. Here obviously this in the arrow function's outer scope is global (no outer function except arrow function), which make this.teacher always undefined.
```
var workshop = {
	teacher: "Kyle",
	ask: (question) => {
		setTimeout(()=>{
			console.log(this.teacher, question);
		},100)
	}
}

workshop.ask("what happened to 'this'")
// undefined what happened to 'this'

workshop.ask.call(workshop, "still no 'this'")
// undefined still no 'this'
```

#### New keyword
Steps behind the scenes 
```
1. Create a brand new empty object
2. Link that object to another object
3. Call function with this set to the new object
4. if function does not return an object assume return of this
```

#### OLOO Pattern (Delegation-Oriented Design)
Objects Linked to Other Objects.

Object.create() do the first two steps of the new keyword.

```
var AuthController = {
  authenticate() {
    server.authenticate([this.username, this.password],
	this.handleResponse.bind(this)
    );
  },
  handleResponse(resp){
    if(!resp.ok) this.display.Error(resp.msg);
  }
}

var LoginFormController = 
       Object.assign(Object.create(AuthController),{
	   onSubmit(){
               this.username = this.$username.val();
               this.password = this.$password.val();
	       this.authenticate();
	   },
           displayError(msg) {
		alert(msg);
           }
       })
```

