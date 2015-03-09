# Regular Expression Notes

<img src="../images/xkcd-208.png" width="50%"
     style="display: block; margin-left: auto; margin-right: auto">

## Overview

A lot of programs can be summarized as:

1. Take some text as input (text possibly defined broadly)
1. Process it
1. Print some text as output

Many of the most powerful tools for working with text rely on or are
at least a lot more powerful with the use of regular expressions.  If
you’re comfortable with regular expressions, they will most likely
make your life as a developer a lot easier.

If you take CMPSCI 250 or 501 (quite probably with Dave Barrington),
you’ll learn all about regular languages, which are the theoretical
underpinnings of regular expressions.  We’re going to focus on the
practical side of regular expressions, though with occasional detours
to the theory. If you’ve taken 250 and/or 501, bear with us: you
might still learn something.

**Theory**: Mathematically speaking, a language is a set of strings.
A regular expression is one way of describing a subset of languages,
the regular languages (the definition for which boils down to
“languages which can be described by regular expressions”).

**Practice**: Regular expressions describe patterns against which we
test text to see if the text matches.  In Scala:

```scala
trait RegularExpression {
  def matches(s: String): Boolean
}
```

(there are other operations, but this is the essential one)

Regular expressions are an essential part of Unix culture;
historically, they’ve tended to only get introduced to other computing
cultures via Unix refugees.

**History**: Stephen Kleene developed regular expressions in the
1950s.  For about a decade they were of primarily theoretical
interest, until Ken Thompson published an efficient algorithm which
ultimately compiled a regular expression into machine language; he
would use this algorithm to efficiently implement regular expression
features in the original Unix text editor ed.

A common use of ed was to print only the lines of a file which matched
(more properly, which had a substring which matched) a regular
expression.  The command in ed to do this was g/re/p, which translates
as “globally search the text for lines matching the regular expression
(delimited by /’s: this is a convention which persists in many
languages to this day) `re` and print those lines".  Eventually,
Thompson would take this functionality out and create a program called
grep.

**Demonstration of grep**: Nearly any Unix system, including the
virtual machines distributed for this class, has grep and a dictionary
file of words.

```bash
$ grep -i 't.*i.*m' /usr/share/dict/words | less
```

This command will print all the words in a dictionary which contain
(ignoring case) a `t` followed somewhere later by an `i` followed
somewhere later by an `m` (the `|` less part simply causes the output
of grep to be fed to `less` which breaks the output into screenfuls so
we can scroll through it (there are a lot of words matching this
pattern!).  This also exhibits part of the Unix philosophy: develop
programs which do a focused task and can be easily combined.
Composability and delegation as design principles!)

**History**: Another pair of Unix tools for sculpting text are sed and
awk, which allow for fairly powerful manipulations of text using
regular expressions.  Their limitations inspired the development of
the Perl programming language where regular expressions take center
stage.  From Perl, regular expressions spread out to Python, Ruby,
Javascript, Java, C#, PHP, and nearly any language above the level of
assembly language you’ll find in use today.

