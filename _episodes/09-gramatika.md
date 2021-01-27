---
title: "Building features based on grannar"
teaching: 60
exercises: 30
questions:
-  "What are some useful text corpora and lexical resources, and how can
   we access them with Python?"
-  "Which Python constructs are most helpful for this work?"
-  "How do we avoid repeating ourselves when writing Python code?"
objectives:
- "Describe the reverse instructional design process."
- "Describe the purpose and implementation of formative assessments."
keypoints:
- "Lessons are design in four stages: conceptual, summative, formative, and connective."
---

## Building features based on grannar

Natural languages have an extensive range of grammatical constructions
which are hard to handle with the simple methods described in
chap-parse\_. In order to gain more flexibility, we change our treatment
of grammatical categories like `S`, `NP` and `V`. In place of atomic
labels, we decompose them into structures like dictionaries, where
features can take on a range of values.

The goal of this chapter is to answer the following questions:

1.  How can we extend the framework of context free grammars with
    features so as to gain more fine-grained control over grammatical
    categories and productions?
2.  What are the main formal properties of feature structures and how do
    we use them computationally?
3.  What kinds of linguistic patterns and grammatical constructions can
    we now capture with feature based grammars?

Along the way, we will cover more topics in English syntax, including
phenomena such as agreement, subcategorization, and unbounded dependency
constructions.

Grammatical Features
--------------------

In chap-data-intensive\_, we described how to build classifiers that
rely on detecting features of text. Such features may be quite simple,
such as extracting the last letter of a word, or more complex, such as a
part-of-speech tag which has itself been predicted by the classifier. In
this chapter, we will investigate the role of features in building
rule-based grammars. In contrast to feature extractors, which record
features that have been automatically detected, we are now going to
*declare* the features of words and phrases. We start off with a very
simple example, using dictionaries to store features and their values.

> &gt;&gt;&gt; kim = {'CAT': 'NP', 'ORTH': 'Kim', 'REF': 'k'}
> &gt;&gt;&gt; chase = {'CAT': 'V', 'ORTH': 'chased', 'REL': 'chase'}

The objects `kim` and `chase` both have a couple of shared features,
`CAT` (grammatical category) and `ORTH` (orthography, i.e., spelling).
In addition, each has a more semantically-oriented feature: `kim['REF']`
is intended to give the referent of `kim`, while `chase['REL']` gives
the relation expressed by `chase`. In the context of rule-based
grammars, such pairings of features and values are known as
feature structures, and we will shortly see alternative notations for
them.

Feature structures contain various kinds of information about
grammatical entities. The information need not be exhaustive, and we
might want to add further properties. For example, in the case of a
verb, it is often useful to know what "semantic role" is played by the
arguments of the verb. In the case of chase, the subject plays the role
of "agent", while the object has the role of "patient". Let's add this
information, using `'sbj'` and `'obj'` as placeholders which will get
filled once the verb combines with its grammatical arguments:

> &gt;&gt;&gt; chase\['AGT'\] = 'sbj' &gt;&gt;&gt; chase\['PAT'\] =
> 'obj'

If we now process a sentence Kim chased Lee, we want to "bind" the
verb's agent role to the subject and the patient role to the object. We
do this by linking to the `REF` feature of the relevant `NP`. In the
following example, we make the simple-minded assumption that the `NP`s
immediately to the left and right of the verb are the subject and object
respectively. We also add a feature structure for Lee to complete the
example.

> &gt;&gt;&gt; sent = "Kim chased Lee" &gt;&gt;&gt; tokens =
> sent.split() &gt;&gt;&gt; lee = {'CAT': 'NP', 'ORTH': 'Lee', 'REF':
> 'l'} &gt;&gt;&gt; def lex2fs(word): ... for fs in \[kim, lee, chase\]:
> ... if fs\['ORTH'\] == word: ... return fs &gt;&gt;&gt; subj, verb,
> obj = lex2fs(tokens\[0\]), lex2fs(tokens\[1\]), lex2fs(tokens\[2\])
> &gt;&gt;&gt; verb\['AGT'\] = subj\['REF'\] &gt;&gt;&gt; verb\['PAT'\]
> = obj\['REF'\] &gt;&gt;&gt; for k in \['ORTH', 'REL', 'AGT', 'PAT'\]:
> ... print("%-5s =&gt; %s" % (k, verb\[k\])) ORTH =&gt; chased REL
> =&gt; chase AGT =&gt; k PAT =&gt; l

The same approach could be adopted for a different verb, say surprise,
though in this case, the subject would play the role of "source" (`SRC`)
and the object, the role of "experiencer" (`EXP`):

> &gt;&gt;&gt; surprise = {'CAT': 'V', 'ORTH': 'surprised', 'REL':
> 'surprise', ... 'SRC': 'sbj', 'EXP': 'obj'}

Feature structures are pretty powerful, but the way in which we have
manipulated them is extremely *ad hoc*. Our next task in this chapter is
to show how the framework of context free grammar and parsing can be
expanded to accommodate feature structures, so that we can build
analyses like this in a more generic and principled way. We will start
off by looking at the phenomenon of syntactic agreement; we will show
how agreement constraints can be expressed elegantly using features, and
illustrate their use in a simple grammar.

Since feature structures are a general data structure for representing
information of any kind, we will briefly look at them from a more formal
point of view, and illustrate the support for feature structures offered
by |NLTK|. In the final part of the chapter, we demonstrate that the
additional expressiveness of features opens up a wide spectrum of
possibilities for describing sophisticated aspects of linguistic
structure.

### Syntactic Agreement

The following examples show pairs of word sequences, the first of which
is grammatical and the second not. (We use an asterisk at the start of a
word sequence to signal that it is ungrammatical.)

In English, nouns are usually marked as being singular or plural. The
form of the demonstrative also varies: this (singular) and these
(plural). Examples
[ex-thisdog](..%20ex::..%20ex::this%20dog..%20ex::\*these%20dog) and
[ex-thesedogs](..%20ex::..%20ex::these%20dogs..%20ex::\*this%20dogs)
show that there are constraints on the use of demonstratives and nouns
within a noun phrase: either both are singular or both are plural. A
similar constraint holds between subjects and predicates:

Here we can see that morphological properties of the verb co-vary with
syntactic properties of the subject noun phrase. This co-variance is
called agreement. If we look further at verb agreement in English, we
will see that present tense verbs typically have two inflected forms:
one for third person singular, and another for every other combination
of person and number, as shown in tab-agreement-paradigm\_.

We can make the role of morphological properties a bit more explicit as
illustrated in
[ex-runs](..%20gloss::the%20%7C%20dog%20%20%20%20%20%20%20%7Crun.s%7C%20dog.3.SG%20%20%7Crun.3.SG)
and
[ex-run](..%20gloss::the%20%7C%20dog-s%20%20%20%20%20%7Crun%7C%20dog.3.PL%20%20%7Crun.3.PL).
These representations indicate that the verb agrees with its subject in
person and number. (We use "3" as an abbreviation for 3rd person, "SG"
for singular and "PL" for plural.)

Let's see what happens when we encode these agreement constraints in a
context-free grammar. We will begin with the simple CFG in
[ex-agcfg0](..%20ex::::).

> S -&gt; NP VP NP -&gt; Det N VP -&gt; V
>
> Det -&gt; 'this' N -&gt; 'dog' V -&gt; 'runs'

Grammar [ex-agcfg0](..%20ex::::) allows us to generate the sentence
this dog runs; however, what we really want to do is also generate
these dogs
run while blocking unwanted sequences like \*this dogs run and
\*these dog runs. The most straightforward approach is to add new
non-terminals and productions to the grammar:

> S -&gt; NP\_SG VP\_SG S -&gt; NP\_PL VP\_PL NP\_SG -&gt; Det\_SG N\_SG
> NP\_PL -&gt; Det\_PL N\_PL VP\_SG -&gt; V\_SG VP\_PL -&gt; V\_PL
>
> Det\_SG -&gt; 'this' Det\_PL -&gt; 'these' N\_SG -&gt; 'dog' N\_PL
> -&gt; 'dogs' V\_SG -&gt; 'runs' V\_PL -&gt; 'run'

