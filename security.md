# C Security

**Warning:** these notes are unfinished and should not be depended on.

## Buffer overflows

#### Never use `gets`; use `fgets` instead

``` c
// Bad
int main( void ) {
    char buf[ 64 ];
    // Buffer overflow if s is longer than 64 characters
    gets( s );
}

// Good
int main( void ) {
    int max = 64;
    char buf[ max ];
    fgets( buf, max, stdin );
}
```

#### `strcpy` is dangerous with an unchecked source

``` c
// If you know the size of the destination buffer, you can add an
// explicit check:
if ( strlen( src ) < dest_size ) {
    strcpy( dest, src );
} else {
    // error
}

// But an easier way to do this is with strncpy():
strncpy( dest, src, dest_size - 1 );
// If src length is more than dest_size - 1, then strncpy won't write
// null bytes to dest, so to be safe:
dest[ dest_size - 1 ] = '\0';

// Another way to protect strcpy() against overflows is to manually
// allocate space as required:
char * const dest = malloc( strlen( src ) + 1 );
strcpy( dest, src );

// String literals are harmless, though.
strcpy( buf, "Hello!" );
```


**Always use `-fstack-protector-strong` to enable stack-smashing protection.**


## Format string vulnerabilities

``` c
int main( void ) {
    int n = 0;
    char buffer[] = "abcdefghijklmnopqrstuvwxyz";
    printf( "%.20x%n", buffer, &n );
    printf( "\nNumber of bytes formatted in previous printf: %d\n", n );
}
```

The format string has written to a value to another memory location. We've lost memory safety!

**Solution:** always enable `-Wformat=2` (which isn't enabled by `-Wall` or `-Wextra`) to warn when format strings aren't string literals.

