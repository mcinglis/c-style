# C Style

I've never done anything useful with C, but I read a lot about C, and play around with it. I often read other people's C, often in the hopes of contributing to some project, only to give up in disgust of the interface and their practices. Then, I consider implementing a greenfield version of that project, and look at what projects they depend on. Rinse repeat, until I'm considering writing wrappers around the standard library, or writing monadic Haskell or [Rust](http://www.rust-lang.org/) bindings. Sigh.

My programming ability is severely damaged by my inability to work with bad interfaces. I think the best engineers are those who can make do with what they're given; I can't, unless it's for work or school.

Anyway, this document describes my favorite C programming practices. Some points are as trivial as style, while others are more intricate. I prioritize readability, simplicity over maintainability over speed, because:

* premature optimization is the root of all evil
* compilers are better at optimizing than humans, and they're only going to get better

**Write readable, simple and maintainable software, and tune it when you're done**, with benchmarks to identify the choke points. Also, modern compilers will often change computational complexities. Simplicity and maintainability can often lead you to the best solution anyway - e.g., it's easier to write a linked list than it is to get an array to grow, but it's harder to index a list than it is to index an array.

Many of these concepts are just good programming practices, and apply outside of C programming. Issues and pull-requests are very welcome.

**This is a work-in-progress.**



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



#### Write comments in full sentences, without abbreviations, properly capitalized, with a full-stop



#### Avoid trailing comments because they're maintenance-heavy



#### Don't comment what you don't need to

You want to fit as much on screen as you can, so if you can make a line of code self-commenting rather than adding a comment line above it, do so. If you can say the same thing in one sentence which is being said in three, do so. Avoid line-expensive comment styles at all costs.



#### Don't use comments to conceal bad naming or bad design you can fix

But certainly use comments to explain bad design or bad naming forced upon you. If your project heavily depends on the bad interface, you should write a wrapper around it to improve it.



#### Comment all `#include`s to say what symbols you use from them

I've never seen any projects that do this, but I think it would be great if more did. Use `man` to find where a standard library symbol is defined. You shouldn't require readers to use ctags or grep to find where a function came from. One of these days, I'm going to write a linter that checks this...



#### No global variables if you can help it (you probably can)

Global variables are just hidden parameters to all the functions that use them. They make it really hard to understand what a function does, and how it is controlled.

Mutable global variables are especially evil and should be avoided at all costs. Conceptually, a global variable assignment is a bunch of `longjmp`s to set hidden, static variables. Yuck.

Even if you have a parameter that will be passed around to lots of a functions - if it affects their computation, it should be a parameter or a member of a parameter. This **always** leads to better code.



#### Immutability saves lives: use `const` everywhere you can

`const` isn't just for documenting read-only pointers. It should be used for every read-only variable. It helps the reader a lot in understanding a piece of functionality. If they can look at an initialization and be sure that that value won't change throughout the scope, they can reason about the rest of the scope much easier. Without `const`, everything is up in the air and you're forced to comprehend the entire scope to understand what is and isn't being modified.

But don't use pointers to get around the `const` - if the value isn't constant, don't make it one.



#### Declare variables when they're needed

This reminds the reader of the type they're working with. It also suggests where to extract a function to minimize variable scope.



#### Use one line per variable definition; don't bunch same types together

This makes the types easier to change in future, and atomic lines are easier to edit. If you'll need to change all their types together, you should use your editor's block editing mode.



#### Don't be afraid of short variable names

If the scope fits on a screen, and the variable is used in a lot of places, and there would be an obvious letter or two to represent it, try it out and see if it helps readability. It probably will!



#### Be consistent in your variable names across functions

Consistency helps your readers understand what's happening. Using different names for the same values in functions is suspicious, and forces them to check that nothing weird is happening.

Also, don't use parameter names like `self` or `this` for object-oriented functions because it doesn't make sense. C only has functions, so name the parameters what they are. I consider C's separation of data and functionality one of its best features. Haskell, at the forefront of language design, makes the same choice. Try to embrace and appreciate what C offers, rather than grafting other paradigms onto it.



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



#### Only put function calls in expressions if it reads naturally

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



#### Avoid unsigned types because the integer conversion rules are complicated

