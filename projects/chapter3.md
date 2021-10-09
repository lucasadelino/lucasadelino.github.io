---
title: Chapter 3 - N-gram Sentence Generator
layout: post
math: true
---
## What it is

A [bot](https://twitter.com/assis_bot) that uses an n-gram language model trained on the [Machado de Assis Digital Corpus](http://machado.byu.edu) to generate random sentences.

## The setup

[Chapter 3 of Jurafsky and Martin](https://web.stanford.edu/~jurafsky/slp3/3.pdf) introduces the reader to n-gram language models. The exercises at the end of the chapter prompt the reader to write a program to compute unsmoothed unigrams and bigrams, and to add an option in the program to generate random sentences and to compute the perplexity of a test set. Contrary to the [previous chapter]({% link projects/chapter2.md %}), this chapter does not provide the algorithm for implementing n-gram models, though it does provide formulas for calculating n-gram probabilities, which meant I would have to come up with the algorithm design myself. I also thought it would be nice to try to implement computations for any *n*-gram, not just unigrams and bigrams. Finally, I was particularly inspired by the following example, which the authors used to show how higher order n-grams perform better (in terms of modeling the corpus on which it was trained):

![higherngrams.PNG](/assets/higherngrams.PNG)

[*Jurafsky & Martin (2021), p. 11*](https://web.stanford.edu/~jurafsky/slp3/3.pdf#figure.3.4)

I thought this was the coolest thing I had seen in my NLP learning journey up to that point, so I wanted to implement a similar bot, but I wanted my bot to model a Brazilian author. Machado de Assis was my first choice, as it is a celebrated Brazilian author whose works are in the public domain. After googling, I came across the Machado the Assis Digital Corpus Project from Brigham Young University, which perfectly suited this project.

## The process

### Challenge #1: Sentence segmentation and tokenization

Before I could even start to think about the design of my n-gram algorithm, I first had to think about how I would segment approach sentence segmentation and tokenization of the corpus. I could (and definitely would, if this were a professional project!) use an establish tokenizer (like, say, [NLTK](http://www.nltk.org/)'s), but because this was a learning project, I thought it would be interesting to write my own (very crude) segmenter and tokenizer using regular expressions.

For sentence segmentation, I decided that I would use periods, question marks, and exclamation marks as sentence delimiters. My sentence segmenter basically returns a list of all matches of the regex below:

```python
sentence_regex = re.compile(r'(\S(?:.+?)[\.\?!]+)')
```

For tokenization, I wanted to have the option of either counting punctuation marks as tokens or not. I wrote a regex for each case and included a parameter in the function to take care of both cases. After finding tokens, the tokenizer converts all tokens to lowercase: 

```python
# Matches either a sequence of word characters or a punctuation mark
token_regex = re.compile(r'(\w+|[“”":;\'-\.\?!,]+)', re.UNICODE)

# Matches a sequence of alphabetic characters
word_regex = re.compile(r'(\w+)', re.UNICODE)

def tokenize(text, punctuation = True):
    token_list = []
    
    for sentence in text:
        if punctuation == True:
            token_list.append(token_regex.findall(sentence))
        else:
            token_list.append(word_regex.findall(sentence))
    
    # Case fold all words to lowercase
    for each_list in token_list:
        for i, word in enumerate(each_list):
            if word.isupper() or word.istitle():
                each_list[i] = word.lower()

    return token_list
```

Now that I knew exactly what my tokenizer would return (a list of lists), I could start thinking about my n-gram algorithm.

### Challenge #2: Designing the n-gram algorithm

I started by laying down all the steps required to implement the n-gram algorithm. For a given list (of lists) of tokens, the program would have to:

1. Add sentence start ```<s>``` and end ```</s>``` markers to each list of characters
2. Count the frequency of n-grams
3. Count the frequency of (n-1)-grams
4. Use steps 2. and 3. above to calculate n-gram probabilities for all n-grams in the list.

Let's look at how to approach each of these steps.

#### 1 - Add sentence start \<s> and end </s> markers

This step was relatively straightforward. For the probability distribution to work, we need to add $n - 1$ sentence start and end markers to our sentences, so for calculating trigrams, we would have: ```<s> <s> an example sentence </s> </s>```. We can write this algorithm as such:

```python
from copy import deepcopy

def add_markers(n, token_list):
    if n > 1:
        token_list = deepcopy(token_list)
        for i, sentence in enumerate(token_list):
            token_list[i] = (['<s>'] * (n-1)) + sentence + (['</s>'] * (n-1))

		return token_list
```

Our list of tokens passed as argument (token_list) is deep copied here to avoid changing it directly, since lists are mutable data types in Python.

#### 2 - Count the frequency of n-grams

The idea here is also simple: first, let's create a dictionary to contain our n-grams and their frequency counts. Now, we'll look at each n-gram in our lists of tokens. We'll then add this n-gram to our dictionary if it isn't already there (using Python's ```setdefault()``` dictionary method) and increase it's frequency count by 1.

But there's a small hitch: we want this to work for *any* $$n$$, which means we have to first form our n-grams, using whatever $$n$$ the user gave us. I knew I'd have to iterate over a list of tokens, but how would I set the parameters so that it worked for any $$n$$? Inspired by [this video](https://youtu.be/XYi2-LPrwm4?t=266) (which I came across while studying the [minimum edit distance algorithm]({% link projects/chapter2.md %}), my approach was to use a pointer (which we'll call $$p$$) to mark where we are in our list of tokens. We can use it to iterate over the preceding words and join them together to form an n-gram.

To figure out the starting values for the pointers, I drew two cases, one for a trigram and another for a 4-gram. In the tables below, there are two pointers, $$i$$ and $$p$$. Let's look at the trigram first $(n = 3)$. In our first iteration, we would have:

<div align="center">
<table>
    <thead>
        <tr>
            <th>&lt;s&gt;</th>
            <th>&lt;s&gt;</th>
            <th>she</th>
            <th>set</th>
            <th>out</th>
            <th>&lt;/s&gt;</th>
            <th>&lt;/s&gt;</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>0</td>
            <td>1</td>
            <td>2</td>
            <td>3</td>
            <td>4</td>
            <td>5</td>
            <td>6</td>
        </tr>
        <tr>
            <td>p</td>
            <td></td>
            <td>i</td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
        </tr>
    </tbody>
</table>
</div>

This would form the trigram ```<s> <s> she```. Now, for our 4-gram $(n = 4)$, in our first iteration would have:

<div align="center">
<table>
    <thead>
        <tr>
            <th>&lt;s&gt;</th>
            <th>&lt;s&gt;</th>
            <th>&lt;s&gt;</th>
            <th>she</th>
            <th>set</th>
            <th>out</th>
            <th>&lt;/s&gt;</th>
            <th>&lt;/s&gt;</th>
            <th>&lt;/s&gt;</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>0</td>
            <td>1</td>
            <td>2</td>
            <td>3</td>
            <td>4</td>
            <td>5</td>
            <td>6</td>
            <td>7</td>
            <td>8</td>
        </tr>
        <tr>
            <td>p</td>
            <td></td>
            <td></td>
            <td>i</td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
        </tr>
    </tbody>
</table>
</div>

This would form the 4-gram ```<s> <s> <s> she```.

Generalizing from the above, we can see that:

- $$i$$ starts at $2$ when $n = 3$, and at $3$ when $n = 4$. Therefore, $$i$$ always starts at $n - 1$.
- $$p$$ starts $(n - 1)$ steps behind $$i$$. Therefore, $p = i - (n - 1)$
