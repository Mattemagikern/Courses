# EDAN40 - Functional programming
# Haskell
Haskell is a functional programming language with a strict typing.


Functional languages use a different paradigm than imperative and
object-oriented languages. They use side-effect-free functions as a basic
building block in the language. This enables lots of things and makes a lot of
things more difficult.
And because functions in functional languages behave very similar to
mathematical functions it's easy to translate those into functional languages.
In some cases, this can make code more readable.

Functional programing can be reduced in short to:
```
a -> f -> b

where a and b are arbitrary values and f is a function.
```
This states that an input a to the function f will produce the output b. Not
so intresting perhaps. 
## Types and classes
Types and classes are the two most fundamental concepts in Haskell.
Haskell is both **statically** and **strongly** typed compared to most programming
languages.

**Static** — types are known at compile time. Java and Haskell are statically typed.

**Dynamic** — Python, Ruby, etc. Some people call it "unityped"; dynamic can be
"emulated" within a static setting but the reverse is not true unless you add
external static analysis tools/plugins to an otherwise dynamically typed
language. 

**Strong** — values that are intended to be treated as Cat always are; trying
to treat them like a Dog will cause a loud meeewww... I mean error.

**Weak** — this effectively boils down to 2 similar but distinct things: type
coercion and memory reinterpretation. Type coercion would look like this in c:
```C
int a = 5;
double b = 3.46;
int c = a + (int) b; //holy crap! We have forced a double to become an int!
-----                // It's like magic!
=> c = 8
```
Memory reinterpretation is pretty clear from the name but this is another
example: 
```C
int f(void* b){
        z = 2 * (*(*int)b); // Holy moly batman, you have manipulated a memory
                            // space to become something it might not be!
}
```
The magic with static and strongly typed is that the compiler will not allow
any missconduct or variable abuse from the programmer. Which garantees the
corecctness of the program. This has been the cuase for discussions among
security proffesionals, that Haskell programs may be unhackable due to this
language feature.   

### Define your own types and typeclasses
It is rediculusly simple, but it took me a lot of convincing to belive that
this is how you declare types becuase it was to easy and very plane.

```Haskell
-- Declareing a new type
data MyList = MyList [Int] deriving (Show)
-- really that simple. In this case it is just a way of grouping data together
-- in a neat fashion. 

-- Here we define functions that expects our newly created data type MyList 
-- as input which gives functions more restrictions and more easily followed
AddElem :: MyList -> Int -> MyList
AddElem list a = list ++ [a]
```
### v :: T syntax
When talking about Types and classes we need to take a look at the v :: T
syntax. It is a way of declaring the variable v to be of a certain type T.
Ex.
```Haskell
False :: Bool
True :: Bool
not :: Bool -> Bool
```

Every expression must have a type in Haskell. The type is calculated prior to
evaluating the expression by a process called type inference. This is what
makes Haskell such a popular language, because of the type inference precedes
the function evaluation it will find all type errors. This makes the language
**type safe**, which means you can never input the wrong input parameters and
cause undefined behaviour as you can in C for example.

Because type inference precedes evaluation, Haskell programs are type safe, in
the sense that type errors can never occur during evaluation.

#### Some eqvivalences 
Allright, becuase we are now playing with right associativity we can acctually
see that the following functions are eqvivalences! 
```Haskell
((.):) == (:)(.)

-- This is eqvivalent becuase haskell by nature is right associative, the
-- function is just describing its first parameter , the (:) function. When you
-- see that it is easy to expand this expression to (:)(.) 
-- where the next argument would be another function it would expand into:
-- (:) . f, its like magic but with more steps


-- The same applies to this function! 
((:):) == (:)(:)
-- The result of the first operation (:) will yield its result to become the first
-- input to the second function -> (:) (:)
-- It will not be the same as the function above becuase non of the functions
-- accept another function as an input parameter
```
#### How to evaluate types from functions #TheHannaLindWallWay
Evaluate the following function: ``(.)(.)``..

