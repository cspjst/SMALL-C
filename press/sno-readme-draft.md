# SNO - SNOBOL4 patterns for C

*I've wanted to do this since 1988!*

A small C99 library that brings the pattern matching of SNOBOL4 to C, as described in *The SNOBOL4 Programming Language*, 2nd edition, by R. E. Griswold, J. F. Poage and I. P. Polonsky (Prentice-Hall, 1971), better known as "The Green Book". 

> **Draft** - status, signatures and examples are for sno_core v0.4 and sno_extra v0.1 (28 October 2026).

## Where it started

I used SNOBOL at university and, quirks and all, parsing files with it was a breeze. However, and at the time, I loved C and wanted a C version of SNOBOL but the task was clearly a daunting one and so I carried on hand coding my parsing needs ad-hoc. (I still do love C btw warts and all!)

After decades of ad-hoc, but still SNOBOL inspired, C parsing using some ersatz and ultimately unsatisfying snobol-esque functions - the time has come to do the job "properly".

So the naive idea was: take the Green Book, work through it chapter by chapter, and turn each SNOBOL4 function into a C function. `LEN` becomes `sno_len`, `ANY` becomes `sno_any`, job done.

It was not job done. Almost every chapter turned up a place where SNOBOL and C disagree about something fundamental - what a string is, what failure is, what an error is, even what a column is. This README is about those clashes and how each one was resolved - the resolutions became the library.

## A little history

SNOBOL was created at Bell Labs in the early 1960s by David Farber, Ralph Griswold and Ivan Polonsky; SNOBOL4 followed in 1967. It is often filed under "esoteric", which is unfair. It was a serious tool for text, and its patterns are *more* powerful than regular expressions: with recursive patterns and `BAL` they can match nested structures that no true regex can.

It was also a language of the 80-column punched card, and that shows everywhere once you look. `TAB` and `POS` take column numbers. `&TRIM` exists because every card was padded with blanks. `INPUT` reads *records*, and longer ones are truncated. Even the Green Book's examples are fixed-field data - chemical elements, and the 1964 list of congressmen from New Jersey.

