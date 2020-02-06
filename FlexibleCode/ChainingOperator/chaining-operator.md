# Chaining operator

Method chaining is a commonly used pattern for writing concise yet readable code,
primarily for builders.

## Problems

We have 2 problems at hand, let's discuss them both.

**Problem #1**: return of own class is ambiguous. Whenever a method is declared
in class ``T`` as one that returns ``T``, it isn't immediately obvious whether
the method returns the object on which it is called, or creates a new object
and returns it instead.

To ensure you understand the issue, acknowledge the following snippet:

```cs
class FluentAPICounter{
    int i = 0;
    public FluentAPICounter Increment(){
        i++;
        return this;
    }
    public FluentAPICounter Merge(FluentAPICounter other){
        var merged = new FluentAPICounter();
        merged.i = this.i + other.i;
        return merged;
    }
}
```

When you're looking at the implementation, it's clear what the intent is
(even though the intent is evidently quite stupid). But suppose you don't have
any access to sources, nor any documentation. Then what you see becomes:

```cs
class FluentAPICounter{
    public FluentAPICounter Increment();
    public FluentAPICounter Merge(FluentAPICounter param1);
}
```

And you have no clue whether either method returns the same object, 
or a new object, or an existing flywheel. Admittedly, one could argue that
the client *shouldn't* know these details, and/or that it should(and likely
would) be obvious from method name and context what the method will return.

Perhaps, but there's an even more convincing argument:

**Problem #2**: if an API is fluent, you can use it as a non-fluent API
(you just ignore the return values), but if it isn't fluent, there's nothing 
you can do.

If the developer of the library you're using felt the need to ``return this``
in each and every of his method, you'll enjoy a nice, optional, concise fluent
API throughout your code. But if he didn't, and he most likely didn't(changing
all ``void``s to own class name and adding a ``return this`` *to almost every 
method* turns out to be quite the effort), you'll be stuck writing pointlessly
verbose APIs.

## Proposed solution

Opponents of fluent API argue that overloading the ``.`` operator is stupid,
since it introduces unnecessary ambiguity: are we going deeper down the 
composition tree, or are we declaring an "and also do" statement?

Fair enough.

But why overload the ``.`` operator? Indeed, i fully agree, ambiguous mixing of
these similar-yet-different concepts is counterproductive. So instead of 
overloading the ``.``, let's create a new operator.

Introducing: the ``.&``

```cs
obj
    .SomeMethodOfOriginalObject();
    .&SomeMethodReturningOtherObject() // result of method ignored
    .&SomeMethodOfOriginalObject();
```

``.&`` is an operator that ignores the result of the method on the left(say, 
``someMethodReturningOtherObject()``) but instead takes the ``this`` of 
that method(in this case, ``obj``) and treats that as the return value of the 
method on the left.

In short, ``.&`` is the "and also do" idea we mentioned earlier.

The operator would improve lives both of those who are developing fluent APIs
(don't have to write ``return this`` anymore), and for those who are using
said APIs(you can chain *any* method you want, as long as it isn't static).

One could even go as far as *prohibiting* methods from returning ``this``,
ever, so as to eliminate all ambiguity. However, that would lower flexibility,
too. It's not unreasonable to imagine a method of sorts:

```cs
public ThisClass Filter(AuthResult auth){
    switch(auth) {
        case AuthResult.INVALID_TOKEN: return INVALID_TOKEN_OBJECT;
        case AuthResult.FORBIDDEN: return FORBIDDEN_OBJECT;
        case AuthResult.SUCCESS: return this;
    }
}
```

Whether that use case would make any sense in real world is certainly up to debate,
but it does showcase the flexibility present in being able to ``return this`` 
conditionally.

As such, i think that languages should still allow an explicit ``return this``
just in case some developer needs it one day.

This proposition does have a slight downside, though. Autocompletion somewhat breaks
when this operator is introduced. You won't see any chainable methods
when you type ``.``, and you won't see any member methods when you type ``.&``.
Worst case, you see *both method types* mixed into the same list, always, which 
would lead to quite a bit of confusion.

All that said, slight autocompletion inconvenience is well worth the benefits.
Furthermore, you won't be using ``.&`` often, so it's probably not too big of
a deal to have separate autocompletion lists at hand.

By the way, Wikipedia mentions [logging](https://en.wikipedia.org/wiki/Fluent_interface#Logging)
as a counter-argument for fluent interfaces, and as usual it's a pretty compelling 
argument. However, i have a solution for that in mind, too. Consider:

```cs
obj.chainMethod1()
    .&chainMethod2()
    .& { Console.WriteLine((this==obj)+" is true!"); }
    .&chainMethod3()
    .&chainMethod4()
    .
    .
    .
```

As you can see, what we do is declare a sort-of-lambda which will execute
after ``chainMethod2()``, before ``chainMethod3()``, and with ``obj`` set to
``this``.

Yes, you could break the chain, create some temp variables and fiddle around
with the code so that you can insert a line of logging inbetween. Or just use
a lambda.

## Alternate syntax

``->`` is sadly completely out of the question, since some languages use it
instead of ``.`` But, it would've fit nicely.

``.&`` is a pretty decent option, but might be a bit confusing, unreadable or
easy to miss. Consider alternatives:

 - ``&.`` because its pronunciation sounds closer to what it does("and do", "and-dot")
 - ``&also.`` might be a bit too verbose, but certainly shows the intent clearly.
 - ``