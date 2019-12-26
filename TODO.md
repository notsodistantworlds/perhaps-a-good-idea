### Method borrowing

Inheritance is often used not for polymorphism(which IMO is the main
reason for that paradigm), but for borrowing functionality.

Developers are lazy, and that's fine. What isn't fine is 
having to either completely commit to inheriting a class, or
having to rewrite the entire damn thing because there's a single
private method you can't override.

Method borrowing could, perhaps, solve it.

### Compile-time validated builders

Fuck constructors. Builders are the way to go, they just need
a tiny bit better integration and more guarantees.

### Properly integrated code generation

Developers should be able to write compile-time scripts
inlined directly into their application source codes.
Imagine something along the lines of cpp preprocessor
(``#include #define``), but much better.

Ideally, the same language the application is written in would
also act as the language of preprocessor scripts.

Although an untrivial problem to solve(the halting problem
and generally the security of the whole ordeal certainly make
this proposal questionable), the boost to a developer's productivity
would, IMO, be worth it.