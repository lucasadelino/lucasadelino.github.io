---
title: Chapter 3 - N-gram Sentence Generator
layout: post
math: true
---
## What it is

A [bot](https://twitter.com/assis_bot) that uses an n-gram language model trained on the [Machado de Assis Digital Corpus](http://machado.byu.edu) to generate random sentences.

## The setup

[Chapter 3 of Jurafsky and Martin](https://web.stanford.edu/~jurafsky/slp3/3.pdf) introduces the reader to n-gram language models. The exercises in chapter prompt the reader to write a program to compute unsmoothed unigrams and bigrams, and to add an option to that program to generate random sentences. As opposed to the [previous chapter]({% link projects/chapter2.md %}), this chapter does not provide the algorithm for implementing n-gram models (though it does provide formulas for calculating n-gram probabilities), which meant I would have to design the algorithm myself. I also thought it would be nice to try to compute any *n*-gram, not just unigrams and bigrams. 

I was particularly inspired by the following example, which the authors used to show how higher order n-grams perform better (in terms of modeling the corpus on which it was trained):

![higherngrams.PNG](/assets/higherngrams.PNG)

[*Jurafsky & Martin (2021), p. 11*](https://web.stanford.edu/~jurafsky/slp3/3.pdf#figure.3.4)

I thought that was the coolest thing I'd seen while learning NLP so far. I wanted to implement a similar bot, but I wanted my bot to model a Brazilian author. Machado de Assis was my first choice because his works are in the public domain and after some searching I found the [Machado the Assis Digital Corpus Project](http://machado.byu.edu) from Brigham Young University, which perfectly suited this project.

## The process

### Challenge #1: Sentence segmentation and tokenization

Before I could even start thinking about the design of my n-gram algorithm, I first had to consider how I would approach sentence segmentation and tokenization. I could (and definitely would, if this were a professional project!) use an established tokenizer (like, say, [NLTK](http://www.nltk.org/)'s), but since this was a learning project I thought it would be interesting to write my own (very crude) segmenter and tokenizer using regular expressions.

For sentence segmentation, I decided to use periods, question marks, and exclamation marks as sentence delimiters. The segmenter basically returns a list of all matches of the regex below:

```python
# Matches everything before a period, question mark, or exclamation mark.
sentence_regex = re.compile(r'(\S(?:.+?)[\.\?!]+)')
```

For tokenization, I wanted to have the option of either counting or not counting punctuation marks as tokens. I wrote a regex for each case and added a parameter in the function to signal which one to use. Before it returns, the tokenizer converts all tokens to lowercase: 

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

Note that this tokenizer doesn't do a great job of parsing clitic contractions (as in *don't*), but since Portuguese those are comparatively rarer in Portuguese, it gets the job done.

Now that I knew exactly what my tokenizer would return (a list of lists), I could start thinking about my n-gram algorithm.

### Challenge #2: Designing the n-gram algorithm

Before writing any code, I tried to consider all the steps necessary to implement the n-gram algorithm. For a given list (of lists) of tokens, the program would:

1. Add sentence start ```<s>``` and end ```</s>``` markers to each list of characters
2. Count the frequency of n-grams
3. Count the frequency of (n-1)-grams
4. Use steps 2 and 3 to calculate n-gram probabilities for all n-grams in the list.

Let's look at how to approach each of these steps.

#### Add sentence start \<s> and end </s> markers

This step was relatively straightforward. For the probability distribution to work, we need to add $n - 1$ sentence start and end markers to our sentences. For instance, so for calculating trigrams, we would have: ```<s> <s> an example sentence </s> </s>```. We can write the algorithm as such:

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

#### Count the frequency of n-grams

The idea here is also simple: first, let's create a dictionary to contain our n-grams and their frequency counts. Then, we'll look at each n-gram in our lists of tokens. We'll add this n-gram to our dictionary if it isn't already there (using ```setdefault()```) and increase it's frequency count by 1.

But there's a small hitch: we want this to work for *any* $$n$$, which means we first have to form n-grams using whatever $$n$$ given. I knew I'd have to iterate over a list of tokens, but how would I set the parameters so that it worked for any $$n$$? Inspired by [this video](https://youtu.be/XYi2-LPrwm4?t=266) (which I came across while studying the [minimum edit distance algorithm]({% link projects/chapter2.md %}), my approach was to use two pointers (which we'll call $$i$$ and $$p$$) to mark where we are in our list of tokens. We can use them to iterate over the preceding words and join them together to form an n-gram.

To figure out exactly how this would work, I drew two cases on paper: one for a trigram and another for a 4-gram. Let's look at the trigram $(n = 3)$ first. In the first iteration, we would have:

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

$$p$$ would then iterate until it met $$i$$, which would form the trigram ```<s> <s> she```. Then, $$p$$ would return to it's starting position and both $$p$$ and $$i$$ would increase by $1$, so that we can form the next trigram (```<s> she set```). 

For our 4-gram $(n = 4)$, in our first iteration would have:

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

We can generalize from the above to get the starting values of $$i$$ and $$p$$:

- $$i$$ starts at $2$ when $n = 3$, and at $3$ when $n = 4$. Therefore, $$i$$ always starts at $n - 1$.
- $$p$$ starts $(n - 1)$ steps behind $$i$$. Therefore, $$p$$ always starts at $i - (n - 1)$

We're almost ready to write our function! We just need one more thing: a parameter that tells the function to add (or not) `<s>` and `</s>` markers to the list of tokens before forming and counting n-grams. The final result looks like this:

```python
def ngram_count(n, token_list, markers=True):

if markers == True:
        token_list = add_markers(n, token_list)

ngram_dict = {}

    for sentence in token_list:
        for i in range((n - 1), len(sentence)):
            # Look at last (i - (n-1)) words for concatenation
            p = i - (n - 1)
            # Form an ngram by concatenating last (i - (n-1)) words, up to i
            ngram = ''
            while pointer <= i: 
                ngram += sentence[p] + ' '
                pointer += 1
            # Add ngram to dict
            ngram_dict.setdefault(ngram, 0)
            ngram_dict[ngram] += 1
    
    return ngram_dict
```

#### Calculate probabilities

Now that we can count n-grams, it's time to start working on a function to calculate probabilities. Again, the basic steps are:

1. Count the frequency of n-grams
2. Count the frequency of (n-1)-grams
3. Use the above to calculate probability

We need to do this for all n-grams in a list of tokens. Once more, the idea is simple, but we need to pay attention to a couple of things[^footnote]: 

* We must add the markers to our list of tokens **only once**. Forgetting this would add extra markers to our list and throw off the computation. We'll manually add the markers using `add_markers()` and remember to set `markers=False` whenever we call `ngram_count()`. 
* We need to consider what to do for unigrams $(n = 1)$. In this case, we need to divide the how many times the unigram appears by the total amount of tokens. Let's create a short function to count the number of tokens in a list:

```python
def get_token_count(token_list):

    token_count = 0
    for value in ngram_count(1, token_list).values():
        token_count += value

    return token_count
```

Now we can write our function:

```python
def ngram_prob(n, token_list):

    ngram_prob = {}

    # Add markers
    token_list = add_markers(n, token_list)

    # Get the counts of ngrams of the same n passed as argument
    ngrams =  ngram_count(n, token_list, markers=False)
    # Get the count of lower-order ngram OR total number of tokens if n == 1
    if n > 1:
        lower_ngrams =  ngram_count(n - 1, token_list, markers=False)
    elif n == 1:
        lower_ngram_count = get_token_count()

    for key, count in ngrams.items():
        if n > 1:
            # Lower order ngram = current ngram minus its last word
            lower_ngram_key = key.rsplit(' ', 1)[0]
            lower_ngram_count = lower_ngrams[lower_ngram_key]
        ngram_prob.update({key: (count / lower_ngram_count)})
    
    return ngram_prob
```

We're not using log probabilities here to make the next step a little easier.

### Challenge #3: Generating sentences

Now that our program can count n-gram probabilities, we can use the `ngram_prob` dictionary the function above returns to generate sentences. The idea for this is not as straightforward as the previous functions, so let's take it piece by piece. 

Before we begin, let's get the value of our $n$. We're getting a dictionary of n-gram probabilities, so we don't need to ask the user to manually type the $n$ when calling the function; we can get that from the dict:

```python
n = len(list(ngram_probs.keys())[0].split(' '))
```

#### Generating the first n-gram

The first thing we'll need to do is choose our first n-gram. Since we're starting a sentence, we must only consider n-grams that start that start with $n-1$ `<s>` markers:

```python
next_keys = []
next_values = []

for k, v in ngram_probs.items():
    if k.startswith(('<s> ' * (n - 1)).rstrip()):
        next_keys.append(k)
        next_values.append(v)
# Choose ngram according to its probability
sentence = choices(next_keys, weights=next_values)[0]
```

Note that we can only use `choices()` with `next_values` as weights because we didn't use log probabilities in our `ngram_prob` function.

Let's also take note of the last word in the n-gram we just generated:

```python
last_word = sentence.rsplit(' ', 1)[1]
```

#### Generating remaining n-grams

Now, let's generate the rest of the sentence. The idea is to keep generating n-grams, using the last word of the n-gram we just got as the first word of the next n-gram. Let's start with by taking note of n-grams whose first word == `last_word`:

```python
next_keys = []
next_values = []

for k, v in ngram_probs.items():
    if k.startswith(last_word):
        next_keys.append(k)
        next_values.append(v)
```

We'll again choose an n-gram according to its probability. We'll add it to the sentence **without** its first word (since it's already in the sentence). To keep the loop going, we'll again take note of the last word of the n-gram we chose:

```python
next_ngram = choices(next_keys, weights=next_values)[0]
sentence += ' ' + next_ngram.split(' ', 1)[1]
last_word = next_ngram.rsplit(' ', 1)[1]
```

We'll keep repeating this process until we generate an n-gram with a `</s>` marker. If there are any `</s>` markers missing when the loop stops, but we can add them to our sentence (or, if you want the output to not contain any `<s>` and `</s>` markers, you can tweak the code below to do so):

```python
endmarker_regex = re.compile(r'</s>')

sentence +=  ' </s>' * ((n - 1) - len(endmarker_regex.findall(sentence)))
```

That's it! Now, adding it all together:

```python
# Matches en dashes surrounded by word characters. Useful for PT-BR parsing
endmarker_regex = re.compile(r'</s>')

def generate_sentence(ngram_probs):
    # Look at keys in ngram_probs to figure out what's the order of our ngrams
    n = len(list(ngram_probs.keys())[0].split(' '))

    # Look for 1st ngram. Consider only ngrams that start with n-1 <s> markers
    next_keys = []
    next_values = []
    for k, v in ngram_probs.items():
        if k.startswith(('<s> ' * (n - 1)).rstrip()):
            next_keys.append(k)
            next_values.append(v)
    # Choose ngram according to its probability
    sentence = choices(next_keys, weights=next_values)[0]
    # Next ngram must start with the last word of this ngram
    last_word = sentence.rsplit(' ', 1)[1]
    
    # Keep generating sentences until we get an n-gram with an </s> end marker 
    # Loop stops after the first </s>; we'll add the remaining markers later 
    while '</s>' not in last_word:
        # Look for next ngram.
        next_keys = []
        next_values = []
        for k, v in ngram_probs.items():
            if k.startswith(last_word):
                next_keys.append(k)
                next_values.append(v)
        
        # Choose ngram. Don't add first word since it's already in the sentence
        next_ngram = choices(next_keys, weights=next_values)[0]
        sentence += ' ' + next_ngram.split(' ', 1)[1]
        last_word = next_ngram.rsplit(' ', 1)[1]

    # Add missing sentence end markers
    sentence +=  ' </s>' * ((n - 1) - len(endmarker_regex.findall(sentence)))

    return sentence
```

Now all we need to do is generate a dictionary of n-gram probabilities from the entire Machado de Assis corpus, and use that dictionary to generate sentences! Here's the [final version of the code](https://t.co/YCUmayR2cx?amp=1), including everything described so far.

[^footnote]: It's worth nothing that, as one can see by the [commit history of this project](https://github.com/lucasadelino/Learning-Compling/commits/main/Textbooks/Speech%20and%20Language%20Processing%20(Jurafsky%2C%20Martin)/Chapter%203%20-%20N-gram%20Language%20Models/ngram.py), I didn't realize these things at first! In particular, I was unsure when to add markers in the program. My [first instinct](https://github.com/lucasadelino/Learning-Compling/commit/1666dda9ac84e0d4861075d43d6809c0cb340c0a#diff-8fffd6dea253d4cfd6e5c12dc954dbf9ef2f31dd2a18fa5572fa1ffaa7700621) was to do it inside `tokenize()`. That works, but it requires the user to specify the value of $n$ twice (once when calling `tokenize()` and once when calling `ngram_prob()`). I also [tried to do it](https://github.com/lucasadelino/Learning-Compling/commit/98bf0bb41462fbe6e6e5ee6b63602fbb8f864bbc#diff-8fffd6dea253d4cfd6e5c12dc954dbf9ef2f31dd2a18fa5572fa1ffaa7700621) inside `ngram_count()` automatically, which also works, but only because it modifies the list passed into it (since Python lists are mutable), which struck me as less then ideal. I eventually decided to create a separate function to add the markers, add a parameter to `ngram_count()` to make adding markers optional, and manually add markers in the `ngram_prob()`, which how it's process described in the main text.