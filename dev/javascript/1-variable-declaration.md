# How to declare a variable

## Introduction

In javascript, you can use three way to declare variables : var, let and const. So what is the difference between the three?

## The historical `var`

That is the historic way to declare variables. Variable declared with `var` are **function scoped**.  
So for example :

```
function test() {
    var someVariable = 'that is a function scoped variable'
    if(true) {
        var someVariable = 'function affectation'
    }
    console.log(someVariable)
}
```

If you try this code in your browser, it will print 'function affectation' in the browser.   
Here is the explanation : Javascript interpreter "optimise" your code before executing it. It will move variable 
declaration at the beginning of the scope of the variable. That behavior is named 
[**hoisting**](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting).    
So in my example, the real executed code would be like that :

```
function test() {
    var someVariable
    someVariable = 'that is a function scoped variable'
    if(true) {
        someVariable = 'function affectation'
    }
    console.log(someVariable)
}
```

Here it becomes clear that there is only one variable. So you should be really careful when using `var` declaration as
it could produce **unintended results**.

## `let` to the rescue

Let is a new way to declare variables. It appeared with ECMAScript 6. `let` is similar to `var` but it declare variables
block scoped.  
So if we take back the example, and replace `var` with `let` :

```
function test() {
    let someVariable = 'that is a block scoped variable'
    if(true) {
        let someVariable = 'function affectation'
    }
    console.log(someVariable)
}
```

Here the output will be "that is a block scoped variable". If we look at what the interpreter executes, it looks like 
that :

```
function test() {
    let someVariable
    someVariable = 'that is a block scoped variable'
    if(true) {
        let someVariable
        someVariable = 'function affectation'
    }
    console.log(someVariable)
}
```

Here, the second `someVariable` lives only while the block `if (true) {...}` lives resulting in two different variables.  
That behavior is more like other programing languages, so it avoid confusions for developers that begin with the language.  
It was not possible to change `var` to behave like `let` as javascript is an old language and some old scripts may rely
on that behavior.

## So what about `const`

`const` behave exactly as `let` in the way it creates block scoped variables. It differs as variables created with `const`
cannot be reaffected.  
For example :

```
const someVariable = 'content'
someVariable = 'another content'
```

In that example, the Javascript interpreter will blow the following error :

```
TypeError: invalid assignment to const `someVariable'
```

If you use IDE like Intellij/Webstorm, it will underline your code with an error mark.

## Cool so what's the catch ?

Nothing comes for free. Despite it's name `const` does not create constants.  
For example, the following code is right :

```
const someVariable = []
someVariable.push('content')
console.log(someVariable)
```  

Here the array is mutated and contains one element : `Array [ "content" ]`. So be careful, `const` only **forbids new
affectations**, you does not manipulate constant. 
 
## My recommendations

`const` is the more restrictive and best way to declare variables in Javascript. Whenever I can't use it, I try to refactor
my code to use it, often leading in more readable code.  
When it is not possible, you can switch to `let` as block scope is more familiar to everyone and easier to reason about.  
`var` should be avoided at all times.

## Learn more

[Documentation MDN on `var`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var)  
[Documentation MDN on `let`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)  
[Documentation MDN on `const`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)