---
title: Chapter 5 - Logistic Regression
layout: post

---

## What it is

A [logistic classifier](https://github.com/lucasadelino/Learning-Compling/tree/main/Textbooks/Speech%20and%20Language%20Processing%20(Jurafsky%2C%20Martin)/Chapter%205%20-%20Logistic%20Regression) trained on a corpus of 105,195 movie reviews in Portuguese, built by scrapping user film reviews from [Filmow](http://filmow.com) (the same corpus used in the [previous chapter]({% link projects/chapter4.md %})). 

## The setup

[Chapter 5 of Jurafsky & Martin](https://web.stanford.edu/~jurafsky/slp3/5.pdf) introduces the reader to logistic regression classifiers. The chapter uses these classifiers to teach foundational machine learning concepts like loss functions, optimization, and regularization. I devoted special attention to this chapter because it taught such important concepts; implementing this classifier was one of the ways to make sure I got everything right. What I really wanted form this project was to get a feel for _how_ a logistic classifier worked (and especially how gradient descent worked in practice).

## The process

### Challenge #1: Understanding the underlying math

The first real hurdle for me was ensuring I was really familiar with the mathematical foundations behind logistic regression and gradient descent. The chapter presents these concepts in a remarkably clear and succinct manner, but I didn't possess all the mathematical knowledge to grasp those concepts immediately. To get up to speed, I found a Youtube channel called [StatQuest](https://www.youtube.com/channel/UCtYLUTtgS3k1Fg4y5tAhLbw) that explained everything very clearly. The most important videos were their linear and logistic regression series, which introduced me to the ideas of fitting a line to data:

<iframe width="400" height="225" src="https://www.youtube.com/embed/PaFPbb66DxQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Another important video was their gradient descent guide:

<iframe width="400" height="225" src="https://www.youtube.com/embed/sDv4f4s2SB8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

I watched those videos each at least twice; you can find the notes I took while watching them [here](https://github.com/lucasadelino/Learning-Compling/blob/main/Textbooks/Speech%20and%20Language%20Processing%20(Jurafsky%2C%20Martin)/Chapter%205%20-%20Logistic%20Regression/statquestnotes.md). 

### Challenge #2: Choosing features 

The Naive Bayes classifier of the last chapter simply compared how often a word appeared in each class to make a classification decision. In contrast, the Logistic regression uses much more sophisticated features, so I wanted to have a go at implementing them. 

I first looked for what features were commonly used in movie review classifications. When I couldn't find much information on that front, I decided to use the same features used in the chapter:

![features](/assets/features.png)

_[Jurafsky & Martin (2021), p. 5](https://web.stanford.edu/~jurafsky/slp3/5.pdf#subsection.5.2.1)_

Implementing the features 3 through 6 above would be relatively simple, but for the first two features, I would need a sentiment lexicon in Portuguese. I found an article by Freitas and Vieira (2015)[^1] describing comparing several Portuguese sentiment lexicons. I chose to use [OpLexicon](https://www.inf.pucrs.br/linatural/wordpress/recursos-e-ferramentas/oplexicon/) since it was comparatively leaner relative to the other lexicons. 

[^1]: Freitas, L. A., Vieira, R. (2015). Exploring Resources for Sentiment Analysis in Portuguese Language. *2015 Brazilian conference on intelligent systems (BRACIS)* , 152-156. https://doi.org/10.1109/BRACIS.2015.52