```Haskell
-- So we have the defenition of (.)
(.) :: (b -> c) -> (a -> b) -> a -> c

-- Notice the difference between (a -> c) and a -> c! The -> operator is
-- right-associative, meaning that you're free to put parens on the 
-- right-side of -> whenever you please. But when you do it on the left side
-- you state that they belonge together, this usualy indicates a function. 

-- write all the defenitions under eachother with different variables.
(.) :: (b -> c) -> (a -> b) -> a -> c
(.) :: (b'-> c') -> (a'-> b') -> a' -> c'

-- Because (.)(.) is a chain function the second function will consume the
-- first function (b->c) in our top function. That will result in the lower
-- function to be equal to the (b->c) statement in the first function.
-- In reallity it would look like this:
(.) :: (a -> b) -> a -> c
(.) :: (b'-> c') -> (a'-> b') -> a' -> c'

b = (b' -> c')
c = (a'-> b') -> a' -> c'
-- by applying this to the top function we will receive the following
(a -> b' -> c') -> a -> (a' -> b') -> a' -> c'

-- Now were done! (or we can make it look nicer like this):

(a -> b -> c) -> a -> (a' -> b) -> a' -> c

-- Speciall thanks to Hanna Lindwall the unsung hero of this exam!
```
In short: 
1. Write the functions as a list (with different types)
2. Consume variables or functions from the top expression if you can. 
3. exchange variables
4. simplify 
#### Some hard typing evaluations 
```Haskell
-- requires an Number and returns a Number
f :: a -> a
f = (8-)

-- Requires a Number and returns a Number
g :: a-> a
g = (+0).(0+)
-- a -> a
-- a' -> a' 
-- a' = a => a-> a

-- requires a function, an argument and a list, returns a list
f :: (a -> a1) -> a -> [a1] -> [a1]
f = (.) (:)
-- (.) :: (b -> c) -> (a -> b) -> a -> c
-- (:) :: a' -> [a'] -> [a']
-- =>
-- (.) :: (a->b) -> a -> c
-- (:) :: a' -> [a'] -> [a']
-- =>
-- c = [a'] -> [a']
-- b = a'
-- => 
-- (a -> a') -> a -> [a'] -> [a']

g :: (([a] -> [a]) -> c) -> a -> c
g = (.(:))  == (:)(.)
-- (:) :: a -> [a] -> [a]
-- (.) :: (b -> c) -> (a -> b) -> a -> c
-- Note a slimy trick here! becuase it is right associative the (:) function is
-- now going to be the (a->b)!
-- b = [a']->[a']
-- a = a'
-- (:) :: a -> [a] -> [a]
-- (.) :: (b -> c) -> a -> c
-- =>
-- (.(:)) :: ([a']->[a'] -> c) -> a' -> c


h :: [a -> [a] -> [a]] -> [a -> [a] -> [a]]
h = ((:):) => (:)(:)
-- (:) :: a -> [a] -> [a]
-- (:) :: a' -> [a'] -> [a']
-- a = a' -> [a'] -> [a']
-- =>
-- [a' -> [a'] -> [a']] -> [a' -> [a'] -> [a']]


i:: (a -> b -> c) -> a -> (a1 -> b) -> a1 -> c
i = (.)(.)
-- See above!


j :: (a -> b) -> a -> b
j = (($)$($))
--- Can be rewritten as ($)($)($)
-- ($) -- (a -> b) -> a -> b
-- ($) -- (a' -> b') -> a' -> b'
-- ($) -- (a'' -> b'') -> a'' -> b''
-- => 
-- a' = (a'' -> b'')
-- b' = a'' -> b''
-- => 
-- ($) -- (a -> b) -> a -> b
-- ($) -- (a'' -> b'') -> a'' -> b''
-- => 
-- a = (a'' -> b'')
-- b = a'' -> b'' 
-- ($) -- a -> b
-- ($) -- (a'' -> b'') -> a'' -> b''
-- =>
-- ($)($)($) :: (a'' -> b'') -> a'' -> b''

k ::  Ord a => [a -> a -> Bool]
k = ([]>>=)(\_->[(>=)])
-- I have no idea where to start! #learn monads
```
## Syntax
Function application has higher priority than all other operators in the
language. An example:

```
f a + b means (f a) + b rather than f (a + b)
```

