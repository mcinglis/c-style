# C Style

I've never done anything useful with C, but I read a lot about C, and play around with it. I often read other people's C, often in the hopes of contributing to some project, only to give up in disgust of the interface and their practices. Then, I consider implementing a greenfield version of that project, and look at what projects they depend on. Rinse repeat, until I'm considering writing wrappers around the standard library, or writing monadic Haskell or [Rust](http://www.rust-lang.org/) bindings. Sigh.

My programming ability is hindered by my inability to work with bad interfaces. I think the best engineers are those who can make do with what they're given; I can't, unless it's for work or school.

Anyway, this document describes what I consider good C. Some points are as trivial as style, while others are more intricate. Some points I adhere to religiously, and others I use as a guideline. I prioritize correctness, readability, simplicity and maintainability over speed, because:

* [premature optimization is the root of all evil](http://c2.com/cgi/wiki?PrematureOptimization)
* compilers are better at optimizing than humans, and they're only going to get better

**Write correct, readable, simple and maintainable software, and tune it when you're done**, with benchmarks to identify the choke points. Also, modern compilers *will* change computational complexities. Simplicity and maintainability can often lead you to the best solution anyway - e.g., it's easier to write a linked list than it is to get an array to grow, but it's harder to index a list than it is to index an array.

Many of these rules are just good programming practices, and apply outside of C programming. Writing this guide made me deeply consider, and reconsider, best C programming practices. I've changed my opinion multiple times on more than a few points here.

So, I'm probably still wrong on even more points. **This is a work-in-progress;** issues and pull-requests are very welcome.  This guide is licensed under the [Creative Commons Attribution-ShareAlike](/license.md), so I'm not liable for anything.


---



#### Write to the most modern standard you can



#### We can't get tabs right, so use spaces everywhere

The idea of tabs was that we'd use tabs for indentation levels, and spaces for alignment. This lets people choose a tab-width to their liking, but it won't break alignment of columns.

``` c
int main( void ) {
|tab   |if ( pigs_can_fly() ) {
|tab   ||tab   |developers_can_use_tabs( "and align columns "
|tab   ||tab   |                         "with spaces!" );
|tab   |}
}
```

But, alas, we (and our editors) rarely get it right, so I prefer to cut the complexity and stick to spaces only, because that's harder to get wrong.



#### Use `//` comments everywhere, never `/* ... */`

Compared to single-line comments, multi-line comments:

- are rarely used with a blank margin, so they're just as verbose
- have a style, which has to be specified and adhered to
- often have `*/` on its own line, so they're more line-expensive



#### Don't comment what you don't need to

You want to fit as much on screen as you can, so if you can make a line of code self-commenting rather than adding a comment line above it, do so. If you can say the same thing in one sentence which is being said in three, do so. Avoid line-expensive comment styles at all costs.



#### Write comments in full sentences, without abbreviations



#### Don't use comments to conceal bad naming or bad design you can fix

But certainly use comments to explain bad design or bad naming forced upon you. If your project heavily depends on the bad interface, you should write a wrapper around it to improve it.



#### Try to write comments after the line referred to

This is just a preference, but I've really taken to writing comments after the code. I find it to be much easier to read, and much more informative. It also encourages the comments to not repeat what the code says. I now declare my structs like this:

``` c
typedef struct Alphabet {
// An Alphabet defines an ordering of characters, such that each
// character in the Alphabet has exactly one corresponding index.

    int size;
    // The number of characters in this Alphabet.

    int ( *index_for )( char );
    // Returns the index for the given character, or -1 if that char
    // isn't in this Alphabet. The index must be less than the
    // Alphabet's size.

    char ( *char_for )( int );
    // Returns the character associated with the given index, or
    // `Trie_ERR` if the index is invalid ( < 0 or >= size ).

} Alphabet;
```

But, if the comment will refer to a multi-line block of code (e.g. a small `if` block), I'll still put the comment at the top.



#### Program in American English

Write `color`, `flavor`, `center`, `meter`, `neighbor`, `defense`, `routing`, `sizable`, `disk`, `tire` and so on ([see more](https://en.wikipedia.org/wiki/American_and_British_English_spelling_differences)). I'm Australian, but I appreciate that most programmers will be learning and using American English. Also, American English spelling is consistently more phonetic than British English. British English tends to evolve towards American English for this reason, I think.



#### Comment all `#include`s to say what symbols you use from them

Namespaces are one of the great advances of software development. Unfortunately, C missed out (scopes aren't namespaces). But, because namespaces are so fantastic, we should emulate them with comments.

``` c
#include <stdlib.h>     // size_t, calloc, free
```

If the name occurs in your source code, it should be declared in that file, or mentioned in a comment beside the header file it's declared in. It's terrible to require readers to refer to documentation or use grep to get this information. I've never seen any projects that do this, but I think it would be great if more did.

You can use `man <func>` to find out where a standard library function is defined, and `man stdlib.h` to get documentation on that header (to see what it defines).



#### Avoid unified headers

Unified headers are bad, because they relieve the library developer of the responsibility to provide loosely-coupled modules clearly separated by their purpose and abstraction. Even if the developer (thinks she) does this anyway, a unified header increases compilation time, and couples the user's program to the entire library, regardless of if they need it.

There was a good expose on unified headers on [Programmers' Stack Exchange](http://programmers.stackexchange.com/questions/185773/library-design-provide-a-common-header-file-or-multiple-headers). An answer mentions that it's reasonable for something like GTK+ to only provide a single header file. I agree, but I think that's due to the bad design of GTK+, and it's not intrinsic to a graphical toolkit.

It's harder for users to write multiple `#include`s just like it's harder for users to write types. Bringing difficulty into it is missing the forest for the trees.



#### No global variables if you can help it (you probably can)

Global variables are just hidden parameters to all the functions that use them. They make it really hard to understand what a function does, and how it is controlled.

Mutable global variables are especially evil and should be avoided at all costs. Conceptually, a global variable assignment is a bunch of `longjmp`s to set hidden, static variables. Yuck.

Even if you have a variable that will have to be passed around to lots of a functions - if it affects their computation, it should be a parameter or a member of a parameter. This **always** leads to better software.



#### Immutability saves lives: use `const` everywhere you can

`const` improves compile-time correctness. `const` isn't just for documenting read-only pointers. It should be used for every read-only variable. It helps the reader a lot in understanding a piece of functionality. If they can look at an initialization and be sure that that value won't change throughout the scope, they can reason about the rest of the scope much easier. Without `const`, everything is up in the air; the reader is forced to comprehend the entire scope to understand what is and isn't being modified. If you consistently use `const`, then your reader will begin to trust you, and will be able to assume that a variable that isn't qualified with `const` is a signal that it *will* be changed at some point.

Using `const` everywhere you can also helps you, as a developer, reason about what's happening in the control flow of your program, and where mutability is spreading. It helps your compiler help you, which is great to have.

The compiler will warn if a pointee loses `const`ness in a function call (because that would let the pointee be modified), but it won't complain if a pointee gains `const`ness. Thus, if you *don't* specify your pointer parameters as `const` when they're read-only anyway, you discourage your users from using `const` in their code:

``` c
// Bad: sum should define its array as const.
int sum( int n, int * xs );

// Because otherwise, this will be a compilation warning:
int const xs[] = { 1, 2, 3 };
sum( 3, xs );

// => warning: passing argument 2 of ‘sum’ discards ‘const’
//             qualifier from pointer target type
```

Using `const` isn't really a choice. Lots of people consider it beneficial, so everyone should consider it required, whether they like it or not. If you don't use `const`, you force your users to either cast all calls to your functions (yuck), ignore `const` warnings (asking for trouble), or remove those `const` qualifiers (lose compile-time correctness).

If you're forced to work with a library that ignores `const`, you can write a macro that casts for you:

``` c
// `sum` will not modify the given array; casts for `const` pointers.
#define sum( n, xs ) sum( n, ( int * ) xs )
```

Only provide `const` qualifiers for pointees in function prototypes - `const` for other parameters is just an implementation detail.

``` c
// Unnecessary
bool Trie_has( Trie const, char const * const );
// Good
bool Trie_has( Trie, char const * );
```

Also, only make constant the pointees of struct members, not the struct members themselves. For example, if any of your struct members should be assignable to a string literal, give that member the type `char const *`. Qualifying other members with const turns all variables of that struct into a const, and that [hurts](http://stackoverflow.com/questions/9691404/how-to-initialize-const-in-a-struct-in-c-with-malloc) more than helps.

Use `const` for return type pointees if you can, but don't worry if you can't.

Finally, never use typecasts or pointers to get around `const` qualifiers. If the variable isn't constant, don't make it one, or if the variable is constant, add qualifiers to taste.



#### Always put `const` on the right, like `*`, and read types right-to-left

``` c
const char * word;              // Bad: not as const-y as it can be
const char * const word;        // Bad: makes types very weird to read
char const* const word;         // Bad: weird * placement

// Good: right-to-left, word is a constant pointer to a constant char
char const * const word;
```

Because of this rule, you should always pad the `*` type qualifier with spaces.



#### Always use `double` instead of `float`

Again, from *21st Century C*, by Ben Klemens:

``` c
printf( "%f\n", ( float )333334126.98 );    // 333334112.000000
printf( "%f\n", ( float )333334125.31 );    // 333334112.000000
```

Space isn't an issue anymore, but floating-point errors still are. It's much harder for numeric drift to cause problems for `double`s than it is for `float`s. `float` is another thing we just don't need anymore, so don't use it and your C programming will be simpler.

Ben Klemens says there is less of imperative to use `long`s over `int`s, but it's still something you should think about:

> Should we use long `int`s everywhere integers are used? The case isn't quite as open and shut. A `double` representation of π is more precise than a `float` representation of π, even though we’re in the ballpark of three; both `int` and `long` representations of numbers up to a few billion are precisely identical. The only issue is overflow, [when `int` will be entirely wrong.]

As well as this argument, I'm partial to using `int` (for now) over `long` just because it reads better. Also, `int` plays a large role in idiomatic C (e.g. return codes), so it would be quite jarring to ditch completely in favor of `long`.



#### Declare variables when they're needed

This reminds the reader of the type they're working with. It also suggests where to extract a function to minimize variable scope. Declaring variables when they're needed almost always leads to initialization (`int x = 1;`), rather than declaration (`int x;`). Initializing a variable usually means you can `const` it, too.

Thus, I consider all declarations shifty.



#### If you have to declare a variable, zero it

Ben Klemens, from *21st Century C*:

> If you declare a variable inside a function, then C won't zero it out automatically (which is perhaps odd for things called automatic variables). I'm guessing that the rationale here is a speed savings: when setting up the frame for a function, zeroing out bits is extra time spent, which could potentially add up if you call the function a million times and it's 1985. But here in the present, leaving a variable undefined is asking for trouble.

"Zeroing" means assigning numbers to `0` and pointers to `NULL`. Note that struct and array initializations zero all non-mentioned fields:

``` c
int zeroed[ 3 ] = { 0 };        // { 0, 0, 0 }
Book book = {};                 // { .name = NULL, .pages = 0 }
// sometimes it's better to explicitly mention your fields, though
// (like if your reader is unlikely to be familiar with the struct)
```

Unfortunately, variable-length arrays can't be initialized, so I'll usually only zero a variable-length array if it's defined in a large scope, or will be passed to other scopes.



#### Use one line per variable definition; don't bunch same types together

This makes the types easier to change in future, and atomic lines are easier to edit. If you'll need to change all their types together, you should use your editor's block editing mode.

I think it's alright to bunch struct members together, though, because struct definitions are much easier to comprehend than active code.

``` c
// Fine
typedef struct { char r, g, b; } Color;
```



#### Don't be afraid of short variable names

If the scope fits on a screen, and the variable is used in a lot of places, and there would be an obvious letter or two to represent it, try it out and see if it helps readability. It probably will!



#### Be consistent in your variable names across functions

Consistency helps your readers understand what's happening. Using different names for the same values in functions is suspicious, and forces them to check that nothing weird is happening.

Also, don't use parameter names like `self` or `this` for object-oriented functions because it doesn't make sense. C doesn't have methods, so name the parameters what they are. I consider C's separation of data and functionality one of its best features. Haskell, at the forefront of language design, makes the same choice. Try to embrace and appreciate what C offers, rather than grafting other paradigms onto it.



#### Use `bool` from `stdbool.h` whenever you have a binary value

``` c
bool print_steps = false;        // Good - intent is clear
int print_steps = 0;             // Bad - is this counting steps?
```



#### Use explicit comparisons when expecting boolean values

Explicit comparisons tell the reader what they're working with, because it's not always obvious in C. Are we working with counts or booleans or pointers?

``` c
if ( !num_kittens );            // Bad, though the name helps
if ( balance != 0 );            // Good

if ( on_fire );                 // Bad; not obvious it's a boolean
if ( is_hostile == true );      // Good

if ( !address );                // Bad; not obvious it's a pointer
if ( address == NULL );         // Good
```



#### Never change state within an expression (e.g. with assignments or `++`)

Please, please, **no state-changes in an expression, and one state-change per statement**.

``` c
Trie_add( *child, ++word );     // Bad
Trie_add( *child, word + 1 );   // Good

// Good, if you need to modify `word`
word += 1;
Trie_add( *child, word );

// Bad
if ( ( x = calc() ) == 0 );
// Good
x = calc();
if ( x == 0 );

// Fine (technically an assignment within an expression)
a = b = c;
```

But don't use multiple assignment unless the variable's values are semantically linked. If there are two variable assignments near each other that coincidentally have the same value, don't throw them into a multiple assignment just to save a line.



#### Only put function calls in expressions if it reads naturally

Assign function calls to a variable to describe what it is, even if the variable is as simple as an `int rv` (return value). This avoids surprising your readers with hidden state changes. Even if you think it's obvious, and it will save you a line - it's not worth the potential for a slip-up. Stick to this rule, and don't think about it.

The only exception is if the function name is short and reads naturally where it will be placed. For example, if the function name is a predicate, like `is_adult` or `in_tree`, then it will read naturally in an `if` expression. It's also probably fine to join these kind of functions in a boolean expression if you need to, but use your judgement. Complex boolean expressions should often be extracted to a function.

``` c
// Good
int rv = listen( fd, backlog );
if ( rv == -1 ) {
    perror( "listen" );
    return 1;
}

// Good
if ( is_tasty( banana ) ) {
    eat( banana );
}
```



#### Avoid unsigned types because the integer conversion rules are complicated

[CERT attempts to explain the integer conversion rules](https://www.securecoding.cert.org/confluence/display/seccode/INT02-C.+Understand+integer+conversion+rules), saying:

> Misunderstanding integer conversion rules can lead to errors, which in turn can lead to exploitable vulnerabilities. Severity: medium, Likelihood: probable.

*Expert C Programming* (a great book that explores the ANSI standard) also explains this in its first chapter. The takeaway is that you shouldn't declare `unsigned` variables even if they shouldn't be negative. If you want a larger maximum value, use a `long` or `long long`. Remember, lots of dynamic languages make do with a single integer type that can be either sign. Don't worry about using `size_t`: an `int` is almost always fine for an array index.

Unsigned values offer no type safety; even with `-Wall` and `-Wextra`, GCC doesn't bat an eyelid at `unsigned int x = -1;`.

*Expert C Programming* also provides an example for why you should cast all macros that will evaluate to an unsigned value.

``` c
#define NELEM( xs ) ( ( sizeof xs ) / ( sizeof xs[0] ) )
int const xs[] = { 1, 2, 3, 4, 5, 6 };

int main( void ) {
    int const d = -1;
    if ( d < NELEM( xs ) - 1 ) {
        return xs[ d + 1 ];
    }
    return 0;
}
```

The `if` branch won't be executed, because `NELEM` will evaluate to an `unsigned int` (via `sizeof`). So, `d` will be promoted to an `unsigned int`. `-1` in [two's complement](https://en.wikipedia.org/wiki/Two%27s_complement) represents the largest possible unsigned value (bit-wise), so the expression will be false, and the program will return `0`.



#### Use `+= 1` and `-= 1` over `++` and `--`

Actually, don't use either form if you can help it. Changing state should always be avoided (within reason). But, when you have to, `+=` and `-=` are obvious, simpler and less cryptic than `++` and `--`, and useful in other contexts. Python does without `++` and `--` operators, and Douglas Crockford excluded them from the good parts of JavaScript, because we don't need them. Sticking to this rule also encourages you to avoid changing state within an expression (below).



#### Use parentheses for expressions where the [operator precedence](https://en.wikipedia.org/wiki/Operators_in_C_and_C%2B%2B#Operator_precedence) isn't obvious


``` c
int x = a * b + c / d;          // Bad
int x = ( a * b ) + ( c / d );  // Good

&( ( struct sockaddr_in* ) sa )->sin_addr;      // Bad
&( ( ( struct sockaddr_in* ) sa )->sin_addr );  // Good
```

You can and should make exceptions for commonly-seen combinations of operations. For example, skipping the operators when combining the equality and boolean operators is fine, because readers are probably used to that, and are confident of the result.

``` c
// Fine
return hungry == true
       || ( legs != NULL && fridge.empty == false );
```




#### Use `if`s instead of `switch`

The `switch` fall-through mechanism is error-prone, and you almost never want the cases to fall through anyway, so the vast majority of `switch`es are longer than the `if` equivalent. Also, `case` values have to be an integral constant expression, so they can't match against another variable. Furthermore, any statement inside a `switch` can be labelled and jumped to, which fosters highly-obscure bugs if, for example, you mistype `defau1t`.

`if` has none of these issues, is simpler, and easier to change.



#### Separate functions and struct definitions with two lines

And limit yourself to a maximum of one blank line within functions - but, try to decompose your functions enough so you don't need internal blank lines.



#### Minimize the scope of variables

If a variable is only used in a contiguous sequence of lines, and only a single value is used after that sequence, then that's a great candidate for extracting to a function.

``` c
// Good: addr was only used in a part of handle_request
int accept_request( int const listenfd ) {
    struct sockaddr addr;
    return accept( listenfd, &addr, &( socklen_t ){ sizeof addr } );
}

int handle_request( int const listenfd ) {
    int const reqfd = accept_request( listenfd );
    // ... stuff not involving addr, but involving reqfd
}
```

If the body of `accept_request` were left in `handle_request`, then the `addr` variable will be in the scope for the remainder of the `handle_request` function even though it's only used for getting the `reqfd`. This kind of thing adds to the cognitive load of understanding a function, and should be fixed wherever possible.

Another tactic to limit the exposure of variables is to break apart complex expressions into blocks, like so:

``` c
// Rather than:
bool Trie_has( Trie const trie, char const * const string ) {
    Trie const * const child = Trie_child( trie, string[ 0 ] );
    return string[ 0 ] == '\0'
           || ( child != NULL
                && Trie_has( *child, string + 1 ) );
}

// child is only used for the second part of the conditional, so we
// can limit its exposure like so:
bool Trie_has( Trie const trie, char const * const string ) {
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

It can often help the readability of your code if you replace variables that are only assigned to constant expressions, with those expressions. Consider the `Trie_has` example above - the `word[ 0 ]` expression is repeated twice. Do you think it would be easier to read if we inserted an extra line to define a `char` variable?



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
setsockopt( fd, SOL_SOCKET, SO_REUSEADDR, &( int ){ 1 }, sizeof( int ) );
```



#### Use macros to eliminate repetition

C can only get you so far. The preprocessor is how you meta-program C. Too many developers don't know about the [advanced features of the preprocessor](https://en.wikibooks.org/wiki/C_Programming/Preprocessor#.23define), like symbol stringification (`#`) and concatenation (`##`). I also like to define macros to make the users' life easier. They don't have to use the macro if they don't want to, but users who do will probably appreciate it.

``` c
// Good - what harm does it do?
#define Trie_EACH( trie, index ) \
    for ( int index = 0; index < trie.size; index += 1 )

Trie_EACH( trie, i ) {
    Trie * const child = trie.children[ i ];
    ...
}
```



#### Only upper-case a macro if will act differently than a function call

By "act differently", I mean if things will break when users wouldn't expect them to. If a macro just looks different (e.g. the named parameters technique), then I don't consider that justification for an upper-case name.

[A macro will break if takes an array literal as a named parameter](http://stackoverflow.com/questions/5503362/passing-array-literal-as-macro-argument), so any such macro should have an upper-case name.

Also, if a macro repeats its parameters in its substitution, it will break things when called with an expression. GCC provides [statement expressions](http://gcc.gnu.org/onlinedocs/gcc-4.8.1/gcc/Statement-Exprs.html#Statement-Exprs) as an extension to deal with this. I think there are other techniques for fixing this, but I'm not aware of them.

Also, I don't capitalize the macro's prefix: I'd call a macro `Apple_SCARY` rather than `APPLE_SCARY`.



#### Initialize strings as arrays, and use `sizeof` for byte size

Always initialize your string literals as arrays, unless you have a very good reason. Then, with an array variable, you can use it with `sizeof` to get the byte size, rather than something error-prone and slow like `strlen( s ) + 1`.

``` c
// Good
char const message[] = "always use arrays for strings!";
write( output, message, sizeof message );
```



#### Pointer parameters only for public modifications, or for nullity

Due to [pass-by-value semantics](http://c-faq.com/ptrs/passbyref.html), structs will be "copied" when passed into functions that don't modify them. If you think this is a problem, consider:

- compilers are smart and can optimize this - if a function only uses one field of a struct, then the compiler can only copy that field
- dereferencing pointers is slow - it's better to give the function frame the actual data straight-up
- most structs are only a few bytes, so copying is negligible

Defining a *modification* is tricky when you introduce structs with pointer members (usually pointer-to-arrays - most other pointers usually aren't needed). I consider a modification to be something that affects the value's public state. This depends on common-decency of users of the interface to respect visibility comments.

So, if a struct will be "modified" by a function, have that function accept a pointer-to-const of that struct even if it doesn't need to. This saves the readers from having to trawl through and memorize every relevant struct definition, to be aware of which of its members are pointers.

``` c
typedef struct {
    int population;
} State;

typedef struct {
    bool fired;
} Missile;

typedef struct {
    State * states;
    int num_states;
    // private
    Missile * missiles;
    int num_missiles;
} Country;

// Good: takes a `Country *` even though it doesn't need to, because
// this will change the public state of `country`.
void Country_grow( Country const * const country, double const percent ) {
    for ( int i = 0; i < country->num_states; i += 1 ) {
        country->states[ i ].population *= percent;
    }
}
```

In the above example, although `missiles` is a "private" member, if there are any public `Country_` functions whose result (or side-effects) is affected by the values of the pointees of `missiles`, then changes to those pointees should be considered as affecting the public state of a `Country` object. Assuming there aren't any such functions, then no one needs to be told about changes to `missiles`.

``` c
// If there are no public methods affected by `missiles`, then this is
// fine, because it doesn't change any public state of `country`.
void Country_fire_ze_missiles( Country const country ) {
    for ( int i = 0; i < country.num_missiles; i += 1 ) {
        country.missiles[ i ].fired = true;
    }
}
```

The other case to use pointer parameters is if the function *needs* nullity (poor man's [Maybe](http://learnyouahaskell.com/making-our-own-types-and-typeclasses)). If so, **use const to signal that the pointer is not for modification**.

``` c
// Good: `NULL` represents an empty list, and list is a pointer-to-const
int List_length( List const * list ) {
    int length = 0;
    for ( ; list != NULL; list = list->next ) {
        length += 1;
    }
    return length;
}
```

If you're reading a codebase that sticks to this rule, and its functions and types are maximally decomposed, you can often tell what a function does just by reading its prototype. Furthermore, when your readers see a dereference in a call to a function, they can be totally certain that it won't be changed by that function (when you can't make the pointee constant).



#### Never use array-syntax for function parameters

[Arrays decay into pointers in most expressions](http://c-faq.com/aryptr/aryptrequiv.html), including [when passed as parameters to functions](http://c-faq.com/aryptr/aryptrparam.html). Functions can't ever receive an array as a parameter; [only a pointer to the array](http://c-faq.com/aryptr/aryptr2.html). `sizeof` won't work like an array parameter declaration would suggest; it would return the size of the pointer, not the array pointed to.

[Static array indices in function parameters are nice](http://hamberg.no/erlend/posts/2013-02-18-static-array-indices.html), but only protect against trivial situations. Also, GCC doesn't warn about their violation, only Clang. I don't consider the confusing, non-obvious syntax to be worth the small compilation check.

Yeah, `[]` hints that the parameter will be treated as an array, but so does a plural name like `pets` or `children`, so do that instead.



#### Use `assert` everywhere your program would fail otherwise

Good software fails fast. Also, assert errors are much more informative than segmentation faults. If a function is given a pointer it will dereference, assert that it's not null. If it's given an array index, assert that it's within bounds. Assert for any consistency that you need between parameters.

Don't mistake asserting for error-reporting, though. Assert things that you won't bother to check otherwise (like null pointers).



#### Use variable-length arrays rather than allocating manual memory

Since C99, arrays can now be allocated to have a length determined at runtime. Unfortunately, variable-length arrays can't be initialized.

``` c
const int num_threads = atoi( argv[ 1 ] );

// Bad
pthread_t * const threads = calloc( num_threads * sizeof pthread_t );
...
free( threads );

// Good (though memory isn't zeroed, so be careful)
pthread_t threads[ num_threads ];
// The memory is released when the scope ends.
```

As mentioned in the rule on zeroing declared variables, variable-length arrays can't be initialized, so I'll usually only zero a variable-length array if it's defined in a large scope, or will be passed to other scopes.



#### Avoid `void *` and `union`s because they harm type safety

`void*` and `union` are useful for polymorphism, but polymorphism is almost never as important as type safety. See if you can avoid them by (ab)using the preprocessor. Read *21st Century C* for inspiration.



#### If you have a `void *`, assign it to a typed variable as soon as possible

Just like working with uninitialized variables is dangerous, working with void pointers is dangerous: you want the compiler on your side. So ditch the `void *` as soon as you can.



#### Don't typecast unless you have to (you probably don't)

If it's valid to assign a value of one type to a variable of another type, then you don't have to cast it. There are only three reasons to use typecasts:

- performing true division (not integer division) of `int` expressions
- making an array index an `int`, but you can do this with assignment anyway
- using compound literals for structs and arrays; C doesn't infer them :(



#### Only typedef structs; never basic types or pointers

``` c
// Bad
typedef double centermeters;
typedef double inches;
typedef struct Apple * Apple;
typedef void * gpointer;
```

This mistake is committed by way too many codebases. It masks what's really going on, and you have to read documentation or find the `typedef` to learn how to work with it. Never do this in your own interfaces, and ignore the typedefs in other interfaces.

Also, pointer typedefs exclude the users from adding `const` qualifiers to the pointee. This is a huge loss for the readability of your codebase.

Even if you intend to be consistent about it, so that all camel-case typedefs are actually pointer to structs, you'll be violating the rule above on using pointers only for arrays and parameters that will be modified (and losing the benefits of that).



#### Give structs CamelCase names, and typedef them

``` c
// Good
typedef struct Person {
    char * name;
    int age;
} Person;
```

CamelCase names should be exclusively used for structs so that they're recognizable. They also let you name struct variables as the same thing as their type without names clashing (e.g. a `banana` of type `Banana`). You should always define the struct name, even if you don't need to, because it helps readability if and when the struct definition becomes large.

My only exception is that I don't typedef structs used for named parameters (see below), however, because the CamelCase naming would be weird. Anyway, if you're using a macro for named parameters, then the typedef is unnecessary and the struct definition is hidden.



#### Only use pointers in structs for nullity, dynamic arrays or self-references

Every pointer in a struct is an opportunity for a segmentation fault.

If the would-be pointer shouldn't be NULL, isn't an array of an unknown size, and isn't of the type of the struct itself, then don't make it a pointer. Just include the type itself in the struct. Don't worry about the size of the containing struct until you've finished development and have done benchmarks.



#### Always prefer to return a value rather than modifying pointers

This encourages immutability, cultivates [pure functions](https://en.wikipedia.org/wiki/Pure_function), and makes things simpler and easier to understand.

``` c
// Bad: unnecessary mutation (probably)
void Drink_mix( Drink * const drink, Ingredient const ingr ) {
    Color_blend( &( drink->color ), ingr.color );
    drink->alcohol += ingr.alcohol;
}

// Good: immutability rocks, pure functions everywhere
Drink Drink_mix( Drink const drink, Ingredient const ingr ) {
    return ( Drink ) {
        .color = Color_blend( drink.color, ingr.color ),
        .alcohol = drink.alcohol + ingr.alcohol
    };
}
```



#### Always use designated initializers in struct literals

``` c
// Bad - will break if struct members are reordered, and it's not
// always clear what the values represent.
Fruit apple = { "red", "medium" };
// Good; future-proof and descriptive
Fruit watermelon = { .color = "green", .size = "large" };
```



#### Use structs to provide named function parameters for optional arguments

``` c
struct run_server_options {
    char * port;
    int backlog;
};

#define run_server( ... ) \
    _run_server( ( struct run_server_options ){ \
        .port = "45680", \
        .backlog = 5, \
        __VA_ARGS__ \
    } )

// default values were specified above

int run_server( struct run_server_options opts ) {
    ...
}

int main( void ) {
    return run_server( .port = "3490", .backlog = 10 );
}
```

I learnt this from *21st Century C*. So many C interfaces could be improved immensely if they took advantage of this technique. You can define a macro to make this easier for multiple functions.

Don't use named parameters everywhere. If a function's only parameter happens to be a struct, that doesn't necessarily mean it should become the named parameters for that function. A good rule of thumb is that if the struct is used outside of that function, you shouldn't hide it with a macro like above.

``` c
// Good; the typecast here is informative and expected.
Trie_new( ( Alphabet ){ .start = 'a', .size = 26 } );
```



#### Prefer `_new()` functions with named parameters to struct literals

When a "required" member is added to a struct, all previous `_new()` calls will become errors, whereas struct literals without that required member will still compile:

``` c
// Suppose we add a required `age` field to the `Character` struct,
// and update the `_new()` function accordingly.

Character Character__new( char name[ static 1 ], int age, Character options );

#define Character_new( name, age, ... ) \
    Character__new( name, age, ( Character ){ __VA_ARGS__ } )

// Then old `_new()` calls are now compilation errors:
Character a = Character_new( "Arthur", .dexterity = 3 );

// But old struct literal definitions still compile:
Character b = { .name = "Brock", .strength = 5 };
```

Still, simplicity can often be more important than maintainability. I'll still use struct literals for trivial structs, or well-defined structs which I'm sure will require no other required parameters.

Also, `_new()` function calls become really hard to decipher when you have more than a few required parameters. I haven't worked out a way to have required, named parameters with compile-time correctness other than to have comments beside the calls. Lots of required parameters is often be a code-smell, anyway.



#### Only provide getters and setters if you need the encapsulation

*Needing* encapsulation in C is often a sign that you're overcomplicating things. Anyway, if you do, use getters and setters to show that extra computation is happening behind the scenes when they're called, and specify fields as private in the struct definition with a comment. Don't prefix private struct members with `_`: you don't need to, it looks ugly, and it ties the access level with the name, which will make it a pain to change later.

If there's nothing special happening behind the scenes, let your users get and set the struct members directly. Struct members should default to public unless you have a good reason to make them private. Yes, this will mean the public interface will have to change often. I consider the simplicity and brevity of direct member access to be worth it.

``` c
// Good
City c = City_new( "Vancouver" );
c = City_set_state( c, "BC" );
printf( "%s is in %s, did you know?\n", c.name, c.country );
```

But you should always give your users the option of [declarative programming](https://en.wikipedia.org/wiki/Declarative_programming):

``` c
// Better
City const c = City_new( "Boston", .state = "MA" );
printf( "I think I'm going to %s\n"
        "Where no one changes my state\n", c.name );
```



#### Always use `calloc` instead of `malloc`

Use `calloc` because undefined memory is dangerous. Computers are so much faster, and compilers are so much better, and `malloc` is just something we don't need anymore. Always use `calloc` and stop caring about the difference - at least, until you've finished development, and have done benchmarks.



#### If you're providing new and free functions only for a struct member, allocate memory for the whole struct

If you're providing `Foo_new` and `Foo_free` methods only so you can allocate memory for a member of the `Foo` struct, you've lost the benefits and safety of automatic storage. You may as well have the new and free methods allocate memory for the whole struct, so users can pass it outside the scope it was defined, if they want.



#### Use GNU Make's implicit rules, `CPPFLAGS`, `CFLAGS`, and `LDLIBS`

GNU Make has an implicit rule to make an object file from the corresponding C source file:

``` make
%.o: %.c
    $(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@
```

This saves you from having to specify the C source file as a dependency of the object file, or how to build most any object file. Use `CPPFLAGS` to control the preprocessor, `CFLAGS` to control the compiler, and `LDFLAGS` to control the linker. Use these variables in your own rules when you need to.

Be explicit, but try to avoid defining something you don't need to.



#### For development, always compile with warnings, optimizations and debugging

``` sh
$ gcc -g -Og -Wall -Wextra -Wpedantic -Werror
```

Optimizations can catch errors that will be missed even with all warnings on, like removing undefined behavior or unused variables. `-Og` enables optimizations that don't interfere with debugging; the GCC manual describes it as *the optimization level of choice for the edit-compile-debug cycle*. `-Werror` makes warnings errors, which speeds up the development cycle when compilation is slow.

