---
title: "Text corpora and lexical resources"
teaching: 60
exercises: 0
questions:
-  "What kind of text corpora can be used with Python"
-  "What kind of Python programming model suit to this task?"
-  "How to avoid repetion for the same task in Python?"
objectives:
- "Describe the reverse instructional design process."
- "Describe the purpose and implementation of formative assessments."
keypoints:
- "Accessing text corpora."
- "Conditional Distribution Frequency (CDF)"
- "Re-use the Python codes."
- "Lexical sources"
- "WordNet"
---

2. Accessing Text Corpora and Lexical Resources
===============================================

Practical work in Natural Language Processing typically uses large
bodies of linguistic data, or corpora. 

This chapter continues to present programming concepts by example, in
the context of a linguistic processing task. We will wait until later
before exploring each Python construct systematically. Don't worry if
you see an example that contains something unfamiliar; simply try it out
and see what it does, and |mdash| if you're game |mdash| modify it by
substituting some part of the code with a different text or word. This
way you will associate a task with a programming idiom, and learn the
hows and whys later.

Accessing Text Corpora
----------------------

As just mentioned, a text corpus is a large body of text. Many corpora
are designed to contain a careful balance of material in one or more
genres. We examined some small text collections in chap-introduction\_,
such as the speeches known as the US Presidential Inaugural Addresses.
This particular corpus actually contains dozens of individual texts
|mdash| one per address |mdash| but for convenience we glued them
end-to-end and treated them as a single text. chap-introduction\_ also
used various pre-defined texts that we accessed by typing
`from nltk.book import *`. However, since we want to be able to work
with other texts, this section examines a variety of text corpora. We'll
see how to select individual texts, and how to work with them.

### Gutenberg Corpus

|NLTK| includes a small selection of texts from the Project Gutenberg
electronic text archive, which contains some 25,000 free electronic
books, hosted at `http://www.gutenberg.org/`. We begin by getting the
Python interpreter to load the |NLTK| package, then ask to see
`nltk.corpus.gutenberg.fileids()`, the file identifiers in this corpus:

> &gt;&gt;&gt; import nltk &gt;&gt;&gt; nltk.corpus.gutenberg.fileids()
> \['austen-emma.txt', 'austen-persuasion.txt', 'austen-sense.txt',
> 'bible-kjv.txt', 'blake-poems.txt', 'bryant-stories.txt',
> 'burgess-busterbrown.txt', 'carroll-alice.txt', 'chesterton-ball.txt',
> 'chesterton-brown.txt', 'chesterton-thursday.txt',
> 'edgeworth-parents.txt', 'melville-moby\_dick.txt',
> 'milton-paradise.txt', 'shakespeare-caesar.txt',
> 'shakespeare-hamlet.txt', 'shakespeare-macbeth.txt',
> 'whitman-leaves.txt'\]

Let's pick out the first of these texts |mdash| *Emma* by Jane Austen
|mdash| and give it a short name, `emma`, then find out how many words
it contains:

> &gt;&gt;&gt; emma = nltk.corpus.gutenberg.words('austen-emma.txt')
> &gt;&gt;&gt; len(emma) 192427

> **note**
>
> In sec-computing-with-language-texts-and-words\_, we showed how you
> could carry out concordancing of a text such as `text1` with the
> command `text1.concordance()`. However, this assumes that you are
> using one of the nine texts obtained as a result of doing
> `from nltk.book import *`. Now that you have started examining data
> from `nltk.corpus`, as in the previous example, you have to employ the
> following pair of statements to perform concordancing and other tasks
> from sec-computing-with-language-texts-and-words\_:

When we defined `emma`, we invoked the `words()` function of the
`gutenberg` object in |NLTK|'s `corpus` package. But since it is
cumbersome to type such long names all the time, Python provides another
version of the `import` statement, as follows:

> &gt;&gt;&gt; from nltk.corpus import gutenberg &gt;&gt;&gt;
> gutenberg.fileids() \['austen-emma.txt', 'austen-persuasion.txt',
> 'austen-sense.txt', ...\] &gt;&gt;&gt; emma =
> gutenberg.words('austen-emma.txt')

Let's write a short program to display other information about each
text, by looping over all the values of `fileid` corresponding to the
`gutenberg` file identifiers listed earlier and then computing
statistics for each text. For a compact output display, we will round
each number to the nearest integer, using `round()`.

> &gt;&gt;&gt; for fileid in gutenberg.fileids(): ... num\_chars =
> len(gutenberg.raw(fileid)) \# \[\_raw-access\] ... num\_words =
> len(gutenberg.words(fileid)) ... num\_sents =
> len(gutenberg.sents(fileid)) ... num\_vocab = len(set(w.lower() for w
> in gutenberg.words(fileid))) ... print(round(num\_chars/num\_words),
> round(num\_words/num\_sents), round(num\_words/num\_vocab), fileid)
> ... 5 25 26 austen-emma.txt 5 26 17 austen-persuasion.txt 5 28 22
> austen-sense.txt 4 34 79 bible-kjv.txt 5 19 5 blake-poems.txt 4 19 14
> bryant-stories.txt 4 18 12 burgess-busterbrown.txt 4 20 13
> carroll-alice.txt 5 20 12 chesterton-ball.txt 5 23 11
> chesterton-brown.txt 5 18 11 chesterton-thursday.txt 4 21 25
> edgeworth-parents.txt 5 26 15 melville-moby\_dick.txt 5 52 11
> milton-paradise.txt 4 12 9 shakespeare-caesar.txt 4 12 8
> shakespeare-hamlet.txt 4 12 7 shakespeare-macbeth.txt 5 36 12
> whitman-leaves.txt

This program displays three statistics for each text: average word
length, average sentence length, and the number of times each vocabulary
item appears in the text on average (our lexical diversity score).
Observe that average word length appears to be a general property of
English, since it has a recurrent value of `4`. (In fact, the average
word length is really `3` not `4`, since the `num_chars` variable counts
space characters.) By contrast average sentence length and lexical
diversity appear to be characteristics of particular authors.

The previous example also showed how we can access the "raw" text of the
book raw-access\_, not split up into tokens. The `raw()` function gives
us the contents of the file without any linguistic processing. So, for
example, `len(gutenberg.raw('blake-poems.txt'))` tells us how many
*letters* occur in the text, including the spaces between words. The
`sents()` function divides the text up into its sentences, where each
sentence is a list of words:

> &gt;&gt;&gt; macbeth\_sentences =
> gutenberg.sents('shakespeare-macbeth.txt') &gt;&gt;&gt;
> macbeth\_sentences \[\['\[', 'The', 'Tragedie', 'of', 'Macbeth', 'by',
> 'William', 'Shakespeare', '1603', '\]'\], \['Actus', 'Primus', '.'\],
> ...\] &gt;&gt;&gt; macbeth\_sentences\[1116\] \['Double', ',',
> 'double', ',', 'toile', 'and', 'trouble', ';', 'Fire', 'burne', ',',
> 'and', 'Cauldron', 'bubble'\] &gt;&gt;&gt; longest\_len = max(len(s)
> for s in macbeth\_sentences) &gt;&gt;&gt; \[s for s in
> macbeth\_sentences if len(s) == longest\_len\] \[\['Doubtfull', 'it',
> 'stood', ',', 'As', 'two', 'spent', 'Swimmers', ',', 'that', 'doe',
> 'cling', 'together', ',', 'And', 'choake', 'their', 'Art', ':', 'The',
> 'mercilesse', 'Macdonwald', ...\]\]

> **note**
>
> Most |NLTK| corpus readers include a variety of access methods apart
> from `words()`, `raw()`, and `sents()`. Richer linguistic content is
> available from some corpora, such as part-of-speech tags, dialogue
> tags, syntactic trees, and so forth; we will see these in later
> chapters.

### Web and Chat Text

Although Project Gutenberg contains thousands of books, it represents
established literature. It is important to consider less formal language
as well. |NLTK|'s small collection of web text includes content from a
Firefox discussion forum, conversations overheard in New York, the movie
script of *Pirates of the Carribean*, personal advertisements, and wine
reviews:

> &gt;&gt;&gt; from nltk.corpus import webtext &gt;&gt;&gt; for fileid
> in webtext.fileids(): ... print(fileid, webtext.raw(fileid)\[:65\],
> '...') ... firefox.txt Cookie Manager: "Don't allow sites that set
> removed cookies to se... grail.txt SCENE 1: \[wind\] \[clop clop
> clop\] KING ARTHUR: Whoa there! \[clop... overheard.txt White guy: So,
> do you have any plans for this evening? Asian girl... pirates.txt
> PIRATES OF THE CARRIBEAN: DEAD MAN'S CHEST, by Ted Elliott & Terr...
> singles.txt 25 SEXY MALE, seeks attrac older single lady, for discreet
> encoun... wine.txt Lovely delicate, fragrant Rhone wine. Polished
> leather and strawb...

There is also a corpus of instant messaging chat sessions, originally
collected by the Naval Postgraduate School for research on automatic
detection of Internet predators. The corpus contains over 10,000 posts,
anonymized by replacing usernames with generic names of the form
"UserNNN", and manually edited to remove any other identifying
information. The corpus is organized into 15 files, where each file
contains several hundred posts collected on a given date, for an
age-specific chatroom (teens, 20s, 30s, 40s, plus a generic adults
chatroom). The filename contains the date, chatroom, and number of
posts; e.g., `10-19-20s_706posts.xml` contains 706 posts gathered from
the 20s chat room on 10/19/2006.

> &gt;&gt;&gt; from nltk.corpus import nps\_chat &gt;&gt;&gt; chatroom =
> nps\_chat.posts('10-19-20s\_706posts.xml') &gt;&gt;&gt;
> chatroom\[123\] \['i', 'do', "n't", 'want', 'hot', 'pics', 'of', 'a',
> 'female', ',', 'I', 'can', 'look', 'in', 'a', 'mirror', '.'\]

### Brown Corpus

The Brown Corpus was the first million-word electronic corpus of
English, created in 1961 at Brown University. This corpus contains text
from 500 sources, and the sources have been categorized by genre, such
as *news*, *editorial*, and so on. tab-brown-sources\_ gives an example
of each genre (for a complete list, see
`http://icame.uib.no/brown/bcm-los.html`).

We can access the corpus as a list of words, or a list of sentences
(where each sentence is itself just a list of words). We can optionally
specify particular categories or files to read:

> &gt;&gt;&gt; from nltk.corpus import brown &gt;&gt;&gt;
> brown.categories() \['adventure', 'belles\_lettres', 'editorial',
> 'fiction', 'government', 'hobbies', 'humor', 'learned', 'lore',
> 'mystery', 'news', 'religion', 'reviews', 'romance',
> 'science\_fiction'\] &gt;&gt;&gt; brown.words(categories='news')
> \['The', 'Fulton', 'County', 'Grand', 'Jury', 'said', ...\]
> &gt;&gt;&gt; brown.words(fileids=\['cg22'\]) \['Does', 'our',
> 'society', 'have', 'a', 'runaway', ',', ...\] &gt;&gt;&gt;
> brown.sents(categories=\['news', 'editorial', 'reviews'\]) \[\['The',
> 'Fulton', 'County'...\], \['The', 'jury', 'further'...\], ...\]

The Brown Corpus is a convenient resource for studying systematic
differences between genres, a kind of linguistic inquiry known as
stylistics. Let's compare genres in their usage of modal verbs. The
first step is to produce the counts for a particular genre. Remember to
`import nltk` before doing the following:

> &gt;&gt;&gt; from nltk.corpus import brown &gt;&gt;&gt; news\_text =
> brown.words(categories='news') &gt;&gt;&gt; fdist =
> nltk.FreqDist(w.lower() for w in news\_text) &gt;&gt;&gt; modals =
> \['can', 'could', 'may', 'might', 'must', 'will'\] &gt;&gt;&gt; for m
> in modals: ... print(m + ':', fdist\[m\], end=' ') ... can: 94 could:
> 87 may: 93 might: 38 must: 53 will: 389

> **note**
>
> We need to include `end=' '` in order for the print function to put
> its output on a single line.

> **note**
>
> |TRY| Choose a different section of the Brown Corpus, and adapt the
> previous example to count a selection of wh words, such as what, when,
> where, who, and why.

Next, we need to obtain counts for each genre of interest. We'll use
|NLTK|'s support for conditional frequency distributions. These are
presented systematically in sec-conditional-frequency-distributions\_,
where we also unpick the following code line by line. For the moment,
you can ignore the details and just concentrate on the output.

> &gt;&gt;&gt; cfd = nltk.ConditionalFreqDist( ... (genre, word) ... for
> genre in brown.categories() ... for word in
> brown.words(categories=genre)) &gt;&gt;&gt; genres = \['news',
> 'religion', 'hobbies', 'science\_fiction', 'romance', 'humor'\]
> &gt;&gt;&gt; modals = \['can', 'could', 'may', 'might', 'must',
> 'will'\] &gt;&gt;&gt; cfd.tabulate(conditions=genres, samples=modals)
> can could may might must will news 93 86 66 38 50 389 religion 82 59
> 78 12 54 71 hobbies 268 58 131 22 83 264 science\_fiction 16 49 4 12 8
> 16 romance 74 193 11 51 45 43 humor 16 30 8 8 9 13

Observe that the most frequent modal in the news genre is will, while
the most frequent modal in the romance genre is could. Would you have
predicted this? The idea that word counts might distinguish genres will
be taken up again in chap-data-intensive\_.

### Reuters Corpus

The Reuters Corpus contains 10,788 news documents totaling 1.3 million
words. The documents have been classified into 90 topics, and grouped
into two sets, called "training" and "test"; thus, the text with fileid
`'test/14826'` is a document drawn from the test set. This split is for
training and testing algorithms that automatically detect the topic of a
document, as we will see in chap-data-intensive\_.

> &gt;&gt;&gt; from nltk.corpus import reuters &gt;&gt;&gt;
> reuters.fileids() \['test/14826', 'test/14828', 'test/14829',
> 'test/14832', ...\] &gt;&gt;&gt; reuters.categories() \['acq', 'alum',
> 'barley', 'bop', 'carcass', 'castor-oil', 'cocoa', 'coconut',
> 'coconut-oil', 'coffee', 'copper', 'copra-cake', 'corn', 'cotton',
> 'cotton-oil', 'cpi', 'cpu', 'crude', 'dfl', 'dlr', ...\]

Unlike the Brown Corpus, categories in the Reuters corpus overlap with
each other, simply because a news story often covers multiple topics. We
can ask for the topics covered by one or more documents, or for the
documents included in one or more categories. For convenience, the
corpus methods accept a single fileid or a list of fileids.

> &gt;&gt;&gt; reuters.categories('training/9865') \['barley', 'corn',
> 'grain', 'wheat'\] &gt;&gt;&gt; reuters.categories(\['training/9865',
> 'training/9880'\]) \['barley', 'corn', 'grain', 'money-fx', 'wheat'\]
> &gt;&gt;&gt; reuters.fileids('barley') \['test/15618', 'test/15649',
> 'test/15676', 'test/15728', 'test/15871', ...\] &gt;&gt;&gt;
> reuters.fileids(\['barley', 'corn'\]) \['test/14832', 'test/14858',
> 'test/15033', 'test/15043', 'test/15106', 'test/15287', 'test/15341',
> 'test/15618', 'test/15648', 'test/15649', ...\]

Similarly, we can specify the words or sentences we want in terms of
files or categories. The first handful of words in each of these texts
are the titles, which by convention are stored as upper case.

> &gt;&gt;&gt; reuters.words('training/9865')\[:14\] \['FRENCH', 'FREE',
> 'MARKET', 'CEREAL', 'EXPORT', 'BIDS', 'DETAILED', 'French',
> 'operators', 'have', 'requested', 'licences', 'to', 'export'\]
> &gt;&gt;&gt; reuters.words(\['training/9865', 'training/9880'\])
> \['FRENCH', 'FREE', 'MARKET', 'CEREAL', 'EXPORT', ...\] &gt;&gt;&gt;
> reuters.words(categories='barley') \['FRENCH', 'FREE', 'MARKET',
> 'CEREAL', 'EXPORT', ...\] &gt;&gt;&gt;
> reuters.words(categories=\['barley', 'corn'\]) \['THAI', 'TRADE',
> 'DEFICIT', 'WIDENS', 'IN', 'FIRST', ...\]

### Inaugural Address Corpus

In sec-computing-with-language-texts-and-words\_, we looked at the
Inaugural Address Corpus, but treated it as a single text. The graph in
fig-inaugural\_ used "word offset" as one of the axes; this is the
numerical index of the word in the corpus, counting from the first word
of the first address. However, the corpus is actually a collection of 55
texts, one for each presidential address. An interesting property of
this collection is its time dimension:

> &gt;&gt;&gt; from nltk.corpus import inaugural &gt;&gt;&gt;
> inaugural.fileids() \['1789-Washington.txt', '1793-Washington.txt',
> '1797-Adams.txt', ...\] &gt;&gt;&gt; \[fileid\[:4\] for fileid in
> inaugural.fileids()\] \['1789', '1793', '1797', '1801', '1805',
> '1809', '1813', '1817', '1821', ...\]

Notice that the year of each text appears in its filename. To get the
year out of the filename, we extracted the first four characters, using
`fileid[:4]`.

Let's look at how the words America and citizen are used over time. The
following code converts the words in the Inaugural corpus to lowercase
using `w.lower()` lowercase-startswith\_, then checks if they start with
either of the "targets" `america` or `citizen` using `startswith()`
lowercase-startswith\_. Thus it will count words like American's and
Citizens. We'll learn about conditional frequency distributions in
sec-conditional-frequency-distributions\_; for now just consider the
output, shown in
[fig-inaugural2](..%20figure::%20../images/inaugural2.png:scale:%2018:22:20).

> &gt;&gt;&gt; cfd = nltk.ConditionalFreqDist( ... (target,
> fileid\[:4\]) ... for fileid in inaugural.fileids() ... for w in
> inaugural.words(fileid) ... for target in \['america', 'citizen'\] ...
> if w.lower().startswith(target)) \# \[\_lowercase-startswith\]
> &gt;&gt;&gt; cfd.plot()

> Plot of a Conditional Frequency Distribution: all words in the
> Inaugural Address Corpus that begin with `america` or `citizen` are
> counted; separate counts are kept for each address; these are plotted
> so that trends in usage over time can be observed; counts are not
> normalized for document length.

### Annotated Text Corpora

Many text corpora contain linguistic annotations, representing POS tags,
named entities, syntactic structures, semantic roles, and so forth. NLTK
provides convenient ways to access several of these corpora, and has
data packages containing corpora and corpus samples, freely downloadable
for use in teaching and research. tab-corpora\_ lists some of the
corpora. For information about downloading them, see
`http://nltk.org/data`. For more examples of how to access |NLTK|
corpora, please consult the Corpus HOWTO at |NLTK-HOWTO-URL|.

### Corpora in Other Languages

NLTK comes with corpora for many languages, though in some cases you
will need to learn how to manipulate character encodings in Python
before using these corpora (see sec-unicode\_).

> &gt;&gt;&gt; nltk.corpus.cess\_esp.words() \['El', 'grupo', 'estatal',
> 'Electricitxe9\_de\_France', ...\] &gt;&gt;&gt;
> nltk.corpus.floresta.words() \['Um', 'revivalismo', 'refrescante',
> 'O', '7\_e\_Meio', ...\] &gt;&gt;&gt;
> nltk.corpus.indian.words('hindi.pos') \['पूर्ण', 'प्रतिबंध', 'हटाओ',
> ':', 'इराक', 'संयुक्त', ...\] &gt;&gt;&gt; nltk.corpus.udhr.fileids()
> \['Abkhaz-Cyrillic+Abkh', 'Abkhaz-UTF8', 'Achehnese-Latin1',
> 'Achuar-Shiwiar-Latin1', 'Adja-UTF8', 'Afaan\_Oromo\_Oromiffa-Latin1',
> 'Afrikaans-Latin1', 'Aguaruna-Latin1', 'Akuapem\_Twi-UTF8',
> 'Albanian\_Shqip-Latin1', 'Amahuaca', 'Amahuaca-Latin1', ...\]
> &gt;&gt;&gt; nltk.corpus.udhr.words('Javanese-Latin1')\[11:\]
> \['Saben', 'umat', 'manungsa', 'lair', 'kanthi', 'hak', ...\]

The last of these corpora, `udhr`, contains the Universal Declaration of
Human Rights in over 300 languages. The fileids for this corpus include
information about the character encoding used in the file, such as
`UTF8` or `Latin1`. Let's use a conditional frequency distribution to
examine the differences in word lengths for a selection of languages
included in the `udhr` corpus. The output is shown in
[fig-word-len-dist](..%20figure::%20../images/word-len-dist.png:scale:%2025)
(run the program yourself to see a color plot). Note that `True` and
`False` are Python's built-in boolean values.

> &gt;&gt;&gt; from nltk.corpus import udhr &gt;&gt;&gt; languages =
> \['Chickasaw', 'English', 'German\_Deutsch', ...
> 'Greenlandic\_Inuktikut', 'Hungarian\_Magyar', 'Ibibio\_Efik'\]
> &gt;&gt;&gt; cfd = nltk.ConditionalFreqDist( ... (lang, len(word)) ...
> for lang in languages ... for word in udhr.words(lang + '-Latin1'))
> &gt;&gt;&gt; cfd.plot(cumulative=True)

> Cumulative Word Length Distributions: Six translations of the
> Universal Declaration of Human Rights are processed; this graph shows
> that words having 5 or fewer letters account for about 80% of Ibibio
> text, 60% of German text, and 25% of Inuktitut text.

> **note**
>
> |TRY| Pick a language of interest in `udhr.fileids()`, and define a
> variable `raw_text = udhr.raw(`*Language-Latin1*`)`. Now plot a
> frequency distribution of the letters of the text using
> `nltk.FreqDist(raw_text).plot()`.

Unfortunately, for many languages, substantial corpora are not yet
available. Often there is insufficient government or industrial support
for developing language resources, and individual efforts are piecemeal
and hard to discover or re-use. Some languages have no established
writing system, or are endangered. (See sec-further-reading-corpora\_
for suggestions on how to locate language resources.)

### Text Corpus Structure

We have seen a variety of corpus structures so far; these are summarized
in
[fig-text-corpus-structure](..%20figure::%20../images/text-corpus-structure.png:scale:%20120:140:140).
The simplest kind lacks any structure: it is just a collection of texts.
Often, texts are grouped into categories that might correspond to genre,
source, author, language, etc. Sometimes these categories overlap,
notably in the case of topical categories as a text can be relevant to
more than one topic. Occasionally, text collections have temporal
structure, news collections being the most common example.

> Common Structures for Text Corpora: The simplest kind of corpus is a
> collection of isolated texts with no particular organization; some
> corpora are structured into categories like genre (Brown Corpus); some
> categorizations overlap, such as topic categories (Reuters Corpus);
> other corpora represent language use over time (Inaugural Address
> Corpus).

|NLTK|'s corpus readers support efficient access to a variety of
corpora, and can be used to work with new corpora. tab-corpus\_ lists
functionality provided by the corpus readers. We illustrate the
difference between some of the corpus access methods below:

> &gt;&gt;&gt; raw = gutenberg.raw("burgess-busterbrown.txt")
> &gt;&gt;&gt; raw\[1:20\] 'The Adventures of B' &gt;&gt;&gt; words =
> gutenberg.words("burgess-busterbrown.txt") &gt;&gt;&gt; words\[1:20\]
> \['The', 'Adventures', 'of', 'Buster', 'Bear', 'by', 'Thornton', 'W',
> '.', 'Burgess', '1920', '\]', 'I', 'BUSTER', 'BEAR', 'GOES',
> 'FISHING', 'Buster', 'Bear'\] &gt;&gt;&gt; sents =
> gutenberg.sents("burgess-busterbrown.txt") &gt;&gt;&gt; sents\[1:20\]
> \[\['I'\], \['BUSTER', 'BEAR', 'GOES', 'FISHING'\], \['Buster',
> 'Bear', 'yawned', 'as', 'he', 'lay', 'on', 'his', 'comfortable',
> 'bed', 'of', 'leaves', 'and', 'watched', 'the', 'first', 'early',
> 'morning', 'sunbeams', 'creeping', 'through', ...\], ...\]

### Loading your own Corpus

If you have your own collection of text files that you would like to
access using the above methods, you can easily load them with the help
of |NLTK|'s `PlaintextCorpusReader`. Check the location of your files on
your file system; in the following example, we have taken this to be the
directory `/usr/share/dict`. Whatever the location, set this to be the
value of `corpus_root` corpus-root-dict\_. The second parameter of the
`PlaintextCorpusReader` initializer corpus-reader\_ can be a list of
fileids, like `['a.txt', 'test/b.txt']`, or a pattern that matches all
fileids, like `'[abc]/.*\.txt'` (see
sec-regular-expressions-word-patterns\_ for information about regular
expressions).

> &gt;&gt;&gt; from nltk.corpus import PlaintextCorpusReader
> &gt;&gt;&gt; corpus\_root = '/usr/share/dict' \#
> \[\_corpus-root-dict\] &gt;&gt;&gt; wordlists =
> PlaintextCorpusReader(corpus\_root, '.\*') \# \[\_corpus-reader\]
> &gt;&gt;&gt; wordlists.fileids() \['README', 'connectives',
> 'propernames', 'web2', 'web2a', 'words'\] &gt;&gt;&gt;
> wordlists.words('connectives') \['the', 'of', 'and', 'to', 'a', 'in',
> 'that', 'is', ...\]

As another example, suppose you have your own local copy of Penn
Treebank (release 3), in `C:\corpora`. We can use the
`BracketParseCorpusReader` to access this corpus. We specify the
`corpus_root` to be the location of the parsed Wall Street Journal
component of the corpus corpus-root-treebank\_, and give a
`file_pattern` that matches the files contained within its subfolders
file-pattern\_ (using forward slashes).

Conditional Frequency Distributions
-----------------------------------

We introduced frequency distributions in
sec-computing-with-language-simple-statistics\_. We saw that given some
list `mylist` of words or other items, `FreqDist(mylist)` would compute
the number of occurrences of each item in the list. Here we will
generalize this idea.

When the texts of a corpus are divided into several categories, by
genre, topic, author, etc, we can maintain separate frequency
distributions for each category. This will allow us to study systematic
differences between the categories. In the previous section we achieved
this using |NLTK|'s `ConditionalFreqDist` data type. A
conditional frequency distribution is a collection of frequency
distributions, each one for a different "condition". The condition will
often be the category of the text.
[fig-tally2](..%20figure::%20../images/tally2.png:scale:%2070:100:80)
depicts a fragment of a conditional frequency distribution having just
two conditions, one for news text and one for romance text.

> Counting Words Appearing in a Text Collection (a conditional frequency
> distribution)

### Conditions and Events

A frequency distribution counts observable events, such as the
appearance of words in a text. A conditional frequency distribution
needs to pair each event with a condition. So instead of processing a
sequence of words seq-words\_, we have to process a sequence of pairs
seq-pairs\_:

Each pair has the form `(condition, event)`. If we were processing the
entire Brown Corpus by genre there would be 15 conditions (one per
genre), and 1,161,192 events (one per word).

### Counting Words by Genre

In sec-extracting-text-from-corpora\_ we saw a conditional frequency
distribution where the condition was the section of the Brown Corpus,
and for each condition we counted words. Whereas `FreqDist()` takes a
simple list as input, `ConditionalFreqDist()` takes a list of pairs.

> &gt;&gt;&gt; from nltk.corpus import brown &gt;&gt;&gt; cfd =
> nltk.ConditionalFreqDist( ... (genre, word) ... for genre in
> brown.categories() ... for word in brown.words(categories=genre))

Let's break this down, and look at just two genres, news and romance.
For each genre each-genre\_, we loop over every word in the genre
each-word\_, producing pairs consisting of the genre and the word
genre-word-pairs\_:

> &gt;&gt;&gt; genre\_word = \[(genre, word) \# \[\_genre-word-pairs\]
> ... for genre in \['news', 'romance'\] \# \[\_each-genre\] ... for
> word in brown.words(categories=genre)\] \# \[\_each-word\]
> &gt;&gt;&gt; len(genre\_word) 170576

So, as we can see below, pairs at the beginning of the list `genre_word`
will be of the form (`'news'`, *word*) start-genre\_, while those at the
end will be of the form (`'romance'`, *word*) end-genre\_.

> &gt;&gt;&gt; genre\_word\[:4\] \[('news', 'The'), ('news', 'Fulton'),
> ('news', 'County'), ('news', 'Grand')\] \# \[\_start-genre\]
> &gt;&gt;&gt; genre\_word\[-4:\] \[('romance', 'afraid'), ('romance',
> 'not'), ('romance', "''"), ('romance', '.')\] \# \[\_end-genre\]

We can now use this list of pairs to create a `ConditionalFreqDist`, and
save it in a variable `cfd`. As usual, we can type the name of the
variable to inspect it inspect-cfd\_, and verify it has two conditions
conditions-cfd\_:

> &gt;&gt;&gt; cfd = nltk.ConditionalFreqDist(genre\_word) &gt;&gt;&gt;
> cfd \# \[\_inspect-cfd\] &lt;ConditionalFreqDist with 2 conditions&gt;
> &gt;&gt;&gt; cfd.conditions() \['news', 'romance'\] \#
> \[\_conditions-cfd\]

Let's access the two conditions, and satisfy ourselves that each is just
a frequency distribution:

> &gt;&gt;&gt; print(cfd\['news'\]) &lt;FreqDist with 14394 samples and
> 100554 outcomes&gt; &gt;&gt;&gt; print(cfd\['romance'\]) &lt;FreqDist
> with 8452 samples and 70022 outcomes&gt; &gt;&gt;&gt;
> cfd\['romance'\].most\_common(20) \[(',', 3899), ('.', 3736), ('the',
> 2758), ('and', 1776), ('to', 1502), ('a', 1335), ('of', 1186),
> ('\`\`', 1045), ("''", 1044), ('was', 993), ('I', 951), ('in', 875),
> ('he', 702), ('had', 692), ('?', 690), ('her', 651), ('that', 583),
> ('it', 573), ('his', 559), ('she', 496)\] &gt;&gt;&gt;
> cfd\['romance'\]\['could'\] 193

### Plotting and Tabulating Distributions

Apart from combining two or more frequency distributions, and being easy
to initialize, a `ConditionalFreqDist` provides some useful methods for
tabulation and plotting.

The plot in
[fig-inaugural2](..%20figure::%20../images/inaugural2.png:scale:%2018:22:20)
was based on a conditional frequency distribution reproduced in the code
below. The condition is either of the words america or citizen
america-citizen\_, and the counts being plotted are the number of times
the word occured in a particular speech. It exploits the fact that the
filename for each speech, e.g., `1865-Lincoln.txt` contains the year as
the first four characters first-four-chars\_. This code generates the
pair `('america', '1865')` for every instance of a word whose lowercased
form starts with america |mdash| such as Americans |mdash| in the file
`1865-Lincoln.txt`.

> &gt;&gt;&gt; from nltk.corpus import inaugural &gt;&gt;&gt; cfd =
> nltk.ConditionalFreqDist( ... (target, fileid\[:4\]) \#
> \[\_first-four-chars\] ... for fileid in inaugural.fileids() ... for w
> in inaugural.words(fileid) ... for target in \['america', 'citizen'\]
> \# \[\_america-citizen\] ... if w.lower().startswith(target))

The plot in
[fig-word-len-dist](..%20figure::%20../images/word-len-dist.png:scale:%2025)
was also based on a conditional frequency distribution, reproduced
below. This time, the condition is the name of the language and the
counts being plotted are derived from word lengths lang-len-word\_. It
exploits the fact that the filename for each language is the language
name followed by `'-Latin1'` (the character encoding).

> &gt;&gt;&gt; from nltk.corpus import udhr &gt;&gt;&gt; languages =
> \['Chickasaw', 'English', 'German\_Deutsch', ...
> 'Greenlandic\_Inuktikut', 'Hungarian\_Magyar', 'Ibibio\_Efik'\]
> &gt;&gt;&gt; cfd = nltk.ConditionalFreqDist( ... (lang, len(word)) \#
> \[\_lang-len-word\] ... for lang in languages ... for word in
> udhr.words(lang + '-Latin1'))

In the `plot()` and `tabulate()` methods, we can optionally specify
which conditions to display with a `conditions=` parameter. When we omit
it, we get all the conditions. Similarly, we can limit the samples to
display with a `samples=` parameter. This makes it possible to load a
large quantity of data into a conditional frequency distribution, and
then to explore it by plotting or tabulating selected conditions and
samples. It also gives us full control over the order of conditions and
samples in any displays. For example, we can tabulate the cumulative
frequency data just for two languages, and for words less than 10
characters long, as shown below. We interpret the last cell on the top
row to mean that 1,638 words of the English text have 9 or fewer
letters.

> &gt;&gt;&gt; cfd.tabulate(conditions=\['English', 'German\_Deutsch'\],
> ... samples=range(10), cumulative=True) 0 1 2 3 4 5 6 7 8 9 English 0
> 185 525 883 997 1166 1283 1440 1558 1638 German\_Deutsch 0 171 263 614
> 717 894 1013 1110 1213 1275

> **note**
>
> |TRY| Working with the news and romance genres from the Brown Corpus,
> find out which days of the week are most newsworthy, and which are
> most romantic. Define a variable called `days` containing a list of
> days of the week, i.e. `['Monday', ...]`. Now tabulate the counts for
> these words using `cfd.tabulate(samples=days)`. Now try the same thing
> using `plot` in place of `tabulate`. You may control the output order
> of days with the help of an extra parameter:
> `samples=['Monday', ...]`.

You may have noticed that the multi-line expressions we have been using
with conditional frequency distributions look like list comprehensions,
but without the brackets. In general, when we use a list comprehension
as a parameter to a function, like `set([w.lower() for w in t])`, we are
permitted to omit the square brackets and just write:
`set(w.lower() for w in t)`. (See the discussion of "generator
expressions" in sec-sequences\_ for more about this.)

### Generating Random Text with Bigrams

We can use a conditional frequency distribution to create a table of
bigrams (word pairs). (We introducted bigrams in
sec-computing-with-language-simple-statistics\_.) The `bigrams()`
function takes a list of words and builds a list of consecutive word
pairs. Remember that, in order to see the result and not a cryptic
"generator object", we need to use the `list()` function:

> &gt;&gt;&gt; sent = \['In', 'the', 'beginning', 'God', 'created',
> 'the', 'heaven', ... 'and', 'the', 'earth', '.'\] &gt;&gt;&gt;
> list(nltk.bigrams(sent)) \[('In', 'the'), ('the', 'beginning'),
> ('beginning', 'God'), ('God', 'created'), ('created', 'the'), ('the',
> 'heaven'), ('heaven', 'and'), ('and', 'the'), ('the', 'earth'),
> ('earth', '.')\]

In code-random-text\_, we treat each word as a condition, and for each
one we effectively create a frequency distribution over the following
words. The function `generate_model()` contains a simple loop to
generate text. When we call the function, we choose a word (such as
`'living'`) as our initial context, then once inside the loop, we print
the current value of the variable `word`, and reset `word` to be the
most likely token in that context (using `max()`); next time through the
loop, we use that word as our new context. As you can see by inspecting
the output, this simple approach to text generation tends to get stuck
in loops; another method would be to randomly choose the next word from
among the available words.

> def generate\_model(cfdist, word, num=15):
>
> :   
>
>     for i in range(num):
>
>     :   print(word, end=' ') word = cfdist\[word\].max()
>
> text = nltk.corpus.genesis.words('english-kjv.txt') bigrams =
> nltk.bigrams(text) cfd = nltk.ConditionalFreqDist(bigrams) \#
> \[\_bigram-condition\]
>
> &gt;&gt;&gt; cfd\['living'\] FreqDist({'creature': 7, 'thing': 4,
> 'substance': 2, ',': 1, '.': 1, 'soul': 1}) &gt;&gt;&gt;
> generate\_model(cfd, 'living') living creature that he said , and the
> land of the land of the land

Conditional frequency distributions are a useful data structure for many
|NLP| tasks. Their commonly-used methods are summarized in
tab-conditionalfreqdist\_.

More Python: Reusing Code
-------------------------

By this time you've probably typed and retyped a lot of code in the
Python interactive interpreter. If you mess up when retyping a complex
example you have to enter it again. Using the arrow keys to access and
modify previous commands is helpful but only goes so far. In this
section we see two important ways to reuse code: text editors and Python
functions.

### Creating Programs with a Text Editor

The Python interactive interpreter performs your instructions as soon as
you type them. Often, it is better to compose a multi-line program using
a text editor, then ask Python to run the whole program at once. Using
|IDLE|, you can do this by going to the `File` menu and opening a new
window. Try this now, and enter the following one-line program:

    print('Monty Python')

Save this program in a file called `monty.py`, then go to the `Run`
menu, and select the command `Run Module`. (We'll learn what modules are
shortly.) The result in the main |IDLE| window should look like this:

You can also type `from monty import *` and it will do the same thing.

From now on, you have a choice of using the interactive interpreter or a
text editor to create your programs. It is often convenient to test your
ideas using the interpreter, revising a line of code until it does what
you expect. Once you're ready, you can paste the code (minus any `>>>`
or `...` prompts) into the text editor, continue to expand it, and
finally save the program in a file so that you don't have to type it in
again later. Give the file a short but descriptive name, using all
lowercase letters and separating words with underscore, and using the
`.py` filename extension, e.g., `monty_python.py`.

> **note**
>
> |IMPORTANT| Our inline code examples include the `>>>` and `...`
> prompts as if we are interacting directly with the interpreter. As
> they get more complicated, you should instead type them into the
> editor, without the prompts, and run them from the editor as shown
> above. When we provide longer programs in this book, we will leave out
> the prompts to remind you to type them into a file rather than using
> the interpreter. You can see this already in code-random-text\_ above.
> Note that it still includes a couple of lines with the Python prompt;
> this is the interactive part of the task where you inspect some data
> and invoke a function. Remember that all code samples like
> code-random-text\_ are downloadable from |NLTK-URL|.

### Functions

Suppose that you work on analyzing text that involves different forms of
the same word, and that part of your program needs to work out the
plural form of a given singular noun. Suppose it needs to do this work
in two places, once when it is processing some texts, and again when it
is processing user input.

Rather than repeating the same code several times over, it is more
efficient and reliable to localize this work inside a function. A
function is just a named block of code that performs some well-defined
task, as we saw in sec-computing-with-language-texts-and-words\_. A
function is usually defined to take some inputs, using special variables
known as parameters, and it may produce a result, also known as a
return value. We define a function using the keyword `def` followed by
the function name and any input parameters, followed by the body of the
function. Here's the function we saw in
sec-computing-with-language-texts-and-words\_ (including the `import`
statement that is needed for Python 2, in order to make division behave
as expected):

> &gt;&gt;&gt; from \_\_future\_\_ import division &gt;&gt;&gt; def
> lexical\_diversity(text): ... return len(text) / len(set(text))

We use the keyword `return` to indicate the value that is produced as
output by the function. In the above example, all the work of the
function is done in the `return` statement. Here's an equivalent
definition which does the same work using multiple lines of code. We'll
change the parameter name from `text` to `my_text_data` to remind you
that this is an arbitrary choice:

> &gt;&gt;&gt; def lexical\_diversity(my\_text\_data): ... word\_count =
> len(my\_text\_data) ... vocab\_size = len(set(my\_text\_data)) ...
> diversity\_score = vocab\_size / word\_count ... return
> diversity\_score

Notice that we've created some new variables inside the body of the
function. These are local variables and are not accessible outside the
function. So now we have defined a function with the name
`lexical_diversity`. But just defining it won't produce any output!
Functions do nothing until they are "called" (or "invoked"):

> &gt;&gt;&gt; from nltk.corpus import genesis &gt;&gt;&gt; kjv =
> genesis.words('english-kjv.txt') &gt;&gt;&gt; lexical\_diversity(kjv)
> 0.06230453042623537

Let's return to our earlier scenario, and actually define a simple
function to work out English plurals. The function `plural()` in
code-plural\_ takes a singular noun and generates a plural form, though
it is not always correct. (We'll discuss functions at greater length in
sec-functions\_.)

> def plural(word):
>
> :   
>
>     if word.endswith('y'):
>
>     :   return word\[:-1\] + 'ies'
>
>     elif word\[-1\] in 'sx' or word\[-2:\] in \['sh', 'ch'\]:
>
>     :   return word + 'es'
>
>     elif word.endswith('an'):
>
>     :   return word\[:-2\] + 'en'
>
>     else:
>
>     :   return word + 's'
>
> &gt;&gt;&gt; plural('fairy') 'fairies' &gt;&gt;&gt; plural('woman')
> 'women'

The `endswith()` function is always associated with a string object
(e.g., `word` in code-plural\_). To call such functions, we give the
name of the object, a period, and then the name of the function. These
functions are usually known as methods.

### Modules

Over time you will find that you create a variety of useful little text
processing functions, and you end up copying them from old programs to
new ones. Which file contains the latest version of the function you
want to use? It makes life a lot easier if you can collect your work
into a single place, and access previously defined functions without
making copies.

To do this, save your function(s) in a file called (say) `text_proc.py`.
Now, you can access your work simply by importing it from the file:

Our plural function obviously has an error, since the plural of fan is
fans. Instead of typing in a new version of the function, we can simply
edit the existing one. Thus, at every stage, there is only one version
of our plural function, and no confusion about which one is being used.

A collection of variable and function definitions in a file is called a
Python module. A collection of related modules is called a package.
|NLTK|'s code for processing the Brown Corpus is an example of a module,
and its collection of code for processing all the different corpora is
an example of a package. |NLTK| itself is a set of packages, sometimes
called a library.

> **caution**
>
> If you are creating a file to contain some of your Python code, do
> *not* name your file `nltk.py`: it may get imported in place of the
> "real" NLTK package. When it imports modules, Python first looks in
> the current directory (folder).

Lexical Resources
-----------------

A lexicon, or lexical resource, is a collection of words and/or phrases
along with associated information such as part of speech and sense
definitions. Lexical resources are secondary to texts, and are usually
created and enriched with the help of texts. For example, if we have
defined a text `my_text`, then `vocab = sorted(set(my_text))` builds the
vocabulary of `my_text`, while `word_freq = FreqDist(my_text)` counts
the frequency of each word in the text. Both of `vocab` and `word_freq`
are simple lexical resources. Similarly, a concordance like the one we
saw in sec-computing-with-language-texts-and-words\_ gives us
information about word usage that might help in the preparation of a
dictionary. Standard terminology for lexicons is illustrated in
[fig-lexicon](..%20figure::%20../images/lexicon.png:scale:%20180). A
lexical entry consists of a headword (also known as a lemma) along with
additional information such as the part of speech and the sense
definition. Two distinct words having the same spelling are called
homonyms.

> Lexicon Terminology: lexical entries for two lemmas having the same
> spelling (homonyms), providing part of speech and gloss information.

The simplest kind of lexicon is nothing more than a sorted list of
words. Sophisticated lexicons include complex structure within and
across the individual entries. In this section we'll look at some
lexical resources included with |NLTK|.

### Wordlist Corpora

|NLTK| includes some corpora that are nothing more than wordlists. The
Words Corpus is the `/usr/share/dict/words` file from Unix, used by some
spell checkers. We can use it to find unusual or mis-spelt words in a
text corpus, as shown in code-unusual\_.

> def unusual\_words(text):
>
> :   text\_vocab = set(w.lower() for w in text if w.isalpha())
>     english\_vocab = set(w.lower() for w in nltk.corpus.words.words())
>     unusual = text\_vocab - english\_vocab return sorted(unusual)
>
> &gt;&gt;&gt;
> unusual\_words(nltk.corpus.gutenberg.words('austen-sense.txt'))
> \['abbeyland', 'abhorred', 'abilities', 'abounded', 'abridgement',
> 'abused', 'abuses', 'accents', 'accepting', 'accommodations',
> 'accompanied', 'accounted', 'accounts', 'accustomary', 'aches',
> 'acknowledging', 'acknowledgment', 'acknowledgments', ...\]
> &gt;&gt;&gt; unusual\_words(nltk.corpus.nps\_chat.words())
> \['aaaaaaaaaaaaaaaaa', 'aaahhhh', 'abortions', 'abou', 'abourted',
> 'abs', 'ack', 'acros', 'actualy', 'adams', 'adds', 'adduser',
> 'adjusts', 'adoted', 'adreniline', 'ads', 'adults', 'afe', 'affairs',
> 'affari', 'affects', 'afk', 'agaibn', 'ages', ...\]

There is also a corpus of stopwords, that is, high-frequency words like
the, to and also that we sometimes want to filter out of a document
before further processing. Stopwords usually have little lexical
content, and their presence in a text fails to distinguish it from other
texts.

> &gt;&gt;&gt; from nltk.corpus import stopwords &gt;&gt;&gt;
> stopwords.words('english') \['i', 'me', 'my', 'myself', 'we', 'our',
> 'ours', 'ourselves', 'you', 'your', 'yours', 'yourself', 'yourselves',
> 'he', 'him', 'his', 'himself', 'she', 'her', 'hers', 'herself', 'it',
> 'its', 'itself', 'they', 'them', 'their', 'theirs', 'themselves',
> 'what', 'which', 'who', 'whom', 'this', 'that', 'these', 'those',
> 'am', 'is', 'are', 'was', 'were', 'be', 'been', 'being', 'have',
> 'has', 'had', 'having', 'do', 'does', 'did', 'doing', 'a', 'an',
> 'the', 'and', 'but', 'if', 'or', 'because', 'as', 'until', 'while',
> 'of', 'at', 'by', 'for', 'with', 'about', 'against', 'between',
> 'into', 'through', 'during', 'before', 'after', 'above', 'below',
> 'to', 'from', 'up', 'down', 'in', 'out', 'on', 'off', 'over', 'under',
> 'again', 'further', 'then', 'once', 'here', 'there', 'when', 'where',
> 'why', 'how', 'all', 'any', 'both', 'each', 'few', 'more', 'most',
> 'other', 'some', 'such', 'no', 'nor', 'not', 'only', 'own', 'same',
> 'so', 'than', 'too', 'very', 's', 't', 'can', 'will', 'just', 'don',
> 'should', 'now'\]

Let's define a function to compute what fraction of words in a text are
*not* in the stopwords list:

> &gt;&gt;&gt; def content\_fraction(text): ... stopwords =
> nltk.corpus.stopwords.words('english') ... content = \[w for w in text
> if w.lower() not in stopwords\] ... return len(content) / len(text)
> ... &gt;&gt;&gt; content\_fraction(nltk.corpus.reuters.words())
> 0.7364374824583169

Thus, with the help of stopwords we filter out over a quarter of the
words of the text. Notice that we've combined two different kinds of
corpus here, using a lexical resource to filter the content of a text
corpus.

> A Word Puzzle: a grid of randomly chosen letters with rules for
> creating words out of the letters; this puzzle is known as "Target."

A wordlist is useful for solving word puzzles, such as the one in
[fig-target](..%20figure::%20../images/target.png:scale:%2030:150:35).
Our program iterates through every word and, for each one, checks
whether it meets the conditions. It is easy to check obligatory letter
obligatory-letter\_ and length constraints length-constraint\_ (and
we'll only look for words with six or more letters here). It is trickier
to check that candidate solutions only use combinations of the supplied
letters, especially since some of the supplied letters appear twice
(here, the letter v). The `FreqDist` comparison method
freqdist-compare\_ permits us to check that the frequency of each
*letter* in the candidate word is less than or equal to the frequency of
the corresponding letter in the puzzle.

> &gt;&gt;&gt; puzzle\_letters = nltk.FreqDist('egivrvonl') &gt;&gt;&gt;
> obligatory = 'r' &gt;&gt;&gt; wordlist = nltk.corpus.words.words()
> &gt;&gt;&gt; \[w for w in wordlist if len(w) &gt;= 6 \#
> \[\_length-constraint\] ... and obligatory in w \#
> \[\_obligatory-letter\] ... and nltk.FreqDist(w) &lt;=
> puzzle\_letters\] \# \[\_freqdist-compare\] \['glover', 'gorlin',
> 'govern', 'grovel', 'ignore', 'involver', 'lienor', 'linger',
> 'longer', 'lovering', 'noiler', 'overling', 'region', 'renvoi',
> 'revolving', 'ringle', 'roving', 'violer', 'virole'\]

One more wordlist corpus is the Names corpus, containing 8,000 first
names categorized by gender. The male and female names are stored in
separate files. Let's find names which appear in both files, i.e. names
that are ambiguous for gender:

> &gt;&gt;&gt; names = nltk.corpus.names &gt;&gt;&gt; names.fileids()
> \['female.txt', 'male.txt'\] &gt;&gt;&gt; male\_names =
> names.words('male.txt') &gt;&gt;&gt; female\_names =
> names.words('female.txt') &gt;&gt;&gt; \[w for w in male\_names if w
> in female\_names\] \['Abbey', 'Abbie', 'Abby', 'Addie', 'Adrian',
> 'Adrien', 'Ajay', 'Alex', 'Alexis', 'Alfie', 'Ali', 'Alix', 'Allie',
> 'Allyn', 'Andie', 'Andrea', 'Andy', 'Angel', 'Angie', 'Ariel',
> 'Ashley', 'Aubrey', 'Augustine', 'Austin', 'Averil', ...\]

It is well known that names ending in the letter a are almost always
female. We can see this and some other patterns in the graph in
[fig-cfd-gender](..%20figure::%20../images/cfd-gender.png:scale:%2025),
produced by the following code. Remember that `name[-1]` is the last
letter of `name`.

> &gt;&gt;&gt; cfd = nltk.ConditionalFreqDist( ... (fileid, name\[-1\])
> ... for fileid in names.fileids() ... for name in names.words(fileid))
> &gt;&gt;&gt; cfd.plot()

> Conditional Frequency Distribution: this plot shows the number of
> female and male names ending with each letter of the alphabet; most
> names ending with a, e or i are female; names ending in h and l are
> equally likely to be male or female; names ending in k, o, r, s, and t
> are likely to be male.

### A Pronouncing Dictionary

A slightly richer kind of lexical resource is a table (or spreadsheet),
containing a word plus some properties in each row. |NLTK| includes the
CMU Pronouncing Dictionary for US English, which was designed for use by
speech synthesizers.

> &gt;&gt;&gt; entries = nltk.corpus.cmudict.entries() &gt;&gt;&gt;
> len(entries) 133737 &gt;&gt;&gt; for entry in entries\[42371:42379\]:
> ... print(entry) ... ('fir', \['F', 'ER1'\]) ('fire', \['F', 'AY1',
> 'ER0'\]) ('fire', \['F', 'AY1', 'R'\]) ('firearm', \['F', 'AY1',
> 'ER0', 'AA2', 'R', 'M'\]) ('firearm', \['F', 'AY1', 'R', 'AA2', 'R',
> 'M'\]) ('firearms', \['F', 'AY1', 'ER0', 'AA2', 'R', 'M', 'Z'\])
> ('firearms', \['F', 'AY1', 'R', 'AA2', 'R', 'M', 'Z'\]) ('fireball',
> \['F', 'AY1', 'ER0', 'B', 'AO2', 'L'\])

For each word, this lexicon provides a list of phonetic codes |mdash|
distinct labels for each contrastive sound |mdash| known as phones.
Observe that fire has two pronunciations (in US English): the
one-syllable `F AY1 R`, and the two-syllable `F AY1 ER0`. The symbols in
the CMU Pronouncing Dictionary are from the *Arpabet*, described in more
detail at `http://en.wikipedia.org/wiki/Arpabet`

Each entry consists of two parts, and we can process these individually
using a more complex version of the `for` statement. Instead of writing
`for entry in entries:`, we replace `entry` with *two* variable names,
`word, pron` word-pron\_. Now, each time through the loop, `word` is
assigned the first part of the entry, and `pron` is assigned the second
part of the entry:

> &gt;&gt;&gt; for word, pron in entries: \# \[\_word-pron\] ... if
> len(pron) == 3: \# \[\_len-pron-three\] ... ph1, ph2, ph3 = pron \#
> \[\_tuple-assignment\] ... if ph1 == 'P' and ph3 == 'T': ...
> print(word, ph2, end=' ') ... pait EY1 pat AE1 pate EY1 patt AE1 peart
> ER1 peat IY1 peet IY1 peete IY1 pert ER1 pet EH1 pete IY1 pett EH1
> piet IY1 piette IY1 pit IH1 pitt IH1 pot AA1 pote OW1 pott AA1 pout
> AW1 puett UW1 purt ER1 put UH1 putt AH1

The above program scans the lexicon looking for entries whose
pronunciation consists of three phones len-pron-three\_. If the
condition is true, it assigns the contents of `pron` to three new
variables `ph1`, `ph2` and `ph3`. Notice the unusual form of the
statement which does that work tuple-assignment\_.

Here's another example of the same `for` statement, this time used
inside a list comprehension. This program finds all words whose
pronunciation ends with a syllable sounding like nicks. You could use
this method to find rhyming words.

> &gt;&gt;&gt; syllable = \['N', 'IH0', 'K', 'S'\] &gt;&gt;&gt; \[word
> for word, pron in entries if pron\[-4:\] == syllable\] \["atlantic's",
> 'audiotronics', 'avionics', 'beatniks', 'calisthenics', 'centronics',
> 'chamonix', 'chetniks', "clinic's", 'clinics', 'conics', 'conics',
> 'cryogenics', 'cynics', 'diasonics', "dominic's", 'ebonics',
> 'electronics', "electronics'", ...\]

Notice that the one pronunciation is spelt in several ways: nics, niks,
nix, even ntic's with a silent t, for the word atlantic's. Let's look
for some other mismatches between pronunciation and writing. Can you
summarize the purpose of the following examples and explain how they
work?

> &gt;&gt;&gt; \[w for w, pron in entries if pron\[-1\] == 'M' and
> w\[-1\] == 'n'\] \['autumn', 'column', 'condemn', 'damn', 'goddamn',
> 'hymn', 'solemn'\] &gt;&gt;&gt; sorted(set(w\[:2\] for w, pron in
> entries if pron\[0\] == 'N' and w\[0\] != 'n')) \['gn', 'kn', 'mn',
> 'pn'\]

The phones contain digits to represent primary stress (`1`), secondary
stress (`2`) and no stress (`0`). As our final example, we define a
function to extract the stress digits and then scan our lexicon to find
words having a particular stress pattern.

> &gt;&gt;&gt; def stress(pron): ... return \[char for phone in pron for
> char in phone if char.isdigit()\] &gt;&gt;&gt; \[w for w, pron in
> entries if stress(pron) == \['0', '1', '0', '2', '0'\]\]
> \['abbreviated', 'abbreviated', 'abbreviating', 'accelerated',
> 'accelerating', 'accelerator', 'accelerators', 'accentuated',
> 'accentuating', 'accommodated', 'accommodating', 'accommodative',
> 'accumulated', 'accumulating', 'accumulative', ...\] &gt;&gt;&gt; \[w
> for w, pron in entries if stress(pron) == \['0', '2', '0', '1',
> '0'\]\] \['abbreviation', 'abbreviations', 'abomination',
> 'abortifacient', 'abortifacients', 'academicians', 'accommodation',
> 'accommodations', 'accreditation', 'accreditations', 'accumulation',
> 'accumulations', 'acetylcholine', 'acetylcholine', 'adjudication',
> ...\]

> **note**
>
> A subtlety of the above program is that our user-defined function
> `stress()` is invoked inside the condition of a list comprehension.
> There is also a doubly-nested `for` loop. There's a lot going on here
> and you might want to return to this once you've had more experience
> using list comprehensions.

We can use a conditional frequency distribution to help us find
minimally-contrasting sets of words. Here we find all the p-words
consisting of three sounds p3-words\_, and group them according to their
first and last sounds group-first-last\_.

> &gt;&gt;&gt; p3 = \[(pron\[0\]+'-'+pron\[2\], word) \#
> \[\_group-first-last\] ... for (word, pron) in entries ... if
> pron\[0\] == 'P' and len(pron) == 3\] \# \[\_p3-words\] &gt;&gt;&gt;
> cfd = nltk.ConditionalFreqDist(p3) &gt;&gt;&gt; for template in
> sorted(cfd.conditions()): ... if len(cfd\[template\]) &gt; 10: ...
> words = sorted(cfd\[template\]) ... wordstring = ' '.join(words) ...
> print(template, wordstring\[:70\] + "...") ... P-CH patch pautsch
> peach perch petsch petsche piche piech pietsch pitch pit... P-K pac
> pack paek paik pak pake paque peak peake pech peck peek perc perk ...
> P-L pahl pail paille pal pale pall paul paule paull peal peale pearl
> pearl... P-N paign pain paine pan pane pawn payne peine pen penh penn
> pin pine pinn... P-P paap paape pap pape papp paup peep pep pip pipe
> pipp poop pop pope pop... P-R paar pair par pare parr pear peer pier
> poor poore por pore porr pour... P-S pace pass pasts peace pearse
> pease perce pers perse pesce piece piss p... P-T pait pat pate patt
> peart peat peet peete pert pet pete pett piet piett... P-UW1 peru
> peugh pew plew plue prew pru prue prugh pshew pugh...

Rather than iterating over the whole dictionary, we can also access it
by looking up particular words. We will use Python's dictionary data
structure, which we will study systematically in sec-dictionaries\_. We
look up a dictionary by giving its name followed by a key (such as the
word `'fire'`) inside square brackets dict-key\_.

> &gt;&gt;&gt; prondict = nltk.corpus.cmudict.dict() &gt;&gt;&gt;
> prondict\['fire'\] \# \[\_dict-key\] \[\['F', 'AY1', 'ER0'\], \['F',
> 'AY1', 'R'\]\] &gt;&gt;&gt; prondict\['blog'\] \# \[\_dict-key-error\]
> Traceback (most recent call last): File "&lt;stdin&gt;", line 1, in
> &lt;module&gt; KeyError: 'blog' &gt;&gt;&gt; prondict\['blog'\] =
> \[\['B', 'L', 'AA1', 'G'\]\] \# \[\_dict-assign\] &gt;&gt;&gt;
> prondict\['blog'\] \[\['B', 'L', 'AA1', 'G'\]\]