When defining a new function, the names of the function and its arguments must
begin with a lower-case letter.
### Functions
#### Pattern Matching and Wildcards
Note the use of the wildcard pattern _ in the recursive case, which reflects
the fact that calculating the length of a list does not depend upon the values
of its elements.
An example: 
```Haskell
f 2 _ = "stuff"
f _ [] = Nothing
```
The first bad exampl in this summary, but hey what do you wan't me to do? 
As you can se we have now created a function that has two "base-cases", where
the first parameter is 2 we are going to return "stuff" and the second
base-case is when we input an empty list we will return Nothing. The "\_" are
wildcards they means more or less: Anything.

Using wildcards or static values to define base-cases above the function is called pattern
matching. This is really usefull for writing easily read code!
### Currying
All functions uses a single input.

Currying is the process of transforming a function that takes multiple
arguments into a function that takes just a single argument and returns
another function if any arguments are still needed.

That is: 
```Haskell
Add a b = a + b
``` 
Unlike imperative or object oriented languages `` Add 1 `` is a valid input to
the function. 

What happens in haskell now is that the Add function is now waiting for a new
function to apply its own function to so to speak. That is called Currying.


#### uncurrying
uncurry converts a curried function to a function on pairs.
This creates a function that can take 2 functions.

It is used to manipulate functions input parameters.
### Guards |
As an alternative to using conditional expressions, functions can also be
defined using guarded equations, in which a sequence of logical expressions
called guards is used to choose between a sequence of results of the same
type. If the first guard is True, then the first result is chosen; otherwise,
if the second is True, then the second result is chosen, and so on.

The symbol | is read as such that, and the guard otherwise is defined in the
standard prelude simply by otherwise = True. Ending a sequence of guards with
otherwise is not necessary, but provides a convenient way of handling all
other cases, as well as avoiding the possibility that none of the guards in
the sequence is True, which would otherwise result in an error.

The main benefit of guarded equations over conditional expressions is that
definitions with multiple guards are easier to read.

An example: 
```Haskell
f a 
        | a < 2 = 1 
        | otherwise = a * f(a-1)
```
This is a stupid comparator but it is just an example. 

### \<- operator
The \<- operator is read as "is drawn from". An expression like: 
`` x <- [1..5]`` is called a generator.

### Lambda expressions
As an alternative to defining functions using equations, functions can also be
constructed using lambda expressions, which comprise a pattern for each of the
arguments, a body that specifies how the result can be calculated in terms of
the arguments, but do not give a name for the function itself. In other words,
lambda expressions are nameless functions.

The symbol \ represents the Greek letter lambda, written as λ. Despite the
fact that they have no names, functions constructed using lambda expressions
can be used in the same way as any other functions.

Secondly, lambda expressions are also useful when defining functions that
return functions as results by their very nature, rather than as a consequence
of currying.

Finally, lambda expressions can be used to avoid having to name a function
that is only referenced once in a program.

An example:
```Haskell
map (\a -> a*2) [1..4]
```
Also idiotic example but hey, I'm not getting payed or anything so your stuck
with this.

### list comprehensions
A list comprehension can have more than one generator, with successive
generators being separated by commas. An example: 
`` [(x,y) | x <- list1, y <- list2] ``

List comprehensions can also use logical expressions called guards to filter
the values produced by earlier generators. If a guard is True, then the
current values are retained; if it is False, then they are discarded. 
An example: 
```Haskell
[(x,y,c) | x <- [1..10], y <- [1..10], c <- [1..10], x^2 + y^2 == c^2]
``` 
This would produce a list of tuples with the elements representing the length
of each side in a pytagoran triangle. 

### Dot and $ operator
The dot operator is a way of chainging functions together, creating compound
functions easily. 
```Haskell
f = (*2)
g = (^3)
h = (/3)

j = (f . g . h)

j 2 => 0.5925925925925926 == f(g(h(2))) == 2* <- ^3 <- 2/3
```

Many confuses the dot operator with the $ operator, they are similar but
they are fundamentaly different. 

