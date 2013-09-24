# C Style

I've never done anything useful with C, but I read a lot about C, and play around with it. I often read other people's C, often in the hopes of contributing to some project, only to give up in disgust of the interface and their practices. Then, I consider implementing a greenfield version of that project, and look at what projects they depend on. Rinse repeat, until I'm considering writing wrappers around the standard library, or writing monadic Haskell or [Rust](http://www.rust-lang.org/) bindings. Sigh.

My programming ability is severely damaged by my inability to work with bad interfaces. I think the best engineers are those who can make do with what they're given; I can't, unless it's for work or school.

Anyway, this document describes my favorite C programming practices. Some points are as trivial as style, while others are more intricate. I prioritize readability, simplicity over maintainability over speed, because:

* premature optimization is the root of all evil
* compilers are better at optimizing than humans, and they're only going to get better

**Write readable, simple and maintainable software, and tune it when you're done**, with benchmarks to identify the choke points. Also, modern compilers will often change computational complexities. Simplicity and maintainability can often lead you to the best solution anyway - e.g., it's easier to write a linked list than it is to get an array to grow, but it's harder to index a list than it is to index an array.

Many of these concepts are just good programming practices, and apply outside of C programming. Issues and pull-requests are very welcome.

**This is a work-in-progress.**



#### Write to the most modern standard you can



#### We can't get tabs right, so use spaces everywhere



#### Separate functions and struct definitions with two lines



#### Use `//` comments everywhere, never `/* ... */`

`/* ... */` is never used with a blank margin, and is more complicated and error-prone anyway; it offers no advantages over `//`.



#### Don't use comments to conceal bad naming or bad design you can fix

But certainly use comments to explain bad design or bad naming forced upon you. If your project heavily depends on the bad interface, you should write a wrapper around it to improve it.



#### Comment all `#include`s to say what symbols you use from them

