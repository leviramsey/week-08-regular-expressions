# Regular Expression Derivative Notes

Written by Tim Richards, based on [Matt Might]'s notes on derivative-based regular expression matching.

[Matt Might]: http://matt.might.net

## Matching Rules

* **R1:** If string to match is empty and the current pattern matches empty, then the match succeeds.
* **R2:** If the string to match is non-empty, the new pattern is the derivative of the current pattern with respect to the first character of the current string, and the new string to match is the remainder of the current string.

## Empty Matching Rules

The following set of recursively defined rules test if a regular expression *accepts* the empty string. The empty string can only match empty (&epsilon;) and no string can match the null pattern (&empty;).

* **E1:** &delta;(&empty;) = &empty;
* **E2:** &delta;(&epsilon;) = &epsilon;
* **E3:** &delta;(c) = &empty;
* **E4:** &delta;(re<sub>1</sub> re<sub>2</sub>) =
  &delta;(re<sub>1</sub>) &delta;(re<sub>2</sub>)
* **E5:** &delta;(re<sub>1</sub>|re<sub>2</sub>) =
  &delta;(re<sub>1</sub>) | &delta;(re<sub>2</sub>)
* **E6:** &delta;(re*) = &epsilon;

## Derivative Rules

Let *D<sub>c</sub>(re)* denote the derivative of the regular expression *re* with respect to the character *c*; then the derivative can be defined recursively:

* **D1:** D<sub>c</sub>(&empty;) = &empty;
* **D2:** D<sub>c</sub>(&epsilon;) = &empty;
* **D3:** D<sub>c</sub>(c) = &epsilon;
* **D4:** D<sub>c</sub>(c&prime;) = &empty; if c &ne; c&prime;
* **D5:** D<sub>c</sub>(re<sub>1</sub> re<sub>2</sub>) =
    &delta;(re<sub>1</sub>)D<sub>c</sub>(re<sub>2</sub>) |
    D<sub>c</sub>(re<sub>1</sub>)re<sub>2</sub>
* **D6:** D<sub>c</sub>(re<sub>1</sub> | re<sub>2</sub>) =
  D<sub>c</sub>(re<sub>1</sub>) | D<sub>c</sub>(re<sub>2</sub>)
* **D7:** D<sub>c</sub>(re\*) = D<sub>c</sub>(re) re\*;

## Examples

Here are a few examples of how we might use these rules to match a given pattern *p* to a string *s*. You could imagine easily implementing this using Scala's pattern matching facilities!

### String Pattern

```
/foo/ match "foo"
1. Df(/foo/) = d(/f/) Df(/oo/) | Df(/f/) /oo/ (R2,D5)
             = NULL | EMPTY /oo/ (E3,D3)
             = /oo/
2. Do(/oo/)  = d(/o/) Do(/o/) | Do(/o/) /o/ (R2,D5)
             = NULL | EMPTY /o/ (E3,D3)
             = /o/
3. Do(/o/)   = EMPTY
4. EMPTY     = match success (R1)
```

```
/foo/ match "bar"
1. Df(/foo/) = d(/f/) Db(/oo/) | Db(/b/) /oo/ (R2,D5)
             = NULL | NULL /oo/ (E3,D4)
             = NULL
2. NULL      = match failed (R1)
```


### Alternation Pattern

```
/foo|bar/ match "bar"
1. Db(/foo|bar/) = Db(/foo/) | Db(/bar/) (D6)
                 = NULL | Db(/bar/)
                 = d(/b/) Db(/ar/) | Db(/b/) /ar/ (D5)
                 = NULL | EMPTY /ar/ (E3,D3)
                 = /ar/
2. Da(/ar/)      = d(/a/) Da(/r/) | Da(/a/) /r/ (D5)
                 = NULL | EMPTY /r/ (E3,D3)
                 = /r/
3. Dr(/r/)       = EMPTY
4. EMPTY         = match success (R1)
```

### Closure Pattern

```
/(a|b)*/ match "aaba"
1. Da(/(a|b)*/) = Da(/(a|b)/) /(a|b)*/ (D7)
                = (Da(/a/) | Da(/b/)) /(a|b)*/ (D6)
                = (EMPTY | NULL) /(a|b)*/ (D3,D2)
                = EMPTY /(a|b)*/
                = /(a|b)*/
2. Da(/(a|b)*/) = Da(/(a|b)/) /(a|b)*/ (D7)
                = (Da(/a/) | Da(/b/)) /(a|b)*/ (D6)
                = (EMPTY | NULL) /(a|b)*/ (D3,D2)
                = EMPTY /(a|b)*/
                = /(a|b)*/
3. Db(/(a|b)*/) = Db(/(a|b)/) /(a|b)*/ (D7)
                = (Db(/a/) | Db(/b/)) /(a|b)*/ (D6)
                = (NULL | EMPTY) /(a|b)*/ (D2,D3)
                = EMPTY /(a|b)*/
                = /(a|b)*/
1. Da(/(a|b)*/) = Da(/(a|b)/) /(a|b)*/ (D7)
                = (Da(/a/) | Da(/b/)) /(a|b)*/ (D6)
                = (EMPTY | NULL) /(a|b)*/ (D3,D2)
                = EMPTY /(a|b)*/
                = /(a|b)*/
                = match success (R1)
```