If we try to look up a non-existent key dict-key-error\_, we get a
`KeyError`. This is similar to what happens when we index a list with an
integer that is too large, producing an `IndexError`. The word blog is
missing from the pronouncing dictionary, so we tweak our version by
assigning a value for this key dict-assign\_ (this has no effect on the
|NLTK| corpus; next time we access it, blog will still be absent).

We can use any lexical resource to process a text, e.g., to filter out
words having some lexical property (like nouns), or mapping every word
of the text. For example, the following text-to-speech function looks up
each word of the text in the pronunciation dictionary.

> &gt;&gt;&gt; text = \['natural', 'language', 'processing'\]
> &gt;&gt;&gt; \[ph for w in text for ph in prondict\[w\]\[0\]\] \['N',
> 'AE1', 'CH', 'ER0', 'AH0', 'L', 'L', 'AE1', 'NG', 'G', 'W', 'AH0',
> 'JH', 'P', 'R', 'AA1', 'S', 'EH0', 'S', 'IH0', 'NG'\]

### Comparative Wordlists

Another example of a tabular lexicon is the comparative wordlist. |NLTK|
includes so-called Swadesh wordlists, lists of about 200 common words in
several languages. The languages are identified using an ISO 639
two-letter code.

> &gt;&gt;&gt; from nltk.corpus import swadesh &gt;&gt;&gt;
> swadesh.fileids() \['be', 'bg', 'bs', 'ca', 'cs', 'cu', 'de', 'en',
> 'es', 'fr', 'hr', 'it', 'la', 'mk', 'nl', 'pl', 'pt', 'ro', 'ru',
> 'sk', 'sl', 'sr', 'sw', 'uk'\] &gt;&gt;&gt; swadesh.words('en') \['I',
> 'you (singular), thou', 'he', 'we', 'you (plural)', 'they', 'this',
> 'that', 'here', 'there', 'who', 'what', 'where', 'when', 'how', 'not',
> 'all', 'many', 'some', 'few', 'other', 'one', 'two', 'three', 'four',
> 'five', 'big', 'long', 'wide', ...\]

