---
title: Site
layout: nonpost
---

## What it is

## The setup

I created my [previous portfolio](https://lcsadelino.wixsite.com/eportfolio) using [Wix](http://wix.com/), a simple online website builder. Wix's greatest pro is also its greatest con: it is What You See Is What You Get ([WYSIWYG](https://en.wikipedia.org/wiki/WYSIWYG)). This means that anyone can build a Wix website, since you don't need to know how to code. It also means that Wix isn't as customizable, easy to update, or rich in features as non-WYSIWYG platforms. Moreover, setting up a custom domain for a Wix website requires you to sign up for their (relatively expensive) premium plan. Since I was learning more about coding, I thought I could take another shot at doing a portfolio (and use a custom domain for a fraction of the price).

Two things pushed me to use the platform that I used: 

- First, I discovered that Notion (which is what I used to write this) uses Markdown for most of its formatting. Whatever I wrote in Notion, I could copy and paste into a code editor and it would look almost exactly the same, which made writing, say, readme files in GitHub a breeze.
- Then, I found out that GitHub pages uses Jekyll. Once I realized that I could use Jekyll to easily turn my markdown files into a website, I knew that's what I would use. I already knew some (very basic) HTML and CSS, but after completing my web scraping project, I felt even more confident in playing around with it.

## The process

### Challenge #1: Learning Jekyll

First, I had to familiarize myself with Jekyll, which I did by watching this entire series by [Giraffe Academy](https://www.youtube.com/c/GiraffeAcademy):

<div align="center">
<iframe width="400" height="225" src="https://www.youtube.com/embed/videoseries?list=PLLAZ4kZ9dFpOPV5C5Ay0pHaa0RJFhcmcB" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

This introduced me to the basic of Jekyll such as front matter, layouts, variables, and includes. 

### Challenge #2: Customizing the theme

With a basic understanding of Jekyll, I set upon finding a suitable theme. I specifically looked for sidebar themes to make it different from my last portfolio. I eventually came across [Chirpy](https://chirpy.cotes.info/), which was just like I wanted.

But there were some specific things in Chirpy that I didn't want. I liked that it emphasized writing, but I didn't necessarily want my site to have the look and feel of a blog. I wanted to get rid of most of the features that contributed to this, like searching, pagination, and comments, but I did want to preserve some stuff, like the table of contents and estimated reading times. I also wanted to change the way the breadcrumb navigation worked.

But how to make changes to Chirpy's layouts? Chirpy differed from what I had seen in the Jekyll tutorial videos in an important respect: it doesn't copy all of its layouts when using the [Chirpy starter](https://chirpy.cotes.info/posts/getting-started/#installation) to generate a website. It still accesses those layouts somehow, but they don't have to be on the repository. So if I wanted to modify the layouts, what would I do?

My first thought was not to modify them but to create new layouts entirely. This didn't play nice with Chirpy (for instance, it broke the way images were displayed). I figured investigating what broke and why (and how to fix it) seemed much more complex and prone to errors than just modifying the original layouts, so I discarded this idea.

Rereading Chirpy's documentation, I came across [these instructions](https://chirpy.cotes.info/posts/getting-started/#customing-stylesheet) on how to customize the style sheet. This gave me an idea. What if I used that same process to change the layouts? I found out that, in [Chirpy's GitHub page](https://github.com/cotes2020/jekyll-theme-chirpy), I had access to the original layouts and includes. What if I copied those to my directory, using the same file paths and names, and modified them? Would the changes work?

Well, they did! So my next step was to carefully read the includes and layouts to find what I wanted to remove. If I needed help locating a specific element in the HTML, I would load up the Chirpy example website and use Firefox's Developer Tools to inspect that element. If something broke, I could always redownload the original include or layout and try again.