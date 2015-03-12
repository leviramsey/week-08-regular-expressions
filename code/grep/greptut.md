# Regular Expression Tutorial

This is a short tutorial on using regular expressions with
grep. Before proceeding you should
[install a dictionary of words][dict].

[dict]: install.md

## Matching Words

The simplest type of pattern is one in which we look for specific
words:

```bash
$ grep 'Zamboni' /usr/share/dict/words
Zamboni
```

`grep` will match the string `Zamboni` in the `words` file and display
the word it found with that pattern. You may also want to find all
words that have a `Z` in them:

```bash
$ grep 'Z' /usr/share/dict/words
...
```

This will display 140 words that have the letter `Z` in them. You can
find out how may words matched using the `wc` command:

```bash
$ grep 'Z' /usr/share/dict/words | wc -l
140
```

If we were to look for all words containing a lowercase `z` we would
do the following:

```bash
$ grep 'z' /usr/share/dict/words | wc -l
2889
```

As you can see, the result is larger as we have more words with a
lowercase `z`.

## Beginning of Line (^)

You can also use *meta-characters* which will allow you to match other
patterns. In particular, we may want to find all words that start with
an upper case `Z`. To do this we use the `^` meta-character:

```bash
$ grep '^Z' /usr/share/dict/words | wc -l
138
```

Or those that begin with a lowercase `z`:

```bash
$ grep '^z' /usr/share/dict/words | wc -l
2889
```

## End of Line ($)

Another interesting pattern are those where characters match the end
of a string. To do this we use the `$` meta-character:

```bash
$ grep 'Z$' /usr/share/dict/words | wc -l
1
```

There is only one match. To find out which line this occurs on you can
use the `-nr` command flag:

```bash
$ grep -nr 'Z$' /usr/share/dict/words
16346:Z
```

As you can see there is only one word that ends with a `Z` which is
`Z` - wow, didn't realize this was a word! So, how many words end with
a lowercase `z`?

If we wanted to know how many empty lines there are in file we can use
both `^` and `$` to achieve that:

```bash
$ grep -nr '^$' /usr/share/dict/words
```
Apparently, in this file there are no empty lines! However, we could
easily find the number of empty lines in this file. Give it a try!

If we wanted to find a word that has exactly the characters of the
word that begin and end on a line we would do this:

```bash
$ grep -nr '^in$' /usr/share/dict/words
52931:in
```

In this case, we have only one. What pattern would you use to identify
all words that end in `in`? How about those that begin with `in`?

What if we wanted to match any character?

## Single Character (.)

The meta-character `.` matches any character except the end of line
character. So, what if we wanted to match all words that had a `t`
followed by any character followed by a `m`?

```bash
$ grep 't.m' /usr/share/dict/words | wc -l
1011
```

That is quiet a few! How about those that end in a `t` followed by
any character followed by a `m`?

```bash
$ grep 't.m$' /usr/share/dict/words | wc -l
46
```

How about the beginning of a line? Apparently there are more of those!

### Exercises

1. How would you match words of exactly 4 characters in length? (*we found 3352*)
1. How would you match words that are 5 characters in length that
   begin with a `t` and end with a `m`? (*we found 2*)
1. How would you match words that start with a `w` followed by any
   character followed by an `a` followed by any character followed by
   a `p`? (*we found 7*)

## Zero or More Occurrence (*)

What if want to match more than a single character? For that we can
use the meta-character `*` which matches 0 or more characters. So, if
we wanted to match all words that begin with an `m`, have 0 or more
`a`s, followed by a `p` we would do this:

```bash
$ grep '^ma*p' /usr/share/dict/words | wc -l
10
```

How about those that end in a `p`:

```bash
$ grep '^ma*p$' /usr/share/dict/words | wc -l
1
```

Guess what the word is?

### Exercises

1. Find all words that begin with a `q`, followed by 0 or more
   characters, followed by an `l` as the last character. (*we found 9*)
1. Find all words that begin with a `q`, followed by 0 or more
   characters, followed by 1 or more `z`s, followed by 0 or more
   characters, followed by 1 or more `l`s, followed by a `y` as the
   last character. (*we found 1*)

## One or More Occurrence (\+)

The previous exercises showed you how to represent 1 or more using
`*`. There happens to be a shorthand to this using the `\+`
meta-character:

```bash
$ grep 'hel\+o$' /usr/share/dict/words
Othello
hello
```

This is equivalent to:

