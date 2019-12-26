# Exhaustive switch

``switch`` is a lovely control structure implemented in most modern languages, 
allowing the programmer to conditionally run one of multiple branches of code
based on the value of some variable, without writing a huge mess of ifs.
The statement is also better suited for optimization when used in this context.

## Problem

Suppose there's an enum and a switch:

```cs
enum SomeEnum{
    VALUE1,
    VALUE2
}

// ...

SomeEnum someEnumVar = /*...*/;
switch(someEnumVar) {
    case SomeEnum.VALUE1: Console.WriteLine("value is 1"); break;
    case SomeEnum.VALUE2: Console.WriteLine("value is 2"); break;
}
```

It is an exhaustive switch, that is, a switch that handles all possible options.
That is, until:

```cs
enum SomeEnum{
    VALUE1,
    VALUE2,
    VALUE3 // <- added
}

// ...

SomeEnum someEnumVar = /*...*/;
switch(someEnumVar) {
    case SomeEnum.VALUE1: Console.WriteLine("value is 1"); break;
    case SomeEnum.VALUE2: Console.WriteLine("value is 2"); break;
}
```

The switch is no longer exhaustive, and may, in some cases(indeed in case 
``someEnumVar==VALUE3``), allow the execution to flow through unhandled.
``default`` is **not** a proper solution:

```cs
enum SomeEnum{
    VALUE1,
    VALUE2,
    VALUE3
}

// ...

SomeEnum someEnumVar = /*...*/;
switch(someEnumVar) {
    case SomeEnum.VALUE1: Console.WriteLine("value is 1"); break;
    case SomeEnum.VALUE2: Console.WriteLine("value is 2"); break;
    default: Console.WriteLine("unexpected value"); break; // <- this is NOT handling!
}
```

Often when writing a ``switch``, the programmer wants to write code that will
handle every possible value of the variable in question. Indeed, there's a reason
both Visual Studio and IntelliJ IDEA provide the "generate missing branches" quick 
fix for switches based on an enum variable.

Code changes and possible values get added or removed sometimes. When a possible
value is removed, the programmer is greeted by a clear as day 
``Could not find symbol EnumName.REMOVED_VALUE`` error, and is thus promptly 
forced to edit the now-useless branch out(or the code won't compile).

But that doesn't happen when a new possibility is **added**. There's no notification,
no error stalling the compilation, nothing. Your code will happily compile, even though
it is not truly functional and does not handle all edge cases. But isn't that
what programming is all about - handling all edge **cases**?

If the enum is part of your project, you might be fine - as the person who added
a possibility approaches you and tells you "hey Matt, EnumName now has NEWLY_ADDED_OPTION,
make sure to handle it everywhere in your code!" Yeah... i wouldn't count on it.
Even worse off is if the enum in question is part of some third party library.
Last time i checked, they don't announce the addition of each enum entry on their 
landing page. Good luck not missing that!

And even if you didn't miss the news about the addition of a new possibility, what
if you have dozens of switches handling the same enum? Unlikely? Sure. And yet,
someone, somewhere, will have this exact problem. Furthermore, what if only a
portion of those switches is destined to be exhaustive? Will you rely on
comments? Those are easily missed.

The ``default`` keyword **doesn't solve this issue**. It's a runtime crutch,
a hotfix, a piece of tape plopped over the ``switch`` "just in case" it breaks.
Because it will break one day. Don't get me wrong, ``default`` is better than 
no ``default``, but it's not the solution. When compile-time guarantees are
possible, one should avoid runtime error handling.

## Proposed solution: exhaustive keyword

Introduce a new keyword, ``exhaustive``. When placed in front of a ``switch``,
this keyword marks it as, well, exhaustive. During compilation the 
compiler must check that such a switch handles all possible cases.

Take our switch again:

```cs
enum SomeEnum{
    VALUE1,
    VALUE2
}

// ...

SomeEnum someEnumVar = /*...*/;
exhaustive switch(someEnumVar) {
    case SomeEnum.VALUE1: Console.WriteLine("value is 1"); break;
    case SomeEnum.VALUE2: Console.WriteLine("value is 2"); break;
}
```

It is now marked as exhaustive. The compiler now is aware the switch **must**
handle all cases, and now stalls the compilation when a new possibility is
added:

```cs
enum SomeEnum{
    VALUE1,
    VALUE2,
    VALUE3 // <- added
}

// ...

SomeEnum someEnumVar = /*...*/;
exhaustive switch(someEnumVar) { 
    // Error: exhaustive switch does not handle all possibilities
    // (missing SomeEnum.VALUE3)
    //  at Example.cs:10
    case SomeEnum.VALUE1: Console.WriteLine("value is 1"); break;
    case SomeEnum.VALUE2: Console.WriteLine("value is 2"); break;
}
```

``exhaustive`` keyword makes proper sense in two contexts: when branching 
based on an enum variable, and on inheritors/implementors(C# supports that
and it is absolutely lovely, Java... doesn't? well, does but in a roundabout 
way):

```cs
// BaseClass is inherited by SomeChildClass, BetterChildClass and MockChildClass
BaseClass baseVar = /*...*/;
exhaustive switch(baseVar){
    // Error: exhaustive switch does not handle all possibilities
    // (missing BetterChildClass)
    //  at Example.cs:10
    case SomeChildClass child: Console.WriteLine("child: "+child); break;
    case MockChildClass mock: Console.WriteLine("mock: "+mock); break;
}
```

However, considering that the high-level language runtimes nowadays support
runtime loading of libraries, the exhaustiveness of such a class-casting switch
cannot be truly guaranteed. That is unfortunate, but, in my opinion, does not
undermine the idea. Indeed, the compile-time guarantees such a construct would
provide would far outweight the occasional non-exhaustiveness because of
a runtime-loaded library.

That said, it does make ``default`` relevant for such a switch. Initially i
thought ``default`` would make no sense in the context of such a switch,
but now it is clear that it might be smart to allow ``default`` at least
for class-casting ``switch`` statements. It doesn't make any sense in 
the context of an enum-based switch, though.

``exhaustive switch`` based on other types of variables seems likely unusable.
Writing an ``exhaustive switch`` for a ``byte`` seems at least somewhat possible,
but already ridiculous. And as for ``string``s... it's completely unviable.

## etc.

TypeScript actually has it! Holy shit;

https://stackoverflow.com/questions/39419170/how-do-i-check-that-a-switch-block-is-exhaustive-in-typescript

It's a hack, but it works

I am not sure if IntelliJ shows warnings about non-exhaustive ``switch`` blocks. If
it does... fair enough. Warnings don't count, though. Developers typically ignore
warnings until the day of not ignoring warnings comes, which is when those get cleared
up, only to pile up again over time. Compile errors are kept at a minimum(read: zero)
because with compile errors, the code... doesn't compile.