[CERT attempts to explain the integer conversion rules](https://www.securecoding.cert.org/confluence/display/seccode/INT02-C.+Understand+integer+conversion+rules), saying:

> Misunderstanding integer conversion rules can lead to errors, which in turn can lead to exploitable vulnerabilities. Severity: medium, Likelihood: probable.

*Expert C Programming* (a great book that explores the ANSI standard) also explains this in its first chapter. The takeaway is that you shouldn't declare `unsigned` variables even if they shouldn't be negative. If you want a larger maximum value, use a `long` or `long long`. Remember, lots of dynamic languages make do with a single integer type that can be either sign.

The *only* benefit of `unsigned` is that it saves a byte or two, which really isn't that important anymore. `unsigned` offers no type safety; even with all warnings on, GCC doesn't bat an eyelid at `unsigned int x = -1;`.

*Expert C Programming* also provides an example for why you should cast all macros that will evaluate to an unsigned value.

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

The `if` branch won't be executed, because the `NUM_ELS` will evaluate to an `unsigned int` (via `sizeof`). So, `d` will be promoted to an `unsigned int`. `-1` in [two's complement](https://en.wikipedia.org/wiki/Two%27s_complement) represents the largest possible unsigned value (bit-wise), so the expression will be false.



#### Use `+= 1` and `-= 1` over `++` and `--`

`+=` and `-=` are obvious, simpler and less cryptic than `++` and `--`, and useful in other contexts. Python does without `++` and `--` operators, and Douglas Crockford excluded them from the good parts of JavaScript, because we don't need them. Sticking to this rule also encourages you to avoid changing state within an expression (below).



#### Never change state within an expression (e.g. with assignments or `++`)

Please, please, **no state-changes in an expression, and one state-change per statement**.

``` c
Trie_add( *child, ++word );     // Bad
Trie_add( *child, word + 1 );   // Good

// Good, if you need to modify word
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



#### If you have an expression that depends on [operator precedence rules](https://en.wikipedia.org/wiki/Operators_in_C_and_C%2B%2B#Operator_precedence), use parentheses


``` c
int x = a * b + c / d;          // Bad
int x = ( a * b ) + ( c / d );  // Good

&( ( struct sockaddr_in* ) sa )->sin_addr;      // Bad
&( ( ( struct sockaddr_in* ) sa )->sin_addr );  // Good
```

You can and should make exceptions for commonly-seen combinations of operations. For example, skipping the operators when combining the equality and boolean operators is fine, because readers are probably used to that.

``` c
// Bad
return hungry == true || legs != NULL && fridge.empty == false;

// Good
return hungry == true
       || ( legs != NULL && fridge.empty == false );
```




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



#### Separate functions and struct definitions with two lines

And limit yourself to a maximum of one blank line within functions - but, try to decompose your functions enough so you don't need internal blank lines.



#### Minimize the scope of variables

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

Another tactic to limit the exposure of variables is to break apart complex expressions into blocks, like so:

``` c
// Rather than:
bool Trie_has( const Trie trie, const char * word ) {
    const char c = word[ 0 ];
    const Trie* child = Trie_child( trie, c );
    return c == '\0'
           || ( child != NULL
                && Trie_has( *child, word + 1 ) );
}

// child is only used for the second part of the conditional, so we
// can limit its exposure like so:
bool Trie_has( const Trie trie, const char * word ) {
    const char c = word[ 0 ];
    if ( c == '\0' ) {
        return true;
    } else {
        const Trie* child = Trie_child( trie, c );
        return child != NULL
            && Trie_has( *child, word + 1 );
    }
}
```



#### Use macros to eliminate repetition

C can only get you so far. The preprocessor is how you meta-program C. Too many developers don't know about the [advanced features of the preprocessor](https://en.wikibooks.org/wiki/C_Programming/Preprocessor#.23define), like symbol stringification (`#`) and concatenation (`##`).



#### Avoid `void*` and `union`s because they harm type safety

`void*` and `union` are useful for polymorphism, but polymorphism is almost never as important as type safety. See if you can avoid them by (ab)using the preprocessor. Read *21st Century C* for inspiration.



#### Use variable-length arrays rather than `malloc`ing arrays

Since C99, arrays can now be allocated to have a length determined at runtime.

``` c
int num_threads = atoi( argv[ 1 ] );

// Rather than:
pthread_t* threads = malloc( num_threads * sizeof pthread_t );
...
free( threads );

// Instead:
pthread_t threads[ num_threads ];
// And the memory is released when the scope ends.
```



#### Never use array types for function parameters, because they're actually pointers

``` c
int main( int argc, char* argv[] );     // Bad; argv is actually a char**
int main( int argc, char** argv );      // Good
```

[Arrays decay into pointers in most expressions](http://c-faq.com/aryptr/aryptrequiv.html), including [when passed as parameters to functions](http://c-faq.com/aryptr/aryptrparam.html), so functions can't ever receive an array as a parameter; [only a pointer to the array](http://c-faq.com/aryptr/aryptr2.html).

So, don't write function parameters as arrays because it masks what they actually receive. `sizeof` won't work like the declaration would suggest; it would return the size of the pointer, not the array pointed to.

`[]` hints that the parameter will be treated as an array, but so does a plural name like `pets` or `children`.



#### Use pointer arguments only for arrays and parameters that will be modified

``` c
// Good
size_t strlen( const char* s );
int Monster_copy( Monster source, Monster* dest );

// Bad
bool Banana_is_ripe( Banana *b ); // just reads - no need for a pointer!
```

Yes, due to [pass-by-value semantics](http://c-faq.com/ptrs/passbyref.html), structs will be "copied" when passed into functions that don't modify them. First, compilers are smart and can optimize this - if a function only uses one field of a struct, then the compiler will only copy that field. Second, giving the function frame the actual data rather than a pointer saves having to fetch that remote memory first. Third, most structs are only a few bytes, so copying is negligible.

If a function can access a pointer, account for that pointee changing. This means that structs with pointer members are liable to "changing" even if they were passed as a value into the function.

``` c
typedef struct { char **cities; } Country;

// May add and remove cities of c, even though it doesn't
// have a Country pointer.
void Country_load_cities( Country c );
```

You should consider that structs with pointer members are liable 

If you're reading a codebase that sticks to this rule, and its functions and types are maximally decomposed, you can usually tell what a function does just by reading its signature. It's almost as good as Haskell!



#### Don't typecast unless you have to (you probably don't)

If it's valid to assign a value of one type to a variable of another type, then you don't have to cast it. There are only three reasons to use typecasts:

- performing true division (not integer division) of `int` expressions
- making an array index an `int`, but you can do this with assignment anyway
- using compound literals for structs and arrays; C doesn't infer them :(



#### Always use `double` instead of `float`

``` c
// from 21st Century C, by Ben Klemens
printf( "%f\n", ( float )333334126.98 );    // 333334112.000000
printf( "%f\n", ( float )333334125.31 );    // 333334112.000000
```

Space isn't an issue anymore, but floating-point errors still are. It's much harder for numeric drift to cause problems for `double`s than it is for `float`s. `float` is another thing we just don't need anymore, so don't use it and your C programming will be simpler.

Ben Klemens says there is less of imperative to use `long`s over `int`s, but it's still something you should think about:

> Should we use long `int`s everywhere integers are used? The case isn't quite
> as open and shut. A `double` representation of π is more precise than a
> `float` representation of π, even though we’re in the ballpark of three; both
> `int` and `long` representations of numbers up to a few billion are precisely
> identical. The only issue is overflow, [when `int` will be entirely wrong.]

As well as this argument, I'm partial to using `int` (for now) over `long` just because it reads better. Also, `int` plays a large role in idiomatic C (e.g. return codes), so it would be quite jarring to ditch completely in favor of `long`.



#### `typedef` structs with CamelCase names and avoid using the `struct` namespace

``` c
typedef struct {
    char *name;
    char *state;
    char *country;
    long population;
} City;
```

I don't `typedef` structs used for named parameters (see below), however, because the CamelCase naming would be weird.

I only define the `struct` name if I have to (for self-referencing pointers).



#### Never `typedef` a pointer type

``` c
// Bad
typedef struct Apple* Apple;
typedef void* gpointer;
```

This mistake is committed by way too many codebases. It masks what's really going on, and you have to read documentation or find the `typedef` to learn how to work with it. Never do this in your own interfaces, and if you have to work with other interfaces that do it, `typedef` it back.

Even if you intend to be consistent about it, so that all camel-case typedefs are actually pointer to structs, you'll be violating the rule above on using pointers only for arrays and parameters that will be modified (and losing the benefits of that).



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


#### Avoid compound `struct` literals without designated initializers

``` c
// Bad - will break if struct members are reordered, and it's not
// always clear what the values represent.
Fruit apple = { "red", "medium" };
// Good; future-proof and descriptive
Fruit watermelon = { .color = "green", .size = "large" };
```



#### Prefer structs over a verbose interface

``` c
// Good
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

If you want encapsulation, you can still be brief and declarative with the named parameters idiom from above:

``` c
// Immutability responsibility is on the users, which makes sense.
// Other languages do the same thing.

City b = City_new( .name = "Boston", .state = "MA" );
printf( "I think I'm going to %s\n"
        "Where no one changes my state", b.name );

City v = City_new( .name = "Vancouver", .state = "BC" );
printf( "Encapsulation means I know %s is in %s\n"
        "Even though I didn't tell it!",
        v.name, v.country );
```



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
