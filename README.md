# Combinatron Specification

A formal specification of the abstract workings of the Combinatron machine.

## Instruction Set

The Combinatron's instruction set is composed of six words. The valid words are
the four combinators: B, C, K, W, and the nesting and returning words N and M
respectively.

The combinator words are relatively straightforward. The only information
encoded in a combinator word is which combinator should be applied. Description
of he actual reduction rules are left to another section.

The nesting words are a little more complicated. They each require a
pointer. The nest instruction points to the sentence that is nested, and the
return instruction points to the sentence that did the nesting. In most cases
the return instruction should not be used directly, and is only created during
normal operation of the Combinatron.

Words are grouped together into sentences that are at most three words in
length.

### Notation

The following notation is what will generally be used in this document.

Sentences are denoted by angle brackets, e.g. `<B C K>`.

Sentences shorter than three words are padded with zeros, e.g. `<B C 0>`

Nesting and unnesting words are written with the pointer juxtaposed with the
word and the sentence being pointed is labelled. E.g.

```
1. <B C K>
2. <N1 W 0>
3. <M2 W W>
```

## Cursors

The Combinatron has two major constructs. The first is the cursor. A cursor is
simply a view of a sentence. There are three cursors in the Combinatron referred
to as the top, middle, and bottom cursors. The cursors are used to provide
context and missing arguments when performing a reduction. Because there are a
maximum of three arguments to any word, only three cursors are necessary. The
primary operation on cursors is to rotate them either up or down. A rotation up
brings a new sentence in to the bottom cursor, if necessary, and writes the
value of the top cursor out to the sentence table. A rotation down brings a new
sentence in to the top cursor, if necessary, and writes the bottom cursor out to
the sentence table.

### Notation

Cursors are denoted by curly brackets, with the location of the sentence first
and the sentence itself second. E.g. `{1 <B B C>}`

## Sentence Index

The second major construct in the Combinatron is the sentence index, or
sometimes just the Index. It is at a high level an index of the sentences that
form the program. The sentence index supports the fetching of a sentence by its
index. It supports the addition of new sentences to the index. And it supports
the overwriting of sentences at an index. In the notation and formal semantics
the Index is 1-based. There is no 0-index.

### Notation

The sentence index is simply noted as an `S`. Fetches are indicated by ASCII
subscript syntax, e.g. `S_1`. Writes are indicated by a fetch with a bind
notation, e.g. `S_1 := <B C K>`.  Additions are indicated by a plus, e.g.
`S + <K W W>`.

## Semantics

Now that the definitions and major concepts are out of the way I'll outline the
actual operational semantics of how the machine works. To start a sentence is
indicated as the starting point and is loaded into the bottom cursor. To
determine what the current word is the machine examines the first word in the
bottom cursor. It then proceeds to examine or manipulate the other cursors as
necessary. To clarify and avoid obscuring concepts by specificity I'll refrain
from using actual words unless appropriate. Relevant words that can take on any
valid word will be denoted by a lowercase variable whose meaning should be
inferred from context. Irrelevant words are denoted by an underscore.

### Cursor Rotation

It was mentioned above that cursors could be rotated upwards or downwards. What
follows is a more formal description of that process.

Rotating the cursors down writes the bottom cursor to the Index while
simultaneously pulling in a new top cursor from the Index. The middle cursor is
copied to the bottom cursor, and the top cursor is copied to the middle cursor.

```
{2 <M1 _ _>}    {1      S_1}
{3 <M2 _ _>} -> {2 <M1 _ _>}, S_4 := <a _ _>
{4 < a _ _>}    {3 <M2 _ _>}
```

If a rotation down happens and the first word in the top cursor points to a
non-existent index, then the top cursor ends with a null cursor.

```
{1 <M0 _ _>}    {0 < 0 0 0>}
{2 <M1 _ _>} -> {1 <M0 _ _>}, S_3 := <a _ _>
{3 < a _ _>}    {2 <M2 _ _>}
```

Similarly, if the top cursor is a null cursor, things proceed intuitively.

```
{0 < 0 0 0>}    {0 < 0 0 0>}
{1 <M0 _ _>} -> {0 < 0 0 0>}, S_2 := <a _ _>
{2 < a _ _>}    {1 <M2 _ _>}
```

