# fk-regex

Regular expressions for [fckt](https://github.com/m4xy/fckt), matched in time linear in the text,
whatever the pattern: no pattern can make it backtrack forever, as `(a*)*b` makes most matchers do on
a long run of `a`s. It's Pike's machine, as RE2 and Rust's regex crate have it: every way the pattern
could match is followed at once, a character at a time, in order of preference, so the match found
is the one a backtracking matcher would find first.

```kotlin
import uk.m4xy.regex.*
import uk.m4xy.regex.Regexes.*

email = "(\\w+)@(\\w+)\\.com".toRegex()          // fails, saying why, if it isn't a pattern

email.find("mail ada@example.com now")           // Some(Match("ada@example.com", 5, 20, [Some("ada"), Some("example")]))
email.matches("ada@example.com")                 // true: all of the text
"\\d+".toRegex().findAll("a1b22c333")            // matches of 1, 22 and 333
email.replace("ada@example.com", "$2:$1")         // "example:ada"
",\\s*".toRegex().split("a, b,c")                // ["a", "b", "c"]
regex("(ab")                                     // Error(RegexError(0, "this ( is never closed"))
```

A [kiln](https://github.com/m4xy/kiln) project uses it as `library("uk.m4xy:fk-regex:0.1.0")`.

## Patterns

| | |
|---|---|
| `abc` | the characters themselves |
| `.` | any character but a line break |
| `[a-z_]`, `[^0-9]` | a character in, or not in, the class; `]` first, or `-` last, is itself |
| `\d \w \s`, `\D \W \S` | a digit, word character (`[A-Za-z0-9_]`) or space, or anything else |
| `^`, `$` | the start or end of the text |
| `\b`, `\B` | at a word's edge, or not |
| `x*`, `x+`, `x?`, `x{n}`, `x{n,}`, `x{n,m}` | repeats, as many as can be; with `?` after, as few |
| `x\|y` | `x`, or else `y` |
| `(x)`, `(?:x)` | a group, numbered from 1, whose match is captured; or one that only groups |
| `\n \t \r \f \v \0`, `\u{1F600}` | a control character, or any by its code point |
| `(?i)` at the start | the letters A to Z in either case |

Any other character after `\` is itself, as `\.` and `\(` are; a letter after `\` that means nothing
is an error, not the letter, so a mistyped escape doesn't quietly match something else.

## Matching

`Regex.find(text, from)`, `findAll`, `matches`, `containsMatchIn`, `replace`, `replaceWith` and
`split`. A `Match` has the text matched, where it `start`s and `end`s, and `group(n)`: what group `n`
matched, if it took part, `0` being the whole match. Indexes count characters, code points, as
fckt's `Char` does.

In a replacement, `$0` is the whole match, `$1` to `$9`, or `${12}`, a group's, and `$$` a `$`.

## How it's built

`Syntax` reads a pattern into a `Node` tree; `Compiler` makes it a program of `Instruction`s, as
Thompson's construction does; `Machine` runs it. `kiln build` runs the tests in `test/`.
