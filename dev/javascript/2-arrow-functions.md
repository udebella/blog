# Arrow functions and their specificity

## Introduction

ES2015 brought a new way to write anonymous functions called arrow functions. So what are the caveat of using this new
notation? Can we replace every "old" notation with this "new" notation?

## Syntax

### "Old" Syntax

Nothing new here :
```javascript
const increment = function(number) {
    return number + 1
}

increment(1) // returns 2
```

### Arrow syntax

```javascript
const increment = number => number + 1

increment(1) // returns 2
```

I'll let [MPJ](https://www.youtube.com/watch?v=6sQDTgOqh-I) explain why we would want a better syntax for anonymous 
functions. All I can say is that it unlocks [functional style in Javascript (FR)](https://www.youtube.com/watch?v=IQ1kDpGeoCk&t=13s)  
In most case, you can replace the "Old" syntax with Arrow syntax, but there are some corner case in Javascript where
this syntax does not work at all.

### This

*Disclaimer: The whole code in this article works in a browser and in nodejs. This whole example rely on global objects
available in these environment. If you start getting errors, just check that it did not add `"use strict";` on top of
your script :wink:*

#### Introduction

`this` in Javascript deserves a whole blog to be understood, so I'm not going to cover every aspect of `this` here. The
only advice I can give is that if you already learnt a language using `this`, well unlearn what you have learnt because
it won't work that way in Javascript. :confused:

#### Giving a **context** to a function

Let's take a simple example:
```javascript
const increment = function() {
    return this.number + 1
}

const example = {
    number: 4
}

example.increment = increment // "Attach" the increment function to the object
example.increment() // returns 5
```

That example is a bit tricky, but nothing really hard to understand. As Javascript is a dynamic language, we can
add behavior to an object at runtime. In order to access the properties of the object, we need to use `this` parameter.
If I call `increment` function without giving it a **context** it won't know how to behave.

```javascript
increment() // returns NaN
```

In the above example, we did not give an execution context to increment function, so it did not know what is the 
`this.number` value.

#### Removing the **context** of a function

```javascript
const example = {
    number: 4,
    increment() {
        return this.number + 1;
    }
}

const increment = example.increment
increment() // returns NaN
```

Okay, so this time we have done the same thing but the other way around. We have defined an object with a function 
`increment`. We have removed the execution context from the method by assigning it to the constant. So when calling it
it does not know what is the value of `this.number` because we removed the **context**.  
*At this point, you should say "Hey, I already encountered that case when using AngularJs/Angular/React/VueJs/...etc".
And you are right, they all use runtime to add functions to objects and extract functions to their context all the time.*
:wink:

#### Forcing a function **context**

That's why, you probably already encountered the following syntax : 
```javascript
const example = {
    number: 4,
    increment() {
        return this.number + 1;
    }
}

const increment = example.increment.bind(example)
increment() // returns 5
```

The `bind` function allow to "freeze" the context for a given function. So even if you call it with or without providing
a context, it will always use the bound context. 

#### Now with Arrow functions

So what happen, if I introduce arrow functions in here ?
```javascript
const example = {
    number: 4,
    increment: () => this.number + 1
}

const increment = example.increment
increment() // returns NaN
```

What the heck? It does not work anymore? :anguished:  
So what happened here? Well, Arrow functions are a bit special about context. **It always use the context in which the
function was defined**. What does it means? 
If we decompose the code written above : first we created an anonymous arrow function, then we attached it to the 
`increment` property of the `example` object. When not in [strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)
in Javascript, we always have a global context. For NodeJs, it is a [Global object](https://nodejs.org/api/globals.html)
and in a browser, it is the [Window object](https://developer.mozilla.org/fr/docs/Web/API/Window), both with a well 
known API.  
So when we defined the anonymous arrow function, it was implicitly bound to that global object. And as "Arrow" function
are always already bound to a context, we cannot use the `bind` trick we used earlier.
```javascript
const example = {
    number: 4,
    increment: () => this.number + 1
}

const increment = example.increment.bind(example)
increment() // still returns NaN
```

So how do we avoid that problem? Well, that's really simple, and that's a rule that will help you keep your sanity when
using Javascript: **Never use `this`**

So the solution in this case is :
```javascript
const example = {
    number: 4,
    increment: () => example.number + 1
}

const increment = example.increment
increment() // returns 5 \o/
```

## My recommendations

In almost all cases, we can avoid `this` problem. I'll explain in the next article, how to create objects in javascript
that do not rely on `this`.  
You should always use arrow function as the syntax is far better and you keep your sanity because you don't have to
deal with moving **contexts**. There are some case, though, where you can't escape using `this` (when dealing with
frameworks for example). In these case, just use the "Old" notation.  
When dealing with functions in Javascript, never forget about **contexts**, as it can lead you in really painful
debugging sessions ! 

## References

[Documentation on MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)