**Expressing regular expressions**: We can build up regular
expressions recursively (or if you prefer, inductively).  For pattern
matching, the lowest level we go is characters (for theoretical
purposes, there are two regular expressions which are, in a sense,
even lower level: an expression which only matches the empty string
("" in most languages you’re familiar with) and an expression which
doesn’t actually match any string).  The regular expression matching a
single character is that character (though, since we’re representing
regular expressions in our code or on the command line as text and as
we’ll see, some characters have special meanings in the expression, we
need to escape some characters.  We do this by sticking a \ in front
of the character (this makes \ such a special character, so the
pattern that matches a single \ is \\).

```bash
$ grep -i 'a' /usr/share/dict/words | wc -l
```

Counts all the words which have an `a` in them.

The `.` character matches any character.

We can concatenate regular expressions by mashing them together:

```bash
$ grep -i 'ab' /usr/share/dict/words | wc -l
```

Counts all the words which have an `a` immediately followed by a `b`.

We can give the regular expression a *choice* by placing a `|` between
two regular expressions:

```bash
$ grep -i 'ab|bc' /usr/share/dict/words | wc -l

# OOPS!  Doesn't actually match anything.  Try egrep
$ egrep -i 'ab|bc' /usr/share/dict/words | wc -l

# Or try escaping the |
$ grep -i 'ab\|bc' /usr/share/dict/words | wc -l
```

grep (and some other Unix tools) defaults to *basic* regular
expressions, where a number of operators need to be escaped.  egrep is
a modification of grep which uses extended regular expressions, where
the operators don't need to be escaped.  We'll use the extended syntax
from here.

We can specify precedence by enclosing regular expressions within
parentheses to get a regular expression which we can more easily embed
in a larger regular expression:

```bash
$ egrep 'a(b|c).d' > scratch
abide
acid
abdomen
acknowledge
$ cat scratch
abide
acid
```

We can specify that we want zero or more instances of a regular
expression with `*`:

```bash
$ egrep '(ab.)*' > scratch
abo
absent
cabo
cabdabo

$ cat scratch
abo
absent
cabo
cabdabo
```

We can say that we want one or more instances of a regular expression
with `+`, zero or one instances with `?`, exactly *n* instances with
`{n}`, at least *m* but at most *n* instances with `{m,n}`, at most
*n* instances with `{,n}`, and at least *m* instances with `{m,}`.
These are all actually reducible to combinations of other regular
expression operators, so they're not technically needed, but they do
make things more concise.

The combination of repetition and the common behavior to look for a
substring which matches the regular expression often means that
regular expressions match more than you thought they would.  To solve
this, there are a pair of assertions for beginning and end of the line
(or *string*, regular expression implementations often allow you to
consider all lines of text as one big string instead of considering
each line individually): `^` and `$`, respectively.  In theoretical
regular expressions, this isn't necessary, as its assumed that the
whole string must match instead of just a substring which matches (the
theoretical behavior is equivalent to putting a `^` before and a `$`
after every regular expression, and you can get the practical behavior
in theory by putting a `.*` before and after the regular expression).

If there's a set of characters you'd like to match, you can use `()`
and `|s` to make a giant alternation.  Because this is so common, most
implementations of regular expressions support character classes,
delimited with `[` and `]`.  Inside a character class, most operators
don't have to be escaped.  You can complement a character class by
having the first character be a `^`, but this can cause bugs, especially
on Unicode-based systems, so try not to do it. `-` becomes an operator
in character classes, meaning a range of characters.  This works, most
of the time, but it can have problems with Unicode.

UNIX: Helping geeks cheat at crossword puzzles since 1971!

The "standard" regular expression facilities have been augmented with
some extra features which turn them into
"sorta-kinda-but-not-really-regular" expressions (because they allow
you to do things that theoretical regular expressions can't ever do).
The most common of these features is called a backreference.  This
allows you to "capture" the part of the string which matched a
parenthesized subexpression and require that some later substring in
the string match it.  This breaks a guarantee that theoretical regular
expressions make (basically that you can determine whether the string
matches without ever using more than some constant amount of memory
plus the length of the string: in order for backreferences to work,
you'd need to remember (on top of the memory for the whole string)
arbitrarily many substrings), but it's so useful that practical
regular expression systems often dispense with the theory.

This does cause confusion. When programmers are talking about regular
expressions, they might mean theoretical regular expressions or they
might mean any of several somewhat related dialects used by different
programming languages.

## Using regular expressions in Scala

Scala's regular expressions are implemented in a class
`scala.util.matching.Regex`.  The regular expression syntax is
documented in exhaustive detail in the documentation for the Java
class [java.util.regex.Pattern], though Scala uses some different
methods to interact with a regular expression.

The easiest way to construct a regular expression in Scala is to apply
the `r` method to a string.  It will interpret the string as a regular
expression and return the `Regex` object corresponding to the string.

```scala
import scala.util.matching.Regex
val Decimal = """(-)?(\d+)\.(\d*)?""".r
```

The triple quotes mean that the string is exactly as typed, without
escape characters interpreted as such.  The parentheses allow for
*captures* to be made.  We're interested in whether or not there was a
leading minus sign, the one or more digits before the decimal point,
and all digits after the decimal point (if there were any).

There are a few different match operators in the `Regex` class.
`findFirstIn` finds the first occurrence in the string (as an
`Option[String]`). `findAllIn` returns an `Iterator[String]` of the
occurrences in the string.  `findPrefixOf` finds an occurrence at the
start of the string (as an `Option[String]`).

```scala
scala> val input = "for -1.0 to 99 by 3"
scala> for (s <- Decimal findAllIn input)
println(s)
-1.0
99
3
scala> Decimal findFirstIn input
res3: Option[String] = Some(-1.0)
scala> Decimal findPrefixOf input
res4: Option[String] = None
```

A neat feature of Scala is extractors:

```scala
scala> val Decimal(sign, integerPart, decimalPart) = "-500.67"
sign: String = -
integerPart: String = 500
decimalPart: String = 67
```

## How it Works

There are three basic strategies known for matching regular
expressions.

Two involve constructing a finite state machine, which *moves* from
state to state as it moves through the string.  If the machine ends up
in a state declared *accepting*, the match succeeded.  If it consumes
the whole string and is stuck in a state not declared accepting, the
match fails.

Regular expressions in the theoretical sense can construct finite
state machines which will always *succeed* or *fail* in time linear in
the length of the string, with no guessing ever required.

However, regular expression matchers which allow "sorta kinda but not
really regular" expressions can't construct such finite state machines
for all expressions, and in practice will often, for any regular
expression, construct a different kind of finite state machine which
may require guessing between potentially exponentially many choices.
These matchers will, in the worst case, take time exponential in the
length of the string, especially if the match will fail.
Unfortunately, `java.util.regex`'s matcher, which is used by Scala, uses
this strategy.

The third strategy is one that was discovered in the early 1960s, and
was rarely used until the past few years.  It is based on the idea of
consuming the string character by character and simultaneously
manipulating the regular expression so that, if and only if it would
have matched the whole string, it will match the remainder of the
string.  This technique only works for regular expressions in the
theoretical sense.  A straightforward implementation exercises some
parts of Scala we've spent time on.

## Derivatives of Regular Expressions

The derivative of a language (remember that a language is just a set
of strings) with respect to some string *s* is the set of strings
which, when *s* is prepended to them, are in the original language.
In a more formal definition:

```
DsL = { w: sw is in L}
```

The derivative of the language { "", "foo", "frak", "foofoo",
"foofrak", "frakfoo", "frakfrak", … } with respect to "fo" is { "o",
"ofoo", "ofrak", "ofoofoo", "ofoofrak", "ofrakfoo", "ofrakfrak",
... }.  It should probably also be easy to see that we can chain the
derivatives:

```
DfoL = Do(DfL)
```

As it happens, we can prove (if you've passed 250, you probably should
be able to) that the derivative of a regular language is another
regular language.  From this it follows that we can apply the
derivative to a regular expression and get another regular expression.
We will need, however, the two special regular expressions that we
previously only said that were needed in theory: one that could never
match any string (which we'll call the *null regular expression*) and
one that only matches the empty string (which we'll call *epsilon*
(some prefer to call it *lambda*; there may have (but probably
haven't) been duels fought by theoretical computer scientists over
this question)).

The derivative of the *null regular expression* with respect to any
string has to be the *null regular expression*: if it was anything else,
it could match a string and that would imply that the *null regular
expression* could match a string, which it can't.  For simplicity, the
rest of the derivatives are only defined with respect to a character.

The derivative of *epsilon* with respect to any character also has to be
the *null regular expression*, by similar reasoning.

The derivative of a character with respect to a character is epsilon
if the two characters are the same and the null regular expression
otherwise.

To take the derivative of an alternation of two regular expressions,
you just create an alternation of their derivatives.  Note that when
creating an alternation, if one of the alternatives is the null
regular expression you can just ignore that alternative.  If the
alternative happens to be `.*`, then that will make all the other
alternatives irrelevant.  If all the alternatives are identical, then
you can dispense with the alternation.

To take the derivative of a concatenation of two regular expressions
(if there's more regular expressions being concatenated, you can add
layers of concatenation so every concatenation is of two regular
expressions) requires a bit of subtlety.  If the first regular
expression could match the empty string, you have to create an
alternation between two cases.  The first such case is that the first
regular expression matched the empty string, so it's just the
derivative of the second regular expression.  The second case is the
concatenation of the derivative of the first regular expression with
the (unchanged) second.  If the first regular expression couldn't
match the empty string, then we only need the second case.

To take the derivative of a starred regular expression, we concatenate
the derivative of the regular expression with the regular expression
starred.

The regular expressions which can match the empty string are epsilon,
any starred regular expression, the concatenation of any number of
regular expressions which can each match the empty string, and an
alternation with any regular expression which can match the empty
string.

An advantage of this approach is that it also allows for a
straightforward interpretation of additional regular expression
operators.  Two of the most useful such operators, which are only
rarely included in regular expression implementations because of their
historic difficulty of implementation, are intersection and
complement.  The intersection of two regular expressions matches if
and only if both regular expressions match.  The complement of a
regular expression matches if and only if the regular expression
doesn't match.

From the definition of the derivative, it's easy to prove that the
derivative of the intersection is the intersection of the derivatives
and that the intersection can only match the empty string if both of
its regular expressions can match the empty string.  Likewise, the
derivative of the complement is the complement of the derivative, and
it can only match the empty string if the regular expression being
complemented doesn't accept the empty string.

The algorithm for matching is thus to check if the regular expression
being matched against is the *null regular expression* (in which case,
the match automatically fails) and then if the string to match is
empty (in which case the match depends on whether the regular
expression matches the empty string).  In all other cases, the matcher
computes the derivative of the regular expression with respect to the
first character of the string and the result of the match is the
result of matching the rest of the string against the derivative.)