In place of a single production expanding `S`, we now have two
productions, one covering the sentences involving singular subject `NP`s
and `VP`s, the other covering sentences with plural subject `NP`s and
`VP`s. In fact, every production in [ex-agcfg0](..%20ex::::) has two
counterparts in [ex-agcfg1](..%20ex::::). With a small grammar, this is
not really such a problem, although it is aesthetically unappealing.
However, with a larger grammar that covers a reasonable subset of
English constructions, the prospect of doubling the grammar size is very
unattractive. Let's suppose now that we used the same approach to deal
with first, second and third person agreement, for both singular and
plural. This would lead to the original grammar being multiplied by a
factor of 6, which we definitely want to avoid. Can we do better than
this? In the next section we will show that capturing number and person
agreement need not come at the cost of "blowing up" the number of
productions.

### Using Attributes and Constraints

We spoke informally of linguistic categories having *properties*; for
example, that a noun has the property of being plural. Let's make this
explicit:

> N\[NUM=pl\]

In [ex-num0](..%20ex::::), we have introduced some new notation which
says that the category `N` has a (grammatical) feature called `NUM`
(short for 'number') and that the value of this feature is `pl` (short
for 'plural'). We can add similar annotations to other categories, and
use them in lexical entries:

> Det\[NUM=sg\] -&gt; 'this' Det\[NUM=pl\] -&gt; 'these'
>
> N\[NUM=sg\] -&gt; 'dog' N\[NUM=pl\] -&gt; 'dogs' V\[NUM=sg\] -&gt;
> 'runs' V\[NUM=pl\] -&gt; 'run'

Does this help at all? So far, it looks just like a slightly more
verbose alternative to what was specified in [ex-agcfg1](..%20ex::::).
Things become more interesting when we allow *variables* over feature
values, and use these to state constraints:

> S -&gt; NP\[NUM=?n\] VP\[NUM=?n\] NP\[NUM=?n\] -&gt; Det\[NUM=?n\]
> N\[NUM=?n\] VP\[NUM=?n\] -&gt; V\[NUM=?n\]

We are using `?n` as a variable over values of `NUM`; it can be
instantiated either to `sg` or `pl`, within a given production. We can
read the first production as saying that whatever value `NP` takes for
the feature `NUM`, `VP` must take the same value.

In order to understand how these feature constraints work, it's helpful
to think about how one would go about building a tree. Lexical
productions will admit the following local trees (trees of depth one):

Now `S -> NP[NUM=?n] VP[NUM=?n]` says that whatever the `NUM` values of
`N` and `Det` are, they have to be the same. Consequently,
`NP[NUM=?n] -> Det[NUM=?n] N[NUM=?n]` will permit ex-this\_ and ex-dog\_
to be combined into an `NP` as shown in ex-good1\_ and it will also
allow ex-these\_ and ex-dogs\_ to be combined, as in ex-good2\_. By
contrast, ex-bad1\_ and ex-bad2\_ are prohibited because the roots of
their subtrees differ in their values for the `NUM` feature; this
incompatibility of values is indicated informally with a *FAIL* value at
the top node.

Production `VP[NUM=?n] -> V[NUM=?n]` says that the `NUM` value of the
head verb has to be the same as the `NUM` value of the `VP` parent.
Combined with the production for expanding `S`, we derive the
consequence that if the `NUM` value of the subject head noun is `pl`,
then so is the `NUM` value of the `VP`'s head verb.

Grammar [ex-agcfg2](..%20ex::::) illustrated lexical productions for
determiners like this and these which require a singular or plural head
noun respectively. However, other determiners in English are not choosy
about the grammatical number of the noun they combine with. One way of
describing this would be to add two lexical entries to the grammar, one
each for the singular and plural versions of determiner such as the :

    Det[NUM=sg] -> 'the' | 'some' | 'any'
    Det[NUM=pl] -> 'the' | 'some' | 'any'

However, a more elegant solution is to leave the `NUM` value
underspecified and letting it agree in number with whatever noun it
combines with. Assigning a variable value to `NUM` is one way of
achieving this result:

    Det[NUM=?n] -> 'the' | 'some' | 'any'

But in fact we can be even more economical, and just omit any
specification for `NUM` in such productions. We only need to explicitly
enter a variable value when this constrains another value elsewhere in
the same production.

The grammar in code-feat0cfg\_ illustrates most of the ideas we have
introduced so far in this chapter, plus a couple of new ones.

Notice that a syntactic category can have more than one feature; for
example, `V[TENSE=pres, NUM=pl]`. In general, we can add as many
features as we like.

A final detail about code-feat0cfg\_ is the statement `%start S`. This
"directive" tells the parser to take `S` as the start symbol for the
grammar.

In general, when we are trying to develop even a very small grammar, it
is convenient to put the productions in a file where they can be edited,
tested and revised. We have saved code-feat0cfg\_ as a file named
`'feat0.fcfg'` in the NLTK data distribution. You can make your own copy
of this for further experimentation using `nltk.data.load()`.

code-featurecharttrace\_ illustrates the operation of a chart parser
with a feature-based grammar. After tokenizing the input, we import the
`load_parser` function load\_parser1\_ which takes a grammar filename as
input and returns a chart parser `cp` load\_parser2\_. Calling the
parser's `parse()` method will iterate over the resulting parse trees;
`trees` will be empty if the grammar fails to parse the input and will
contain one or more parse trees, depending on whether the input is
syntactically ambiguous or not.

The details of the parsing procedure are not that important for present
purposes. However, there is an implementation issue which bears on our
earlier discussion of grammar size. One possible approach to parsing
productions containing feature constraints is to compile out all
admissible values of the features in question so that we end up with a
large, fully specified CFG along the lines of [ex-agcfg1](..%20ex::::).
By contrast, the parser process illustrated above works directly with
the underspecified productions given by the grammar. Feature values
"flow upwards" from lexical entries, and variable values are then
associated with those values, via bindings (i.e., dictionaries) such as
`{?n: 'sg', ?t: 'pres'}`. As the parser assembles information about the
nodes of the tree it is building, these variable bindings are used to
instantiate values in these nodes; thus the underspecified
`VP[NUM=?n, TENSE=?t] -> TV[NUM=?n, TENSE=?t] NP[]` becomes instantiated
as `VP[NUM='sg', TENSE='pres'] -> TV[NUM='sg', TENSE='pres'] NP[]` by
looking up the values of `?n` and `?t` in the bindings.

Finally, we can inspect the resulting parse trees (in this case, a
single one).

### Terminology

So far, we have only seen feature values like `sg` and `pl`. These
simple values are usually called atomic |mdash| that is, they can't be
decomposed into subparts. A special case of atomic values are boolean
values, that is, values that just specify whether a property is true or
false. For example, we might want to distinguish auxiliary verbs such as
can, may, will and do with the boolean feature `AUX`. For example, the
production `V[TENSE=pres, AUX=+] -> 'can'` means that can receives the
value `pres` for `TENSE` and `+` or `true` for `AUX`. There is a widely
adopted convention which abbreviates the representation of boolean
features `f`; instead of `AUX=+` or `AUX=-`, we use `+AUX` and `-AUX`
respectively. These are just abbreviations, however, and the parser
interprets them as though `+` and `-` are like any other atomic value.
[ex-lex](..%20ex::::) shows some representative productions:

> V\[TENSE=pres, +AUX\] -&gt; 'can' V\[TENSE=pres, +AUX\] -&gt; 'may'
>
> V\[TENSE=pres, -AUX\] -&gt; 'walks' V\[TENSE=pres, -AUX\] -&gt;
> 'likes'

