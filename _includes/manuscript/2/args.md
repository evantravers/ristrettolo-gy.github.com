
## Ah. I'd Like to Have an Argument, Please.[^mp] {#fargs}

[^mp]: [The Argument Sketch](http://www.mindspring.com/~mfpatton/sketch.htm) from "Monty Python's Previous Record" and "Monty Python's Instant Record Collection"

Up to now, we've looked at functions without arguments. We haven't even said what an argument *is*, only that our functions don't have any.

A> Most programmers are perfectly familiar with arguments (often called "parameters"). Secondary school mathematics discusses this. So you know what they are, and I know that you know what they are, but please be patient with the explanation!

Let's make a function with an argument:

    (room) ->
  
This function has one argument, `room`, and no body. Here's a function with two arguments and no body:

    (room, board) ->
  
I'm sure you are perfectly comfortable with the idea that this function has two arguments, `room`, and `board`. What does one do with the arguments? Use them in the body, of course. What do you think this is?

    (diameter) -> diameter * 3.14159265

It's a function for calculating the circumference of a circle given the radius. I read that aloud as "When applied to a value representing the diameter, this function *gives* (that's my word for the arrow) the diameter times 3.14159265."

Remember that to apply a function with no arguments, we wrote `(->)()`. To apply a function with an argument (or arguments), we put the argument (or arguments) within the parentheses, like this:

    ((diameter) -> diameter * 3.14159265)(2)
      #=> 6.2831853
      
You won't be surprised to see how to write and apply a function to two arguments:

    ((room, board) -> room + board)(800, 150)
      #=> 950
      
T> ### a quick summary of functions and bodies
T>
T> How arguments are used in a body's expression is probably perfectly obvious to you from the examples, especially if you've used any programming language (except, possibly for the dialect of BASIC I recall from my secondary school that didn't allow parameters when you called a procedure).
T>
T> Expressions consist either of representations of values (like `3.14159265`, `true`, and `undefined`), operators that combine expressions (like `3 + 2`), and some special forms like `[1, 2, 3]` for creating arrays out of expressions and `(`*arguments*`) ->`*body-expression* for creating functions.
T>
T> This loose definition is recursive, so we can intuit (or use our experience with other languages) that since a function has an expression on its right hand side, we can write a function that has a function as its expression, or an array that contains another array expression. Or a function that gives an array, an array of functions, a function that gives an array of functions, and so forth:
T>
T> <<(code/f1.coffee)

### call by value {#call-by-value}

Like most contemporary programming languages, CoffeeScript uses the "call by value" [evaluation strategy]. That's a $2.75 way of saying that when you write some code that appears to apply a function to an expression or expressions, CoffeeScript evaluates all of those expressions and applies the functions to the resulting value(s).

[evaluation strategy]: http://en.wikipedia.org/wiki/Evaluation_strategy

So when you write:

    ((diameter) -> diameter * 3.14159265)(1 + 1)
      #=> 6.2831853

What happened internally is that the expression `1 + 1` was evaluated first, resulting in `2`. Then our circumference function was applied to `2`.[^f2f]

[^f2f]: We said that you can't apply a function to an expression. You *can* apply a function to one or more functions. Functions are values! This has interesting applications, and they will be explored much more thoroughly in [Functions That Are Applied to Functions](#consumers).

### variables and bindings

Right now everything looks simple and straightforward, and we can move on to talk about arguments in more detail. And we're going to work our way up from `(diameter) -> diameter * 3.14159265` to functions like:

    (x) -> (y) -> x
    
A> `(x) -> (y) -> x` just looks crazy, as if we are learning English as a second language and the teacher promises us that soon we will be using words like *antidisestablishmentarianism*. Besides a desire to use long words to sound impressive, this is not going to seem attractive until we find ourselves wanting to discuss the role of the Church of England in 19th century British politics.
A>
A> But there's another reason for learning the word *antidisestablishmentarianism*: We might learn how prefixes and postfixes work in English grammar. It's the same thing with `(x) -> (y) -> x`. It has a certain important meaning in its own right, and it's also an excellent excuse to learn about functions that make functions, environments, variables, and more.
    
In order to talk about how this works, we should agree on a few terms (you may already know them, but let's check-in together and "synchronize our dictionaries"). The first `x`, the one in `(x) ->`, is an *argument*. The `y` in `(y) ->` is another argument. The second `x`, the one in `-> x`, is not an argument, *it's an expression referring to a variable*. Arguments and variables work the same way whether we're talking about `(x) -> (y) -> x`  or just plain `(x) -> x`.

Every time a function is invoked ("invoked" is a synonym for "applied to zero or more arguments"), a new *environment* is created. An environment is a (possibly empty) dictionary that maps variables to values by name. The `x` in the expression that we call a "variable" is itself an expression that is evaluated by looking up the value in the environment.

How does the value get put in the environment? Well for arguments, that is very simple. When you apply the function to the arguments, an entry is placed in the dictionary for each argument. So when we write:

    ((x) -> x)(2)
      #=> 2

What happens is this:

1. CoffeeScript parses this whole thing as an expression made up of several sub-expressions.
1. It then starts evaluating the expression, including evaluating sub-expressions
1. One sub-expression, `(x) -> x` evaluates to a function.
1. Another, `2`, evaluates to the number 2.
1. CoffeeScript now evaluates applying the function to the argument `2`. Here's where it gets interesting...
1. An environment is created.
1. The value '2' is bound to the name 'x' in the environment.
1. The expression 'x' (the right side of the function) is evaluated within the environment we just created.
1. The value of a variable when evaluated in an environment is the value bound to the variable's name in that environment, which is '2'
1. And that's our result.

When we talk about environments, we'll use an [unsurprising syntax][json] for showing their bindings: `{x: 2, ...}`. meaning, that the environment is a dictionary, and that the value `2` is bound to the name `x`, and that there might be other stuff in that dictionary we aren't discussing right now.

[json]: http://json.org/

### call by sharing

Earlier, we distinguished CoffeeScript's *value types* from its *reference types*. At that time, we looked at how CoffeeScript distinguishes objects that are identical from objects that are not. Now it is time to take another look at the distinction between value and reference types.

There is a property that CoffeeScript strictly maintains: When a value--any value--is passed as an argument to a function, the value bound in the function's environment must be identical to the original.

We said that CoffeeScript binds names to values, but we didn't say what it means to bind a name to a value. Now we can elaborate: When CoffeeScript binds a name to a value type value, it makes a copy of the value and places the copy in the environment. As you recall, value types like strings and numbers are identical to each other if they have the same content. So CoffeeScript can make as many copies of strings, numbers, or booleans as it wishes.

What about reference types? CoffeeScript cannot place a copy of an array or object in an environment, because the copy would not be identical to the original. So instead, CoffeeScript does not place reference values in any environment. CoffeeScript places *references* to reference types in environments, and when the value needs to be used, CoffeeScript uses the reference to obtain the original.

Because many references can share the same value, and because CoffeeScript passes references as arguments, CoffeeScript can be said to implement "call by sharing" semantics. Call by sharing is generally understood to be a specialization of call by value, and it explains why some values are known as value types and other values are known as reference types.

And with that, we're ready to look at *closures*. When we combine our knowledge of value types, reference types, arguments, and closures, we'll understand why this function always evaluates to `true` no matter what argument you apply it to:

    (value) ->
      ((copy) ->
        copy is value
      )(value)