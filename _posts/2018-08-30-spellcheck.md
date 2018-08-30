---
layout: post
title: 5 algorithms you need to know to build a fast spellchecker
description: >
  Spellchecking is an essential part of many products and business applications. However, it's working characteristics (speed, quality, memory consumption) are not optimal - let's see how to make your spell-checker fast and furious.
tags: [NLP, spellchecking, spelling correction]
---


It is more than 10 years already Peter Norvig has put his sterlingly simple spell-checker into 21 line of [https://norvig.com/spell-correct.html](python code). A pure probability model selects the most probable replacement for an unknown word from the list of words collected in a book – this minimalistic approach is still quite relevant and can be considered as a baseline and a prototype for further development, however, it has a number of imperfections:

1)	if your dictionary is big enough, (like, millions of words), the candidate list will be too big to range it properly with this simple technique

2)	no restrictions on the speed and memory are provided for this solution

3)	it cannot find real-word errors

We could name, of course, other problems of the original code, but let us consider these two the main ones, as this is just a prototype – and we should now clarify, how to adapt the algorithm so it can become a production solution.

So, if we just want our spell-checker to work fast despite having a big dictionary (what is a usual situation, when you collect a dictionary from a big web-corpus or your language has rich morphology – like Russian, Hungarian, etc), then you have to search for a more courteous manner to refine the dictionary storage and time a new word is compared to dictionary words you have.

I will now try to introduce 5 new algorithms and data structures that I use to optimize spell-checking. In this article, all examples are given in python 3, but I will also provide links to their realizations on other languages. 

So, getting started: there are two steps in spelling correction: firstly, you find error in the text, secondly, you choose the best correction.

## Finding an error

### Storing dictionary:
Bk-trees
Trie
metaphone

### Real-word errors

## Choosing correction 

Distance measures
SVM

## References