We have spoken of attaching "feature annotations" to syntactic
categories. A more radical approach represents the whole category
|mdash| that is, the non-terminal symbol plus the annotation |mdash| as
a bundle of features. For example, `N[NUM=sg]` contains part of speech
information which can be represented as `POS=N`. An alternative notation
for this category therefore is `[POS=N, NUM=sg]`.

In addition to atomic-valued features, features may take values that are
themselves feature structures. For example, we can group together
agreement features (e.g., person, number and gender) as a distinguished
part of a category, grouped together as the value of `AGR`. In this
case, we say that `AGR` has a complex value. [ex-agr0](..%20ex::::)
depicts the structure, in a format known as an attribute value matrix
(AVM).

> \[POS = N \] \[ \] \[AGR = \[PER = 3 \]\] \[ \[NUM = pl \]\] \[ \[GND
> = fem \]\]

> Rendering a Feature Structure as an Attribute Value Matrix

In passing, we should point out that there are alternative approaches
for displaying AVMs;
[fig-avm1](..%20figure::%20../images/avm1.png:scale:%2060) shows an
example. Athough feature structures rendered in the style of
[ex-agr0](..%20ex::::) are less visually pleasing, we will stick with
this format, since it corresponds to the output we will be getting from
|NLTK|.

On the topic of representation, we also note that feature structures,
like dictionaries, assign no particular significance to the *order* of
features. So [ex-agr0](..%20ex::::) is equivalent to:

Once we have the possibility of using features like `AGR`, we can
refactor a grammar like code-feat0cfg\_ so that agreement features are
bundled together. A tiny grammar illustrating this idea is shown in
[ex-agr2](..%20ex::::).

> S -&gt; NP\[AGR=?n\] VP\[AGR=?n\] NP\[AGR=?n\] -&gt; PropN\[AGR=?n\]
> VP\[TENSE=?t, AGR=?n\] -&gt; Cop\[TENSE=?t, AGR=?n\] Adj
>
> Cop\[TENSE=pres, AGR=\[NUM=sg, PER=3\]\] -&gt; 'is'
> PropN\[AGR=\[NUM=sg, PER=3\]\] -&gt; 'Kim' Adj -&gt; 'happy'

Processing Feature Structures
-----------------------------

In this section, we will show how feature structures can be constructed
and manipulated in |NLTK|. We will also discuss the fundamental
operation of unification, which allows us to combine the information
contained in two different feature structures.

Feature structures in |NLTK| are declared with the `FeatStruct()`
constructor. Atomic feature values can be strings or integers.

> &gt;&gt;&gt; fs1 = nltk.FeatStruct(TENSE='past', NUM='sg')
> &gt;&gt;&gt; print(fs1) \[ NUM = 'sg' \] \[ TENSE = 'past' \]

A feature structure is actually just a kind of dictionary, and so we
access its values by indexing in the usual way. We can use our familiar
syntax to *assign* values to features:

> &gt;&gt;&gt; fs1 = nltk.FeatStruct(PER=3, NUM='pl', GND='fem')
> &gt;&gt;&gt; print(fs1\['GND'\]) fem &gt;&gt;&gt; fs1\['CASE'\] =
> 'acc'

We can also define feature structures that have complex values, as
discussed earlier.

> &gt;&gt;&gt; fs2 = nltk.FeatStruct(POS='N', AGR=fs1) &gt;&gt;&gt;
> print(fs2) \[ \[ CASE = 'acc' \] \] \[ AGR = \[ GND = 'fem' \] \] \[
> \[ NUM = 'pl' \] \] \[ \[ PER = 3 \] \] \[ \] \[ POS = 'N' \]
> &gt;&gt;&gt; print(fs2\['AGR'\]) \[ CASE = 'acc' \] \[ GND = 'fem' \]
> \[ NUM = 'pl' \] \[ PER = 3 \] &gt;&gt;&gt;
> print(fs2\['AGR'\]\['PER'\]) 3

An alternative method of specifying feature structures is to use a
bracketed string consisting of feature-value pairs in the format
`feature=value`, where values may themselves be feature structures:

> &gt;&gt;&gt; print(nltk.FeatStruct("\[POS='N', AGR=\[PER=3, NUM='pl',
> GND='fem'\]\]")) \[ \[ GND = 'fem' \] \] \[ AGR = \[ NUM = 'pl' \] \]
> \[ \[ PER = 3 \] \] \[ \] \[ POS = 'N' \]

Feature structures are not inherently tied to linguistic objects; they
are general purpose structures for representing knowledge. For example,
we could encode information about a person in a feature structure:

> &gt;&gt;&gt; print(nltk.FeatStruct(NAME='Lee', TELNO='01 27 86 42 96',
> AGE=33)) \[ AGE = 33 \] \[ NAME = 'Lee' \] \[ TELNO = '01 27 86 42 96'
> \]

In the next couple of pages, we are going to use examples like this to
explore standard operations over feature structures. This will briefly
divert us from processing natural language, but we need to lay the
groundwork before we can get back to talking about grammars. Hang on
tight!

It is often helpful to view feature structures as graphs; more
specifically, directed acyclic graphs (DAGs).
[ex-dag01](..%20ex::..%20image::%20../images/dag01.png:scale:%2040) is
equivalent to the above AVM.

The feature names appear as labels on the directed arcs, and feature
values appear as labels on the nodes that are pointed to by the arcs.

Just as before, feature values can be complex:

When we look at such graphs, it is natural to think in terms of paths
through the graph. A feature path is a sequence of arcs that can be
followed from the root node. We will represent paths as tuples. Thus,
`('ADDRESS', 'STREET')` is a feature path whose value in
[ex-dag02](..%20ex::..%20image::%20../images/dag02.png:scale:%2040) is
the node labeled `'rue Pascal'`.

Now let's consider a situation where Lee has a spouse named *Kim*, and
Kim's address is the same as Lee's. We might represent this as
[ex-dag04](..%20ex::..%20image::%20../images/dag04.png:scale:%2040).

However, rather than repeating the address information in the feature
structure, we can "share" the same sub-graph between different arcs:

In other words, the value of the path `('ADDRESS')` in
[ex-dag03](..%20ex::..%20image::%20../images/dag03.png:scale:%2040) is
identical to the value of the path `('SPOUSE', 'ADDRESS')`. DAGs such as
[ex-dag03](..%20ex::..%20image::%20../images/dag03.png:scale:%2040) are
said to involve structure sharing or reentrancy. When two paths have the
same value, they are said to be equivalent.

In order to indicate reentrancy in our matrix-style representations, we
will prefix the first occurrence of a shared feature structure with an
integer in parentheses, such as `(1)`. Any later reference to that
structure will use the notation `->(1)`, as shown below.

> &gt;&gt;&gt; print(nltk.FeatStruct("""\[NAME='Lee',
> ADDRESS=(1)\[NUMBER=74, STREET='rue Pascal'\], ...
> SPOUSE=\[NAME='Kim', ADDRESS-&gt;(1)\]\]""")) \[ ADDRESS = (1) \[
> NUMBER = 74 \] \] \[ \[ STREET = 'rue Pascal' \] \] \[ \] \[ NAME =
> 'Lee' \] \[ \] \[ SPOUSE = \[ ADDRESS -&gt; (1) \] \] \[ \[ NAME =
> 'Kim' \] \]

The bracketed integer is sometimes called a tag or a coindex. The choice
of integer is not significant. There can be any number of tags within a
single feature structure.

> &gt;&gt;&gt; print(nltk.FeatStruct("\[A='a', B=(1)\[C='c'\],
> D-&gt;(1), E-&gt;(1)\]")) \[ A = 'a' \] \[ \] \[ B = (1) \[ C = 'c' \]
> \] \[ \] \[ D -&gt; (1) \] \[ E -&gt; (1) \]

### Subsumption and Unification

It is standard to think of feature structures as providing partial
information about some object, in the sense that we can order feature
structures according to how much information they contain. For example,
ex-fs01\_ has less information than ex-fs02\_, which in turn has less
information than ex-fs03\_.

