---
layout: post
title: 5 weird tricks for a good spellchecker
comments: true
description: >
  Spellchecking is an essential part of many products and business applications. However, it's working characteristics (speed, quality, memory consumption) are often not optimal - let's see how to make your spell-checker fast and furious.
tags: [NLP, spellchecking, spelling correction]
---

**A quick intro**

It is more than 10 years already Peter Norvig has put his sterlingly simple spell-checker into 21 line of [python code](https://norvig.com/spell-correct.html). A pure probability model selects the most probable replacement for an unknown word from the list of words collected in a book – this minimalistic approach is still quite relevant and can be considered as a baseline and a prototype for further development, however, it has a number of imperfections:

1.	if your dictionary is big enough, (like, millions of words), the candidate list will be too big to range it properly with this simple technique
1. no restrictions on the speed and memory are provided for this solution
1.	it cannot find real-word errors

We could name, of course, other problems of the original code, but let us consider these two the main ones, as this is just a prototype – and we should now clarify, how to adapt the algorithm so it can become a production solution.

So, if we just want our spell-checker to work fast despite having a big dictionary (what is a usual situation, when you collect a dictionary from a big web-corpus or your language has rich morphology – like Russian, Hungarian, etc), then you have to search for a more courteous manner to refine the dictionary storage and time a new word is compared to dictionary words you have.

I will now try to introduce 5 new algorithms and data structures that I use to optimize spell-checking. 

So, getting started: there are two steps in spelling correction: 
  - firstly, you get a dictionary and find errors in the text, 
  - secondly, you choose the best correction, ranging dictionary candidates


## Storing a dictionary

Searching if a word is in a plain dictionary or list structure can take a while, and it can take even more, if you should measure a distance between an out-of-vocabulary word and every word in dictionary to find corrections.

### 1. Trie
Trie, aka radix tree or prefix tree, is a kind of search tree — an ordered tree data structure used to store a dynamic set or associative array where the keys are usually strings. It is kind of similar to binary search tree, but for language data - you have no >< conditions, but variants - what substring would be the next:
![](https://i.stack.imgur.com/f9Q3u.jpg)

So, all the descendants of a node have a common prefix of the string associated with that node, and the root is associated with the empty string. In Python kind of logics, that corresponds to a recursive dictionary in a dictionary, where keys are letters. This kind of structure is extremely useful when you have to store a dictionary in memory to check if a word is in dictionary - you do not only the amount of memory, but also the search time decreases tens of times, as the search for a key in a dictionary of millions of words is orders of magnitude slower than the sequential search in that descending embedded dictionaries.

You can build an independent trie-vocabulary for every list of words beginning with the same letter (A-, B-, etc), or having this letter last, etc - and thus limit the search for candidates only by these trie-structures.

Realizations:

[Python](https://github.com/pytries/datrie)

[C](https://linux.thai.net/~thep/datrie/datrie.html)

[Java](https://github.com/digitalstain/DoubleArrayTrie)


## Approximate string matching

Now let's move to tricks with word distance and their fast 

### 2. Bk-trees
A BK-tree is a metric tree suggested by Walter Austin Burkhard and Robert M. Keller[1] specifically adapted to discrete metric spaces. For simplicity, let us consider integer discrete metric d (x, y). Then, BK-tree is defined in the following way: an arbitrary element a is selected as root node. The root node may have zero or more subtrees. The k-th subtree is recursively built of all elements b such that d(a,b)=k. BK-trees can be used for approximate string matching in a dictionary:

![](https://nullwords.files.wordpress.com/2013/03/bk31-e1363207034407.png)

The main profit of using bk-trees instead of plain dictionaries is that measuring distance   between an out-of-vocabulary word and every word in dictionary is much faster in this kind of structure - it's now O(log n) instead of O(n).

Implementations:

[Python](https://github.com/Jetsetter/pybktree)

[C](https://github.com/tyler/BkTree)

[Ruby](https://github.com/threedaymonk/bktree)

[Java](https://github.com/gtri/bk-tree)


### 3. Phonetic Algorithms

Phonetic algorithms are widely used in spellchecking, as they can make the search of a close vocabulary word much more precise:
  -  if you use some standard distance measure, which is based on letter alighnment, the search is  usually limited to candidates standing 1-2 letters from a word with error. If you increase this distance, you propably get  too much unrelevant candidates.
  - a lot of typical errors, caused by intentional distortion or slangy language gamification, outstand from a relevant candidate  more than 2 letters away: _riiiiigtht_ --> _right_, _donut_ --> _doughnut_, _ave_--> _avenue_
  - these far-from-a-right-candidate examples seem to be somehow systematicly located: actually, these errors we can call "phonetic" or "abbreviative", and instead of using ordinary distance measures we can make a new, phonetic distance. Phonetic algorithms solve this problem quite well.
  
* **Metaphone**

Metaphone algorithm [2]-[3] is one of the most popular phonetic algorithms on spelling correction. 
It makes a phonetic hash out of each word in vocabulary, and getting this kind of a hash for a word with error, you can search for a better candadate through your hashed dictionary using standard distance measures.

Original Metaphone codes use the 16 consonant symbols 0BFHJKLMNPRSTWXY. The '0' represents "th" (as an ASCII approximation of Θ), 'X' represents "sh" or "ch", and the others represent their usual English pronunciations. The vowels AEIOU are also used, but only at the beginning of the code. Text below summarizes most of the rules in the original implementation:

    Drop duplicate adjacent letters, except for C.
    If the word begins with 'KN', 'GN', 'PN', 'AE', 'WR', drop the first letter.
    Drop 'B' if after 'M' at the end of the word.
    'C' transforms to 'X' if followed by 'IA' or 'H' (unless in latter case, it is part of '-SCH-', in which case it transforms to 'K'). 'C' transforms to 'S' if followed by 'I', 'E', or 'Y'. Otherwise, 'C' transforms to 'K'.
    'D' transforms to 'J' if followed by 'GE', 'GY', or 'GI'. Otherwise, 'D' transforms to 'T'.
    Drop 'G' if followed by 'H' and 'H' is not at the end or before a vowel. Drop 'G' if followed by 'N' or 'NED' and is at the end.
    'G' transforms to 'J' if before 'I', 'E', or 'Y', and it is not in 'GG'. Otherwise, 'G' transforms to 'K'.
    Drop 'H' if after vowel and not before a vowel.
    'CK' transforms to 'K'.
    'PH' transforms to 'F'.
    'Q' transforms to 'K'.
    'S' transforms to 'X' if followed by 'H', 'IO', or 'IA'.
    'T' transforms to 'X' if followed by 'IA' or 'IO'. 'TH' transforms to '0'. Drop 'T' if followed by 'CH'.
    'V' transforms to 'F'.
    'WH' transforms to 'W' if at the beginning. Drop 'W' if not followed by a vowel.
    'X' transforms to 'S' if at the beginning. Otherwise, 'X' transforms to 'KS'.
    Drop 'Y' if not followed by a vowel.
    'Z' transforms to 'S'.
    Drop all vowels unless it is the beginning.


The Double Metaphone phonetic encoding algorithm is the second generation of this algorithm. It makes a number of fundamental design improvements over the original Metaphone algorithm -  it uses a much more complex ruleset for coding than its predecessor; for example, it tests for approximately 100 different contexts of the use of the letter C alone. 

Implementations: (you can write rules for your own language with implementations below)

[Brazilian Portuguese in C](https://sourceforge.net/projects/metaphoneptbr/)

[Brazilian Portuguese in Java](https://github.com/ruliana/MTFN)

[Spanish in Python](https://github.com/amsqr/Spanish-Metaphone)

[Russian Metaphone in Ruby](https://github.com/pavlo/russian_metaphone)


* **Match rating approach**

Match rating approach (MRA) is a phonetic algorithm developed in 1977 and firstly used for the indexation and comparison of homophonous names. Unlike Metaphone, MRA includes both hashing rules and their distance measure. 
It is suitable for small vocabularies and searching for abbreviations and acronyms.

Encoding rules

    Delete all vowels unless the vowel begins the word
    Remove the second consonant of any double consonants present
    Reduce codex to 6 letters by joining the first 3 and last 3 letters only

Comparison rules

In this section, the words "string(s)" and "name(s)" mean "encoded string(s)" and "encoded name(s)".

    If the length difference between the encoded strings is 3 or greater, then no similarity comparison is done.
    Obtain the minimum rating value by calculating the length sum of the encoded strings and using table A
    Process the encoded strings from left to right and remove any identical characters found from both strings respectively.
    Process the unmatched characters from right to left and remove any identical characters found from both names respectively.
    Subtract the number of unmatched characters from 6 in the longer string. This is the similarity rating.
    If the similarity rating equal to or greater than the minimum rating then the match is considered good.
    
Implementations:

[C#](https://archive.codeplex.com/?p=sounditout)

[Javascript](https://github.com/nodef/english-mraphonetic)

    
### 4. Distance measures

There are some non-standard measures of string distance besides Levenshtein distance, which are calculated using a different set of allowable edit operations. Depending on which mistakes you want to correct, while ranking the candidates using the distance function, you can choose more sophisticated distance metric.  For instance, there are

*  [Jaro–Winkler Similarity](https://en.wikipedia.org/wiki/Jaro%E2%80%93Winkler_distance) measures the minimum number of single-character transpositions required to change one word into the other, giving more favorable ratings to strings that match from the beginning. 
*  the [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance) allows deletion, insertion and substitution;
*  the [Damerau–Levenshtein distance](https://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance) allows insertion, deletion, substitution, and the transposition of two adjacent characters;
*  the [longest common subsequence (LCS)](https://en.wikipedia.org/wiki/Longest_common_subsequence_problem) distance allows only insertion and deletion, not substitution;
*  the [Hamming distance](https://en.wikipedia.org/wiki/Hamming_distance) allows only substitution, hence, it only applies to strings of the same length.

You can, of course, implement your own function - for example, take Damerau–Levenshtein distance as a baseline and add some special operations to it, like rearranging the candidates according to keyboard distances.


## Finding real-word errors

### 5. Machine Learning for spelling

"See you in five minuets" - this type of errors, when a spelling error leads to appearance of another normal word, is called Real-Word Error.
Real-word errors (RWE) are harder to find, as you cannot spot them with a dictionary lookup, but the most common approach to find them is context-based.

I will not recommend this type of algorithm for spellcheckers with real-time speed limits, as reachecking every word in a sentence and deciding whether the context is typycal for it or not, is not a fast way to check the spelling.

Yet sometimes RWE are about 15% of all the errors, and in this case you  should deal with them some way. Most successful model dealing with RWEs is described in [4] - there you can find pseudocode based on word ngrams as a context. Other way to find RWEs is noisy channel model, which is not considered a xlassic approach anymore, yet can be effective in this kind of error detection.

Implementations:

[Pseudo-Java](http://old.atala.org/IMG/pdf/Flor-TAL53-3.pdf)

[Noisy-channel Python](https://github.com/ofrik/NLP-HW1/blob/master/spell_checker.py)

[Another solution for Java](https://github.com/UKPLab/spelling-experiments)






## References

[1] [W. Burkhard and R. Keller. Some approaches to best-match file searching, CACM, 1973](https://dl.acm.org/citation.cfm?doid=362003.362025)
[2] [Hanging on the Metaphone, Lawrence Philips. Computer Language, Vol. 7, No. 12 (December), 1990.](https://scinapse.io/papers/152126889)
[3] [The Double Metaphone Search Algorithm, By Lawrence Phillips, June 1, 2000, Dr Dobb's](http://www.drdobbs.com/the-double-metaphone-search-algorithm/184401251?pgno=2)
[4] [Flor M.(2012) Four types of context for automatic spelling correction //TAL.—Vol. 53.—Vol. 3.—pp. 61–99.](http://old.atala.org/IMG/pdf/Flor-TAL53-3.pdf)
