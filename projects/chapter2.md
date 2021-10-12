---
title: Chapter 2 - Minimum Edit Distance Calculator
layout: post
math: true
---
## What it is

A program that calculates the minimum edit distance between two strings. It supports custom operation costs and outputting alignments in either plain text or LaTeX.

## The setup

Towards the end of [Chapter 2](https://web.stanford.edu/~jurafsky/slp3/2.pdf), Jurafsky and Martin introduce the reader to the minimum edit distance algorithm. They provide a description of how the algorithm works (including pseudo code) as well as several examples. The end-of-chapter exercises ask the reader to implement the edit distance algorithm, and to with an option to the algorithm to output an alignment between two strings.

## The process

### Challenge #1 - Understanding the algorithm

Perhaps the most challenging part of this entire endeavor was wrapping my head around the idea of dynamic programming. It was the first time I had seen this approach, so reading the textbook wasn't enough; there was still a lot I didn't understand.

I decided to look for other sources. I searched for 'edit distance' on YouTube and came across these two videos:

<div id="yt-videos" align="center">
<iframe width="560" height="315" src="https://www.youtube.com/embed/XYi2-LPrwm4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/MiqoA-yF-0M" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

After watching each video a couple times, I felt better equipped to deal with the problem, but to gain a little more confidence, I drew edit distance matrixes by hand:

![WhatsApp Image 2021-09-25 at 1.47.26 PM.jpeg](/assets/ledadeal.jpeg){: width="300" height="400" }

![WhatsApp Image 2021-09-25 at 1.47.26 PM(1).jpeg](/assets/drivedivers.jpeg){: width="300" height="400" }

Now, using these edit matrixes (as well as the examples provided by the book), I felt ready to start coding.

### Challenge #2 - Implementing the algorithm

For reference, here's the pseudo code provided by the textbook:

![editdistalgorithm.PNG](/assets/editdistalgorithm.PNG)
*[Jurafsky & Martin (2021), p. 24](https://web.stanford.edu/~jurafsky/slp3/2.pdf#figure.2.17)*

My first approach was to implement this code as directly as possible, which you can find [here](https://github.com/lucasadelino/Learning-Compling/blob/main/Textbooks/Speech%20and%20Language%20Processing%20(Jurafsky%2C%20Martin)/Chapter%202%20-%20Regular%20Expressions%2C%20Text%20Normalization%2C%20Edit%20Distance/mineditdist.py). This gave me a solid foundation to build upon, which I used to implement the features below:

#### Storing pointers

First, I needed to modify my matrix to store pointers as well as values. These pointers would later be used to output an alignment. To see what that looks like, here's an example from the textbook:

![backtrace.PNG](/assets/backtrace.PNG)
*[Jurafsky & Martin (2021), p. 25](https://web.stanford.edu/~jurafsky/slp3/2.pdf#figure.2.19)*

Since each cell stores not only a number but a series of pointers, I changed my original matrix: instead of each item being a number, it would be a dictionary consisting of values and pointers, like so:

```python
{'number': 11, 'pointers': ['up', 'left']}
```

This would certainly be more confusing if I hadn't done that first version of the program! Thankfully, I had, so I was able to use most of the existing code with just a few modifications. One of these modifications was to have the function return not only the minimum edit distance cost but the edit distance matrix as well. We need that matrix for the functions below.

Something else that really helped (not only for this step but throughout the whole project) was using the textbook examples to recognize patterns. Thankfully this chapter provided many such examples, but one could use the same approach even if didn't[^1]. 

#### Generating alignments

For this one, I created another function, `align()`, which would take source and target strings and generate an edit distance matrix. Then, starting from the last value, the function would **backtrace:** look at the pointers in that cell and move in the corresponding direction until it arrived at the first cell. If there was more than one available pointer, the function would choose between them randomly.

That's the overall idea, but how to transform the backtrace into a set of strings? Ultimately, the result we want would look like this:

![alignment.PNG](/assets/alignment.PNG)
*[Jurafsky & Martin (2021), p. 22](https://web.stanford.edu/~jurafsky/slp3/2.pdf#figure.2.14)*

From looking at the end-result I knew the output would consist of three strings: one for the source (`i n t e * n t i o n`), one for the target (`* e x e c u t i o n`), and one for the operations performed `d s s   i s`). My approach was to generate these three strings *during* the backtrace: with each step towards the first cell, we use the pointer we chose to tell us what operations need to happen, which in turn tells us what we need to write for the source, target, and operation strings:

```python
if direction == 'up':
    source_output = source[i] + source_output 
    target_output = '*' + target_output
    operations = 'd' + operations
if direction == 'nw': # Diagonal up & left
    source_output = source[i] + source_output 
    target_output = target[j] + target_output
    if source[i] == target[j]:
        operations = ' ' + operations
    else:
        operations = 's' + operations
if direction == 'left':
    source_output = '*' + source_output
    target_output = target[j] + target_output
    operations = 'i' + operations
```

#### Implementing pretty printing

Last but not at least, I wanted an option for my program to print "pretty" versions of the edit distance matrixes, akin to the examples provided by the book. I wanted my pretty printer function to either output to plain text or LaTeX, and, if outputting to LaTeX, optionally output the pointers as well.

To do this, I once more took to the examples provided by the textbook. Consider this example:

![Untitled](/assets/matrix.PNG)
*[Jurafsky & Martin (2021), p. 24](https://web.stanford.edu/~jurafsky/slp3/2.pdf#figure.2.18)*

In the example, the source and target strings are *part* of the table as well. I decided to do the same for my pretty printer: I'd take the matrix generated by `min_edit_dist()` and add to it an extra row (for the target string) and column (for the source string)[^2]. To make this less daunting, I implemented one feature at a time. The [first version](https://github.com/lucasadelino/Learning-Compling/commit/564c0b1d1e983a4ceb3df57f192f61cb53ffe1a2#diff-31942b37ac4a742902375197872524e4bf4340472e4b264796e4a0b2852cd4b7) outputted only to LaTeX; outputting [plain text](https://github.com/lucasadelino/Learning-Compling/commit/97715b71466a2fe49531a1aaed2ba1f41f598083#diff-31942b37ac4a742902375197872524e4bf4340472e4b264796e4a0b2852cd4b7) and [pointers](https://github.com/lucasadelino/Learning-Compling/commit/ad79e45cab2eb68140b97a9a2202fb76364598ca#diff-31942b37ac4a742902375197872524e4bf4340472e4b264796e4a0b2852cd4b7) was implemented later.

[^1]: Which is what happened in [Chapter 3]({% link projects/chapter3.md %}): the textbook provides formulas for calculating n-grams but does not provide an algorithm. In this case, I came up with the examples myself (like the [tables]({% link projects/chapter3.md %}#count-the-frequency-of-n-grams) I used to map out iteration)
[^2]: Actually, what I did was to initialize an empty matrix, append the first row (the target string), and then append the remaining rows (consisting of a letter from the source string + the value of the current cell in the edit distance matrix). It's much easier do to this by converting the source and target strings to lists (don't forget to add an empty character!). Again, the examples greatly helped me figure this out!
http://localhost:4000/projects/chapter3.html