We can access cognate words from multiple languages using the
`entries()` method, specifying a list of languages. With one further
step we can convert this into a simple dictionary (we'll learn about
`dict()` in sec-dictionaries\_).

> &gt;&gt;&gt; fr2en = swadesh.entries(\['fr', 'en'\]) &gt;&gt;&gt;
> fr2en \[('je', 'I'), ('tu, vous', 'you (singular), thou'), ('il',
> 'he'), ...\] &gt;&gt;&gt; translate = dict(fr2en) &gt;&gt;&gt;
> translate\['chien'\] 'dog' &gt;&gt;&gt; translate\['jeter'\] 'throw'

We can make our simple translator more useful by adding other source
languages. Let's get the German-English and Spanish-English pairs,
convert each to a dictionary using `dict()`, then *update* our original
`translate` dictionary with these additional mappings:

> &gt;&gt;&gt; de2en = swadesh.entries(\['de', 'en'\]) \# German-English
> &gt;&gt;&gt; es2en = swadesh.entries(\['es', 'en'\]) \#
> Spanish-English &gt;&gt;&gt; translate.update(dict(de2en))
> &gt;&gt;&gt; translate.update(dict(es2en)) &gt;&gt;&gt;
> translate\['Hund'\] 'dog' &gt;&gt;&gt; translate\['perro'\] 'dog'