The **$ operator** can be viewed as a parentesis from the $ till the
end. It is comonly used as to reduce the parentesis needed in an function
declaration. 
```Haskell
f = (*2)
g = (^3)
h = (/3)

j a b = f . g . h $ a + b 

j 2 3 => 9.259259259259261 == 2* <- ^3 <- (2+3)/3
```
### Do and >> (then) cluases
The do and >> cluases are monadic. 
### Pointfree style
Pointfree style has to be the worst name for the purpouse! Pointfree should
acctually be named exessivepoint style. Due to how it accutally works. 

The aim of pointfree style is to only display the functions in a function (
OMG is this really a thing? its like inception). 

An example:
```haskell
f x y = (3+x) / y

-- This can also be written as
f = (/) . (+3)
```
This works because currying will applie the first argument to +3 due to + only
has 2 operands and the remaining inparameter to the division between the result
of our + operator and our remaining inparameter. This is called chaining of
functions and it is like everything else read from right to left. 

To summarize:
**USE DOTS WHERE EVER POSSIBLE! CHAINING IS KING! INPARAMETERS IS BAD!**
## Higher-order functions
Formally speaking, a function that takes a function as an argument or returns
a function as a result is called a higher-order function. In practice,
however, because the term curried already exists for returning functions as
results, the term higher-order is often just used for taking functions as
arguments.

### map
### foldl and faldr
foldl captures the common pattern of combining successive elements of a list. It folds up a list from the **left**.
```Haskell
fldl (+) 0 [5..10] => ((((((0+5)+6)+7)+8)+9)+10) = 45
```
As you can see it "folds" the itterable and applies the function to its
inital value and the result of its initial value to the following items in
the itterable. Also, the function above produces the same result as the sum
function in the standard prelude.

**foldr** on the other hand folds up a list from the **right**. 
```Haskell
fldr (+) 0 [5..10] => ((((((0+10)+9)+8)+7)+6)+5)
```
In this example it dosen't matter the result remains 45 but if we were to use
the function ``(-)`` it would produce **-15** for foldl and **3** for folr.

### Polymorphic functions & Polytypic functions
Polymorphism is a key concept of Haskell.

A value is polymorphic if there is more than one type it can have.
Polymorphism is widespread in Haskell and is a key feature of its type system.
Most polymorphism in Haskell falls into one of two broad categories:
parametric polymorphism and ad-hoc polymorphism.

Parametric polymorphism refers to when the type of a value contains one or
more (unconstrained) type variables, so that the value may adopt any type that
results from substituting those variables with concrete types. this means any
type in which a type variable, denoted by a name in a type beginning with a
lowercase letter, appears without constraints (i.e. does not appear to the
left of a =>).

Since a parametrically polymorphic value does not "know" anything about the
unconstrained type variables, it must behave the same regardless of its type.
This is a somewhat limiting but extremely useful property known as
parametricity. 


Ad-hoc polymorphism refers to when a value is able to adopt any one of several
types because it, or a value it uses, has been given a separate definition for
each of those types.
## Functors, Applicatives and Monads
### Functors
"The Functor typeclass represents the mathematical functor: a mapping between
categories in the context of category theory. In practice a functor represents
a type that can be mapped over." is Hoogles answer to what a functor is and
this is roughly translated into:
"Functors abstract the idea of mapping a function over each element of a
structure." or applies functions to an itterable.

An example of a functor: 
```haskell
fmap :: Functor f => (a -> b) -> f a -> f b
```
The difference between a functor and a higher-order function like map is that
a functor can be applied to data structures that is not defined in the standard
prelude.

**All functors has to obay the following!**
```haskell
fmap id      = id
fmap (p . q) = (fmap p) . (fmap q)
``` 
### Monad

"In category theory a monad is an endofunctor (a functor mapping a category to
itself), together with two natural transformations. Monads are used in the
theory of pairs of adjoint functors, and they generalize closure operators on
partially ordered sets to arbitrary categories." - Wikipedia (Monad (category
theory)) 


The term monad is a bit vacuous if you are not a mathematician. An alternative
term is computation builder. It is used to separate *Pure* and *impure* code.

Pure code is defiend as: 
* takes input only from it's parameters
* outputs only via it's return value

That is, a pure function in haskell is only operates within its function scope.

