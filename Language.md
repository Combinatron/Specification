# Combinatron Programming Language

The language to program the Combinatron is really just a slightly modified form
of the BCKW calculus notation.

## Tokens

The language is whitespace insensitive and case insensitive. The few valid
tokens are as follows:

* Basic combinators: `b` `c` `k` `w`
* Side effect combinators, either `g` or `p` followed immediately (no
  whitespace) by a number.
* Parentheses for grouping.

## Grammar

The language grammar is similarly simple. Here it is in a recursive grammar
notation.

```
Expr : empty
     | Basic Expr
     | Nested Expr

Basic : basic_combinator
      | side_effect_combinator

Nested : '(' Expr ')'
```

A valid expression is either empty, a Basic expression followed by another valid
expression, or a Nested expression followed by another valid expression. A Basic
expression is either a basic combinator token or a side effect token. A Nested
expression is any valid expression grouped by parentheses.