We can compare words in various Germanic and Romance languages:

> &gt;&gt;&gt; languages = \['en', 'de', 'nl', 'es', 'fr', 'pt', 'la'\]
> &gt;&gt;&gt; for i in \[139, 140, 141, 142\]: ...
> print(swadesh.entries(languages)\[i\]) ... ('say', 'sagen', 'zeggen',
> 'decir', 'dire', 'dizer', 'dicere') ('sing', 'singen', 'zingen',
> 'cantar', 'chanter', 'cantar', 'canere') ('play', 'spielen', 'spelen',
> 'jugar', 'jouer', 'jogar, brincar', 'ludere') ('float', 'schweben',
> 'zweven', 'flotar', 'flotter', 'flutuar, boiar', 'fluctuare')

### Shoebox and Toolbox Lexicons

Perhaps the single most popular tool used by linguists for managing data
is *Toolbox*, previously known as *Shoebox* since it replaces the field
linguist's traditional shoebox full of file cards. Toolbox is freely
downloadable from `http://www.sil.org/computing/toolbox/`.

A Toolbox file consists of a collection of entries, where each entry is
made up of one or more fields. Most fields are optional or repeatable,
which means that this kind of lexical resource cannot be treated as a
table or spreadsheet.

Here is a dictionary for the Rotokas language. We see just the first
entry, for the word kaa meaning "to gag":

