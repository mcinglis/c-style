# C Style

These are my favorite C programming practices. Some rules are as trivial as style, while others are more intricate. I follow a few rules religiously, and others I use as a guideline. I prioritize correctness, readability, simplicity and maintainability over speed because [premature optimization is the root of all evil](http://c2.com/cgi/wiki?PrematureOptimization).

**Write correct, readable, simple and maintainable software, and tune it when you're done**, with benchmarks to identify the choke points. Also, modern compilers *will* change computational complexities. Simplicity can often lead you to the best solution anyway: it's easier to write a linked list than it is to get an array to grow, but it's harder to index a list than it is to index an array.

Backwards compatibility (e.g. ANSI C) is rarely important to me. In my opinion, backwards compatibility holds everyone back. I think we should use new technologies and new techniques if we can, to move everyone forward, if only a bit.

If you don't agree with something here, that's perfectly fine. Pick and choose what you like, and what works for your own situation. These rules aren't intended to be universal admonitions about quality: they're just my preferences, and work well for what I do, and what I care about.

Writing this guide has made me deeply consider, and reconsider, best C programming practices. I've changed my opinion multiple times on many of the rules in this document.

So, I'm certain I'm wrong on even more points. This is a constant work-in-progress; issues and pull-requests are very welcome. This guide is licensed under the [Creative Commons Attribution-ShareAlike](/license.md), so I'm not liable for anything you do with this, etc.

---



#### Always develop and compile with all warnings (and more) on

No excuses here. Always develop and compile with warnings on. It turns out, though, that `-Wall` and `-Wextra` actually don't enable "all" warnings. There are a few others that can be really helpful:

``` make
CFLAGS += -Wall -Wextra -Wpedantic \
          -Wformat=2 -Wno-unused-parameter -Wshadow \
          -Wwrite-strings -Wstrict-prototypes -Wold-style-definition \
          -Wredundant-decls -Wnested-externs -Wmissing-include-dirs

# GCC warnings that Clang doesn't provide:
ifeq ($(CC),gcc)
    CFLAGS += -Wjump-misses-init -Wlogical-op
endif
```

Compiling with optimizations on can also help to detect errors:

``` make
CFLAGS += -O2
```



#### Use GCC's and Clang's `-M` to automatically generate object file dependencies

The GNU Make Manual [touches](https://www.gnu.org/software/make/manual/make.html#Automatic-Prerequisites) on how to automatically generate the dependencies of your object files from the source file's `#include`s. The example rule given in the manual is a bit complicated. Here's the rules I use:

``` make
depfiles = $(objects:.o=.d)

# Have the compiler output dependency files with make targets for each
# of the object files. The `MT` option specifies the dependency file
# itself as a target, so that it's regenerated when it should be.
%.dep.mk: %.c
	$(CC) -M -MP -MT '$(<:.c=.o) $@' $(CPPFLAGS) $< > $@

# Include each of those dependency files; Make will run the rule above
# to generate each dependency file (if it needs to).
-include $(depfiles)
```



#### Write to the most modern standard you can

C11 is better than C99, which is (far) better than C89. C11 support is still coming along in GCC and Clang, but many features are there. If you need to support other compilers in the medium-term, write to C99.

Always write to a *standard*, as in `-std=c11`. Don't write to a dialect, like `gnu11`. Try to make do without non-standard language extensions: you'll thank yourself later.



#### We can't get tabs right, so use spaces everywhere

The idea of tabs was that we'd use tabs for indentation levels, and spaces for alignment. This lets people choose an indentation width to their liking, without breaking alignment of columns.

``` c
int main( void ) {
|tab   |if ( pigs_can_fly() == true ) {
|tab   ||tab   |developers_can_use_tabs( "and align columns "
|tab   ||tab   |                         "with spaces!" );
|tab   |}
}
```

But, alas, we (and our editors) rarely get it right. There are four main problems posed by using tabs and spaces:

- Tabs for indentation lead to inconsistencies between opinions on line lengths. Someone who uses a tab width of 8 will hit 80 characters much sooner than someone who uses a tab width of 2. The only way to avoid this is to require a tab-width, which eliminates the benefit of tabs.
- It's much harder to configure your editor to correctly handle tabs and spaces for each project, than it is to just handle spaces. See also: [Tabs vs Spaces: An Eternal Holy War](http://www.jwz.org/doc/tabs-vs-spaces.html)
- It's harder to align things using only the space bar. It's much easier to hit tab twice than to hold the space bar for eight characters. A developer on your project *will* make this mistake eventually. If you use spaces for indentation and alignment, you can hit the tab key in either situation, which is quick, easy and not prone to errors.
- It's easier to prevent tab/space errors on projects that use only spaces, because all they need to do is detect for any tabs at all. To prevent against tabs used for alignment on a project that uses tabs, you'll need to come up with a regular expression.

Cut the complexity, and use spaces everywhere. You may have to adjust to someone else's indent width every now and then. Tough luck!


#### Never have more than 79 characters per line

Never write lines longer than 79 characters.

80-characters-per-line is a de-facto standard for viewing code. Readers of your code who rely on that standard, and have their terminal or editor sized to 80 characters wide, can fit more on the screen by placing windows side-by-side.

You should stick to a maximum of 79 characters so that there's always a space in the last column. This makes it more obvious the line doesn't continue onto the next line. It also provides a right-hand margin.

If you go over 80 characters, you're making your code significantly harder to read for people who try to rely on the 80-columns standard. Either your line will wrap, which is hard to read, or your readers will have to scroll the window to the right to get the last few characters. Either of these results in code that's harder to read than if you had just worked out a line-break yourself.

It's harder to read long lines because your eyes have to travel further to get to the start of the next line, and the further they have to go, the more likely you'll have to visually readjust. 100-wide and 120-wide styles are easier to write, but harder to read.

It can be very tempting to let a line here or there go over 79 characters, but your readers will pay the price every time they have to read such a line. Treat 79 characters as a hard limit - no ifs or buts. Work out how best to break long lines, and your readers will thank you.

Do what everyone else is doing, and write for 80-column views, and we'll all be better off.

* [Emacs Wiki: Eighty Column Rule](http://www.emacswiki.org/emacs/EightyColumnRule)
* [Programmers' Stack Exchange: Is the 80 character limit still relevant?](http://programmers.stackexchange.com/questions/604/is-the-80-character-limit-still-relevant-in-times-of-widescreen-monitors)


#### Use `//` comments everywhere, never `/* ... */`

Stick to single-line comments, and cut the complexity. Compared to single-line comments, multi-line comments:

- are rarely used with a blank margin, so they're just as character-heavy
- have a style, which has to be specified and adhered to
- often have `*/` on its own line, so they're more line-expensive
- have weird rules about embedded `/*` and `*/`
- are harder/impossible to block-edit, and to extend
- are more visually-cluttering than `//`

You have to use `/* ... */` for inline comments in multi-line `#define`s, though:

``` c
#define MAGIC( x ) \
    /* Voodoo magic happens here. */ \
    ...
```

But I often prefer to just add `//` comments after the macro body describing the tricky bits. I think this makes the macro body easier to read, but still provides the (much-needed) documentation.



#### Program in American English

Developing in the same language, using the same spelling and vocabulary, is important. This is especially true in free-software projects with contributors from around the world. You should use the same language consistently for your project, in code, comments and documentation.

So, for American English, write `color`, `flavor`, `center`, `meter`, `neighbor`, `defense`, `routing`, `sizable`, `burned`, and so on ([see more](https://en.wikipedia.org/wiki/American_and_British_English_spelling_differences)). I'm Australian, but I appreciate that most programmers will be learning and using American English. Also, American English spelling is consistently more phonetic and consistent than British English. British English tends to evolve towards American English for this reason, I think.



#### Comment non-standard-library `#include`s to say what symbols you use from them

Namespaces are one of the great advances of software development. Unfortunately, C missed out (scopes aren't namespaces). But, because namespaces are so fantastic, we should try to simulate them with comments.

``` c
#include <test.c/test.h> // Test, tests_run
#include "trie.h" // Trie, Trie_*
```

This provides a few benefits:

- readers aren't forced to refer to documentation or use `grep` to find out where a symbol is defined (or, if you don't follow the rule below, where it comes from): your code just tells them
- developers have a hope of being able to determine which `#include`s can be removed and which can't
- developers are forced to consider namespace pollution (which is otherwise ignored in most C code), and encourages them to only provide small, well-defined headers

The downside is that the `#include` comments aren't checked or enforced. I've been intending to write a checker for this for quite some time, but for now, there's nothing to stop the comments from becoming wrong - either mentioning symbols that aren't used anymore, or not mentioning symbols that are used. In your project, try to nip these problems in the bud, to stop it from spreading. You should always be able to trust your code.  This maintenance is annoying, for sure, but I think `#include` comments are worth it in aggregate.

Finding where things come from is always one of my main challenges when learning a codebase. It could be a whole lot easier. I've never seen any projects that write `#include` comments like this, but I'd love to see it become a thing.



#### `#include` the definition of everything you use

Don't depend on what your headers include. If your code uses a symbol, include the header file where that symbol is defined. Then, if your headers change their inclusions, your code won't break.

Also, combined with the `#include` comment rule above, this saves your readers and fellow developers from having to follow a trail of includes just to find the definition of a symbol you're using. Your code should just tell them where it comes from.



#### Avoid unified headers

Unified headers are generally bad, because they relieve the library developer of the responsibility to provide loosely-coupled modules clearly separated by their purpose and abstraction. Even if the developer (thinks she) does this anyway, a unified header increases compilation time, and couples the user's program to the entire library, regardless of if they need it. There are numerous other disadvantages, touched on in the points above.

There was a good exposé on unified headers on the [Programmers' Stack Exchange](http://programmers.stackexchange.com/questions/185773/library-design-provide-a-common-header-file-or-multiple-headers). An answer mentions that it's reasonable for something like GTK+ to only provide a single header file. I agree, but I think that's due to the bad design of GTK+, and it's not intrinsic to a graphical toolkit.

It's harder for users to write multiple `#include`s just like it's harder for users to write types. Bringing difficulty into it is missing the forest for the trees.



#### Provide include guards for all headers to prevent double inclusion

[Include guards](https://en.wikipedia.org/wiki/Include_guard) let you include a header file "twice" without it breaking compilation.

``` c
// Good
#ifndef INCLUDED_ALPHABET_H
#define INCLUDED_ALPHABET_H

...

#endif // ifndef INCLUDED_ALPHABET_H
```

[Rob Pike argues against include guards](http://www.lysator.liu.se/c/pikestyle.html), saying you should just never include files in include files. He says that include guards still "result in thousands of needless lines of code passing through the lexical analyzer".

In fact, [GCC will detect include guards](http://gcc.gnu.org/onlinedocs/cppinternals/Guard-Macros.html), and won't read such files a second time. I don't know if other compilers perform this optimization.

I don't think it's a good idea to require your users include the dependencies of your header files. Your header file's dependencies shouldn't really be considered "public". It would enforce the rule "don't depend on what your header files include", but it falls apart as soon as header files are using things you don't need, like `FILE` or `bool`. Users shouldn't have to care about that if they don't need it themselves.

So, always write include guards, and make your users' lives easy.



#### Always comment `#endif`s of large conditional sections



#### No global or static variables if you can help it (you probably can)

Global variables are just hidden arguments to all the functions that use them. They make it really hard to understand what a function does, and how it is controlled.

Mutable global variables are especially evil and should be avoided at all costs. Conceptually, a global variable assignment is a bunch of `longjmp`s to set hidden, static variables. Yuck.

You should always try to design your functions to be completely controllable by their arguments. Even if you have a variable that will have to be passed around to lots of a functions - if it affects their computation, it should be a argument or a member of a argument. This *always* leads to better code and better design.

For example, removing global variables and constants from my [Trie.c](https://github.com/mcinglis/trie.c) project resulted in the `Alphabet` struct, which lets users tune the storage structure to their needs. It also opened up some really cool dynamic abilities, like swapping alphabets on the fly for the same trie.

Static variables in functions are just global variables scoped to that function; the arguments above apply equally to them. Just like global variables, static variables are often used as an easy way out of providing modular, pure functions. They're often defended in the name of performance (benchmarks first!). You don't need static variables, just like you don't need global variables. If you need persistent state, have the function accept that state as a argument. If you need to return something persistent, allocate memory for it.



#### Minimize what you expose; declare top-level names `static` where you can

Your header files should *only* include things that users need to use your library. Internal functions or structs or macros should not be provided here; declare them in their respective source files. If it's needed among multiple source files, provide an internal header file.

If a function or global variable isn't exported in the header, declare it as `static` in the source file to give it internal linkage. This eliminates the chance of name-clashes among object files, enables a few optimizations, and can improve the linking speed.



#### Immutability saves lives: use `const` everywhere you can

`const` improves compile-time correctness. It isn't only for documenting read-only pointers. It should be used for every read-only variable and pointee.

`const` helps the reader *immensely* in understanding a piece of functionality. If they can look at an initialization and be sure that that value won't change throughout the scope, they can reason about the rest of the scope much easier. Without `const`, everything is up in the air; the reader is forced to comprehend the entire scope to understand what is and isn't being modified. If you consistently use `const`, your reader will begin to trust you, and will be able to assume that a variable that isn't qualified with `const` is a signal that it will be changed at some point in the scope.

Using `const` everywhere you can also helps you, as a developer, reason about what's happening in the control flow of your program, and where mutability is spreading. It's amazing, when using `const`, how much more helpful the compiler is, especially regarding pointers and pointees. You always want the compiler on your side.

The compiler will warn if a pointee loses `const`ness in a function call (because that would let the pointee be modified), but it won't complain if a pointee gains `const`ness. Thus, if you *don't* specify your pointer arguments as `const` when they're read-only anyway, you discourage your users from using `const` in their own code:

``` c
// Bad: sum should define its array as const.
int sum( int * xs, int n );

// Because otherwise, this will be a compilation warning:
int const xs[] = { 1, 2, 3 };
return sum( xs, sizeof xs );
// => warning: passing argument 2 of ‘sum’ discards ‘const’
//             qualifier from pointer target type
```

Thus, using `const` isn't really a choice, at least for function signatures. Lots of people consider it beneficial, so everyone should consider it required, whether they like it or not. If you don't use `const`, you force your users to either cast all calls to your functions (yuck), ignore `const` warnings (asking for trouble), or remove those `const` qualifiers (lose compile-time correctness).

If you're forced to work with a library that ignores `const`, you can write a macro that casts for you:

``` c
// `sum` will not modify the given array; casts for `const` pointers.
#define sum( xs, n ) sum( ( int * ) xs, n )
```

Only provide `const` qualifiers for pointees in function prototypes - `const` for the argument names themselves is just an implementation detail.

``` c
// Unnecessary
bool Trie_has( Trie const, char const * const );
// Good
bool Trie_has( Trie, char const * );
```

Unfortunately, C can't handle conversions from non-const pointee-pointees to const pointee-pointees. Thus, I recommend against making pointee-pointees `const`.

``` c
char ** const xss = malloc( 3 * ( sizeof char * ) );
char const * const * const yss = xss;
// Warning: initialization from incompatible pointer type

char * const * const zss = xss;
// <no warning>
```

If you can `const` the pointees of your *internal* structs, do. Non-constant pointees can cause mutability to needlessly spread, which makes it harder to glean information from the remaining `const` qualifiers. Because you have total control over your internal structs, if you need to remove the `const` in future, you can.

You usually shouldn't `const` the pointees of your external structs. Flexibility is important when they're part of the public interface. Consider it carefully. An exception to this that I often make is for fields are best assignable to string literals, such as `error` fields. In this case, a `char const *` type prevents you and your users from modifying the underlying string literals, which would prompt a segmentation fault.

While it can be reasonable to `const` the *pointees* of struct fields, it's never beneficial to `const` the struct fields themselves. For example, [it makes it painful to `malloc`](http://stackoverflow.com/questions/9691404/how-to-initialize-const-in-a-struct-in-c-with-malloc) a value of that struct. If it really makes sense to stop the fields from changing beyond their original values, just define [invariants](#document-your-struct-invariants-and-provide-invariant-checkers) that enforce whatever qualities you need. Also, you and your users can just define individual variables of that struct as `const` to get the same effect.

Only make return-type pointees `const` if you need to, and after careful consideration. I've found that when the compiler is hinting to add a `const` to a return type, it often means that a `const` should be *removed* somewhere; not added. It can harm flexibility, so be careful.

Finally, never use typecasts or pointers to get around `const` qualifiers - at least, for things you control. If the variable isn't constant, don't make it one.



#### Always put `const` on the right and read types right-to-left

``` c
const char * word;              // Bad: not as const-y as it can be
const char * const word;        // Bad: makes types very weird to read
char const* const word;         // Bad: weird * placement

// Good: right-to-left, word is a constant pointer to a constant char
char const * const word;
```

Because of this rule, you should always pad the `*` type qualifier with spaces.



#### Don't write argument names in function prototypes if they just repeat the type

But, always declare the name of any pointer argument to communicate if it's a pointer-to-array (plural name) or a pointer-to-value (singular name).

``` c
bool trie_eq( Trie trie1, Trie trie2 );         // Bad
bool trie_eq( Trie, Trie );                     // Good

// Bad - are these pointers for modification, nullity, or arrays?
void trie_add( Trie const *, char const * );

// Good
void trie_add( Trie const * trie, char const * string );
```



#### Use `double` rather than `float`, unless you have a specific reason otherwise

From *21st Century C*, by Ben Klemens:

``` c
printf( "%f\n", ( float )333334126.98 );    // 333334112.000000
printf( "%f\n", ( float )333334125.31 );    // 333334112.000000
```

For the vast majority of applications nowadays, space isn't an issue, but floating-point errors can still pose a threat. It's much harder for numeric drift to cause problems for `double`s than it is for `float`s. Unless you have a very specific reason to use `float`s, use `double`s instead. Don't use `float`s "because they will be faster", because without benchmarks, you can't know if it actually makes any discernible difference. Finish development, then perform benchmarks to identify the choke-points, then use `float`s in those areas, and see if it actually helps. Before then, prioritize everything else over any supposed performance improvements. Don't prematurely optimize.



#### Declare variables as late as possible

Declaring variables where they're used reminds the reader of the type they're working with. It also suggests where to extract a function to minimize variable scope. Furthermore, it informs the reader as to where each variables are relevant. Declaring variables when they're needed almost always leads to initialization (`int x = 1;`), rather than just declaration (`int x;`). Initializing a variable usually often means you can `const` it, too.

To me, all declarations (i.e. non-initializations) are shifty.



#### Use one line per variable definition; don't bunch same types together

This makes the types easier to change in future, because atomic lines are easier to edit. If you'll need to change all their types together, you should use your editor's block editing mode.

I think it's alright to bunch semantically-connected struct members together, though, because struct definitions are much easier to comprehend than active code.

``` c
// Fine
typedef struct Color {
    char r, g, b;
} Color;
```



#### Don't be afraid of short variable names

If the scope fits on a screen, and the variable is used in a lot of places, and there would be an obvious letter or two to represent it, try it out and see if it helps readability. It probably will!



#### Be consistent in your variable names across functions

Consistency helps your readers understand what's happening. Using different names for the same values in functions is suspicious, and forces your readers to reason about unimportant things.



#### Use `bool` from `stdbool.h` whenever you have a boolean value

``` c
int print_steps = 0;             // Bad - is this counting steps?
bool print_steps = false;        // Good - intent is clear
```



#### Explicitly compare values; don't rely on truthiness

Explicit comparisons tell the reader what they're working with, because it's not always obvious in C, and it *is* always important. Are we working with counts or characters or booleans or pointers? The first thing I do when I see a variable being tested for truthiness in C is to hunt down the declaration to find its type. I really wish the programmer had just told me in the comparison.

``` c
// Bad - what are these expressions actually testing for (if at all?)
if ( on_fire );
return !character;
something( first( xs ) );
while ( !at_work );

// Good - informative, and eliminates ambiguity
if ( on_fire > 0 );
return character == NULL;
something( first( xs ) != '\0' );
while ( at_work == false );
```

I'll often skip this rule for boolean functions named as a predicate, like `is_edible` or `has_client`. It's still not *completely* obvious what the conditional is checking for, but I usually consider the visual clutter of a `== true` or `== false` to be more of a hassle than a help to readers in this situation. Use your judgement.



#### Never change state within an expression (e.g. with assignments or `++`)

Readable (imperative) programs flow from top to bottom: not right to left. Unfortunately, this happens way too much in C programming. I think the habit and practice was started by *The C Programming Language*, and it's stuck with much of the culture ever since. It's a really bad habit, and makes it so much harder to follow what your program is doing. Never change state in an expression.

``` c
trie_add( *child, ++word );     // Bad
trie_add( *child, word + 1 );   // Good

// Good, if you need to modify `word`
word += 1;
trie_add( *child, word );

// Bad
if ( ( x = calc() ) == 0 );
// Good
x = calc();
if ( x == 0 );

// Fine; technically an assignment within an expression
a = b = c;

while ( --atoms > 0 );          // Bad
while ( atoms -= 1,             // Good
        atoms > 0 );

// Fine; there's no better way, without repetition
int w;
while ( w = calc_width( shape ),
        !valid_width( w ) ) {
    shape = reshape( shape, w );
}
```

Don't use multiple assignment unless the variables' values are semantically linked. If there are two variable assignments near each other that coincidentally have the same value, don't throw them into a multiple assignment just to save a line.

Use the comma operator, as above, judiciously. Do without it if you can:

``` c
// Bad
for ( int i = 0, limit = get_limit( m ); i < limit; i += 1 ) {
    ...
}

// Better
int const limit = get_limit( x );
for ( int i = 0; i < limit; i += 1 ) {
    ...
}
```



#### Avoid non-pure or non-trivial function calls in expressions

Assign function calls to a variable to describe what it is, even if the variable is as simple as an `int result`. This avoids surprising your readers with state changes from non-pure functions hidden inside conditional contexts. To me, it's really unnatural to think about the expression inside an `if ( ... )` changing things on the outside world. It's much clear to assign the result of that state change to a variable, and then check that value.

Even if you think it's obvious, and it will save you a line - it's not worth the potential for a slip-up. Stick to this rule, and don't think about it.

If the function name is a predicate, like `is_adult` or `in_tree`, and will read naturally in a conditional context, then I think it's alright to skip assigning its result. It's also probably fine to join these kind of functions in a boolean expression if you need to, but use your judgement. Complex boolean expressions should often be extracted to a function.

``` c
// Good
int r = listen( fd, backlog );
if ( r == -1 ) {
    perror( "listen" );
    return 1;
}

// Good
if ( is_tasty( banana ) ) {
    eat( banana );
}
```


#### Always use brackets, even for single-statement blocks

Always use brackets, because it's safer, easier to change, and easier to read because it's more consistent. For the same reasons, don't put a single-line statement on the same line as the condition.

What follows is actual code from *The C Programming Language*. Don't do this:

``` c
while (--argc > 0 && (*++argv)[0] == '-')
    while (c = *++argv[0])
        switch (c) {
            ...
        }
if (argc != 1)
    printf("Usage: find -x -n pattern\n");
else
    while (getline(line, MAXLINE) > 0) {
        ...
    }
```



#### Avoid unsigned types because the integer conversion rules are complicated

[CERT attempts to explain the integer conversion rules](https://www.securecoding.cert.org/confluence/display/seccode/INT02-C.+Understand+integer+conversion+rules), saying:

> Misunderstanding integer conversion rules can lead to errors, which in turn can lead to exploitable vulnerabilities. Severity: medium, Likelihood: probable.

*Expert C Programming* (a great book that explores the ANSI standard) also explains this in its first chapter. The takeaway is that you shouldn't declare `unsigned` variables just because they shouldn't be negative. If you want a larger maximum value, use a `long` or `long long` (the next size up).

If your function will fail with a negative number, it will probably also fail with a large number - which is what it will get if passed a negative number. If your function will fail with a negative number, just assert that it's positive. Remember, lots of dynamic languages make do with a single integer type that can be either sign.

Unsigned values offer no type safety; even with `-Wall` and `-Wextra`, GCC doesn't bat an eyelid at `unsigned int x = -1;`.

*Expert C Programming* also provides an example for why you should cast all macros that will evaluate to an unsigned value.

``` c
#define NELEM( xs ) ( ( sizeof xs ) / ( sizeof xs[0] ) )
int const xs[] = { 1, 2, 3, 4, 5, 6 };

int main( void )
{
    int const d = -1;
    if ( d < NELEM( xs ) - 1 ) {
        return xs[ d + 1 ];
    }
    return 0;
}
```

The `if` branch won't be executed, because `NELEM` will evaluate to an `unsigned int` (via `sizeof`). So, `d` will be promoted to an `unsigned int`. `-1` in [two's complement](https://en.wikipedia.org/wiki/Two%27s_complement) represents the largest possible unsigned value (bit-wise), so the expression will be false, and the program will return `0`. The solution in this case would be to cast the result of `NELEM`:

``` c
#define NELEM( xs ) ( long )( ( sizeof xs ) / ( sizeof xs[ 0 ] ) )
```

You will need to use unsigned values to provide [well-defined bit operations](http://stackoverflow.com/questions/4009885/arithmetic-bit-shift-on-a-signed-integer) and modular arithmetic overflow. But, try to keep those values contained, and don't let them interact with signed values.



#### Use `+= 1` and `-= 1` over `++` and `--`

Actually, don't use either form if you can help it. Changing state should always be avoided (within reason). But, when you have to, `+=` and `-=` are obvious, simpler and less cryptic than `++` and `--`, and useful in other contexts and with other values. Also, there are no tricks about the evaluation of `+=` and `-=` and they don't have weird twin operators to provide alternative evaluations. Python does without `++` and `--` operators, and Douglas Crockford excluded them from the Good Parts of JavaScript, because we don't need them. Sticking to this rule also encourages you to avoid changing state within an expression.



#### Use parentheses for expressions where the [operator precedence](https://en.wikipedia.org/wiki/Operators_in_C_and_C%2B%2B#Operator_precedence) isn't obvious

``` c
int x = a * b + c / d;          // Bad
int x = ( a * b ) + ( c / d );  // Good

&sockaddr->sin_addr;            // Bad
&( sockaddr->sin_addr );        // Good
```

You can and should make exceptions for commonly-seen combinations of operations. For example, skipping the operators when combining the equality and boolean operators is fine, because readers are probably used to that, and are confident of the result.

``` c
// Fine
return hungry == true
    || ( legs != NULL && fridge.empty == false );
```



#### Don't use `switch`, and avoid complicated conditionals

The `switch` fall-through mechanism is error-prone, and you almost never want the cases to fall through anyway, so the vast majority of `switch`es are longer than the `if` equivalent. Worse, a missing `break` will still compile: this tripped me up all the time when I used `switch`. Also, `case` values have to be an integral constant expression, so they can't match against another variable. This discourages extractions of logic to functions. Furthermore, any statement inside a `switch` can be labelled and jumped to, which fosters highly-obscure bugs if, for example, you mistype `defau1t`.

If you need to map different constant values to behavior, like:

``` c
switch ( x ) {
case A:
    do_something_for_a( x, y, z );
    break;
case B:
    do_something_for_b( x, y, z ):
    break;
default:
    error( x, y, z );
    break;
}
// These functions might not be explicit functions (i.e. they might
// just be a series of statements using some of those variables).
```

A more explicit, testable and reusable approach is to define a function that uses ternary expressions to return a function pointer of the right type:

``` c
action_fn get_x_action( x ) {
    return ( x == A ) ? do_something_for_a
         : ( x == B ) ? do_something_for_b
         : error;
}

action_fn action = get_x_action( x );
action( x, y, z );

// or just:
get_x_action( x )( x, y, z );

// `action` is a terrible name and is only used as an example. You
// should try to think of a more-informative name for your code.
```

You should do a similar thing if you need to map between two sets of uncorrelated constant values, like:

``` c
// Bad
switch ( x ) {
    case A: return X;
    case B: return Y;
    case C: return Z;
    default: return ERR;
}

// Good
return ( x == A ) ? X
     : ( x == B ) ? Y
     : ( x == C ) ? Z
     : ERR;
```

Don't use a `switch` where you can just use a boolean expression:

``` c
// Bad
switch ( x ) {
    case A: case B: case C:
        return true;
    default:
        return false;
}

// Good
return x == A || x == B || x == C;

// Or, if the names are longer, this usually reads better:
return t == JSON_TYPE_null
    || t == JSON_TYPE_boolean
    || t == JSON_TYPE_number;
```

If you need the fall-through behavior of `switch`, like:

``` c
switch ( x ) {
    case A:
        // A stuff, fall through to B
    case B:
        // B stuff
        break;
    default:
        // default stuff
}
```

The equivalent `if` is much more readable and it's obvious what's going to happen and why. The "B stuff" actually applies when `x == A` too, and this is explicitly declared when you use an `if`.

``` c
if ( x == A ) {
    // A stuff
}

if ( x == A || x == B ) {
    // B stuff
} else {
    // default stuff
}
```

You should only need to use `switch` for performance tuning (once you've done benchmarks to identify hotspots!). Otherwise, there's always a safer, shorter, more-testable, and reusable alternative.



#### Separate functions and struct definitions with two lines

If you limit yourself to a maximum of one blank line within functions, this rule provides clear visual separation of global elements. This is a habit I learned from Python's PEP8 style guide.



#### Minimize the scope of variables

If a few variables are only used in a contiguous sequence of lines, and only a single value is used after that sequence, then those first lines are a great candidate for extracting to a function.

``` c
// Good: addr was only used in the first part of handle_request
int accept_request( int const listenfd )
{
    struct sockaddr addr;
    return accept( listenfd, &addr, &( socklen_t ){ sizeof addr } );
}

int handle_request( int const listenfd )
{
    int const reqfd = accept_request( listenfd );
    // ... stuff not involving addr, but involving reqfd
}
```

If the body of `accept_request` were left in `handle_request`, then the `addr` variable will be in the scope for the remainder of the `handle_request` function even though it's only used for getting the `reqfd`. This kind of thing adds to the cognitive load of understanding a function, and should be fixed wherever possible.

Another tactic to limit the exposure of variables is to break apart complex expressions into blocks, like so:

``` c
// Rather than:
bool trie_has( Trie const trie, char const * const string )
{
    Trie const * const child = Trie_child( trie, string[ 0 ] );
    return string[ 0 ] == '\0'
           || ( child != NULL
                && Trie_has( *child, string + 1 ) );
}

// child is only used for the second part of the conditional, so we
// can limit its exposure like so:
bool trie_has( Trie const trie, char const * const string )
{
    if ( string[ 0 ] == '\0' ) {
        return true;
    } else {
        Trie const * const child = Trie_child( trie, string[ 0 ] );
        return child != NULL
            && Trie_has( *child, string + 1 );
    }
}
```



#### Simple constant expressions can be easier to read than variables

It can often help the readability of your code if you replace variables that are only assigned to constant expressions, with those expressions.

Consider the `trie_has` example above - the `string[ 0 ]` expression is repeated twice. It would be harder to read and follow if we inserted an extra line to define a `char` variable. It's just another thing that the readers would have to keep in mind. Many programmers of other languages wouldn't think twice about repeating an array access.



#### Prefer compound literals to superfluous variables

This is beneficial for the same reason as minimizing the scope of variables.

``` c
// Bad, if `sa` is never used again.
struct sigaction sa = {
    .sa_handler = sigchld_handler,
    .sa_flags = SA_RESTART
};
sigaction( SIGCHLD, &sa, NULL );

// Good
sigaction( SIGCHLD, &( struct sigaction ){
    .sa_handler = sigchld_handler,
    .sa_flags = SA_RESTART
}, NULL );

// Bad
int v = 1;
setsockopt( fd, SOL_SOCKET, SO_REUSEADDR, &v, sizeof v );

// Good
setsockopt( fd, SOL_SOCKET, SO_REUSEADDR, &( int ){ 1 }, sizeof int );
```



#### Never use or provide macros that wrap control structures like `for`

Macros that loop over the elements of a data structure are extremely confusing, because they're extra-syntactic and readers can't know the control flow without looking up the definition.

To understand your program, it's crucial that your readers can understand its control flow.

Don't provide control-macros even as an option. They're universally harmful, so don't enable it. Users can define their own if they really want to.

``` c
// Bad
#define TRIE_EACH( TRIE, INDEX ) \
    for ( int INDEX = 0; INDEX < ( TRIE ).alphabet.size; INDEX += 1 )

// Not at all obvious what's actually going to happen here.
TRIE_EACH( trie, i ) {
    Trie * const child = trie.children[ i ];
    ...
}
```



#### Only upper-case a macro if will act differently than a function call

By "act differently", I mean if things will break when users wouldn't expect them to. If a macro just looks different (e.g. the named arguments technique), then I don't consider that justification for an upper-case name. A macro should have an upper-case name if it:

- repeats its arguments in its body, because this will break for non-pure expressions. Many compilers provide [statement expressions](http://stackoverflow.com/questions/6440021/compiler-support-of-gnu-statement-expression) to prevent this, but it's non-standard. If you do use statement expressions, then you don't need to upper-case your macro name, because it's not relevant to your users.
- is wrapped in blocks or a control structure, because it can't be used as an expression then.
- modifies the surrounding context, e.g., with a `return` or `goto`.
- takes an array literal as a named argument. ([why](http://stackoverflow.com/questions/5503362/passing-array-literal-as-macro-argument))



#### If a macro is specific to a function, `#define` it in the body

For the same reasons why we should always minimize the scope of our variables, if it makes sense to limit the scope of a macro, we should.



#### Initialize strings as arrays, and use `sizeof` for byte size

Always initialize your string literals as arrays, because it lets you get the byte size with just `sizeof str`. If you initialize them as pointers, you have to get the byte size with `strlen( str ) + 1` - I know I've been bitten more than once by forgetting the `+ 1` there.

``` c
// Good
char const message[] = "always use arrays for strings!";
write( output, message, sizeof message );
```

Also, pointer initializations are less safe than array initializations, *unless* you compile with `-Wwrite-strings` to ensure string literals are initialized with the type `char const *`. Unfortunately, `-Wwrite-strings` isn't included in `-Wall` or `-Wextra`: you have to explicitly enable it.  Without `-Wwrite-strings`, you can assign string literals to a `char *`. But your program will seg-fault when you re-assign the elements of that pointer.

``` c
// Without -Wwrite-strings, this will compile without warnings, but
// it will prompt a segmentation fault at the second line.
char * xs = "hello";
xs[ 0 ] = 'c';

// This program will compile and execute fine.
char xs[] = "hello";
xs[ 0 ] = 'c';
```

The benefit of initializing string literals as pointers is that those pointers will point to read-only memory, potentially allowing some optimizations. Initializing string literals as arrays essentially creates a mutable string is only "artificially" protected against modifications with `const` - but this can be defeated with a cast.

Again, I advise against prematurely optimizing. Until you've finished development and have done benchmarks, performance should be your lowest priority. I haven't seen any tests on string literal definitions, but I'd be very surprised to see any notable speed improvements by defining string literals as pointers.

As mentioned in the rule on `const`ing everything: never ever cast away a `const`. Remove the `const` instead. Don't worry about "artificial" protections. I know I'd much prefer my constant values to be protected by explicit, syntactic constructs that will warn when compiling, rather than implicit, obscure rules that will seg-fault when violated.

Finally, sticking to array initializations saves you and your readers the conceptual overhead of switching between pointer initializations and array initializations, depending on if you need mutability or not.

Just always initialize string literals as arrays, and keep it simple.



#### Where possible, use `sizeof` on the variable; not the type

Then, if you change the type of the variable later, you only have to change it once. You'll always get the correct size.

``` c
// Good
int * a = malloc( n * ( sizeof *a ) );
```

You can't do this with compound literals, though. I think it's a worth-while trade-off to remove a variable that's only used once.

``` c
setsockopt( fd, SOL_SOCKET, SO_REUSEADDR, &( int ){ 1 }, ( sizeof int ) );
```


#### Never use array syntax for function arguments definitions

[Arrays become pointers in most expressions](http://c-faq.com/aryptr/aryptrequiv.html), including [when passed as arguments to functions](http://c-faq.com/aryptr/aryptrparam.html). Functions can never receive an array as a argument: [only a pointer to the array](http://c-faq.com/aryptr/aryptr2.html). `sizeof` won't work like an array argument declaration would suggest: it would return the size of the pointer, not the array pointed to.

[Static array indices in function arguments are nice](http://hamberg.no/erlend/posts/2013-02-18-static-array-indices.html), but only protect against very trivial situations, like when given a literal `NULL`. Also, GCC doesn't warn about their violation [yet](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=50584), only Clang. I don't consider the confusing, non-obvious syntax to be worth the small compilation check.

Yeah, `[]` hints that the argument will be treated as an array, but so does a plural name like `requests`, so do that instead.



#### Always prefer array indexing over pointer arithmetic

If you're working with an array of things, treat them as an array. Pointer arithmetic is confusing and bug-prone. Sticking to array indexing often lets you keep the important variables constant, and only the index variables non-constant.

``` c
// Bad
for ( ; *str != '\0'; str += 1 );

// Good
for ( int i = 0; str[ i ] != '\0'; i += 1 );
```



#### Document your struct invariants, and provide invariant checkers

> An **invariant** is a condition that can be relied upon to be true during execution of a program.

For any function that takes a struct (or a pointer to a struct), all invariants of that struct should be true before and after the execution of the function. Invariants make it the caller's responsibility to provide valid data, and the function's responsibility to return valid data. Invariants save those functions from having to repeat assertions of those conditions, or worse, not even checking and working with invalid data.

Provide an "invariants" comment section at the end of your struct definition, and list all the invariants you can think of. Also, implement `is_valid` and `assert_valid` functions for users to check those assertions on values of the structs they create on their own. These functions are crucial to being able to trust that the invariants hold for values of that struct. Without them, how will the users know?

[Here's an example](https://github.com/mcinglis/trie.c/blob/master/alphabet.h#L10) of a struct invariant.

My university faculty is [pretty big](http://www.itee.uq.edu.au/sse/projects) on software correctness. It certainly rubbed off on me.



#### Use `assert` everywhere your program would fail otherwise

Write assertions to meaningfully crash your program before it does something stupid, like deleting data, or to prevent a security vulnerability, or just to prevent a segmentation fault. Good software fails fast.

If a function is given a pointer it will dereference, assert that it's not null. If it's given an array index, assert that it's within bounds. Assert for any consistency that you need between arguments.

That said, never depend on assertions for correctness. Your program should still work correctly when the assertion lines are removed.

Don't mistake assertions for error-reporting. Assert things that you won't bother to check otherwise. If user input (not code) can invalidate an assertion, that's a bug. You should be filtering it before-hand, and reporting the errors in a readable fashion for your users.



#### Repeat `assert` calls; don't `&&` them together

Repeating your `assert` calls improves the assertion error reporting. If you chain assertions together with `&&`, you won't know which condition failed.



#### Don't use variable-length arrays

Variable-length arrays were introduced in C99 as a way to define dynamic-length arrays with automatic storage; no need for `malloc`. For a few reasons, they've been made optional in C11. Thus, if you want to use variable-length arrays in C11, you'll have to write the `malloc` version anyway. Instead, just don't use variable-length arrays.

I'd advise against using variable-length arrays in C99, too. First, you have to [check the values](https://www.securecoding.cert.org/confluence/display/seccode/ARR32-C.+Ensure+size+arguments+for+variable+length+arrays+are+in+a+valid+range) that control their size to protect against stack-smashing. Second, they can't be initialized. Finally, avoiding them will make it easier to upgrade to newer standards later on.



#### Avoid `void *` because it harms type safety

`void *` is useful for polymorphism, but polymorphism is almost never as important as type safety. Void pointers are indispensable in many situations, but you should consider other, safer alternatives first.



#### If you have a `void *`, assign it to a typed variable as soon as possible

Just like working with uninitialized variables is dangerous, working with void pointers is dangerous: you want the compiler on your side. So ditch the `void *` as soon as you can.



#### Use C11's anonymous structs and unions rather mutually-exclusive fields

If only certain fields of your struct should be set when certain other fields have certain values, use C11's anonymous structs and unions:

``` c
enum AUTOMATON_TYPE {
    AUTOMATON_TYPE_char,
    AUTOMATON_TYPE_split,
    AUTOMATON_TYPE_match
};
#define NUM_AUTOMATON_TYPES ( 3 )

typedef struct Automaton {
    enum AUTOMATON_TYPE type;
    union {
        struct { // type = char
            char c;
            struct Automaton * next;
        };
        struct { // type = split
            struct Automaton * left;
            struct Automaton * right;
        };
    };
} Automaton;
```

This is much more explicit and obvious than something like:

``` c
typedef struct Automaton {
    enum AUTOMATON_TYPE type;
    char c;
    struct Automaton * left;
    struct Automaton * right;
} Automaton;
```



#### Don't typecast unless you have to (you probably don't)

If it's valid to assign a value of one type to a variable of another type, then you don't have to cast it. You should only use typecasts when you need to, like:

- performing true division (not integer division) of `int` expressions
- making an array index an integer, but you can do this with assignment anyway
- using compound literals for structs and arrays

``` c
// This compiles fine:
struct Apple * apples = malloc( sizeof *apples );
```



#### Give structs TitleCase names, and typedef them

``` c
// Good
typedef struct Person {
    char * name;
    int age;
} Person;
```

TitleCase names should be used for structs so that they're recognizable without the `struct` prefix. They also let you name struct variables as the same thing as their type without names clashing (e.g. a `banana` of type `Banana`). You should always define the struct name, even if you don't need to, because you might need to later (e.g. to use as an incomplete type). Also, having a name at the top helps readability when comments are inserted, or the struct definition becomes large.

I don't typedef structs used for named arguments (see below), however, because the TitleCase naming would be weird. Anyway, if you're using a macro for named arguments, then the typedef is unnecessary and the struct definition is hidden.

If a user dislikes this practice of typedefing structs (which is fair, because it does have drawbacks - see below), they can always use the `struct` namespace instead.



#### Only typedef structs; never basic types or pointers

``` c
// Bad
typedef double centermeters;
typedef double inches;
typedef struct Apple * Apple;
typedef void * gpointer;
```

This mistake is committed by way too many codebases. It masks what's really going on, and you have to read documentation or find the `typedef` to learn how to work with it. Never do this in your own interfaces, and try to ignore the typedefs in other interfaces.

These criticisms apply equally to struct typedefs, as advised above. In my opinion, the visual clarity achieved by removing all the `struct` declarations is worth requiring users be aware of (or realize) the convention. Also, having a consistent naming scheme for structs, with TitleCase names, helps recognizability.

Pointer typedefs are particularly nefarious because they exclude the users from qualifying the pointee with `const`. This is a huge loss, for reasons enumerated in other rules.

Function pointer typedefs are justified when you need to declare a function that returns a function pointer; the syntax without a typedef is unbearable. I'll also typedef a function pointer if the type is being repeated in many locations (more than three, or so). Some people like to typedef all function pointers, but this often masks what's going on and what's expected. Carefully consider if a function pointer typedef will actually help people understand what that type represents.



#### Give enums `UPPERCASE_SNAKE` names, and lowercase their values

Because enums are mostly just integer constants, it's natural to name them the same way as `#define`d constants. The `enum` type prefix will communicate that it expects an enum value, and the lowercase value suffixes will communicate that they aren't quite integer constants.

``` c
enum JSON_TYPE {
    JSON_TYPE_null,
    JSON_TYPE_boolean,
    JSON_TYPE_number,
    ...
};
```



#### Define a constant for the size of every enum

There's no versatile, future-proof way to work with loops, arrays, or bit-fields of the `enum` otherwise. Always define a constant to denote the size of the enumeration, to avoid hard-coded values (by you or your users).

``` c
enum SUIT {
    SUIT_hearts,
    SUIT_diamonds,
    SUIT_clubs,
    SUIT_spades
};
#define NUM_SUITS 4
```

I like to `#define` the size explicitly, rather than making it the last enum value. It seems natural to exclude the size of the enum from the actual enum itself - `NUM_SUITS` isn't a card suit I've ever heard of! It also protects against one of the previous enum values from being explicitly set (e.g. `SUIT_hearts = 1`), which would mean the last enum value wouldn't represent the size of the enum.



#### Never begin names with `_` or end them with `_t`: they're reserved for standards

[Here's a list](https://www.gnu.org/software/libc/manual/html_node/Reserved-Names.html) of the names reserved by future ISO C standards. `types_like_this_t` and `_anything` are identifiers that are reserved by future standards of C, so don't use them for your own identifiers.

These kinds of names *could've* provided a nice way to tell which types are part of the language standard and which types are provided by libraries. Unfortunately, [it's](https://github.com/facebook/libphenom) [not](https://github.com/joyent/libuv) [hard](https://github.com/liuliu/ccv) to find popular C libraries and projects that make this mistake, which dilutes the helpfulness of such a rule.

This mistake is made way too often: don't make the same mistake in your library!



#### Only use pointers in structs for nullity, dynamic arrays or incomplete types

Every pointer in a struct is an opportunity for a segmentation fault.

If the would-be pointer shouldn't be NULL, isn't an array of an unknown size, and isn't of the type of the struct itself, then don't make it a pointer. Just include a member of the type itself in the struct. Don't worry about the size of the containing struct until you've done benchmarks.



#### Only use pointer arguments for nullity, arrays or modifications

This rule helps readers reason about where values are being modified. It also improves the safety by making it impossible for functions that shouldn't receive `NULL` from receiving `NULL` -- this is a huge benefit over languages that require pass-by-reference semantics (and thus `NULL` as a valid value almost everywhere).

When you're reading a codebase that sticks to this rule, and its functions and types are maximally decomposed, you can often tell what a function does just by reading its prototype. This is in stark contrast to projects that pass pointers everywhere: you have no certainty anywhere.

In C, you can pass struct values to functions, and by [pass-by-value semantics](http://c-faq.com/ptrs/passbyref.html), they'll be copied into the stack frame of the receiving function. The original struct can't be modified by that function (although it can return the modification). Like `const`, using this feature wherever you can makes it easier for your readers to reason about your program.

Defining a "modification" gets tricky when you introduce structs with pointer members. I consider a modification to be something that affects the struct itself, or the pointees of the struct.

If a struct will be "modified" by a function, have that function accept a pointer of that struct even if it doesn't need to. This saves the readers from having to find and memorize every relevant struct definition, to be aware of which structs have pointer members.

``` c
typedef struct {
    int population;
} State;

typedef struct {
    State * states;
    int num_states;
} Country;

// Good: takes a `Country *` even though it *could* modify the array
// pointed to by the `states` member with just a `Country` value.
void country_grow( Country const * const country, double const percent ) {
    for ( int i = 0; i < country->num_states; i += 1 ) {
        country->states[ i ].population *= percent;
    }
}
```

Note the const-ness of the `country` argument above: this communicates that the country itself won't be modified, but a pointee (although it could also be taken to suggest that the pointer is for nullity, but the function name suggests otherwise). It also allows callers to pass in a pointer to a `Country const`.

The other situation to use pointer arguments is if the function needs to accept `NULL` as a valid value (i.e. the poor man's [Maybe](http://learnyouahaskell.com/making-our-own-types-and-typeclasses)). If so, be sure use `const` to signal that the pointer is not for modification, and so it can accept `const` arguments.

``` c
// Good: `NULL` represents an empty list, and list is a pointer-to-const
int list_length( List const * list ) {
    int length = 0;
    for ( ; list != NULL; list = list->next ) {
        length += 1;
    }
    return length;
}
```

Sticking to this rule means ditching incomplete struct types, but I don't really like them anyway. (see the "[C isn't object-oriented](#c-isnt-object-oriented-and-you-shouldnt-pretend-it-is)" rule)



#### Prefer to return a value rather than modifying pointers

This encourages immutability, cultivates [pure functions](https://en.wikipedia.org/wiki/Pure_function), and makes things simpler and easier to understand. It also improves safety by eliminating the possibility of a `NULL` argument.

``` c
// Bad: unnecessary mutation (probably), and unsafe
void drink_mix( Drink * const drink, Ingredient const ingr ) {
    assert( drink != NULL );
    color_blend( &( drink->color ), ingr.color );
    drink->alcohol += ingr.alcohol;
}

// Good: immutability rocks, pure and safe functions everywhere
Drink drink_mix( Drink const drink, Ingredient const ingr ) {
    return ( Drink ){
        .color = color_blend( drink.color, ingr.color ),
        .alcohol = drink.alcohol + ingr.alcohol
    };
}
```

This isn't always the best way to go, but it's something you should always consider.



#### Use structs to name functions' optional arguments

``` c
struct run_server_options {
    char * port;
    int backlog;
};

#define run_server( ... ) \
    run_server_( ( struct run_server_options ){ \
        /* default values */ \
        .port = "45680", \
        .backlog = 5, \
        __VA_ARGS__ \
    } )

int run_server_( struct run_server_options opts )
{
    ...
}

int main( void )
{
    return run_server( .port = "3490", .backlog = 10 );
}
```

I learnt this from *21st Century C*. So many C interfaces could be improved immensely if they took advantage of this technique. The importance and value of (syntactic) named arguments is all-too-often overlooked in software development. If you're not convinced, read Bret Victor's [Learnable Programming](http://worrydream.com/LearnableProgramming/).

Don't use named arguments everywhere. If a function's only argument happens to be a struct, that doesn't necessarily mean it should become the named arguments for that function. A good rule of thumb is that if the struct is used outside of that function, you shouldn't hide it with a macro like above.

``` c
// Good; the typecast here is informative and expected.
book_new( ( Author ){ .name = "Dennis Ritchie" } );
```



#### Always use designated initializers in struct literals

``` c
// Bad - will break if struct members are reordered, and it's not
// always clear what the values represent.
Fruit apple = { "red", "medium" };
// Good; future-proof and descriptive
Fruit watermelon = { .color = "green", .size = "large" };
```

Sometimes I'll bend this rule for named arguments, by having a particular field be at the top of the struct, so that callers can call the function without having to name that single argument:

``` c
run_server( "3490" );
run_server( .port = "3490", .backlog = 10 );
```

If you want to allow this, document it explicitly. It's then your responsibility to version your library correctly, if you change the ordering of the fields.



#### If you're providing allocation and free functions only for a struct member, allocate memory for the whole struct

If you're providing `foo_alloc` and `foo_free` functions only so you can allocate memory for a member of the `Foo` struct, you've lost the benefits and safety of automatic storage. You may as well have the allocation and free methods allocate memory for the whole struct, so users can pass it outside the scope it was defined (without dereferencing it), if they want.



#### Avoid getters and setters

If you're seeking encapsulation in C, you're probably overcomplicating things. Encourage your users to access and set struct members directly; never prefix members with `_` to denote an access level. Declare your struct invariants, and you don't need to worry about your users breaking things - it's their responsibility to provide a valid struct.

As advised in [another rule](#always-prefer-to-return-a-value-rather-than-modifying-pointers), avoid mutability wherever you can.

``` c
// Rather than:
void city_set_state( City * const c, char const * const state )
{
    c->state = state;
    c->country = country_of_state( state );
}

// Always prefer immutability and purity:
City city_with_state( City c, char const * const state )
{
    c.state = state;
    c.country = country_of_state( state );
    return c;
}

City c = { .name = "Vancouver" };
c = city_with_state( c, "BC" );
printf( "%s is in %s, did you know?\n", c.name, c.country );
```

But you should always provide an interface that allows for [declarative programming](https://en.wikipedia.org/wiki/Declarative_programming):

``` c
City const c = city_new( .name = "Boston", .state = "MA" );
printf( "I think I'm going to %s,\n"
        "Where no one changes my state\n", c.name, c.country );
```



#### C isn't object-oriented, and you shouldn't pretend it is

C doesn't have classes, methods, inheritance, (nice) object encapsulation, or real polymorphism. Not to be rude, but: **deal with it**. C might be able to achieve crappy, complicated imitations of those things, but it's just not worth it.

As it turns out, C already has an entirely-capable language model. In C, we define data structures, and we define functionality that uses combinations of those data structures. Data and functionality aren't intertwined in complicated contraptions, and this is a good thing.

Haskell, at the forefront of language design, made the same decision to separate data and functionality. Learning Haskell is one of the best things a programmer can do to improve their technique, but I think it's especially beneficial for C programmers, because of the underlying similarities between C and Haskell. Yes, C doesn't have anonymous functions, and no, you won't be writing monads in C anytime soon. But by learning Haskell, you'll learn how to write good software without classes, without mutability, and with modularity. These qualities are very beneficial for good C programming.

Embrace and appreciate what C offers, rather than attempting to graft other paradigms onto it.