```bash
$ grep 'hell*o$' /usr/share/dict/words
Othello
hello
```

### Exercises

1. Find all words that begin with a `q`, followed by 0 or more
   characters, followed by an `l` as the last character. (*we found 9*)
1. Find all words begin with a `q`, followed by 0 or more characters,
   followed by 1 or more `z`s, followed by 0 or more characters,
   followed by 1 or more `l`s, followed by a `y` as the last
   character. (*we found 1*)

## Zero or One Occurrence (\?)

It is often useful to be able to find 0 or 1 occurrence of a
character. We can do this with the `\?` meta-character:

```bash
$ grep 'c\?ha' /usr/share/dict/words
...
```

### Exercises

1. Find all words that begin with a `w`, followed by an optional
   character, followed by 1 or more `e`s, followed by an `l`, followed
   by 0 or more characters, followed by `ed` at the end of the word.
   (*we found 11*)

## Escaping Special Characters (\)

What if we want to find out if a particular version of a package was
installed on a debian-based linux system (Ubuntu) on March 03? When
you install packages on Ubuntu you use the `apt-get` command which in
turn uses the `dpkg` command. This command logs all of its
installations to the file `/var/log/dpkg.log`. The format of each line
in this file is:

```
YYYY-MM-DD HH:MM:SS COMMANDS PACKAGE VERSION
```

Where the VERSION is split into three numbers X.Y.Z-R (R is the
revision). As you can see, we need to be able to match `.`. To do this
we need to *escape* the `.` so it is matched as `.`:


```bash
$ grep '3\.3\.02-6' /var/log/dpkg.log
...
```

If we want to refine this to only those packages installed on March 10
between 6PM and 6:59PM we could do this:

```bash
$ grep '....-03-10 18:..:...*install.*3\.3\.02-6' /var/log/dpkg.log
...
```

## Character Classes ([0-9])

The character class is nothing but a list of characters mentioned with
in square brackets that match only one out of several characters:

```bash
$ grep '^[abc].*[abc]$' /usr/share/dict/words | wc -l
333
```

We can also specify a range:

```bash
$ grep '^[a-c].*[a-c]$' /usr/share/dict/words | wc -l
333
```

We can also use the complement `^` meta-character inside of the
character class to match anything but the characters specified:

```bash
$ grep '^[^b-y].*[^b-y]$' /usr/share/dict/words | wc -l
1334
```

### Exercises

1. Find all words that do not begin or end with a vowel.
1. Find all words that begin with a `j` or a `k`, followed by a vowel,
   followed by 1 or more characters, followed by not a vowel at the
   end of the word. (*we found 3977*)

## OR Operation |

Regular expressions allow us to specify patterns that match
alternatives. To do this we use the `|` operator. Imagine we want to
find all lines in a file that do not match a line comment for a
variety of languages (see `comments.txt`), we could do something like
this:

```bash
$ grep -v "^#\|^'\|^\/\/" comments.txt
This file shows the comment character in various programming/scripting languages
If the Line starts with single hash symbol,
then its a comment in Perl and shell scripting.
The line should start with a single quote to comment in VB scripting.
Double slashes in the beginning of the line for single line comment in C.
```

The `-v` flag tells `grep` to output the lines that did not match
rather than those that did. You can drop the `-v` to see which ones
matched.

## Character Class Expression

It is also possible to express common character classes with these
shorthands:

* [:digit:] *Only the digits 0 to 9*
* [:alnum:] *Any alphanumeric character 0 to 9 OR A to Z OR a to z*
* [:alpha:] *Any alpha character A to Z OR a to z*
* [:blank:] *Space and TAB characters only*

There are several others that you can look up in `grep`'s man page.


## M to N, M, and M or more Occurrences

A regular expression followed by {m,n} indicates that the preceding
item is matched at least m times, but not more than n times.

```bash
$ grep "^[0-9]\{1,5\}$" number.txt
```

A regular expression followed by {m} indicates that the preceding item
is matched exactly m occurrences.

```bash
$ grep "^[0-9]\{5\}$" number.txt
```

A regular expression followed by {m,} matches m or more occurrences of
the preceding item.

```bash
$ grep "^[0-9]\{5,\}$" number.txt
```

## Grouping

It is often necessary that you would like to group regular expressions
within a single regular expression. For this you use the ().

```bash
$ grep "^\(12\|19\)[[:digit:]]\{4,\}" number.txt
123456
19816282
```