> &gt;&gt;&gt; from nltk.corpus import toolbox &gt;&gt;&gt;
> toolbox.entries('rotokas.dic') \[('kaa', \[('ps', 'V'), ('pt', 'A'),
> ('ge', 'gag'), ('tkp', 'nek i pas'), ('dcsv', 'true'), ('vx', '1'),
> ('sc', '???'), ('dt', '29/Oct/2005'), ('ex', 'Apoka ira kaaroi aioa-ia
> reoreopaoro.'), ('xp', 'Kaikai i pas long nek bilong Apoka bikos em i
> kaikai na toktok.'), ('xe', 'Apoka is gagging from food while
> talking.')\]), ...\]

Entries consist of a series of attribute-value pairs, like `('ps', 'V')`
to indicate that the part-of-speech is `'V'` (verb), and `('ge', 'gag')`
to indicate that the gloss-into-English is `'gag'`. The last three pairs
contain an example sentence in Rotokas and its translations into Tok
Pisin and English.

The loose structure of Toolbox files makes it hard for us to do much
more with them at this stage. XML provides a powerful way to process
this kind of corpus and we will return to this topic in chap-data\_.

> **note**
>
> The Rotokas language is spoken on the island of Bougainville, Papua
> New Guinea. This lexicon was contributed to |NLTK| by Stuart Robinson.
> Rotokas is notable for having an inventory of just 12 phonemes
> (contrastive sounds), `http://en.wikipedia.org/wiki/Rotokas_language`

WordNet
-------

WordNet is a semantically-oriented dictionary of English, similar to a
traditional thesaurus but with a richer structure. |NLTK| includes the
English WordNet, with 155,287 words and 117,659 synonym sets. We'll
begin by looking at synonyms and how they are accessed in WordNet.

### Senses and Synonyms

Consider the sentence in ex-car1\_. If we replace the word motorcar in
ex-car1\_ by automobile, to get ex-car2\_, the meaning of the sentence
stays pretty much the same:

Since everything else in the sentence has remained unchanged, we can
conclude that the words motorcar and automobile have the same meaning,
i.e. they are synonyms. We can explore these words with the help of
WordNet:

> &gt;&gt;&gt; from nltk.corpus import wordnet as wn &gt;&gt;&gt;
> wn.synsets('motorcar') \[Synset('car.n.01')\]

Thus, motorcar has just one possible meaning and it is identified as
`car.n.01`, the first noun sense of car. The entity `car.n.01` is called
a synset, or "synonym set", a collection of synonymous words (or
"lemmas"):

> &gt;&gt;&gt; wn.synset('car.n.01').lemma\_names() \['car', 'auto',
> 'automobile', 'machine', 'motorcar'\]

Each word of a synset can have several meanings, e.g., car can also
signify a train carriage, a gondola, or an elevator car. However, we are
only interested in the single meaning that is common to all words of the
above synset. Synsets also come with a prose definition and some example
sentences:

> &gt;&gt;&gt; wn.synset('car.n.01').definition() 'a motor vehicle with
> four wheels; usually propelled by an internal combustion engine'
> &gt;&gt;&gt; wn.synset('car.n.01').examples() \['he needs a car to get
> to work'\]

Although definitions help humans to understand the intended meaning of a
synset, the words of the synset are often more useful for our programs.
To eliminate ambiguity, we will identify these words as
`car.n.01.automobile`, `car.n.01.motorcar`, and so on. This pairing of a
synset with a word is called a lemma. We can get all the lemmas for a
given synset get-lemmas\_, look up a particular lemma lookup-lemma\_,
get the synset corresponding to a lemma get-synset\_, and get the "name"
of a lemma get-name\_:

> &gt;&gt;&gt; wn.synset('car.n.01').lemmas() \# \[\_get-lemmas\]
> \[Lemma('car.n.01.car'), Lemma('car.n.01.auto'),
> Lemma('car.n.01.automobile'), Lemma('car.n.01.machine'),
> Lemma('car.n.01.motorcar')\] &gt;&gt;&gt;
> wn.lemma('car.n.01.automobile') \# \[\_lookup-lemma\]
> Lemma('car.n.01.automobile') &gt;&gt;&gt;
> wn.lemma('car.n.01.automobile').synset() \# \[\_get-synset\]
> Synset('car.n.01') &gt;&gt;&gt; wn.lemma('car.n.01.automobile').name()
> \# \[\_get-name\] 'automobile'

Unlike the word motorcar, which is unambiguous and has one synset, the
word car is ambiguous, having five synsets:

> &gt;&gt;&gt; wn.synsets('car') \[Synset('car.n.01'),
> Synset('car.n.02'), Synset('car.n.03'), Synset('car.n.04'),
> Synset('cable\_car.n.01')\] &gt;&gt;&gt; for synset in
> wn.synsets('car'): ... print(synset.lemma\_names()) ... \['car',
> 'auto', 'automobile', 'machine', 'motorcar'\] \['car', 'railcar',
> 'railway\_car', 'railroad\_car'\] \['car', 'gondola'\] \['car',
> 'elevator\_car'\] \['cable\_car', 'car'\]

For convenience, we can access all the lemmas involving the word car as
follows.

> &gt;&gt;&gt; wn.lemmas('car') \[Lemma('car.n.01.car'),
> Lemma('car.n.02.car'), Lemma('car.n.03.car'), Lemma('car.n.04.car'),
> Lemma('cable\_car.n.01.car')\]

> **note**
>
> |TRY| Write down all the senses of the word dish that you can think
> of. Now, explore this word with the help of WordNet, using the same
> operations we used above.

### The WordNet Hierarchy

WordNet synsets correspond to abstract concepts, and they don't always
have corresponding words in English. These concepts are linked together
in a hierarchy. Some concepts are very general, such as *Entity*,
*State*, *Event* |mdash| these are called unique beginners or root
synsets. Others, such as *gas guzzler* and *hatchback*, are much more
specific. A small portion of a concept hierarchy is illustrated in
[fig-wn-hierarchy](..%20figure::%20../images/wordnet-hierarchy.png:scale:%2025:120:25).

> Fragment of WordNet Concept Hierarchy: nodes correspond to synsets;
> edges indicate the hypernym/hyponym relation, i.e. the relation
> between superordinate and subordinate concepts.

WordNet makes it easy to navigate between concepts. For example, given a
concept like *motorcar*, we can look at the concepts that are more
specific; the (immediate) hyponyms.

> &gt;&gt;&gt; motorcar = wn.synset('car.n.01') &gt;&gt;&gt;
> types\_of\_motorcar = motorcar.hyponyms() &gt;&gt;&gt;
> types\_of\_motorcar\[0\] Synset('ambulance.n.01') &gt;&gt;&gt;
> sorted(lemma.name() for synset in types\_of\_motorcar for lemma in
> synset.lemmas()) \['Model\_T', 'S.U.V.', 'SUV', 'Stanley\_Steamer',
> 'ambulance', 'beach\_waggon', 'beach\_wagon', 'bus', 'cab', 'compact',
> 'compact\_car', 'convertible', 'coupe', 'cruiser', 'electric',
> 'electric\_automobile', 'electric\_car', 'estate\_car',
> 'gas\_guzzler', 'hack', 'hardtop', 'hatchback', 'heap',
> 'horseless\_carriage', 'hot-rod', 'hot\_rod', 'jalopy', 'jeep',
> 'landrover', 'limo', 'limousine', 'loaner', 'minicar', 'minivan',
> 'pace\_car', 'patrol\_car', 'phaeton', 'police\_car',
> 'police\_cruiser', 'prowl\_car', 'race\_car', 'racer', 'racing\_car',
> 'roadster', 'runabout', 'saloon', 'secondhand\_car', 'sedan',
> 'sport\_car', 'sport\_utility', 'sport\_utility\_vehicle',
> 'sports\_car', 'squad\_car', 'station\_waggon', 'station\_wagon',
> 'stock\_car', 'subcompact', 'subcompact\_car', 'taxi', 'taxicab',
> 'tourer', 'touring\_car', 'two-seater', 'used-car', 'waggon',
> 'wagon'\]

We can also navigate up the hierarchy by visiting hypernyms. Some words
have multiple paths, because they can be classified in more than one
way. There are two paths between `car.n.01` and `entity.n.01` because
`wheeled_vehicle.n.01` can be classified as both a vehicle and a
container.

> &gt;&gt;&gt; motorcar.hypernyms() \[Synset('motor\_vehicle.n.01')\]
> &gt;&gt;&gt; paths = motorcar.hypernym\_paths() &gt;&gt;&gt;
> len(paths) 2 &gt;&gt;&gt; \[synset.name() for synset in paths\[0\]\]
> \['entity.n.01', 'physical\_entity.n.01', 'object.n.01', 'whole.n.02',
> 'artifact.n.01', 'instrumentality.n.03', 'container.n.01',
> 'wheeled\_vehicle.n.01', 'self-propelled\_vehicle.n.01',
> 'motor\_vehicle.n.01', 'car.n.01'\] &gt;&gt;&gt; \[synset.name() for
> synset in paths\[1\]\] \['entity.n.01', 'physical\_entity.n.01',
> 'object.n.01', 'whole.n.02', 'artifact.n.01', 'instrumentality.n.03',
> 'conveyance.n.03', 'vehicle.n.01', 'wheeled\_vehicle.n.01',
> 'self-propelled\_vehicle.n.01', 'motor\_vehicle.n.01', 'car.n.01'\]

We can get the most general hypernyms (or root hypernyms) of a synset as
follows:

> &gt;&gt;&gt; motorcar.root\_hypernyms() \[Synset('entity.n.01')\]

> **note**
>
> |TRY| Try out |NLTK|'s convenient graphical WordNet browser:
> `nltk.app.wordnet()`. Explore the WordNet hierarchy by following the
> hypernym and hyponym links.

### More Lexical Relations

Hypernyms and hyponyms are called lexical relations because they relate
one synset to another. These two relations navigate up and down the
"is-a" hierarchy. Another important way to navigate the WordNet network
is from items to their components (meronyms) or to the things they are
contained in (holonyms). For example, the parts of a tree are its trunk,
crown, and so on; the `part_meronyms()`. The *substance* a tree is made
of includes heartwood and sapwood; the `substance_meronyms()`. A
collection of trees forms a forest; the `member_holonyms()`:

> &gt;&gt;&gt; wn.synset('tree.n.01').part\_meronyms()
> \[Synset('burl.n.02'), Synset('crown.n.07'), Synset('limb.n.02'),
> Synset('stump.n.01'), Synset('trunk.n.01')\] &gt;&gt;&gt;
> wn.synset('tree.n.01').substance\_meronyms()
> \[Synset('heartwood.n.01'), Synset('sapwood.n.01')\] &gt;&gt;&gt;
> wn.synset('tree.n.01').member\_holonyms() \[Synset('forest.n.01')\]

To see just how intricate things can get, consider the word mint, which
has several closely-related senses. We can see that `mint.n.04` is part
of `mint.n.02` and the substance from which `mint.n.05` is made.

> &gt;&gt;&gt; for synset in wn.synsets('mint', wn.NOUN): ...
> print(synset.name() + ':', synset.definition()) ... batch.n.02: (often
> followed by \`of') a large number or amount or extent mint.n.02: any
> north temperate plant of the genus Mentha with aromatic leaves and
> small mauve flowers mint.n.03: any member of the mint family of plants
> mint.n.04: the leaves of a mint plant used fresh or candied mint.n.05:
> a candy that is flavored with a mint oil mint.n.06: a plant where
> money is coined by authority of the government &gt;&gt;&gt;
> wn.synset('mint.n.04').part\_holonyms() \[Synset('mint.n.02')\]
> &gt;&gt;&gt; wn.synset('mint.n.04').substance\_holonyms()
> \[Synset('mint.n.05')\]

There are also relationships between verbs. For example, the act of
walking involves the act of stepping, so walking entails stepping. Some
verbs have multiple entailments:

> &gt;&gt;&gt; wn.synset('walk.v.01').entailments()
> \[Synset('step.v.01')\] &gt;&gt;&gt;
> wn.synset('eat.v.01').entailments() \[Synset('chew.v.01'),
> Synset('swallow.v.01')\] &gt;&gt;&gt;
> wn.synset('tease.v.03').entailments() \[Synset('arouse.v.07'),
> Synset('disappoint.v.01')\]

Some lexical relationships hold between lemmas, e.g., antonymy:

> &gt;&gt;&gt; wn.lemma('supply.n.02.supply').antonyms()
> \[Lemma('demand.n.02.demand')\] &gt;&gt;&gt;
> wn.lemma('rush.v.01.rush').antonyms() \[Lemma('linger.v.04.linger')\]
> &gt;&gt;&gt; wn.lemma('horizontal.a.01.horizontal').antonyms()
> \[Lemma('inclined.a.02.inclined'), Lemma('vertical.a.01.vertical')\]
> &gt;&gt;&gt; wn.lemma('staccato.r.01.staccato').antonyms()
> \[Lemma('legato.r.01.legato')\]

You can see the lexical relations, and the other methods defined on a
synset, using `dir()`, for example: `dir(wn.synset('harmony.n.02'))`.

### Semantic Similarity

We have seen that synsets are linked by a complex network of lexical
relations. Given a particular synset, we can traverse the WordNet
network to find synsets with related meanings. Knowing which words are
semantically related is useful for indexing a collection of texts, so
that a search for a general term like vehicle will match documents
containing specific terms like limousine.

Recall that each synset has one or more hypernym paths that link it to a
root hypernym such as `entity.n.01`. Two synsets linked to the same root
may have several hypernyms in common (cf
[fig-wn-hierarchy](..%20figure::%20../images/wordnet-hierarchy.png:scale:%2025:120:25)).
If two synsets share a very specific hypernym |mdash| one that is low
down in the hypernym hierarchy |mdash| they must be closely related.

> &gt;&gt;&gt; right = wn.synset('right\_whale.n.01') &gt;&gt;&gt; orca
> = wn.synset('orca.n.01') &gt;&gt;&gt; minke =
> wn.synset('minke\_whale.n.01') &gt;&gt;&gt; tortoise =
> wn.synset('tortoise.n.01') &gt;&gt;&gt; novel =
> wn.synset('novel.n.01') &gt;&gt;&gt;
> right.lowest\_common\_hypernyms(minke)
> \[Synset('baleen\_whale.n.01')\] &gt;&gt;&gt;
> right.lowest\_common\_hypernyms(orca) \[Synset('whale.n.02')\]
> &gt;&gt;&gt; right.lowest\_common\_hypernyms(tortoise)
> \[Synset('vertebrate.n.01')\] &gt;&gt;&gt;
> right.lowest\_common\_hypernyms(novel) \[Synset('entity.n.01')\]

Of course we know that whale is very specific (and baleen whale even
more so), while vertebrate is more general and entity is completely
general. We can quantify this concept of generality by looking up the
depth of each synset:

> &gt;&gt;&gt; wn.synset('baleen\_whale.n.01').min\_depth() 14
> &gt;&gt;&gt; wn.synset('whale.n.02').min\_depth() 13 &gt;&gt;&gt;
> wn.synset('vertebrate.n.01').min\_depth() 8 &gt;&gt;&gt;
> wn.synset('entity.n.01').min\_depth() 0

Similarity measures have been defined over the collection of WordNet
synsets which incorporate the above insight. For example,
`path_similarity` assigns a score in the range `0`|ndash|
`1` based on the shortest path that connects the concepts in the
hypernym hierarchy (`-1` is returned in those cases where a path cannot
be found). Comparing a synset with itself will return `1`. Consider the
following similarity scores, relating right whale to minke whale, orca,
tortoise, and novel. Although the numbers won't mean much, they decrease
as we move away from the semantic space of sea creatures to inanimate
objects.

> &gt;&gt;&gt; right.path\_similarity(minke) 0.25 &gt;&gt;&gt;
> right.path\_similarity(orca) 0.16666666666666666 &gt;&gt;&gt;
> right.path\_similarity(tortoise) 0.07692307692307693 &gt;&gt;&gt;
> right.path\_similarity(novel) 0.043478260869565216

> **note**
>
> Several other similarity measures are available; you can type
> `help(wn)` for more information. |NLTK| also includes VerbNet, a
> hierarhical verb lexicon linked to WordNet. It can be accessed with
> `nltk.corpus.verbnet`.

Summary
-------

-   A text corpus is a large, structured collection of texts. NLTK comes
    with many corpora, e.g., the Brown Corpus, `nltk.corpus.brown`.
-   Some text corpora are categorized, e.g., by genre or topic;
    sometimes the categories of a corpus overlap each other.
-   A conditional frequency distribution is a collection of frequency
    distributions, each one for a different condition. They can be used
    for counting word frequencies, given a context or a genre.
-   Python programs more than a few lines long should be entered using a
    text editor, saved to a file with a `.py` extension, and accessed
    using an `import` statement.
-   Python functions permit you to associate a name with a particular
    block of code, and re-use that code as often as necessary.
-   Some functions, known as "methods", are associated with an object
    and we give the object name followed by a period followed by the
    function, like this: `x.funct(y)`, e.g., `word.isalpha()`.
-   To find out about some variable `v`, type `help(v)` in the Python
    interactive interpreter to read the help entry for this kind
    of object.
-   WordNet is a semantically-oriented dictionary of English, consisting
    of synonym sets |mdash| or synsets |mdash| and organized into
    a network.
-   Some functions are not available by default, but must be accessed
    using Python's `import` statement.

Further Reading
---------------

Extra materials for this chapter are posted at |NLTK-URL|, including
links to freely available resources on the web. The corpus methods are
summarized in the Corpus HOWTO, at |NLTK-HOWTO-URL|, and documented
extensively in the online API documentation.

Significant sources of published corpora are the
Linguistic Data Consortium (LDC) and the
European Language Resources Agency (ELRA). Hundreds of annotated text
and speech corpora are available in dozens of languages. Non-commercial
licences permit the data to be used in teaching and research. For some
corpora, commercial licenses are also available (but for a higher fee).

A good tool for creating annotated text corpora is called Brat, and
available from `http://brat.nlplab.org/`.

These and many other language resources have been documented using OLAC
Metadata, and can be searched via the OLAC homepage at |OLAC-URL|.
Corpora List is a mailing list for discussions about corpora, and you
can find resources by searching the list archives or posting to the
list. The most complete inventory of the world's languages is
*Ethnologue*, `http://www.ethnologue.com/`. Of 7,000 languages, only a
few dozen have substantial digital resources suitable for use in |NLP|.

This chapter has touched on the field of Corpus Linguistics. Other
useful books in this area include \[Biber1998\]\_, \[McEnery2006\]\_,
\[Meyer2002\]\_, \[Sampson2005\]\_, \[Scott2006\]\_. Further readings in
quantitative data analysis in linguistics are: \[Baayen2008\]\_,
\[Gries2009\]\_, \[Woods1986\]\_.

The original description of WordNet is \[Fellbaum1998\]\_. Although
WordNet was originally developed for research in psycholinguistics, it
is now widely used in NLP and Information Retrieval. WordNets are being
developed for many other languages, as documented at
`http://www.globalwordnet.org/`. For a study of WordNet similarity
measures, see \[Budanitsky2006EWB\]\_.

Other topics touched on in this chapter were phonetics and lexical
semantics, and we refer readers to chapters 7 and 20 of
\[JurafskyMartin2008\]\_.

Exercises
---------

1.  |easy| Create a variable `phrase` containing a list of words. Review
    the operations described in the previous chapter, including
    addition, multiplication, indexing, slicing, and sorting.
2.  |easy| Use the corpus module to explore `austen-persuasion.txt`. How
    many word tokens does this book have? How many word types?
3.  |easy| Use the Brown corpus reader `nltk.corpus.brown.words()` or
    the Web text corpus reader `nltk.corpus.webtext.words()` to access
    some sample text in two different genres.
4.  |easy| Read in the texts of the *State of the Union* addresses,
    using the `state_union` corpus reader. Count occurrences of `men`,
    `women`, and `people` in each document. What has happened to the
    usage of these words over time?
5.  |easy| Investigate the holonym-meronym relations for some nouns.
    Remember that there are three kinds of holonym-meronym relation, so
    you need to use: `member_meronyms()`, `part_meronyms()`,
    `substance_meronyms()`, `member_holonyms()`, `part_holonyms()`, and
    `substance_holonyms()`.
6.  |easy| In the discussion of comparative wordlists, we created an
    object called `translate` which you could look up using words in
    both German and Spanish in order to get corresponding words in
    English. What problem might arise with this approach? Can you
    suggest a way to avoid this problem?
7.  |easy| According to Strunk and White's *Elements of Style*, the word
    however, used at the start of a sentence, means "in whatever way" or
    "to whatever extent", and not "nevertheless". They give this example
    of correct usage:
    However you advise him, he will probably do as he thinks best.
    (`http://www.bartleby.com/141/strunk3.html`) Use the concordance
    tool to study actual usage of this word in the various texts we have
    been considering. See also the *LanguageLog* posting "Fossilized
    prejudices about 'however'" at
    `http://itre.cis.upenn.edu/~myl/languagelog/archives/001913.html`
8.  |soso| Define a conditional frequency distribution over the Names
    corpus that allows you to see which *initial* letters are more
    frequent for males vs. females (cf.
    [fig-cfd-gender](..%20figure::%20../images/cfd-gender.png:scale:%2025)).
9.  |soso| Pick a pair of texts and study the differences between them,
    in terms of vocabulary, vocabulary richness, genre, etc. Can you
    find pairs of words which have quite different meanings across the
    two texts, such as monstrous in *Moby Dick* and in *Sense and
    Sensibility*?
10. |soso| Read the BBC News article: *UK's Vicky Pollards 'left
    behind'* `http://news.bbc.co.uk/1/hi/education/6173441.stm`. The
    article gives the following statistic about teen language: "the top
    20 words used, including yeah, no, but and like, account for around
    a third of all words." How many word types account for a third of
    all word tokens, for a variety of text sources? What do you conclude
    about this statistic? Read more about this on *LanguageLog*, at
    `http://itre.cis.upenn.edu/~myl/languagelog/archives/003993.html`.
11. |soso| Investigate the table of modal distributions and look for
    other patterns. Try to explain them in terms of your own
    impressionistic understanding of the different genres. Can you find
    other closed classes of words that exhibit significant differences
    across different genres?
12. |soso| The CMU Pronouncing Dictionary contains multiple
    pronunciations for certain words. How many distinct words does it
    contain? What fraction of words in this dictionary have more than
    one possible pronunciation?
13. |soso| What percentage of noun synsets have no hyponyms? You can get
    all noun synsets using `wn.all_synsets('n')`.
14. |soso| Define a function `supergloss(s)` that takes a synset `s` as
    its argument and returns a string consisting of the concatenation of
    the definition of `s`, and the definitions of all the hypernyms and
    hyponyms of `s`.
15. |soso| Write a program to find all words that occur at least three
    times in the Brown Corpus.
16. |soso| Write a program to generate a table of lexical diversity
    scores (i.e. token/type ratios), as we saw in tab-brown-types\_.
    Include the full set of Brown Corpus genres
    (`nltk.corpus.brown.categories()`). Which genre has the lowest
    diversity (greatest number of tokens per type)? Is this what you
    would have expected?
17. |soso| Write a function that finds the 50 most frequently occurring
    words of a text that are not stopwords.
18. |soso| Write a program to print the 50 most frequent bigrams (pairs
    of adjacent words) of a text, omitting bigrams that
    contain stopwords.
19. |soso| Write a program to create a table of word frequencies by
    genre, like the one given in sec-extracting-text-from-corpora\_ for
    modals. Choose your own words and try to find words whose presence
    (or absence) is typical of a genre. Discuss your findings.
20. |soso| Write a function `word_freq()` that takes a word and the name
    of a section of the Brown Corpus as arguments, and computes the
    frequency of the word in that section of the corpus.
21. |soso| Write a program to guess the number of syllables contained in
    a text, making use of the CMU Pronouncing Dictionary.
22. |soso| Define a function `hedge(text)` which processes a text and
    produces a new version with the word `'like'` between every
    third word.
23. |hard| **Zipf's Law**: Let *f(w)* be the frequency of a word *w* in
    free text. Suppose that all the words of a text are ranked according
    to their frequency, with the most frequent word first. Zipf's law
    states that the frequency of a word type is inversely proportional
    to its rank (i.e. *f* |times| *r = k*, for some constant *k*). For
    example, the 50th most common word type should occur three times as
    frequently as the 150th most common word type.
    a)  Write a function to process a large text and plot word frequency
        against word rank using `pylab.plot`. Do you confirm Zipf's law?
        (Hint: it helps to use a logarithmic scale). What is going on at
        the extreme ends of the plotted line?
    b)  Generate random text, e.g., using `random.choice("abcdefg ")`,
        taking care to include the space character. You will need to
        `import random` first. Use the string concatenation operator to
        accumulate characters into a (very) long string. Then tokenize
        this string, and generate the Zipf plot as before, and compare
        the two plots. What do you make of Zipf's Law in the light of
        this?

24. |hard| Modify the text generation program in code-random-text\_
    further, to do the following tasks:
    a)  Store the *n* most likely words in a list `words` then randomly
        choose a word from the list using `random.choice()`. (You will
        need to `import random` first.)
    b)  Select a particular genre, such as a section of the Brown
        Corpus, or a genesis translation, one of the Gutenberg texts, or
        one of the Web texts. Train the model on this corpus and get it
        to generate random text. You may have to experiment with
        different start words. How intelligible is the text? Discuss the
        strengths and weaknesses of this method of generating
        random text.
    c)  Now train your system using two distinct genres and experiment
        with generating text in the hybrid genre. Discuss
        your observations.

25. |hard| Define a function `find_language()` that takes a string as
    its argument, and returns a list of languages that have that string
    as a word. Use the `udhr` corpus and limit your searches to files in
    the Latin-1 encoding.
26. |hard| What is the branching factor of the noun hypernym
    hierarchy? I.e. for every noun synset that has hyponyms |mdash| or
    children in the hypernym hierarchy |mdash| how many do they have on
    average? You can get all noun synsets using `wn.all_synsets('n')`.
27. |hard| The polysemy of a word is the number of senses it has. Using
    WordNet, we can determine that the noun *dog* has 7 senses with:
    `len(wn.synsets('dog', 'n'))`. Compute the average polysemy of
    nouns, verbs, adjectives and adverbs according to WordNet.
28. |hard| Use one of the predefined similarity measures to score the
    similarity of each of the following pairs of words. Rank the pairs
    in order of decreasing similarity. How close is your ranking to the
    order given here, an order that was established experimentally by
    \[MillerCharles1998\]\_: car-automobile, gem-jewel, journey-voyage,
    boy-lad, coast-shore, asylum-madhouse, magician-wizard, midday-noon,
    furnace-stove, food-fruit, bird-cock, bird-crane, tool-implement,
    brother-monk, lad-brother, crane-implement, journey-car,
    monk-oracle, cemetery-woodland, food-rooster, coast-hill,
    forest-graveyard, shore-woodland, monk-slave, coast-forest,
    lad-wizard, chord-smile, glass-magician,
    rooster-voyage, noon-string.

