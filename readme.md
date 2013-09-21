# C Style

I've never done anything useful with C, but I read a lot about C, and play around with it. I often read other people's C, often in the hopes of contributing to some project, only to give up in disgust of the API and their practices. Then, I consider implementing a greenfield version of that project, and look at what projects they depend on. Rinse repeat, until I'm considering writing wrappers around the standard library, or writing [Rust](http://www.rust-lang.org/) bindings.

My programming ability is severely damaged by my inability to work with sub-optimal codebases. I think the best engineers are those who can make do with what they're given; I can't, unless it's for work or school.

Anyway, this document describes my favorite C programming practices. I usually prioritize simplicity over maintainability over readability over speed. Discussion, issues and pull-requests are very welcome.

**This is a work-in-progress.**


## Programming

### Only use assignments as expressions when assigning multiple variables

``` c
a = b = 0;                      // Good
if ( ( x = get_x() ) == 0 )     // Bad
```

### Rarely put function calls in conditional expressions

Assign the function call to a variable to describe what it is, even if the variable is as simple as an `int rv`.

The exception is if the function name is short and reads naturally where it will be placed. For example, if the function name is a predicate, like `is_adult` or `in_tree`, then it will read naturally in an `if` expression.

``` c
// Good
pid_t child_pid = fork();
if ( child_pid == -1 ) {
    ...
} else if ( child_pid == 0 ) {
    ...
}

// Good
if ( is_tasty( banana ) ) {
    eat( banana );
}
```

### Minimize the scope of variables by extracting functions

``` c
// Good: addr was only used in a part of handle_request
int accept_request( int listenfd ) {
    struct sockaddr addr;
    return accept( listenfd, &addr, &( socklen_t ){ sizeof addr } );
}

int handle_request( int listenfd ) {
    int 

```

### Declare variables where they're used

This reminds the reader of the type they're working with. It also suggests where to extract a function to minimize variable scope.

### Immutability saves lives; use `const` everywhere you can

### Use pointer arguments only for arrays and parameters that will be modified

``` c
// Good
size_t strlen( const char *s );
int Monster_copy( Monster source, Monster *dest );

// Bad
void Banana_is_ripe( Banana *b ); // just reads - no need for a pointer!
```

So long as your `struct`s are small, don't worry about copying.

### Use `bool` from `stdbool.h` whenever you have a binary value

``` c
bool print_steps = false;        // Good - intent is clear
int print_steps = 0;             // Bad - is this counting steps?
```

### Short names for small scopes, long names for large scopes

### `typedef` object-oriented `struct`s with CamelCase names

``` c
typedef struct {
    char *name;
    char *country;
    unsigned long population;
} City;
```

### Use and abuse designated `struct` initializers

#### Prefer designated initializers over a verbose interface

``` c
// Good
City c = { "Vancouver", "Canada" };
City c = {
    .name = "Manchester",
    .country = "United Kingdom",
    .population = 2_500_000
};

// Bad
City c = City_new();
City_set_name( c, "Brisbane" );
City_set_country( c, "Australia" );
```

Keep (required) mutability to a minimum; prefer declarative variable definitions rather than initializing and setting members.

``` c
// Good

```

## Compiler options

### Compile with debugging, optimizations and all warnings

``` sh
$ gcc -g -Og -Wall -Wextra -Wpedantic
```

`-Wpedantic` might not be justified; it

### Write to the most modern standard you can

``` sh
$ gcc -std=gnu11 source.c
```