I really wish some projects did this (I've never seen any that do). Use `man` to find where a standard library symbol is defined. You shouldn't require readers to use ctags or grep to find where a function came from. One of these days, I'm going to write a linter that checks this...

I like doing this so I can Ctrl+F on any keyword to find where it came from. I really wish more codebases had this. It's terrible to require readers use ctags or grep to find where a function comes from.



#### Use `if`s instead of `switch`

The `switch` fall-through mechanism is error-prone, and you almost never want the cases to fall through anyway, so the vast majority of `switch`es are longer than the `if` equivalent. Also, `case` values have to be an integral constant expression, so they can't match against another variable. Furthermore, any statement inside a `switch` can be labelled and jumped to.

Spot the bug:

``` c
switch ( x ) {
    case 0:
        printf( "Halt!" );
        break;
    defau1t:
        printf( "Carry on" );
        break;
    case 42:
        printf( "Apologies!" )
        break;
}
```

Actually, the `case`s and the `default` can be defined in arbitrary order, so that's not the bug. The bug is that the `default` above has a typo: it's actually a label called `defau1t` (with a one instead of an el).

`if` has none of these issues, is simpler, and easier to change.



#### Avoid unsigned types because the type promotion rules are complicated

*Expert C Programming* (a great book that explores the ANSI standard) explains this in its first chapter. The takeaway is that you shouldn't define `unsigned` variables even if it wouldn't make sense for them to be negative.

The book also provides an example for why you should cast all macros that will evaluate to an unsigned value.

``` c
#define NUM_ELS( xs ) ( ( sizeof xs ) / ( sizeof xs[0] ) )
const int xs[] = { 0, 1, 2, 3, 4, 5, 6 };

main() {
    int x;
    int d = -1;
    if ( d < NUM_ELS( xs ) - 1 )
        x = xs[ d + 1 ];
}
```

The `if` branch won't be executed, because the `NUM_ELS` will evaluate to an `unsigned int` (via `sizeof`). So, `d` will be promoted to an `unsigned int`. `-1` in [two's complement](https://en.wikipedia.org/wiki/Two%27s_complement) represents the largest possible unsigned value, so the expression will be false.



#### Be explicit when expecting boolean values

Explicit comparisons tell the reader what they're working with, because it's not always obvious in C. Are we working with counts or booleans or pointers?

``` c
// Good
if ( balance != 0 );
if ( is_hostile == true );
if ( address == NULL );

// Bad
if ( on_fire );
if ( !num_kittens );
if ( !address );
```



#### Use one line per variable definition; don't bunch same types together

This makes the types easier to change in future, and atomic lines are easier to edit.



#### Don't be afraid of short variable names

If the scope fits on a screen, and the variable is used in a lot of places, and there would be an obvious letter or two to represent it, try it out and see if it helps.



#### Use `+= 1` and `-= 1` over `++` and `--`

`+=` and `-=` are obvious, simpler and less cryptic than `++` and `--`, and useful in other contexts. Python does without `++` and `--` operators, and Douglas Crockford excluded them from the good parts of JavaScript, because we don't need them. Sticking to this rule also encourages you to avoid changing state within an expression (below).


#### Never change state within an expression (e.g. with assignments or `++`)

``` c
Trie_add( *child, ++word );     // Bad
Trie_add( *child, word + 1 );   // Good
// And, if you need to:
word += 1;
Trie_add( *child, word );

if ( ( x = calc() ) == 0 );     // Bad
// Good
x = calc();
if ( x == 0 );

// Fine (technically an assignment within an expression)
a = b = c;
```

But don't use multiple assignment unless the variable's values are semantically linked. If there are two variable assignments near each other that coincidentally have the same value, don't throw them into a multiple assignment just to save a line.


#### Only put function calls in conditional expressions if it reads naturally

Assign the function call to a variable to describe what it is, even if the variable is as simple as an `int rv` (return value).

The exception is if the function name is short and reads naturally where it will be placed. For example, if the function name is a predicate, like `is_adult` or `in_tree`, then it will read naturally in an `if` expression. It's also probably fine to join these kind of functions in a boolean expression if you need to, but use your judgement. Complex boolean expressions should often be extracted to a function.

``` c
// Good
int rv = listen( fd, backlog );
if ( rv == -1 ) {
    perror( "listen" );
    return -1;
}

// Good
if ( is_tasty( banana ) ) {
    eat( banana );
}
```


#### Minimize the scope of variables by extracting functions

If a variable is only used in a contiguous sequence of lines, and only a single value is used after that sequence, then that's a great candidate for extracting to a function.

``` c
// Good: addr was only used in a part of handle_request
int accept_request( const int listenfd ) {
    struct sockaddr addr;
    return accept( listenfd, &addr, &( socklen_t ){ sizeof addr } );
}

int handle_request( const int listenfd ) {
    const int reqfd = accept_request( listenfd );
    // ... stuff not involving addr, but involving reqfd
}
```

If the body of `accept_request` were left in `handle_request`, then the `addr` variable will be in the scope for the remainder of the `handle_request` function even though it's only used for getting the `reqfd`. This kind of thing adds to the cognitive load of understanding a function, and should be fixed wherever possible.


#### Declare variables where they're used

This reminds the reader of the type they're working with. It also suggests where to extract a function to minimize variable scope.


#### Immutability saves lives; use `const` everywhere you can

`const` isn't just for documenting read-only pointers. It should be used for every read-only variable. It helps the reader a lot in understanding a piece of functionality. If they can look at an initialization and be sure that that value won't change throughout the scope, they can reason about the rest of the scope a lot easier. Without `const`, everything is up in the air and you can't trust anything.

But, don't use pointers to get around the `const` - if it's not actually a constant, don't make it one.


#### If a macro will give you better type safety, do it

`void*` should be avoided with reasonable effort. Not only do you lose type safety with it, but it can also lead to sub-optimal solutions. If you can achieve what you need without it, do so. Polymorphism isn't as important as type safety.


#### Use macros to eliminate repetition

C can only get you so far. The preprocessor is how you meta-program C. Too many developers don't know about the [advanced features of the preprocessor](https://en.wikibooks.org/wiki/C_Programming/Preprocessor#.23define), like symbol stringification (`#`) and concatenation (`##`).



#### Never `typedef` a pointer type

``` c
// Bad
typedef struct Apple* Apple;
typedef void* gpointer;
```

This mistake is committed by way too many codebases. It masks what's really going on, and you have to read documentation or find the `typedef` to work out how to work with it. Never do this in your own interfaces, and if you have to work with other interfaces that do it, `typedef` it back.

Even if you intend to be consistent about it, so that all camel-case typedefs are actually pointer to structs, you'll be violating the rule below on using pointers only for arrays and parameters that will be modified (and losing the benefits of that).



#### Use pointer arguments only for arrays and parameters that will be modified

``` c
// Good
size_t strlen( const char* s );
int Monster_copy( Monster source, Monster* dest );

// Bad
bool Banana_is_ripe( Banana *b ); // just reads - no need for a pointer!
```

Yes, this means structs will be "copied" when passed into functions that don't modify them. First, compilers are smart and can optimize this - if a function only uses one field of a struct, then the compiler will only copy that field. Second, giving the function frame a pointer rather than data means the computer has to fetch that remote memory first. Third, most structs are only a few bytes, co copying is negligible.

If you're reading a codebase that sticks to this rule, and its functions and types are maximally decomposed, you can usually tell what a function does just by reading its signature. It's almost as good as Haskell!



#### Use `bool` from `stdbool.h` whenever you have a binary value

``` c
bool print_steps = false;        // Good - intent is clear
int print_steps = 0;             // Bad - is this counting steps?
```



#### No global variables if you can help it (you probably can)

Global variables are just hidden parameters to all the functions that use them. They make it really hard to understand what a function does, and how it is controlled.

Mutable global variables are especially evil and should be avoided at all costs. They effectively function as `longjmp`s for hidden parameters. Yuck.

Even if you have a parameter that will be passed around to lots of a functions, if it affects their computation, it should be a parameter; either for the functions, or of a struct passed into the functions.



#### `typedef` structs with CamelCase names and avoid using the `struct` namespace

``` c
typedef struct {
    char *name;
    char *country;
    long population;
} City;
```

My only exception to this is when defining structs for named parameters, as described below, because the CamelCase naming wouldn't work then.

I only define the `struct` name if I have to (for self-referencing pointers).



#### Use structs to provide named function parameters for optional arguments

``` c
struct run_server_options {
    char *port;
    int backlog;
};

#define run_server( ... ) \
    _run_server( ( struct run_server_options ){ \
        // Default values \
        .port = "45680", \
        .backlog = 5, \
        __VA_ARGS__ \
    } )

int run_server( struct run_server_options opts ) {
    ...
}

int main( void ) {
    return run_server( .port = "3490", .backlog = 10 );
}
```

I learnt this from *21st Century C*. So many C interfaces could be improved immensely if they took advantage of this technique. You can define a macro to make this easier for multiple functions.

You can use this for required arguments too, obviously, but you lose compile-time checking. I haven't worked out a way to get the best of both worlds: named arguments and compile-time checks.

I'd recommend you *don't* use this technique if the named parameters are more like properties of an object. A good signal of this is if you're defining variables of the same struct type and not using them for calls to the function. Then, you should probably explicitly define the struct, and pass it in anonymously (if you need to do that) without the macro. This keeps the struct's usage consistent.

``` c
// Good
Character_greet( ( struct Character ){
    .name = "Bob",
    .hometown = "Birmingham"
} );
```



#### Prefer structs over a verbose interface

``` c
// Good
City c1 = { "Vancouver", "Canada" };
City c2 = {
    .name = "Manchester",
    .country = "United Kingdom",
    .population = 2_500_000
};
printf( "%s is in %s\n", c2.name, c2.country );

// Bad
City c = City_new();
City_set_name( c, "Brisbane" );
City_set_country( c, "Australia" );
printf( "%s is a pretty cool guy\n", City_get_name( c ) );
```

Keep (required) mutability to a minimum; prefer declarative variable definitions rather than initializing and setting members.



#### Compile with debugging, optimizations and all warnings.

``` sh
$ gcc -g -Og -Wall -Wextra
```

Optimizations can catch errors that will be missed even with all warnings on. `-Og` means optimize for debugging, so you get all the benefits, but none of the drawbacks (except compilation speed).



#### Use `CFLAGS`, `LDLIBS` and Make's knowledge of C

I have a directory of GTK examples I've written, and this is the Makefile I use to compile each source file to its corresponding binary. Make knows how to do the rest.

``` make
CFLAGS = `pkg-config --cflags gtk+-3.0` -std=gnu11 -g -0g -Wall -Wextra
LDLIBS = `pkg-config --libs gtk+-3.0`

all: $(basename $(wildcard *.c))
```