This ordering is called subsumption; $FS$
~0~ subsumes $FS$~1~ if all the information contained in $FS$
~0~ is also contained in $FS$~1~. We use the symbol |SquareSubsetEqual|
to represent subsumption.

When we add the possibility of reentrancy, we need to be more careful
about how we describe subsumption: if $FS$~0~ |SquareSubsetEqual| $FS$
~1~, then $FS$~1~ must have all the paths and reentrancies of $FS$~0~.
Thus,
[ex-dag02](..%20ex::..%20image::%20../images/dag02.png:scale:%2040)
subsumes
[ex-dag03](..%20ex::..%20image::%20../images/dag03.png:scale:%2040),
since the latter has additional reentrancies. It should be obvious that
subsumption only provides a partial ordering on feature structures,
since some feature structures are incommensurable. For example,
[ex-fs04](..%20ex::::) neither subsumes nor is subsumed by ex-fs01\_.

> \[TELNO = 01 27 86 42 96\]

So we have seen that some feature structures carry more information than
others. How do we go about adding more information to a given feature
structure? For example, we might decide that addresses should consist of
not just a street number and a street name, but also a city. That is, we
might want to *merge* graph ex-dag042\_ with ex-dag041\_ to yield
ex-dag043\_.

Merging information from two feature structures is called unification
and is supported by the `unify()` method.

> &gt;&gt;&gt; fs1 = nltk.FeatStruct(NUMBER=74, STREET='rue Pascal')
> &gt;&gt;&gt; fs2 = nltk.FeatStruct(CITY='Paris') &gt;&gt;&gt;
> print(fs1.unify(fs2)) \[ CITY = 'Paris' \] \[ NUMBER = 74 \] \[ STREET
> = 'rue Pascal' \]

Unification is formally defined as a (partial) binary operation: $FS$~0~
|SquareUnion| $FS$~1~. Unification is symmetric, so $FS$~0~
|SquareUnion| $FS$~1~ = $FS$~1~ |SquareUnion| $FS$~0~. The same is true
in Python:

> &gt;&gt;&gt; print(fs2.unify(fs1)) \[ CITY = 'Paris' \] \[ NUMBER = 74
> \] \[ STREET = 'rue Pascal' \]

> only works with repr()

If we unify two feature structures which stand in the subsumption
relationship, then the result of unification is the most informative of
the two:

For example, the result of unifying ex-fs02\_ with ex-fs03\_ is
ex-fs03\_.

Unification between $FS$~0~ and $FS$
~1~ will fail if the two feature structures share a path |pi|, but the
value of |pi| in $FS$~0~ is a distinct atom from the value of |pi| in
$FS$~1~. This is implemented by setting the result of unification to be
`None`.

> &gt;&gt;&gt; fs0 = nltk.FeatStruct(A='a') &gt;&gt;&gt; fs1 =
> nltk.FeatStruct(A='b') &gt;&gt;&gt; fs2 = fs0.unify(fs1) &gt;&gt;&gt;
> print(fs2) None

Now, if we look at how unification interacts with structure-sharing,
things become really interesting. First, let's define
[ex-dag04](..%20ex::..%20image::%20../images/dag04.png:scale:%2040) in
Python:

> &gt;&gt;&gt; fs0 = nltk.FeatStruct("""\[NAME=Lee, ...
> ADDRESS=\[NUMBER=74, ... STREET='rue Pascal'\], ... SPOUSE=
> \[NAME=Kim, ... ADDRESS=\[NUMBER=74, ... STREET='rue Pascal'\]\]\]""")
> &gt;&gt;&gt; print(fs0) \[ ADDRESS = \[ NUMBER = 74 \] \] \[ \[ STREET
> = 'rue Pascal' \] \] \[ \] \[ NAME = 'Lee' \] \[ \] \[ \[ ADDRESS = \[
> NUMBER = 74 \] \] \] \[ SPOUSE = \[ \[ STREET = 'rue Pascal' \] \] \]
> \[ \[ \] \] \[ \[ NAME = 'Kim' \] \]

What happens when we augment Kim's address with a specification for
`CITY`? Notice that `fs1` needs to include the whole path from the root
of the feature structure down to `CITY`.

> &gt;&gt;&gt; fs1 = nltk.FeatStruct("\[SPOUSE = \[ADDRESS = \[CITY =
> Paris\]\]\]") &gt;&gt;&gt; print(fs1.unify(fs0)) \[ ADDRESS = \[
> NUMBER = 74 \] \] \[ \[ STREET = 'rue Pascal' \] \] \[ \] \[ NAME =
> 'Lee' \] \[ \] \[ \[ \[ CITY = 'Paris' \] \] \] \[ \[ ADDRESS = \[
> NUMBER = 74 \] \] \] \[ SPOUSE = \[ \[ STREET = 'rue Pascal' \] \] \]
> \[ \[ \] \] \[ \[ NAME = 'Kim' \] \]

By contrast, the result is very different if `fs1` is unified with the
structure-sharing version `fs2` (also shown earlier as the graph
[ex-dag03](..%20ex::..%20image::%20../images/dag03.png:scale:%2040)):

> &gt;&gt;&gt; fs2 = nltk.FeatStruct("""\[NAME=Lee,
> ADDRESS=(1)\[NUMBER=74, STREET='rue Pascal'\], ... SPOUSE=\[NAME=Kim,
> ADDRESS-&gt;(1)\]\]""") &gt;&gt;&gt; print(fs1.unify(fs2)) \[ \[ CITY
> = 'Paris' \] \] \[ ADDRESS = (1) \[ NUMBER = 74 \] \] \[ \[ STREET =
> 'rue Pascal' \] \] \[ \] \[ NAME = 'Lee' \] \[ \] \[ SPOUSE = \[
> ADDRESS -&gt; (1) \] \] \[ \[ NAME = 'Kim' \] \]

Rather than just updating what was in effect Kim's "copy" of Lee's
address, we have now updated both their addresses at the same time. More
generally, if a unification adds information to the value of some path
|pi|, then that unification simultaneously updates the value of
any path that is equivalent to |pi|.

As we have already seen, structure sharing can also be stated using
variables such as `?x`.

> &gt;&gt;&gt; fs1 = nltk.FeatStruct("\[ADDRESS1=\[NUMBER=74,
> STREET='rue Pascal'\]\]") &gt;&gt;&gt; fs2 =
> nltk.FeatStruct("\[ADDRESS1=?x, ADDRESS2=?x\]") &gt;&gt;&gt;
> print(fs2) \[ ADDRESS1 = ?x \] \[ ADDRESS2 = ?x \] &gt;&gt;&gt;
> print(fs2.unify(fs1)) \[ ADDRESS1 = (1) \[ NUMBER = 74 \] \] \[ \[
> STREET = 'rue Pascal' \] \] \[ \] \[ ADDRESS2 -&gt; (1) \]

Extending a Feature based Grammar
---------------------------------

In this section, we return to feature based grammar and explore a
variety of linguistic issues, and demonstrate the benefits of
incorporating features into the grammar.

### Subcategorization

In chap-parse\_, we augmented our category labels to represent different
kinds of verb, and used the labels `IV` and `TV` for intransitive and
transitive verbs respectively. This allowed us to write productions like
the following:

> VP -&gt; IV VP -&gt; TV NP

Although we know that `IV` and `TV` are two kinds of `V`, they are just
atomic nonterminal symbols from a CFG, as distinct from each other as
any other pair of symbols. This notation doesn't let us say anything
about verbs in general, e.g. we cannot say "All lexical items of
category `V` can be marked for tense", since walk, say, is an item of
category `IV`, not `V`. So, can we replace category labels such as `TV`
and `IV` by `V` along with a feature that tells us whether the verb
combines with a following `NP` object or whether it can occur without
any complement?

A simple approach, originally developed for a grammar framework called
Generalized Phrase Structure Grammar (GPSG), tries to solve this problem
by allowing lexical categories to bear a `SUBCAT` which tells us what
subcategorization class the item belongs to. While GPSG used integer
values for `SUBCAT`, the example below adopts more mnemonic values,
namely `intrans`, `trans` and `clause`:

