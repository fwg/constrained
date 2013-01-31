constrained.js
==============

This is an entry for the January 2013 issue of 
[PLT Games](http://pltgames.com/competition/2013/01).

Due to an `E_NOTIME` resource allocation failure, this month's submission 
will only be this README (at least before the deadline). Feel free to give 
this a completeness score of 0.

At the beginning of this month I had the sentence "Constraints as first-class
citizens" stuck in my head.

So here goes: an extension to JavaScript for function constraints. 

## General idea

Have a way to define and compose pre- and postconditions for a function. These
would be checked when the function is called. A set of preconditions and a set
of postconditions make up a `Constraint`. A constraint's postconditions are only
evaluated when all preconditions evaluate to true.

If a constraint's preconditions match and any of its postconditions evaluates
to false, there should be a runtime exception of type `ConstraintError`.

Applying a constraint to a function produces a `Constrained` callable object.

## The pieces

### Constraint

A constraint is an object with `[[Class]]` `Constraint`. It has a literal
contructor form but can also be created through `new Constraint`.

The literal is a `§` followed by a `ConstraintBody` that starts with `{`, ends
with `}` and has multiple `PrimaryExpressions` separated by `;` in it. 
Note that these are _not_ `Statements`!

Any contained expression may bind names from the current scope. Additionally
the parameters of the constrained function are available by the names `$0` to
`$n` where `n` is the function's arity minus 1. The function call's
`arguments` is available through an immutable `$arguments` object of type
`Array`. Using `arguments` will bind to the `arguments` of the enclosing
function. (Throwing a `ReferenceError` when not in a function.)

The return value can be accessed through `$_`.

Any expression that binds `$_` is a postcondition. Any other expression is a
precondition. Conditions are evaluated in the order they appear in the source.
Though of course the preconditions are checked before the function execution
and the posconditions after.

Example:

    var c1 = §{ $0 > 0; someVariable.hasProperty(); $_ == 0 };

Using `Constraint` as a constructor with `new` expects the first parameter to 
be a callable. `new Constraint(func)` will produce a constraint with no 
preconditions and a single postcondition `func($_)`.

### Applying constraints to a function

There is a new operator `<>` that expects a function or `Constrained` as the
LHS and a constraint as the RHS. It produces a `Constrained`.

### Composing constraints

There is a new operator `#` that composes constraints to a new constraint
object and ensures that the *evaluation order* is the same as the *composition
order*. LHS and RHS must be constraints. Obviously `#` has precedence over
`<>` so that following example is valid:

    function fun(x) { return x/0.5 } <> §{ typeof $_ == 'number' }
                                      # §{ $_ < $0 }

This makes `fun` be the _constrained_ function.

(So there is probably a distinction between `ConstrainedExpression` and
`ConstrainedStatement`... though I would rather only have expressions. That
would make the above not define `fun` in the surrounding scope though.)

### Constrained

Interface should be mostly the same as `Function` except it cannot be used as
a constructor. `typeof` should be ideally be `function`. (Or is there another
test for `[[Callable]]`?)

### That's it.

It's just ideas right now. There are quite some rough corners and implementing
this consistently with ecma-262 will be a challenge.

Some day, some day.





