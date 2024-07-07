---
title: Paraop
layout: nonpost
---
The Paraop corpus and models are available on [GitHub](https://github.com/lucasadelino/paraop).

---

## What is it?

Paraop is a framework for annotating _paraphrase operations_: the specific changes that occur in a sentence in the production of a paraphrase. 

To illustrate, if we have the following pair of sentences:
	
> John gave Mary a book
> 
> John gave a book to Mary

Then, the Paraop annotations for this sentence pair would be:

| John | gave | Mary   | a     | book   |
| :--: | :--: | :----: | :---: | :----: |
|  0   |  0   | 3      | 3     | 3      |

| John | gave | a     | book   | to    | Mary   |
| :--: | :--: | :---: | :----: | :---: | :----: |
|  0   |  0   | 3     | 3      | 1     | 3      |

The numbers below each word in the example above denote which Paraop operation applies to that word. For instance, 1 represents the addition of a function word, 3 represents a change of order, and 0 represents no change.

## Why annotate paraphrase operations?
Applications of paraphrase operation detection include:

- Data augmentation
- Machine translation
- Textual entailment detection
- Text summarization and simplification
- Plagiarism detection

## What resources are available?
The [Paraop repository on GitHub](https://github.com/lucasadelino/paraop) contains:
- The Paraop corpus
- Automatic Paraop classifiers

The Paraop corpus is based on the Extended Typology Paraphrase Corpus [(ETPC)](https://github.com/venelink/ETPC), which, in turn, is based on the Microsoft Research Paraphrase Corpus [(MRPC)](https://www.microsoft.com/en-us/download/details.aspx?id=52398).

The automatic Paraop classifiers are [BERT](https://arxiv.org/abs/1810.04805) models fine-tuned on the Paraop corpus.