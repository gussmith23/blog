How I understand dependent types. An example.

I like the simple example of a type that depends on a nat whose value is in a certain range.
Done in Coq.
First we can explain that greater-than/less-than are types themselves, to be constructed.
We can only construct a greater-than with nats that actually uphold the relation.

Now, we can define "strict" nat subtraction (i.e. a-b where a >= b, so we don't have to handle negative).
Its type will include two nats, of course.
We could define it as nat -> nat -> nat.
But then we couldn't actually reason about the values.
So instead we'll cosntruct it as forall a b, greater-than-or-equal a b -> nat.
This new syntax still confuses me to this day, and I should explain it thoroughly.
Explain how -> is just sugar; forall is for when you need to "name" the "variable" because you're going to refer to it later.
This doesn't happen in Haskell, for example; you always just use arrows.
Why? Because you only need to name the variable when you're going to refer to it in a later part of the type declaration.
Which can only happen in a dependent type system.
So that type can be read as:
take two nats, return a nat,
but _also_ take a piece of evidence that shows that a >= b.
We construct this evidence exactly the same way that we'd construct

Hm, wait, is that actually how you'd implement that?
If you wanted to actually call it at runtime, you don't provide proof...do you?
I'm not sure, I haven't written enough actual, real Coq.
