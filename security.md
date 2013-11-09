# C Security

This guide is licensed under the [Creative Commons Attribution-ShareAlike](/license.md) license, so I'm not liable for anything you do with this.

**Warning:** these notes are unfinished and should not be depended on. Pull requests and issues are very welcome.

Always use `-fstack-protector-strong` to enable stack-smashing protection.

### Never use `gets`; use `fgets` instead

``` c
// Bad
int main( void ) {
    char buf[ 64 ];
    // Buffer overflow if the received string doesn't have `\0` in the
    // first 64 bytes, which is totally possible.
    gets( s );
}

// Good
int main( void ) {
    int max = 64;
    char buf[ max ];
    fgets( buf, max, stdin );
}
```

## Classic string functions

All of the standardized `str` functions are very hard to use correctly, and thus are extremely dangerous. C11's Annex K functions are a huge improvement, but they aren't yet available in any C standard library implementation.

* [Use the bounds-checking interfaces for remediation of existing string manipulation code](https://www.securecoding.cert.org/confluence/display/seccode/STR07-C.+Use+the+bounds-checking+interfaces+for+remediation+of+existing+string+manipulation+code)
* [Functions that read or write to or from an array should take an argument to specify the source or target size](https://www.securecoding.cert.org/confluence/display/seccode/API02-C.+Functions+that+read+or+write+to+or+from+an+array+should+take+an+argument+to+specify+the+source+or+target+size)

You should always **be very careful** when processing strings in C.


### Duplicating and copying strings

If you're creating an empty buffer, and copying an existing string into that buffer, you're duplicating it. If you're copying an existing string into an existing buffer, you're copying it. Carefully consider what you're doing, because duplication is far less error-prone than copying.

To duplicate a string `src`:

``` c
char * const copy = strndup( src, MAX_COPY_SIZE );
...
free( copy );
```

If the length of `src` is controlled by your program, you can use `strdup`. If `src` could come from an external computer (consider this possibility carefully), use `strndup` so clients can't get you to allocate all your memory.

On the copying functions, `strcpy()` and `strncpy()`, the Linux man-pages say:

> If the destination string of a `strcpy()` is not large enough, then anything might happen. Overflowing fixed-length string buffers is a favorite cracker technique for taking complete control of the machine. Any time a program reads or copies data into a buffer, the program first needs to check that there's enough space.

`strcpy( src, dest )` will copy each byte of `src` into `dest` up to and including the terminating null byte `\0`. You should always check that `dest` has enough space to hold `src`, *including the null byte*. This means that `dest` needs to be at least `strlen( src ) + 1` bytes large.

``` c
if ( strlen( src ) < dest_size ) {
    strcpy( dest, src );
} else {
    // error
}
```

If you don't need to prompt an error if `dest` can't contain all of `src`, and instead just want to copy over as many bytes as you can, you could use `strncpy( src, dest n )` to copy at most `n` bytes into `dest`. There are two common downfalls with `strncpy`, though:

1. If `strlen( src ) >= n` (that is, if there's no null byte in the first `n` bytes of `src`), `dest` won't be null-terminated.
2. If `strlen( src ) < n` (that is, if `src` is shorter than `n`), there will be `n - strlen( src )` null bytes written into `dest` so that `n` bytes are written in total. This can slow your program down if `n` is large and `src` is small, and the `strncpy` operation is in an execution hot-spot.

`strncpy` should only be used if you need the second behavior (which is rare; consider alternative approaches like initialization to `{ 0 }` or `calloc`). If you need the first behavior, just use `memcpy( dest, src, n )` which explicitly communicates what you're doing (i.e. byte-agnostic copying). If you are going to use `strncpy`, you should write a wrapper function to insert the null-terminator in case `src` is longer than `n`:

``` c
void strncpy_term( char * const dest, char const * const src, size_t n )
{
    if ( n == 0 ) {
        return;
    }
    strncpy( dest, src, n - 1 );
    dest[ n - 1 ] = '\0';
}
```

`strncpy` is not secure version of `strcpy`: their use cases are different, and they have their own pitfalls.

* <http://the-flat-trantor-society.blogspot.com/2012/03/no-strncpy-is-not-safer-strcpy.html>
* <http://stackoverflow.com/questions/2114896/why-is-strlcpy-and-strlcat-considered-to-be-insecure/2115015#2115015>

What should you use then, if not `strncpy`, to copy over as many bytes as you can from `src` to `dest`? Linus Torvalds [suggested](http://yarchive.net/comp/linux/strncpy.html) a function for such a use case like this (rewritten to be clearer):

``` c
int copy_string( char * const dest, char const * const src, size_t const n )
{
    if ( n == 0 ) {
        return 0;
    }
    size_t const src_len = strlen( src );
    size_t const max = ( src_len < n ) ? src_len : n - 1;
    memcpy( dest, src, max );   // src[i] = dest[i] from 0 to max-1
    dest[ max ] = '\0';
    return max;
}
```


### Concatenation


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