> VP\[TENSE=?t, NUM=?n\] -&gt; V\[SUBCAT=intrans, TENSE=?t, NUM=?n\]
> VP\[TENSE=?t, NUM=?n\] -&gt; V\[SUBCAT=trans, TENSE=?t, NUM=?n\] NP
> VP\[TENSE=?t, NUM=?n\] -&gt; V\[SUBCAT=clause, TENSE=?t, NUM=?n\] SBar
>
> V\[SUBCAT=intrans, TENSE=pres, NUM=sg\] -&gt; 'disappears' | 'walks'
> V\[SUBCAT=trans, TENSE=pres, NUM=sg\] -&gt; 'sees' | 'likes'
> V\[SUBCAT=clause, TENSE=pres, NUM=sg\] -&gt; 'says' | 'claims'
>
> V\[SUBCAT=intrans, TENSE=pres, NUM=pl\] -&gt; 'disappear' | 'walk'
> V\[SUBCAT=trans, TENSE=pres, NUM=pl\] -&gt; 'see' | 'like'
> V\[SUBCAT=clause, TENSE=pres, NUM=pl\] -&gt; 'say' | 'claim'
>
> V\[SUBCAT=intrans, TENSE=past, NUM=?n\] -&gt; 'disappeared' | 'walked'
> V\[SUBCAT=trans, TENSE=past, NUM=?n\] -&gt; 'saw' | 'liked'
> V\[SUBCAT=clause, TENSE=past, NUM=?n\] -&gt; 'said' | 'claimed'

When we see a lexical category like `V[SUBCAT=trans]`, we can interpret
the `SUBCAT` specification as a pointer to a production in which
`V[SUBCAT=trans]` is introduced as the head child in a `VP` production.
By convention, there is a correspondence between the values of `SUBCAT`
and the productions that introduce lexical heads. On this approach,
`SUBCAT` can *only* appear on lexical categories; it makes no sense, for
example, to specify a `SUBCAT` value on `VP`. As required, walk and like
both belong to the category `V`. Nevertheless, walk will only occur in
`VP`s expanded by a production with the feature `SUBCAT=intrans` on the
right hand side, as opposed to like, which requires a `SUBCAT=trans`.

In our third class of verbs above, we have specified a category `SBar`.
This is a label for subordinate clauses such as the complement of claim
in the example You claim that you like
children. We require two further productions to analyze such sentences:

> SBar -&gt; Comp S Comp -&gt; 'that'

The resulting structure is the following.

An alternative treatment of subcategorization, due originally to a
framework known as categorial grammar, is represented in feature based
frameworks such as PATR and Head-driven Phrase Structure Grammar. Rather
than using `SUBCAT` values as a way of indexing productions, the
`SUBCAT` value directly encodes the valency of a head (the list of
arguments that it can combine with). For example, a verb like put that
takes `NP` and `PP` complements (put the
book on the table) might be represented as
[ex-subcathpsg0](..%20ex::::):

This says that the verb can combine with three arguments. The leftmost
element in the list is the subject `NP`, while everything else |mdash|
an `NP` followed by a `PP` in this case |mdash| comprises the
subcategorized-for complements. When a verb like put is combined with
appropriate complements, the requirements which are specified in the
`SUBCAT` are discharged, and only a subject `NP` is needed. This
category, which corresponds to what is traditionally thought of as `VP`,
might be represented as follows.

> V\[SUBCAT=&lt;NP&gt;\]

Finally, a sentence is a kind of verbal category that has no
requirements for further arguments, and hence has a `SUBCAT` whose value
is the empty list. The tree
[ex-subcathpsg2](..%20ex::..%20tree::%20(V%5BSUBCAT=\%3C\%3E%5D%20(NP%20Kim)(V%5BSUBCAT=\%3CNP\%3E%5D(V%5BSUBCAT=\%3CNP,\%20NP,\%20PP\%3E%5D%20put)%3CNP%20the\%20book%3E%3CPP%20on\%20the\%20table%3E)))
shows how these category assignments combine in a parse of
Kim put the book on the table.

### Heads Revisited

We noted in the previous section that by factoring subcategorization
information out of the main category label, we could express more
generalizations about properties of verbs. Another property of this kind
is the following: expressions of category `V` are heads of phrases of
category `VP`. Similarly, `N`s are heads of `NP`s, `A`s (i.e.,
adjectives) are heads of `AP`s, and `P`s (i.e., prepositions) are heads
of `PP`s. Not all phrases have heads |mdash| for example, it is standard
to say that coordinate phrases (e.g., the book and the bell) lack heads
|mdash| nevertheless, we would like our grammar formalism to express the
parent / head-child relation where it holds. At present, `V` and `VP`
are just atomic symbols, and we need to find a way to relate them using
features (as we did earlier to relate `IV` and `TV`).

X-bar Syntax addresses this issue by abstracting out the notion of
phrasal level. It is usual to recognize three such levels. If `N`
represents the lexical level, then `N`' represents the next level up,
corresponding to the more traditional category Nom, while `N`''
represents the phrasal level, corresponding to the category `NP`.
ex-xbar0\_ illustrates a representative structure while ex-xbar01\_ is
the more conventional counterpart.

