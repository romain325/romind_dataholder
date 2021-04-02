# Let, Const & Var in Javascript ES6

Since Javascript ES6, we can define multiple variable type, but how do we use them and most importantly: in which cases !  
This is really not hard to understand but we need to clarify all that a lil bit !  

## Just before we start  

But before we start we need to clarify some concepts to completely understand the differences between those types.  
We need to understand the difference between **global** and **function** scope!  

A global scope represent a variable accessible from everywhere who appear from the moment it has been initialized to the moment the program ends.  
In opposite, a function scoped variable, the variable is only accessible from the function it has been initialized in and his lifespan ends at the moment the function ends.  

Let's take an example:  

```js
var global = "You";
function helloWorld() {
    let func = "Me";
    console.log("Hello World from " + func + " to " + global);
}

helloWorld() // ==> "Hello World from Me to You"
// The global var is global and so accessible from everywhere  
// The func var is function scoped and so accessible from the current function

console.log(global) // ==> "You"  
// This var still exists as it is a global one

console.log(func) // ==> ERROR
// At the moment, we're not in the scope and the variable just doesn't exist
// Like so the langage just return an error

```

Ok so now it's time to take those types one by one and understand how to use them

## Var

The well known `var` !  
Var is a globally scoped variable and is modifiable by everyone and from everywhere.  
A `var` can also be redeclared multiple times in the same code multiple times.  
This type of variable is really useful and really flexible !  
But don't overuse it, it is not the best fit for every situation and you may want to use stg that really fits your need instead of having a variable that hold everything and that won't die !  
Is that useful to keep an iterator after you've been trough your list for example ? **No!**  

## Let

From my pov, I think that let is the most useful type of variable that exists actually.  
The `let` is a function scoped variable, it is modifiable and you can interact with it from his current scope!  
His lifespan is limited to his scope too !!  
Finally I can have an iterator that just stay there and die when my function end !!  
Thanks, I can now avoid lots of silly bugs and just use variables normally :)  
But! Don't be as chill as with the `var`, you can't redeclare a `let` !  

## Const

`const` does looks like `let` !  
It is function scoped, and so interactable from his own scope. And you can't redeclare it either.  
*BUUUUUUUUUUT* be aware, as his name describe it, Const is a constant and so once defined, you can't change his value!  

_**More advanced about CONST**_:  
> If you've never used object in JS then you'll probably not understand this one.  

Of course, as I said you can't change a value once it has been declared.  
But if you declare an object as const, you can still change their inner value (with methods for exemple!)  
To be clear, the allocated space can't be changed BUT you can change what is inside of it!  
If you don't want to see what's inside your object, then you will have to use `myObject.freeze()`.  