Rotations down can only happen if the top cursor contains a null cursor or the
first word of the top cursor is an unnesting word. Any other rotations down are
invalid.

Rotating the cursors up is the same process but in reverse. The top cursor is
written out to the Index, while a new bottom cursor is fetched from the Index.
The middle cursor is copied to the top cursor, and the bottom cursor is copied
to the middle cursor. Rotations up can only happen if the first word in the
bottom cursor is a nesting word.

```
{1 <M0 _ _>}    {2 <M1 _ _>}
{2 <M1 _ _>} -> {3 <N4 _ _>}, S_1 := <M0 _ _>
{3 <N4 _ _>}    {4      S_4}
```

### Nesting and Unnesting

Recall that the nesting word encodes a pointer to a sentence in the Index. The
basic idea of nesting is simply to rotate the cursors up by pulling the pointed
to sentence into the bottom cursor.

```
{1 < a _ _>}    {2 < b _ _>}
{2 < b _ _>} -> {3 <N4 _ _>}, S_1 := <a _ _>
{3 <N4 _ _>}    {4      S_4}
```

This is appropriate until a reduction forces the stack to be unwound and the
cursors to be rotated down, like so:

```
{1 < a _ _>}    {   ?     }
{2 < b _ _>} -> {1 <a _ _>}, S_2 := <N4 _ _>
{3 <N4 _ _>}    {2 <b _ _>}
```

As can be seen it is not immediately apparent where the new top cursor should
come from. This is the role of the M word, and for this reason when nesting the
N word responsible for the nest is always transformed into an M word. See:

```
{1 < a _ _>}    {2 < b _ _>}
{2 < b _ _>} -> {3 <M2 _ _>}, S_1 := <a _ _>
{3 <N4 _ _>}    {4      S_4}
```

The new M word always points to the location in the middle cursor. If the middle
cursor is empty, the new M word points to the 0th sentence in the Index, which
doesn't exist. Because this transformation from N to M always happens, the above
semantic block can be rephrased more accurately.

```
{1 <M0 _ _>}    {2 <M1 _ _>}
{2 <M1 _ _>} -> {3 <M2 _ _>}, S_1 := <M0 _ _>
{3 <N4 _ _>}    {4      S_4}
```

Unnesting is a similar operation, but in reverse. The cursors are rotated down,
and the sentence pointed to by the top cursor is fetched from the Index. If that
sentence doesn't exist then the top cursor remains empty. Unnesting only happens
when the bottom cursor has a single word in its sentence. That word is then
copied to the middle cursor, overwriting the M word, before the cursors are
rotated.

```
{2 <M1 _ _>}    {1      S_1}
{3 <M2 _ _>} -> {2 <M1 _ _>}, S_4 := <a _ _>
{4 < a 0 0>}    {3 < a _ _>}
```

```
{2 <M0 _ _>}    {0 < 0 0 0>}
{3 <M2 _ _>} -> {2 <M1 _ _>}, S_4 := <a _ _>
{4 < a 0 0>}    {3 < a _ _>}
```

### Combinator Reductions

With nesting and unnesting out of the way I can proceed to covering the
reduction rules for the individual combinators. These are a bit more involved
and almost always deal with multiple cursors. I've split them up for sections on
each combinator.

#### K Combinator

The K combinator has probably the simplest reduction rules. There are two
versions. The first version only spans a single cursor and has no effect on the
Index.

```
{1 <K a b>} -> {1 <a 0 0>}
```

The second version is a nested version and spans two cursors. In addition to
performing the reduction, the cursors are also rotated down.

```
{0 < 0 0 0>}    {0 <0 0 0>}
{1 <M0 b c>} -> {0 <0 0 0>}, S_2 := <K a 0>
{2 < K a 0>}    {1 <a c 0>}
```

#### W Combinator

The W combinator is the second simplest combinator. It has two reduction rules,
like the K combinator, but also modifies the Index in one rule. The first rule
only spans a single cursor:

```
{1 <W a b>} -> {1 <a b b>}
```

The second rule spans two cursors, and must write to the Index. It must do this
because it cannot overwrite the bottom cursor and cannot use the entire top
cursor. Therefore a nested sentence is created to allow room. The cursors are
also rotated down.