Impure code on the other hand is any thing that can operate outside its
function scope as a function using a external variable. This is considdered
impure due to the result of the function can't be predicted just by looking at
the function, becuase you also has to know the external variables parameter.
This opens up for sideeffects which is the main reason to why we use a
functional language instead of a imperative language.

*My rule of thumb is if you can plot your result of the function on a graph
calculator it is pure code.*

In layman's terms, a monad is just a type for which the >>= operation is
defined.  In itself >>= is just a cumbersome way of chaining functions, but
with the presence of the do-notation which hides the "plumbing", the monadic
operations turns out to be a very nice and useful abstraction, useful many
places in the language, and useful for creating your own mini-languages in the
language.

**do notation => monadic function**
### do blocks and >> 
Every line in a do loop is a monadic value. 

do loop is used to glue together monadic values in sequence.
do notations are just different syntax from chaining monadic syntax ( >> ),
mainly used to replace lambda expressions. 

do loops and >> are the same thing. 

```Haskell
do [1..4]; return "hello" 

-- is the same as 

[1..4] >> return "hello"
```
### >>=
This is the legendary bind operator, all monads have to implement this
function.

The bind operator itterates through a monadic object and performes a function
upon it. The exact defenition is:

```Haskell
>>= :: contact (map f list)
```
It takes an itterable and a function.

An example: 
```Haskell
do [1,2,3];
    return ("Stuff")
-- will return ["stuff", "stuff", "stuff"]

[1,2,3] >>= (\_ -> return ("stuff"))
-- will return ["stuff", "stuff", "stuff"]

[1,2,3] >>= return ("stuff")
-- will return "stuffstuffstuff"

why?

According to the grandmaster of Haskell, Emil Hammarström C14, the difference
between these functions is that the (\_ -> return "Stuff") will by the
definintion of return(
return :: Monad m => a -> m a
) produce a list of its imput value.. There fore the type difference becomes:

:t ([1,2,3] >>= return ("stuff"))  == [Char]
while :

:t ([1..3] >>= (\_-> return "stuff") == [[Char]]
```
### Applicatives
"Describes a structure intermediate between a functor and a monad
(technically, a strong lax monoidal functor). Compared with monads, this
interface lacks the full power of the binding operation >>=, but it has more
instances.  It is sufficient for many uses, e.g. context-free parsing, or the
Traversable class.  Instances can perform analysis of computations before they
are executed, and thus produce shared optimizations."


In addition to providing the functions pure and \<*\>, applicative functors are
also required to satisfy four equational laws: The first equation states that
pure preserves the identity function, in the sense that applying pure to this
function gives an applicative version of the identity function. The second
equation states that pure also preserves function application, in the sense
that it distributes over normal function application to give applicative
application. The third equation states that when an effectful function is
applied to a pure argument, the order in which we evaluate the two components
doesn’t matter. And finally, the fourth equation states that, modulo the types
that are involved, the operator <*> is associative. It is a useful exercise to
work out the types for the variables in each of these laws. The applicative
laws together formalise our intuition regarding the function pure :: a -> f a,
namely that it embeds values of type a into the pure fragment of an effectful
world of type f a. The laws also ensure that every well-typed expression that
is built using the function pure and the operator <*> can be rewritten in
applicative.

## Multithreding in haskell
**seq** - is usually introduced to improve performance by avoiding unneeded
laziness. seq solely forces evaluation of its first argument, returning the
second one.


**par** - par is generally used when the value of a is likely to be required
later, but not immediately. Also it is a good idea to ensure that a is not a
trivial computation, otherwise the cost of spawning it in parallel overshadows
the benefits obtained by running it in parallel.


**pseq** - Semantically identical to seq, but with a subtle operational
difference: seq is strict in both its arguments, so the compiler may, for
example, rearrange a `seq` b into b `seq` a `seq` b. This is normally no
problem when using seq to express strictness, but it can be a problem when
annotating code for parallelism, because we need more control over the order of
evaluation; we may want to evaluate a before b, because we know that b has
already been sparked in parallel with par.

**spark** - Sparks create entries in the work queues for each thread, from
which they'll take tasks to execute if the thread becomes idle. It is a form of
appending a entrie to a thread pool in java. 
