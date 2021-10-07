---
title: Chapter 4 - Naïve Bayes Sentiment Classifier
layout: post
---

## The setup:

[Chapter 4 of Jurafsky & Martin](https://web.stanford.edu/~jurafsky/slp3/4.pdf) introduces the reader to Naïve Bayes for sentiment classification. This was the first chapter so far that that did not have any exercises, so I knew I had to come up with my own. Fortunately, the chapter provided pseudocode for the Naïve Bayes classifier, so I thought that trying my had at implementing an NB classifier in Python would make for interesting practice. 

The chapter provided examples on how to use a Naïve Bayes classifier for analyzing the sentiment of movie reviews. I decided that I would test my implementation using movie reviews as well. To put my own spin on it, I decided to only use movie reviews written in Portuguese.

## The process:

### Challenge #1: Working with dictionaries to implement the classifier in Python

In the pseudocode, the function to train the Naïve Bayes classifier takes two arguments: a list of documents, and their associated classes. Since each list of documents would belong to a class, I decided that my Python function would take a dictionary as its argument in which class names would serve as keys and their document lists as values. Here's how that would look like, using an example from the textbook:

```python
class_dict = {'negative': [['just', 'plain', 'boring'], 
                           ['entirely', 'predictable', 'and', 'lacks', 'energy'], 
                           ['no', 'surprises', 'and', 'very', 'few', 'laughs']],
              'positive': [['very', 'powerful'], 
                           ['the', 'most', 'fun', 'film', 'of', 'the', 'summer']]}
```

Having decided on how to represent classes and documents, it was time to actually implement the algorithm. I quickly realized that the process of computing priors, calculating word counts for each class, and generating a vocabulary for all classes, would require the creation of more dictionaries. Some of these would be invisible (like the word counts by class), some would be returned (like the list of log priors for each class), but all of them would require a bunch of iteration to be created. This was a little confusing! I knew I would need to take a step back figure out how to do this without getting lost. I did this in the following ways: 

- I made several 'for' loops — even if they were redundant. I did this to teach myself what would need to happen in each iteration, so that later, once I had that figured out, I could collapse any redundant loops together.
- In comments throughout the code, I wrote example dictionaries just so I was 100% sure how each dictionary would be structured. These really helped! Whenever I was unsure of what to do in an iterator, I looked to them for reference.
- I used random variable names just to keep the flow of writing going. I knew these could be confusing, but stopping to think of a better name would interrupt my train of thought.

You can check out that first version of the code [here](https://git.io/Ja5xG).

### Challenge #2: Web scraping movie reviews in Portuguese

Once I had the Naïve Bayes classifier working, it was time to figure out how I would get my hands on a corpus of film reviews to train the classifier. For my n-gram project, I was fortunate enough to find a corpus on the internet. I hoped to do the same for this project, but if there are any corpora of movie reviews in Portuguese out there, I could not find any. This looked like a good opportunity to build my own. Plus, it would get me to practice web scraping with Python, which I knew virtually nothing about. 

First, I had to decide where I would get my data from. I first thought about using IMDB or Letterboxd, both of which have official APIs, but most of the film reviews in these platforms are written in English. I then set my sights on [Filmow](https://filmow.com/), a film-centered Brazilian social network. Filmow did not have an API, so I would have to come up with a way to scrape it from scratch.

![Filmow](/assets/filmowss.png){: width="988" height="909" } 
_Profile of Dune (2021) on Filmow_

I first asked for help from a [web developer friend](https://github.com/hugobrancowb) to make sense of Filmow's website using Firefox's Developer Tools (which I didn't know how to use that well; all I knew was how to find specific elements of the website using the inspector). My friend showed me how to use the dev tools' Console tab to run custom scripts and the Network tab to track requests sent to Filmow. 

Then, I studied the [Web Scraping chapter](https://automatetheboringstuff.com/2e/chapter12/) of Automate the Boring Stuff with Python, which introduced me to the [Requests](https://docs.python-requests.org/en/latest/) and [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) modules. 

With that, I felt I had enough knowledge to try my hand at scraping some comments off of Filmow. I first tried making a simple script using Requests to grab the HTML of the comments section of a Filmow page. I found out that if I gave the Requests module the link to such a page (like [this one](https://filmow.com/comentarios/22/284498/) for instance), the module returned an HTML with no comments at all. I didn't understand why, but this prompted me to have another look at the page using Firefox's dev tools. Looking at the requests received, I found that one of the requests was for a .JSON object, and that .JSON object was what contained the HTML code for the comments! What was more, the same kind of object was returned whenever I clicked on the "Load more comments" button in each comment page. I also found out I could get a direct link to that object, which looked like this on my browser:

![Json](/assets/jsonss.png){: width="300" height="400" } 
_Example of a JSON object in Firefox_

Now, at the time I didn't quite know what .JSON objects were, but after quick googling I learned that they behaved just like Python dictionaries. I also found that the Requests module could represent .JSON objects using the .json() method. That was enough to allow me to write a Python scraper.

I first wrote a simple scraper to find all the comments of a single film, which would eventually become [filmowcommentscraper.py](https://git.io/Jab1J). It took some tinkering to make sure it worked (reading the Requests and Beautiful Soup documentation helped a lot), but after that, I knew enough about scraping to very quickly write [another scraper](https://git.io/Jab5T). This one would look through the film catalog in Filmow and save a bunch of film codes (a 1 to 6-digit string of numbers that Filmow uses to identify each film). I saved the film codes of 2400 films, which I could then feed to the comment scraper to automatically get the comments of any number of films.

### Challenge #3: Testing the program

Since I wanted to use these comments to train a Naive Bayes classifier, I used the scraper to target only positive (3 out of 5 stars or more) or negative (2/5 stars or less) comments, which I saved in different .txt files, which you can find here: [positivo.txt](https://git.io/JaN4X) / [negativo.txt](https://git.io/JaNB0).

Now here's something interesting: after running the comment scraper on 100 movies, the .txt file for negative comments had about 1MB. Meanwhile, the file for positive comments had 21MB! This is because user ratings for films on Filmow tend to be more positive than negative on average. I knew that the Naive Bayes classifier took class size into account, but I fed the lopsided data to the classifier just to see what I could find. 

I did some testing by coming up with film reviews (I also asked my colleaguesmakes it favor the class that has more documents.