The Green Book survives online thanks to the scan at [regressive.org](https://www.regressive.org/snobol4/doc/books.html). Section numbers throughout this library (and its comments) refer to it.

## When Worlds Collide!

### The SNOBOL to C clashes and how they were resolved

#### Clash 1 - What *is* a string?

To SNOBOL a string is a value. To C it's a pointer and a promise that a `'\0'` is out there somewhere - the answer?...

The SNO library uses **views**: a half-open interval `[begin, end)` over characters someone else owns.

```c
typedef struct {
    sno_cursor_t begin;
    sno_cursor_t end;    // one past the last char (exclusive)
} sno_view_t;
```

A view never owns or copies text and *never* includes a terminating `'\0'` (it may contain NUL bytes, though).

#### Clash 2 - Moving between worlds

 `sno_bind` makes a view of a C string

 `sno_unbind` copies one back into a buffer when C needs it.

When matching, the subject's `begin` *is* the cursor. A pattern function advances it on success and leaves it alone on failure. That's the whole scanner.

#### Clash 3 - The null string is not failure

In SNOBOL a statement can succeed with the null string `""`, or it can *fail* and take its `:F` branch. Those are different things, and C's `NULL` can't tell them apart, at least not by itself.

So, the SNO view as ```sno_view_t``` encodes *both*:

##### Table 1. Null string and failure

|             | `begin`  | `end`    | made by                      |
| ----------- | -------- | -------- | ---------------------------- |
| null string | not NULL | == begin | `sno_null()`, `sno_bind("")` |
| failure     | NULL     | NULL     | `sno_fail()`                 |

NB Every function accepts a failure view and passes failure on, so a chain of matches simply stops making progress when one of them fails. 

This sounds obvious. Yet, I still managed to return `sno_null()` instead of `sno_fail()` from `sno_any` and spent a while wondering why "no match" was succeeding. The tests caught it, which is what they're for.

#### Clash 4 - Patterns are not functions and SNO does not backtrack

I used to like ARB but I probably over and inappropriately used it.

SNOBOL patterns backtrack. `ARB` matches nothing at first, and each time something *after* it fails, it grows by one character and tries again. A C function that has already returned can't be "backed into", so a stand-alone `sno_arb` would match the null string forever.

The first sketch was a recursive matcher with backtracking built in. It worked, but it wasn't C - it was an interpreter. So SNO does without implicit backtracking altogether: functions are greedy, commit on success, and chain by passing views. Alternatives are written by the caller with ordinary `||`, saving and restoring the view.

The Green Book turns out to be on my side here. "SPAN never matches a string shorter than the maximum span," and when later components fail, "BREAK does not skip over the break character to continue streaming". SNOBOL itself doesn't backtrack into `SPAN` or `BREAK`.

ARB's uses can readily map onto `BREAK`, a literal search ("everything up to the next `;`"), or a loop in the caller. The book itself advises against ARB for fields with known delimiters.

#### Clash 5 - Reading a line

`INPUT` is a SNOBOL variable - reading it reads a record. Its SNO equivalent `sno_input` is the function that took four rewrites to get right, because it's exactly where C strings and SNOBOL strings collide.

However, `sno_input` is where I learned the most about how SNO in C can model the semantics and, ultimately, the *utility* of SNOBOL:

- one call reads exactly one line - so a read loop always terminates
- `'\n'` is not stored; the buffer is terminated, the view excludes the `'\0'`
- an empty line is the null string but end of file is failure
- a final line without `'\n'` is a normal line
- a line longer than the buffer is truncated and the rest discarded - SNOBOL's record length, minus the card reader

#### Clash 6 - Errors are not failures

Failure means the text doesn't match whereas an error means the program is wrong

`ANY('')` - a null set - is not a failure in SNOBOL. It's error 4, NULL STRING IN ILLEGAL CONTEXT, and the program stops.

My first instinct was to return failure; my second was `assert` - neither is right. Failure would make a bug look like a non-match, and `assert` vanishes with `NDEBUG` (I've [complained about that before](https://libfbp.blogspot.com/2025/07/applying-design-by-contract-to-c.html)).

Chapter 10 of the Green Book settles it. SNOBOL's errors 1 to 16 are *conditionally* fatal: with the default `&ERRLIMIT = 0` they terminate the program. So SNO's preconditions call `sno_error()`, which prints the Green Book's own message and exits with the error number:

```
ERROR 14 -- NEGATIVE NUMBER IN ILLEGAL CONTEXT
```

##### Table 2. SNO Errors

| condition                                     | Green Book error                          |
| --------------------------------------------- | ----------------------------------------- |
| null set to `ANY`, `NOTANY`, `SPAN`, `BREAK`  | 4 NULL STRING IN ILLEGAL CONTEXT          |
| negative `LEN`, `TAB`, `RTAB`, `RPOS`         | 14 NEGATIVE NUMBER IN ILLEGAL CONTEXT     |
| NULL `FILE*`                                  | 12 ILLEGAL I/O UNIT                       |
| read error                                    | 11 READING ERROR                          |
| C-only bad arguments (NULL buffers, subjects) | 10 ILLEGAL ARGUMENT TO PRIMITIVE FUNCTION |
| result too big for the buffer (`sno_str_replace`) | 15 STRING OVERFLOW                        |

NB SNOBOL's `&ERRLIMIT` came too: `sno_err_limit(n)` lets *n* errors through, each one counted and recorded in `sno_err_type()`, and the function with the error simply fails - which is how the tests check every precondition without stopping the program.

#### Clash 7 - Identical, but not the same

`IDENT(X,Y)` in SNOBOL compares pointers. That works because SNOBOL stores each distinct string "once and only once", so equal strings share storage. 

SNO views ```sno_view_t``` are not interned. Two views of `"BCD"` in different buffers have different pointers - so `sno_ident` compares *content*. 

Copy SNOBOL's mechanism and you get the wrong answer but copy its meaning and you get the right one.

`DIFFER` is defined as the complement of `IDENT`, with one twist: in SNOBOL a failed argument means the function is never called, so in SNO *both* return false when either argument is a failure view.

#### Clash 8 - Columns and the punched card legacy

SNOBOL was built to work with 80 column punched cards and its functions and functionality are predicated on that fact. Not only is that fact an anachronism, it is also a problem for the implementation of the SNO library - a tricky one at that.

`TAB(29)` means "column 29". That needs to know where the subject started, and a view only knows its cursor and its end.

I tried adding the origin to the view, and then a separate cursor, and both cost every view, everywhere, 50% more size to help three functions. 

When in doubt fall back to the ideas behind SNOBOL. Ergo, I want to parse lines of free-format text i.e. fields split by `BREAK` and `SPAN`, not by column. Columns were a punched-card feature.

Therefore, views stay two pointers and the consequences for tabbing:

- **`RTAB` and `RPOS`** measure from the end, which every view has - they work anywhere in a chain
- **`TAB`** counts columns of the view you give it - on a freshly read line that's column *n*; after other matches it's the same as `LEN`. For true columns, `TAB` from a fresh copy of the line
- **`POS`** isn't a primitive - every view is at its own column 0, so a ported `POS(10)` would fail silently

#### Clash 9 - Returning true

SNOBOL predicates "return the null string if the condition is met and fail otherwise". There is no true or false - success *is* true.

The SNO library keeps that wherever it matters for chaining - recall that *a function that takes a subject returns a view* - the matched view, or for a cursor test like `RPOS` a null string for true and `sno_fail()` for false. 

However...

Functions that take *no* subject (`sno_ident`, `sno_differ`, `sno_failed`, `sno_succeeded`, `sno_is_null`) return `bool`, because they cross back into C semantics for the program itself.

And a list returns an index - `sno_find_lit` says *which* entry matched, so the caller can `switch` on it. A literal says what; a list says which.

## Putting it all together

The Green Book's congressmen cards, in SNO (fields TABbed from a fresh copy of the line):

```c
char buf[81];
sno_view_t card, u, name, po;

while (!sno_failed(card = sno_input(stdin, buf, sizeof buf))) {
    u = card; sno_tab(&u, 3);   name = sno_trim(sno_len(&u, 26));
    u = card; sno_tab(&u, 35);  po = sno_rem(&u);
    sno_output(stdout, name);
    // ...pad and output po...
}
```

N.B. a short card makes `sno_len` fail, `sno_trim` passes the failure on, and `sno_output` writes nothing - no checks needed in between. That's clash 3 (null versus failure) paying for itself.

## Why not just use regex?

For a quick search, regex is shorter and you should indeed use it - but for parsing perhaps not?

## The epiphany

Somewhere around clash 4 (backtracking) with SNOBOL as C functions, I realised that SNOBOL patterns with greedy, committing primitives and ordered choice are a *Parsing Expression Grammar* - this was formalised by [Bryan Ford in 2004](https://bford.info/pub/lang/peg.pdf), and the model behind a good many modern parser generators. 

In this way the SNO library evolved as a 2004 idea built out of a 1971 book, which is either very old or very new, depending how you look at it. 

## Status (sno_core v0.4, sno_extra v0.1)

| Green Book   | Function                                                     |             |
| ------------ | ------------------------------------------------------------ | ----------- |
| 1.1          | `sno_bind`, `sno_unbind`                                     | done        |
| 1.1          | `sno_bind_list` - on death row, for review at v1.0           | done        |
| 1.3.1        | `sno_null`, `sno_is_null`                                    | done        |
| 1.3.4        | `sno_input`, `sno_in`, `sno_output`, `sno_out`               | done        |
| 1.4          | `sno_lit`                                                    | done        |
| 1.4          | `sno_find_lit`                                               | in progress |
| 1.5          | `sno_str_replace`                                            | done        |
| 1.8          | `sno_fail`, `sno_failed`, `sno_succeeded`, `S_`, `F_`        | done        |
| 1.10.1, 1.11 | `sno_size`, `sno_trim`                                       | done        |
| 2.8          | `sno_len`                                                    | done        |
| 2.9          | `sno_break`                                                  | done        |
| 2.10         | `sno_any`, `sno_notany`                                      | done        |
| 2.11         | `sno_tab`, `sno_rtab`, `sno_rem`                             | done        |
| 2.12         | `sno_rpos`                                                   | done        |
| 3.3.1        | `sno_ident`, `sno_differ`                                    | done        |
| 3.4.2        | `sno_chr_replace`                                            | done        |
| 10.2.2       | `sno_err_limit`, `sno_err_count`, `sno_err_type`             | done        |
| 10.2.3       | `sno_error`                                                  | done        |
| 2.9          | `sno_span`                                                   | next        |
| 2.19         | `sno_bal` - with the bracket pair as an argument, so `[bx+si]` works as well as `(a+b)` | planned     |
| 2.7          | `sno_at`                                                     | parked      |

## Colophon

C99. Built with Open Watcom for 16-bit DOS (tiny model, .COM) and with gcc/clang on modern machines. Tested with plain `assert.h` - two test programs, one per header, with every precondition checked under `sno_err_limit` - plus a DOS batch file that checks the real halt's `ERRORLEVEL`, under DOSBox. Written as a general-purpose library for parsing, translating and reasoning about information stored in strings, with a bent towards language compilers and interpreters - the first being a Small-C compiler targeting the 8086 via p-code, assembled with Open Watcom's WASM and WLINK.

The Green Book: Griswold, R. E., Poage, J. F. and Polonsky, I. P., *The SNOBOL4 Programming Language*, 2nd ed., Prentice-Hall, 1971.

I hope this has been of some use to anyone interested in these ideas.

Jeremy Simon Thornton - [libfbp.blogspot.com](https://libfbp.blogspot.com/)

MIT Licence