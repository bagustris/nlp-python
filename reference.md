---
layout: reference
---

## Glossary
{:auto_ids}

Token
: Before any real processing can be done on the input text, it
needs to be segmented into linguistic units such as words, punctuation,
numbers or alphanumerics. These units are known as tokens.

Sentence
: An ordered sequence of tokens.

Tokenization
: The process of splitting a sentence into its constituent
tokens.  For segmented languages such as English, the existence of
whitespace makes tokenization relatively easier and uninteresting. How-
ever, for languages such as Chinese and Arabic, the task is more dif-
ficult since there are no explicit boundaries. Furthermore, almost all
characters in such non-segmented languages can exist as one-character
words by themselves but can also join together to form multi-character
words.

Corpus
: A body of text, usually containing a large number of sen-
tences.

Part-of-speech  (POS)  Tag
: A word can be classified into one or
more of a set of lexical or part-of-speech categories such as
Nouns, Verbs, Adjectives and Articles, to name a few. A POS tag is a symbol
representing such a lexical category - NN (Noun),  VB (Verb), JJ (Adjective),
AT (Article). One of the oldest and most commonly used tag sets is
the Brown Corpus

Parse Tree
: A tree defined over a given sentence that represents the
syntactic structure of the sentence as defined by a formal grammar.etail below

Chunking
: how to combine word into phrase/sentences

Named entity recognition
: Given sentence, to identify the person, loc, org

Corereference
: Given a corpus, one refer another

Semantic (role labelling)
: Labelling sentence to give (different) meaning
  e.q: /I saw a man/ with telescope/ or /I saw/ a man with telescope/

Lexicon
: Dictionary of words and its type

Syntatic
: Relation of word to other words (structure).

Discourse (analysis)
: Analyze not a single sentence but a set of sentences

Natural language generation
: generation of text from semantic interpretation

Bag-of-words
: Modelling a text (such as a sentence or a document) which is represented as the bag (multiset) of its words, disregarding grammar and even word order but keeping multiplicity.