```
{0 < 0 0 0>}    {0 < 0 0 0>}
{1 <M0 b c>} -> {0 < 0 0 0>}, S_3 := <a b 0>, S_2 := <W a 0>
{2 < W a 0>}    {1 <N3 b c>}
```

#### C Combinator

The C combinator, because it takes three arguments, doesn't have a single cursor
rule. Instead it has two double cursor rules and a single triple cursor rule.
The first double cursor rule is applied when the C combinator has two arguments
already.

```
{0 < 0 0 0>}    {0 < 0 0 0>}
{1 <M0 c d>} -> {0 < 0 0 0>}, S_2 := <C a b>, S_3 := <a c 0>
{2 < C a b>}    {1 <N3 b d>}
```

When the C combinator only has a single argument it doesn't need an extra
sentence because all words in the middle cursor are used as arguments.

```
{0 < 0 0 0>}    {0 <0 0 0>}
{1 <M0 b c>} -> {0 <0 0 0>}, S_2 := <C a 0>
{2 < C a 0>}    {1 <a c b>}
```

Finally, when each cursor has a single argument of the C combinator. This works
similarly to when the C combinator has two arguments but over a larger span. The
middle cursor has its M word rewritten to an N word, and the cursors are rotated
down twice. This reduction rule is broken up into steps to make more clear what
is happening. As an implementation it happens all at once. The first step is to
perform the reduction.

```
{1 <M0 c d>}    {1 <N4 b d>}
{2 <M1 b 0>} -> {2 <N3 b 0>}, S_4 := <a c 0>
{3 < C a 0>}    {3 < C a 0>}
```

The next step is to rotate the cursors down twice.

```
{1 <N4 b d>}    {0 < 0 0 0>}
{2 <N3 b 0>} -> {0 < 0 0 0>}, S_3 := <C a 0>, S_2 := <N3 b 0>
{3 < C a 0>}    {1 <N4 b d>}
```

#### B Combinator

The B combinator has a similar complexity to the C combinator, but doesn't
require the manual creation of a space saving sentence as sentence creation is
already part of the reduction rule.

As before, when the B combinator has two arguments in its cursor.

```
{0 < 0 0 0>}    {0 <0  0 0>}
{1 <M0 c d>} -> {0 <0  0 0>}, S_2 := <B a b>, S_3 := <b c 0>
{2 < B a b>}    {1 <a N3 d>}
```

And when it has a single argument in its cursor.

```
{0 < 0 0 0>}    {0 <0  0 0>}
{1 <M0 b c>} -> {0 <0  0 0>}, S_2 := <B a 0>, S_3 := <b c 0>
{2 < B a 0>}    {1 <a N3 0>}
```

And when each cursor holds an argument. This is again broken up into two steps
to aid clarity. First the reduction:

```
{1 <M0 c d>}    {1 < a N4 d>}
{2 <M1 b 0>} -> {2 <N3  b 0>}, S_4 := <b c 0>
{3 < B a 0>}    {3 < B  a 0>}
```

And the double rotation down.

```
{1 < a N4 d>}    {0 <0  0 0>}
{2 <N3  b 0>} -> {0 <0  0 0>}, S_3 := <B a 0>, S_2 := <N3 b 0>
{3 < B  a 0>}    {1 <a N4 d>}
```

### Halting

Computation halts if any of the above reduction forms aren't met.

## Todos

### Major Additions

There are still a couple additions that need to be worked in to the formal
semantics. Primarily these include side effecting operations and the operation
of a garbage collector. I already have some work done on two side effectful
words but that work is based on an older version of the semantics and needs some
tweaks to be brought up to date. Regarding garbage collection, I'm not as sure
I'm on solid ground. The Index is the only component that needs garbage
collected. I have considered a variation of a mark and compact garbage collector
that also uses reference counting to mark dead sentences in the Index, but I
don't have anything formally defined yet.

### Minor Refinements

Another refinement that is worth exploring deals with the creation of the
space-saving sentences. Right now they are created as partial sentences, but
they could be created as full sentences. This might be better but the tradeoffs
should be explored.

## Contributing

Corrections are welcome in the form of pull requests. Clarifications or
questions are welcome in the form of issues. In that manner questions will be
available to be answered by newcomers.
