---
layout: post
title: 5 algorithms you need to know to build a fast spellchecker
description: >
  Spellchecking is an essential part of many products and business applications. However, it's working characteristics (speed, quality, memory consumption) are often not optimal - let's see how to make your spell-checker fast and furious.
tags: [NLP, spellchecking, spelling correction]
comments: true
---


It is more than 10 years already Peter Norvig has put his sterlingly simple spell-checker into 21 line of [python code](https://norvig.com/spell-correct.html). A pure probability model selects the most probable replacement for an unknown word from the list of words collected in a book – this minimalistic approach is still quite relevant and can be considered as a baseline and a prototype for further development, however, it has a number of imperfections:

1)	if your dictionary is big enough, (like, millions of words), the candidate list will be too big to range it properly with this simple technique

2)	no restrictions on the speed and memory are provided for this solution

3)	it cannot find real-word errors

We could name, of course, other problems of the original code, but let us consider these two the main ones, as this is just a prototype – and we should now clarify, how to adapt the algorithm so it can become a production solution.

So, if we just want our spell-checker to work fast despite having a big dictionary (what is a usual situation, when you collect a dictionary from a big web-corpus or your language has rich morphology – like Russian, Hungarian, etc), then you have to search for a more courteous manner to refine the dictionary storage and time a new word is compared to dictionary words you have.

I will now try to introduce 5 new algorithms and data structures that I use to optimize spell-checking. In this article, all examples are given in python 3, but I will also provide links to their realizations on other languages. 

So, getting started: there are two steps in spelling correction: firstly, you find error in the text, secondly, you choose the best correction.

## Finding an error

### Storing a dictionary:
#### Trie
Trie, aka radix tree or prefix tree, is a kind of search tree — an ordered tree data structure used to store a dynamic set or associative array where the keys are usually strings. It is kind of similar to binary search tree, but for language data - you have no >< conditions, but variants - what substring would be the next:
![](https://i.stack.imgur.com/f9Q3u.jpg)

So, all the descendants of a node have a common prefix of the string associated with that node, and the root is associated with the empty string. In Python kind of logics, that corresponds to a recursive dictionary in a dictionary, where keys are letters. This kind of structure is extremely useful when you have to store a dictionary in memory to check if a word is in dictionary - you do not only the amount of memory, but also the search time decreases tens of times, as the search for a key in a dictionary of millions of words is orders of magnitude slower than the sequential search in that descending embedded dictionaries.

Realizations:
[Python](https://github.com/pytries/datrie)
[C](https://linux.thai.net/~thep/datrie/datrie.html)
[Java](https://github.com/digitalstain/DoubleArrayTrie)

### Approximate string matching
#### Bk-trees
A BK-tree is a metric tree suggested by Walter Austin Burkhard and Robert M. Keller[1] specifically adapted to discrete metric spaces. For simplicity, let us consider integer discrete metric d ( x , y ) {\displaystyle d(x,y)} d(x,y). Then, BK-tree is defined in the following way. An arbitrary element a is selected as root node. The root node may have zero or more subtrees. The k-th subtree is recursively built of all elements b such that d ( a , b ) = k {\displaystyle d(a,b)=k} d(a,b)=k. BK-trees can be used for approximate string matching in a dictionary

#### Metaphone

### Real-word errors

## Choosing correction 

#### Distance measures
#### SVM

## References
[1] [ W. Burkhard and R. Keller. Some approaches to best-match file searching, CACM, 1973](https://dl.acm.org/citation.cfm?doid=362003.362025)
