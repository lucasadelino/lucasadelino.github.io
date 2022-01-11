---
title: Chapter 4 - Naive Bayes Sentiment Classifier
layout: post
---
## What it is

A Naive Bayes classifier trained on a corpus of 105,195 movie reviews in Portuguese, built by scrapping user film reviews from [Filmow](http://filmow.com). 

## The setup

[Chapter 4 of Jurafsky & Martin](https://web.stanford.edu/~jurafsky/slp3/4.pdf) introduces the reader to Naive Bayes classifiers. The chapter provided a description of the algorithm for the classifier (in pseudocode), so I thought that trying my hand at implementing it in Python would make for interesting practice. 

The chapter provided examples on how to use the classifier for analyzing the sentiment of movie reviews. I decided that I would test my implementation using movie reviews as well, but, to put my own spin on it, I would use movie reviews written in Portuguese.

## The process

### Challenge #1: Working with dictionaries to implement the classifier

For reference, here's the algorithm provided by Jurafsky and Martin:

![nbalgorithm](/assets/nbalgorithm.PNG)
_[Jurafsky & Martin (2021), p. 6](https://web.stanford.edu/~jurafsky/slp3/4.pdf#figure.4.2)_

The function to train the Naive Bayes classifier takes two arguments: the list of documents **D**, and their associated classes **C**. While the pseudocode outlines the overall structure of the function, we still have to decide how to represent D and C in Python. Since each document necessarily belongs to a class, we can represent both D and C as a single dictionary in which each key is the name of a class and each value is the list of documents that belongs to that class. Each document is itself a list of words. Here's how that would look like, using an example from the textbook:

```python
class_dict = {'negative': [['just', 'plain', 'boring'], 
                           ['entirely', 'predictable', 'and', 'lacks', 'energy'], 
                           ['no', 'surprises', 'and', 'very', 'few', 'laughs']],
              'positive': [['very', 'powerful'], 
                           ['the', 'most', 'fun', 'film', 'of', 'the', 'summer']]}
```

Having decided on how to represent classes and documents, it was time to actually implement the algorithm. The process of computing priors, calculating word counts for each class, and generating a vocabulary for all classes would require the creation of more dictionaries. Some of these would be "invisible" (like the total number of words of each class), some would be returned by the function (like the list of log priors for each class), but all of them would require a bunch of iteration to be created. This was a little confusing! I knew I would need to take a step back figure out how to do this without getting lost. I did this in the following ways: 

- I made several 'for' loops — even if they were redundant. I did this to teach myself what would need to happen in each iteration, so that later, once I had that figured out, I could collapse any redundant loops together.
- In comments throughout the code, I wrote example dictionaries, just so I was 100% sure how each dictionary would be structured. These really helped! Whenever I was unsure of what to do in an iterator, I looked to them for reference.
- I used random variable names just to keep the flow of writing going. I knew these could be confusing, but stopping to think of a better name would interrupt my train of thought.

You can check out that first version of the code [here](https://git.io/Ja5xG).

### Challenge #2: Building a corpus of movie reviews in Portuguese

Once I had the Naive Bayes classifier working, it was time to find a corpus of film reviews to train the classifier. For my n-gram project, I was fortunate enough to find a corpus on the internet. I had hoped to do the same for this project, but if there are any corpora of movie reviews in Portuguese out there, I could not find them. This meant that I would have to build my own, which looked like a good opportunity to learn web scraping with Python (which I knew virtually nothing about). 

First, I had to decide where I would get my data from. My first instinct was [IMDB](http://www.imdb.com) or [Letterboxd](http://www.letterboxd.com), both of which have official APIs, but most user reviews in these platforms are written in English. I then set my sights on [Filmow](https://filmow.com/), a film-centered Brazilian social network. 

![Filmow](/assets/filmowss.png){: width="988" height="909" } 
_Profile of Dune (2021) on Filmow_

Filmow does not have an API, so I needed to think of a way to scrape the website from scratch. I first asked for help from a [web developer friend](https://github.com/hugobrancowb) to make sense of Filmow's website, using Firefox's Developer Tools (all I knew was how to find specific HTML elements in a website using the inspector). My friend showed me how to use the 'dev tools' Console tab to run custom scripts and the 'network' tab to track requests sent to Filmow. Then, I studied the [Web Scraping chapter](https://automatetheboringstuff.com/2e/chapter12/) of Automate the Boring Stuff with Python, which introduced me to the [Requests](https://docs.python-requests.org/en/latest/) and [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) modules. 

With that, I felt I had enough knowledge to try my hand at scraping some comments. I first tried to make a simple script, using the Requests module, to grab the HTML from the user reviews of a film page in Filmow. I found out that if I gave Requests the link to such a page (like [this one](https://filmow.com/comentarios/22/284498/) for instance), the module returned HTML with no comments at all. I didn't understand why, but this prompted me to have another look at the page using Firefox's dev tools. Looking at the requests received, I found that one of the requests was for a .JSON object, and that .JSON object was what contained the HTML code for the comments! What was more, I got the same kind of object whenever I clicked on the "Load more comments" button in each comment page. I also found out I could get a direct link to that object, which looked like this on my browser:

![Json](/assets/jsonss.png){: width="300" height="400" } 
_Example of a JSON object in Firefox_

At the time I didn't quite know what .JSON objects were, but after quick googling I learned that they behaved just like Python dictionaries, and that the Requests module could represent .JSON objects using the .json() method. This was enough to allow me to write the code.

I first wrote a [simple scraper](https://git.io/Jab1J) to find all the comments of a single film. It took some tinkering to make sure it worked (reading the Requests and Beautiful Soup documentation helped a lot), but after that, I knew enough about scraping to quickly write [another scraper](https://git.io/Jab5T). This one would look through the Filmow's film catalog and return a bunch of film codes (a 1 to 6-digit string of numbers that Filmow uses to identify each film). I saved the [film codes of 2400 films](https://git.io/JrrOA), which I could then feed to the comment scraper to automatically get the comments of any number of films.

### Challenge #3: Testing the program

Since I wanted to use these comments to train a Naive Bayes classifier, I used the scraper to only target the text from positive (3 out of 5 stars or more) or negative (2/5 stars or less) reviews. I saved these in different .txt files, which you can find here: [positivo.txt](https://git.io/JaN4X) / [negativo.txt](https://git.io/JaNB0).

Now here's something interesting: after running the comment scraper on 100 movies, the .txt file for negative comments had about 1MB. Meanwhile, the file for positive comments had 21MB! This is because user ratings for films on Filmow, on average, tend to be more positive than negative. I knew that the Naive Bayes classifier takes the number of documents in a class into account, but I was still curious to see how it behaved with such lopsided data. I tested the classifier using random reviews I wrote, and found that it would sometimes flag positive reviews as negative. For instance, it flagged this review as positive:

> "Esse filme é muito ruim" *(This movie is really bad)*
> 

Here's what I think is happening here: because there is so much more data in the 'positive' class, words such as 'ruim' *(bad)* occur often enough to make the classifier flag it as positive. In reality, the context in which 'ruim' occurs in positive comments can be something like "Este filme **não** é ruim" *(this film is **not** bad)*, but since the classifier only cares about how often words appear, it thinks 'ruim' is very likely to show up on a positive review.

To remedy this effect, I added more data to the negative reviews corpus (negative reviews from 500 films, positive reviews from 100 films). In the end, the data used to train the classifier had 26,932 negative documents and 78,263 positive comments. To test the classifier, I asked some friends what they thought about the last movie they had seen. The classifier more accurately predicted the sentiment of these reviews.