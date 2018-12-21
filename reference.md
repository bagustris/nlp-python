---
layout: reference
---

## Glossary
{:auto_ids}

Annotated Corpus
: Corpora to which some information is attached (POS, word boundaries, etc)

Arc
: Connector of two nodes,represents a partial structure between them. Buffer of arcs to be added to the chart in future (stack)

Bag-of-words
: Modeling a text (such as a sentence or a document) which is represented as the bag (multiset) of its words, disregarding grammar and even word order but keeping multiplicity.

Chart 
: Figure showing a process of parsing (nodes and arcs)

Corpus
: A collection of (text) sentences, usually containing a large number of sentences.

Chunking
: how to combine word into phrase/sentences

Corereference
: Given a corpus, one refer another

Dependency annotated
: annotation based on dependency in a sentence

Document frequency
: Number of documents containing an index term j

Dictionary
: Database of words including various kinds of information

Discourse (analysis)
: Analyze not a single sentence but a set of sentences

Grammar
: Set of rule/formalism that is used to specify what sentence are possible in language

Hash function, H(q)
: Functino calculating the index for a search word q

History
: List of arcs showing all generated arcs during parsing

Indexing
: Procedure to extract index from documents

Information retrieval (IR)
: process to find documents which are relevant to an users's query from a document collection

Information (Compiled in Dictionary
: Necessary information for NLP i.e: Morphological analysis, Syntatic analysis, Semantic analysis

Inverted Index
: List of index term for each document in inverted form (row=index, column=document)

Lexicon
: Dictionary of words and its type

Multimodal data
: Data collected from various way from the same target, e.g: text, speech, video

Part-of-speech  (POS)  Tag
: A word can be classified into one or
more of a set of lexical or part-of-speech categories such as
Nouns, Verbs, Adjectives and Articles, to name a few. A POS tag is a symbol
representing such a lexical category - NN (Noun),  VB (Verb), JJ (Adjective),
AT (Article). One of the oldest and most commonly used tag sets is
the Brown Corpus

Parsing
: How to determine the structure of sentence according to grammar

Parse Tree
: A tree defined over a given sentence that represents the
syntactic structure of the sentence as defined by a formal grammar.etail below

Parallel Corpus
: Collection of (translation) corpus of two or more language in parallel.

PCFG (Probabilistic Context Free Grammar)
: Product (multiplication) of probabilities of rules in parse tree

Precision
: Number of relevant documents in system output per number of documents in system output (how many selected items are relevant?)

Recall
: Number of relevant document in system output per number of all relevant document in overall collection (how many relevant are the selected items?)

Relevance feedback
: Asking user to judge whether retrieved documents are relevant or not

Sentence
: An ordered sequence of tokens.

Semantic (role labelling)
: Labelling sentence to give (different) meaning
  e.q: /I saw a man/ with telescope/ or /I saw/ a man with telescope/

Semantic primitive
: Basic primitive of sense of word, e.g: human, abstract, concrete

Stop word
: word not to be index term (function word; be, have, symbol; etc)

Syntactic
: Relation of word to other words (structure).

Thesaurus
: Database showing *semantic relation* between words

Tokenization
: The process of splitting a sentence into its constituent
tokens.  For segmented languages such as English, the existence of
whitespace makes tokenization relatively easier and uninteresting. 
However, for languages such as Chinese and Arabic, the task is more 
difficult since there are no explicit boundaries. Furthermore, almost all
characters in such non-segmented languages can exist as one-character
words by themselves but can also join together to form multi-character
words.

Token
: Before any real processing can be done on the input text, it
needs to be segmented into linguistic units such as words, punctuation,
numbers or alphanumerics. These units are known as tokens.

Trie
: A tree structure such that common prefix of words are shares

N-gram
: Collocation frequency of n words, e.g: 2-gram, 3-gram, etc.

N-gram model
: Markov model for generation probability of a sentence by 
  calculating probality of a word Ci depends only preceding N-1 
  words (for 2-gram)

Morphology (Morphological analysis)
: Divide word into morphemes

Morphemes
: Minimum linguistic Unit smaller than word, e.g. play-ing, un-kind-ly.

Named entity recognition
: Given sentence, to identify the person, location, organization

Natural language generation
: Generation of text from semantic interpretation

Node
: Virtual node between words, 

Vector space model
: Vector represented both documents and query (from inverted index)

Query
: An index terms or combination of index term

Query expansion
: Automatic procedure to add related words to a query. 
E.g: Q=(car) --> Q=(car, automobile, auto, motorcar)

Zipf's law
: Law about distribution of frequency of English words. There are two ways 
to count words: type (how many types of word) and token (how many word apperas)