The head of the structure ex-xbar0\_ is `N` while `N`' and `N`'' are
called (phrasal) projections of `N`. `N`'' is the maximal projection,
and `N` is sometimes called the zero projection. One of the central
claims of X-bar syntax is that all constituents share a structural
similarity. Using `X` as a variable over `N`, `V`, `A` and `P`, we say
that directly subcategorized complements of a lexical head `X` are
always placed as siblings of the head, whereas adjuncts are placed as
siblings of the intermediate category, `X`'. Thus, the configuration of
the two `P`'' adjuncts in
[ex-xbar1](..%20ex::..%20tree::%20(N''(Det%20a)(N'(N'(N'(N%20student))(P''%20from\%20France))(P''%20with\%20good\%20grades))))
contrasts with that of the complement `P`'' in ex-xbar0\_.

The productions in [ex-xbar2](..%20ex::::) illustrate how bar levels can
be encoded using feature structures. The nested structure in
[ex-xbar1](..%20ex::..%20tree::%20(N''(Det%20a)(N'(N'(N'(N%20student))(P''%20from\%20France))(P''%20with\%20good\%20grades))))
is achieved by two applications of the recursive rule expanding
`N[BAR=1]`.

> S -&gt; N\[BAR=2\] V\[BAR=2\] N\[BAR=2\] -&gt; Det N\[BAR=1\]
> N\[BAR=1\] -&gt; N\[BAR=1\] P\[BAR=2\] N\[BAR=1\] -&gt; N\[BAR=0\]
> P\[BAR=2\] N\[BAR=1\] -&gt; N\[BAR=0\]XS

### Auxiliary Verbs and Inversion

Inverted clauses |mdash| where the order of subject and verb is switched
|mdash| occur in English interrogatives and also after 'negative'
adverbs:

> Do you like children?

However, we cannot place just any verb in pre-subject position:

> \*Like you children?

Verbs that can be positioned initially in inverted clauses belong to the
class known as auxiliaries, and as well as do, can and have include be,
will and shall. One way of capturing such structures is with the
following production:

> S\[+INV\] -&gt; V\[+AUX\] NP VP

That is, a clause marked as \[+INV\] consists of an auxiliary verb
followed by a `VP`. (In a more detailed grammar, we would need to place
some constraints on the form of the `VP`, depending on the choice of
auxiliary.)
[ex-invtree](..%20ex::..%20tree::%20(S%5B+INV%5D(V%5B+AUX,\%20SUBCAT=3%5D%20do)(NP%20you)(VP(V%5B-AUX,\%20SUBCAT=1%5D%20like)(NP%20children))))
illustrates the structure of an inverted clause.

### Unbounded Dependency Constructions

Consider the following contrasts:

> You like Jody.

The verb like requires an `NP` complement, while put requires both a
following `NP` and `PP`. [ex-gap1](..%20ex::..%20_ex-gap1a:..%20ex::)
and [ex-gap2](..%20ex::..%20_ex-gap2a:..%20ex::) show that these
complements are *obligatory*: omitting them leads to ungrammaticality.
Yet there are contexts in which obligatory complements can be omitted,
as [ex-gap3](..%20ex::..%20_ex-gap3a:..%20ex::) and
[ex-gap4](..%20ex::..%20_ex-gap4a:..%20ex::) illustrate.

> Kim knows who you like.

That is, an obligatory complement can be omitted if there is an
appropriate filler in the sentence, such as the question word who in
ex-gap3a\_, the preposed topic this music in ex-gap3b\_, or the wh
phrases which card/slot in [ex-gap4](..%20ex::..%20_ex-gap4a:..%20ex::).
It is common to say that sentences like
[ex-gap3](..%20ex::..%20_ex-gap3a:..%20ex::) |ndash|
[ex-gap4](..%20ex::..%20_ex-gap4a:..%20ex::) contain gaps where the
obligatory complements have been omitted, and these gaps are sometimes
made explicit using an underscore:

> Which card do you put \_\_ into the slot?

So, a gap can occur if it is licensed by a filler. Conversely, fillers
can only occur if there is an appropriate gap elsewhere in the sentence,
as shown by the following examples.

> \*Kim knows who you like Jody.

The mutual co-occurence between filler and gap is sometimes termed a
"dependency". One issue of considerable importance in theoretical
linguistics has been the nature of the material that can intervene
between a filler and the gap that it licenses; in particular, can we
simply list a finite set of sequences that separate the two? The answer
is No: there is no upper bound on the distance between filler and gap.
This fact can be easily illustrated with constructions involving
sentential complements, as shown in
[ex-gap8](..%20ex::..%20_ex-gap8a:..%20ex::).

> Who do you like \_\_?

Since we can have indefinitely deep recursion of sentential complements,
the gap can be embedded indefinitely far inside the whole sentence. This
constellation of properties leads to the notion of an
unbounded dependency construction; that is, a filler-gap dependency
where there is no upper bound on the distance between filler and gap.

A variety of mechanisms have been suggested for handling unbounded
dependencies in formal grammars; here we illustrate the approach due to
Generalized Phrase Structure Grammar that involves slash categories. A
slash category has the form `Y/XP`; we interpret this as a phrase of
category `Y` that is missing a sub-constituent of category `XP`. For
example, `S/NP` is an `S` that is missing an `NP`. The use of slash
categories is illustrated in
[ex-gaptree1](..%20ex::..%20tree::%20(S(NP%5B+WH%5D%20who)(S%5B+INV%5D/NP%20(V%5B+AUX%5D%20do)(NP%5B-WH%5D%20you)(VP/NP(V%5B-AUX,\%20SUBCAT=trans%5D%20like)(NP/NP))))).

The top part of the tree introduces the filler who (treated as an
expression of category `NP[+wh]`) together with a corresponding
gap-containing constituent `S/NP`. The gap information is then
"percolated" down the tree via the `VP/NP` category, until it reaches
the category `NP/NP`. At this point, the dependency is discharged by
realizing the gap information as the empty string, immediately dominated
by `NP/NP`.

Do we need to think of slash categories as a completely new kind of
object? Fortunately, we can accommodate them within our existing feature
based framework, by treating slash as a feature, and the category to its
right as a value; that is, `S/NP` is reducible to `S[SLASH=NP]`. In
practice, this is also how the parser interprets slash categories.

The grammar shown in code-slashcfg\_ illustrates the main principles of
slash categories, and also includes productions for inverted clauses. To
simplify presentation, we have omitted any specification of tense on the
verbs.

> &gt;&gt;&gt; nltk.data.show\_cfg('grammars/book\_grammars/feat1.fcfg')
> % start S \# \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# \# Grammar
> Productions \# \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# S\[-INV\] -&gt;
> NP VP S\[-INV\]/?x -&gt; NP VP/?x S\[-INV\] -&gt; NP S/NP S\[-INV\]
> -&gt; Adv\[+NEG\] S\[+INV\] S\[+INV\] -&gt; V\[+AUX\] NP VP
> S\[+INV\]/?x -&gt; V\[+AUX\] NP VP/?x SBar -&gt; Comp S\[-INV\]
> SBar/?x -&gt; Comp S\[-INV\]/?x VP -&gt; V\[SUBCAT=intrans, -AUX\] VP
> -&gt; V\[SUBCAT=trans, -AUX\] NP VP/?x -&gt; V\[SUBCAT=trans, -AUX\]
> NP/?x VP -&gt; V\[SUBCAT=clause, -AUX\] SBar VP/?x -&gt;
> V\[SUBCAT=clause, -AUX\] SBar/?x VP -&gt; V\[+AUX\] VP VP/?x -&gt;
> V\[+AUX\] VP/?x \# \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# \# Lexical
> Productions \# \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#
> V\[SUBCAT=intrans, -AUX\] -&gt; 'walk' | 'sing' V\[SUBCAT=trans,
> -AUX\] -&gt; 'see' | 'like' V\[SUBCAT=clause, -AUX\] -&gt; 'say' |
> 'claim' V\[+AUX\] -&gt; 'do' | 'can' NP\[-WH\] -&gt; 'you' | 'cats'
> NP\[+WH\] -&gt; 'who' Adv\[+NEG\] -&gt; 'rarely' | 'never' NP/NP -&gt;
> Comp -&gt; 'that'

The grammar in code-slashcfg\_ contains one "gap-introduction"
production, namely `S[-INV] -> NP S/NP`. In order to percolate the slash
feature correctly, we need to add slashes with variable values to both
sides of the arrow in productions that expand `S`, `VP` and `NP`. For
example, `VP/?x -> V SBar/?x` is the slashed version of `VP -> V SBar`
and says that a slash value can be specified on the `VP` parent of a
constituent if the same value is also specified on the `SBar` child.
Finally, `NP/NP ->` allows the slash information on `NP` to be
discharged as the empty string. Using code-slashcfg\_, we can parse the
sequence who do you claim that you
like

> &gt;&gt;&gt; tokens = 'who do you claim that you like'.split()
> &gt;&gt;&gt; from nltk import load\_parser &gt;&gt;&gt; cp =
> load\_parser('grammars/book\_grammars/feat1.fcfg') &gt;&gt;&gt; for
> tree in cp.parse(tokens): ... print(tree) (S\[-INV\] (NP\[+WH\] who)
> (S\[+INV\]/NP\[\] (V\[+AUX\] do) (NP\[-WH\] you) (VP\[\]/NP\[\]
> (V\[-AUX, SUBCAT='clause'\] claim) (SBar\[\]/NP\[\] (Comp\[\] that)
> (S\[-INV\]/NP\[\] (NP\[-WH\] you) (VP\[\]/NP\[\] (V\[-AUX,
> SUBCAT='trans'\] like) (NP\[\]/NP\[\] )))))))

A more readable version of this tree is shown in
[ex-gapparse](..%20ex::..%20tree::%20(S%5B-INV%5D(NP%5B+WH%5D%20who)(S%5B+INV%5D/NP(V%5B+AUX%5D%20do)(NP%5B-WH%5D%20you)(VP/NP(V%5B-AUX,\%20SUBCAT=clause%5D%20claim)(SBar/NP(Comp%20that)(S%5B-INV%5D/NP(NP%5B-WH%5D%20you)(VP/NP(V%5B-AUX,\%20SUBCAT=trans%5D%20like)(NP/NP))))))):scale:%2060:60:50).

The grammar in code-slashcfg\_ will also allow us to parse sentences
without gaps:

> &gt;&gt;&gt; tokens = 'you claim that you like cats'.split()
> &gt;&gt;&gt; for tree in cp.parse(tokens): ... print(tree) (S\[-INV\]
> (NP\[-WH\] you) (VP\[\] (V\[-AUX, SUBCAT='clause'\] claim) (SBar\[\]
> (Comp\[\] that) (S\[-INV\] (NP\[-WH\] you) (VP\[\] (V\[-AUX,
> SUBCAT='trans'\] like) (NP\[-WH\] cats))))))

In addition, it admits inverted sentences which do not involve wh
constructions:

> &gt;&gt;&gt; tokens = 'rarely do you sing'.split() &gt;&gt;&gt; for
> tree in cp.parse(tokens): ... print(tree) (S\[-INV\] (Adv\[+NEG\]
> rarely) (S\[+INV\] (V\[+AUX\] do) (NP\[-WH\] you) (VP\[\] (V\[-AUX,
> SUBCAT='intrans'\] sing))))

### Case and Gender in German

Compared with English, German has a relatively rich morphology for
agreement. For example, the definite article in German varies with case,
gender and number, as shown in tab-german-def-art\_.

Subjects in German take the nominative case, and most verbs govern their
objects in the accusative case. However, there are exceptions like
helfen that govern the dative case:

The grammar in code-germancfg\_ illustrates the interaction of agreement
(comprising person, number and gender) with case.

As you can see, the feature objcase is used to specify the case that a
verb governs on its object. The next example illustrates the parse tree
for a sentence containing a verb which governs dative case.

> &gt;&gt;&gt; tokens = 'ich folge den Katzen'.split() &gt;&gt;&gt; cp =
> load\_parser('grammars/book\_grammars/german.fcfg') &gt;&gt;&gt; for
> tree in cp.parse(tokens): ... print(tree) (S\[\] (NP\[AGR=\[NUM='sg',
> PER=1\], CASE='nom'\] (PRO\[AGR=\[NUM='sg', PER=1\], CASE='nom'\]
> ich)) (VP\[AGR=\[NUM='sg', PER=1\]\] (TV\[AGR=\[NUM='sg', PER=1\],
> OBJCASE='dat'\] folge) (NP\[AGR=\[GND='fem', NUM='pl', PER=3\],
> CASE='dat'\] (Det\[AGR=\[NUM='pl', PER=3\], CASE='dat'\] den)
> (N\[AGR=\[GND='fem', NUM='pl', PER=3\]\] Katzen))))

In developing grammars, excluding ungrammatical word sequences is often
as challenging as parsing grammatical ones. In order to get an idea
where and why a sequence fails to parse, setting the `trace` parameter
of the `load_parser()` method can be crucial. Consider the following
parse failure:

> &gt;&gt;&gt; tokens = 'ich folge den Katze'.split() &gt;&gt;&gt; cp =
> load\_parser('grammars/book\_grammars/german.fcfg', trace=2)
> &gt;&gt;&gt; for tree in cp.parse(tokens): ... print(tree)
> |.ich.fol.den.Kat.| Leaf Init Rule: |\[---\] . . .| \[0:1\] 'ich' |.
> \[---\] . .| \[1:2\] 'folge' |. . \[---\] .| \[2:3\] 'den' |. . .
> \[---\]| \[3:4\] 'Katze' Feature Bottom Up Predict Combine Rule:
> |\[---\] . . .| \[0:1\] PRO\[AGR=\[NUM='sg', PER=1\], CASE='nom'\]
> -&gt; 'ich' \* Feature Bottom Up Predict Combine Rule: |\[---\] . . .|
> \[0:1\] NP\[AGR=\[NUM='sg', PER=1\], CASE='nom'\] -&gt;
> PRO\[AGR=\[NUM='sg', PER=1\], CASE='nom'\] \* Feature Bottom Up
> Predict Combine Rule: |\[---&gt; . . .| \[0:1\] S\[\] -&gt;
> NP\[AGR=?a, CASE='nom'\] \* VP\[AGR=?a\] {?a: \[NUM='sg', PER=1\]}
> Feature Bottom Up Predict Combine Rule: |. \[---\] . .| \[1:2\]
> TV\[AGR=\[NUM='sg', PER=1\], OBJCASE='dat'\] -&gt; 'folge' \* Feature
> Bottom Up Predict Combine Rule: |. \[---&gt; . .| \[1:2\] VP\[AGR=?a\]
> -&gt; TV\[AGR=?a, OBJCASE=?c\] \* NP\[CASE=?c\] {?a: \[NUM='sg',
> PER=1\], ?c: 'dat'} Feature Bottom Up Predict Combine Rule: |. .
> \[---\] .| \[2:3\] Det\[AGR=\[GND='masc', NUM='sg', PER=3\],
> CASE='acc'\] -&gt; 'den' \* |. . \[---\] .| \[2:3\]
> Det\[AGR=\[NUM='pl', PER=3\], CASE='dat'\] -&gt; 'den' \* Feature
> Bottom Up Predict Combine Rule: |. . \[---&gt; .| \[2:3\] NP\[AGR=?a,
> CASE=?c\] -&gt; Det\[AGR=?a, CASE=?c\] \* N\[AGR=?a, CASE=?c\] {?a:
> \[NUM='pl', PER=3\], ?c: 'dat'} Feature Bottom Up Predict Combine
> Rule: |. . \[---&gt; .| \[2:3\] NP\[AGR=?a, CASE=?c\] -&gt;
> Det\[AGR=?a, CASE=?c\] \* N\[AGR=?a, CASE=?c\] {?a: \[GND='masc',
> NUM='sg', PER=3\], ?c: 'acc'} Feature Bottom Up Predict Combine Rule:
> |. . . \[---\]| \[3:4\] N\[AGR=\[GND='fem', NUM='sg', PER=3\]\] -&gt;
> 'Katze' \*

The last two `Scanner` lines in the trace show that den is recognized as
admitting two possible categories:
`Det[AGR=[GND='masc', NUM='sg', PER=3], CASE='acc']` and
`Det[AGR=[NUM='pl', PER=3], CASE='dat']`. We know from the grammar in
code-germancfg\_ that `Katze` has category
`N[AGR=[GND=fem, NUM=sg, PER=3]]`. Thus there is no binding for the
variable `?a` in production
`NP[CASE=?c, AGR=?a] -> Det[CASE=?c, AGR=?a] N[CASE=?c, AGR=?a]` which
will satisfy these constraints, since the `AGR` value of `Katze` will
not unify with either of the `AGR` values of den, that is, with either
`[GND='masc', NUM='sg', PER=3]` or `[NUM='pl', PER=3]`.

Summary
-------

-   The traditional categories of context-free grammar are
    atomic symbols. An important motivation for feature structures is to
    capture fine-grained distinctions that would otherwise require a
    massive multiplication of atomic categories.
-   By using variables over feature values, we can express constraints
    in grammar productions that allow the realization of different
    feature specifications to be inter-dependent.
-   Typically we specify fixed values of features at the lexical level
    and constrain the values of features in phrases to unify with the
    corresponding values in their children.
-   Feature values are either atomic or complex. A particular sub-case
    of atomic value is the Boolean value, represented by convention as
    \[+/- `f`\].
-   Two features can share a value (either atomic or complex).
    Structures with shared values are said to be re-entrant. Shared
    values are represented by numerical indexes (or tags) in AVMs.
-   A path in a feature structure is a tuple of features corresponding
    to the labels on a sequence of arcs from the root of the
    graph representation.
-   Two paths are equivalent if they share a value.
-   Feature structures are partially ordered by subsumption. $FS$~0~
    subsumes $FS$~1~ when all the information contained in $FS$~0~ is
    also present in $FS$~1~.
-   The unification of two structures $FS$~0~ and $FS$~1~, if
    successful, is the feature structure $FS$~2~ that contains the
    combined information of both $FS$~0~ and $FS$
    ~1~.
-   If unification adds information to a path |pi| in $FS$, then it also
    adds information to every path |pi|' equivalent to |pi|.
-   We can use feature structures to build succinct analyses of a wide
    variety of linguistic phenomena, including verb subcategorization,
    inversion constructions, unbounded dependency constructions and
    case government.

Further Reading
---------------

Please consult |NLTK-URL| for further materials on this chapter,
including feature structures, feature grammars, and grammar test suites.

X-bar Syntax: \[Chomsky1970RN\]\_, \[Jackendoff1977XS\]\_ (The primes we
use replace Chomsky's typographically more demanding horizontal bars.)

For an excellent introduction to the phenomenon of agreement, see
\[Corbett2006A\]\_.

The earliest use of features in theoretical linguistics was designed to
capture phonological properties of phonemes. For example, a sound like
/**b**/ might be decomposed into the structure `[+labial, +voice]`. An
important motivation was to capture generalizations across classes of
segments; for example, that /**n**/ gets realized as /**m**/ preceding
any `+labial` consonant. Within Chomskyan grammar, it was standard to
use atomic features for phenomena like agreement, and also to capture
generalizations across syntactic categories, by analogy with phonology.
A radical expansion of the use of features in theoretical syntax was
advocated by Generalized Phrase Structure Grammar (GPSG;
\[Gazdar1985GPS\]\_), particularly in the use of features with complex
values.

Coming more from the perspective of computational linguistics,
\[Kay1984UG\]\_ proposed that functional aspects of language could be
captured by unification of attribute-value structures, and a similar
approach was elaborated by \[Shieber1983FIP\]\_ within the PATR-II
formalism. Early work in Lexical-Functional grammar (LFG;
\[Kaplan1982LFG\]\_) introduced the notion of an f-structure that was
primarily intended to represent the grammatical relations and
predicate-argument structure associated with a constituent structure
parse. \[Shieber1986IUB\]\_ provides an excellent introduction to this
phase of research into feature based grammars.

One conceptual difficulty with algebraic approaches to feature
structures arose when researchers attempted to model negation. An
alternative perspective, pioneered by \[Kasper1986LSF\]\_ and
\[Johnson1988AVL\]\_, argues that grammars involve descriptions of
feature structures rather than the structures themselves. These
descriptions are combined using logical operations such as conjunction,
and negation is just the usual logical operation over feature
descriptions. This description-oriented perspective was integral to LFG
from the outset (cf. \[Kaplan1989FAL\]\_, and was also adopted by later
versions of Head-Driven Phrase Structure Grammar (HPSG;
\[Sag1999ST\]\_). A comprehensive bibliography of HPSG literature can be
found at `http://www.cl.uni-bremen.de/HPSG-Bib/`.

Feature structures, as presented in this chapter, are unable to capture
important constraints on linguistic information. For example, there is
no way of saying that the only permissible values for `NUM` are `sg` and
`pl`, while a specification such as `[NUM=masc]` is anomalous.
Similarly, we cannot say that the complex value of `AGR` must contain
specifications for the features `PER`, `NUM` and `gnd`, but cannot
contain a specification such as `[SUBCAT=trans]`.
Typed feature structures were developed to remedy this deficiency. To
begin with, we stipulate that feature values are always typed. In the
case of atomic values, the values just are types. For example, we would
say that the value of `NUM` is the type `num`. Moreover, `num` is the
most general type of value for `NUM`. Since types are organized
hierarchically, we can be more informative by specifying the value of
`NUM` is a subtype of `num`, namely either `sg` or `pl`.

In the case of complex values, we say that feature structures are
themselves typed. So for example the value of `AGR` will be a feature
structure of type `AGR`. We also stipulate that all and only `PER`,
`NUM` and `GND` are appropriate features for a structure of type `AGR`.
A good early review of work on typed feature structures is
\[Emele1990TUG\]\_. A more comprehensive examination of the formal
foundations can be found in \[Carpenter1992LTF\]\_, while
\[Copestake2002ITF\]\_ focuses on implementing an HPSG-oriented approach
to typed feature structures.

There is a copious literature on the analysis of German within feature
based grammar frameworks. \[Nerbonne1994GHD\]\_ is a good starting point
for the HPSG literature on this topic, while \[Mueller2002CP\]\_ gives a
very extensive and detailed analysis of German syntax in HPSG.

Chapter 15 of \[JurafskyMartin2008\]\_ discusses feature structures, the
unification algorithm, and the integration of unification into parsing
algorithms.

Exercises
---------

1.  |easy| What constraints are required to correctly parse word
    sequences like I am
    happy and she is happy but not \*you is happy or \*they am happy?
    Implement two solutions for the present tense paradigm of the verb
    be in English, first taking Grammar [ex-agcfg1](..%20ex::::) as your
    starting point, and then taking Grammar [ex-agr2](..%20ex::::) as
    the starting point.
2.  |easy| Develop a variant of grammar in code-feat0cfg\_ that uses a
    feature count to make the distinctions shown below:
3.  |easy| Write a function subsumes() which holds of two feature
    structures `fs1` and `fs2` just in case `fs1` subsumes `fs2`.
4.  |easy| Modify the grammar illustrated in
    [ex-subcatgpsg](..%20ex::::) to incorporate a bar feature for
    dealing with phrasal projections.
5.  |easy| Modify the German grammar in code-germancfg\_ to incorporate
    the treatment of subcategorization presented
    in sec-extending-a-feature-based-grammar\_.
6.  |soso| Develop a feature based grammar that will correctly describe
    the following Spanish noun phrases:
7.  |soso| Develop your own version of the `EarleyChartParser` which
    only prints a trace if the input sequence fails to parse.
8.  |soso| Consider the feature structures shown
    in code-featstructures\_.

    Work out on paper what the result is of the following unifications.
    (Hint: you might find it useful to draw the graph structures.)

    1)  `fs1` and `fs2`
    2)  `fs1` and `fs3`
    3)  `fs4` and `fs5`
    4)  `fs5` and `fs6`
    5)  `fs5` and `fs7`
    6)  `fs8` and `fs9`
    7)  `fs8` and `fs10`

    Check your answers using Python.

9.  |soso| List two feature structures that subsume \[A=?x, B=?x\].
10. |soso| Ignoring structure sharing, give an informal algorithm for
    unifying two feature structures.
11. |soso| Extend the German grammar in code-germancfg\_ so that it can
    handle so-called verb-second structures like the following:
12. |soso| Seemingly synonymous verbs have slightly different syntactic
    properties \[Levin1993\]\_. Consider the patterns of grammaticality
    for the verbs loaded, filled, and dumped below. Can you write
    grammar productions to handle such data?

13. |hard| Morphological paradigms are rarely completely regular, in the
    sense of every cell in the matrix having a different realization.
    For example, the present tense conjugation of the lexeme walk only
    has two distinct forms: walks for the 3rd person singular, and walk
    for all other combinations of person and number. A successful
    analysis should not require redundantly specifying that 5 out of the
    6 possible morphological combinations have the same realization.
    Propose and implement a method for dealing with this.
14. |hard| So-called head features are shared between the parent node
    and head child. For example, `TENSE` is a head feature that is
    shared between a `VP` and its head `V` child. See
    \[Gazdar1985GPS\]\_ for more details. Most of the features we have
    looked at are head features |mdash| exceptions are `SUBCAT` and
    `SLASH`. Since the sharing of head features is predictable, it
    should not need to be stated explicitly in the grammar productions.
    Develop an approach that automatically accounts for this regular
    behavior of head features.
15. |hard| Extend |NLTK|'s treatment of feature structures to allow
    unification into list-valued features, and use this to implement an
    HPSG-style analysis of subcategorization, whereby the `SUBCAT` of a
    head category is the concatenation its complements' categories with
    the `SUBCAT` value of its immediate parent.
16. |hard| Extend |NLTK|'s treatment of feature structures to allow
    productions with underspecified categories, such as
    `S[-INV] --> ?x S/?x`.
17. |hard| Extend |NLTK|'s treatment of feature structures to allow
    typed feature structures.
18. |hard| Pick some grammatical constructions described in
    \[Huddleston2002CGE\]\_, and develop a feature based grammar to
    account for them.

