# Perhaps a good idea

Modern-day programming languages are... surprisingly lacking. Tedious,
rigid, verbose. Our saviors are the tools, the IDEs we use, which are
surprisingly capable and are often the side on which the true magic
happens.

But that doesn't have to be that way, and indeed i hope things will
change sooner than later. 

So many things can be abstracted away, code-generated, simplified,
guaranteed... So many bugs to be caught during compilation and not 
during sluggish parsing of huge monotonous logs, so many hours
to be saved not writing stuff that's obvious, had been written plenty 
of times before. It can all be better, and it should. ASAP.

Programming languages come and go, and the lacking languages of today
will unquestionably be replaced by the capable languages of tomorrow.
Except Java, Java is eternal. However, for someone that's planning
to design a better language, there really doesn't seem to be a list
of what to do and what not to do(not that i would qualify as
that someone... hopefully yet).

I skimmed the web a couple of times, trying to find a complete
list of programming language features, ideas, solutions. Admittedly,
i didn't find any, so perhaps i missed it. But i found none.

So i'll make my own.

### Easy but impactful

These features are likely rather easy to implement, but will greatly improve
the workflow and/or capabilities of your programming language.

- [Exhaustive switch](/TypeChecking/ExhaustiveSwitch/exhaustive-switch.md) - 
compile-time guarantee of complete edge case handling
- [Chaining operator](/FlexibleCode/ChainingOperator/chaining-operator.md) -
a convenient and explicitly understandable way for writing fluent APIs 
and builders