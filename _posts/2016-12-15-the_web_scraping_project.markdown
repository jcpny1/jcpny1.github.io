---
layout: post
title:  "The Web Scraping Project"
date:   2016-12-14 23:38:17 -0500
---

This is the first do-it-yourself from scratch Ruby coding project. The previous projects were kind of paint-by-numbers, where you’re completing a part of an existing project or being guided as to what to create. They also had a fairly complete test suite to guide what you coded. This project had none of that, except a brief description of what the project needed to accomplish - In this case, to get some data from a public website of your choosing, place that data in well thought out objects, display it in a summary fashion, and provide the ability to select an item for a detailed view. For my project, I chose to use classified ads.

I first did a quick survey of several classified websites, like my local newspaper, autotrader.com, and kbb.com. What I quickly discovered was, that although the websites presented an orderly view of a wealth of information, under the covers, things were a lot different. Each website had some HTML that was very well defined and some HTML that poorly tagged or well ordered. I tried to find websites that had very specific tags for the data types I was looking for. For example, one website would have a Price class for the sale price of an item, but then use free-form text in generic header tags for other things, like vehicle’s make and model. Some would have the word make or model in the free-form text and others wouldn’t. In the end, I found no website that did it all. So I just picked two and forged ahead.

My concept was that a listing could be selling anything and that various subclasses would handle the attribute differences between a boat and a car, for instance. By scraping a boats-for-sale website and a cars-for-sale website, I was going to get vastly different website data than if I had chosen a single website that offered both. By scraping both, I would also be able to prove out a class design that could, in the future, effectively handle any type of item with minimal incremental effort.

I won’t bore you with all the details, but will say that scraping more than just trivial data elements is a very tedious endeavor. I suspect the designers of these sites did not take the ease-of-scraping effort into consideration when they designed the HTML output `</sarcasm>`.

The following paragraphs describe some of the lessons learned -

### *Fragility*
The code to scrape a website is very fragile. Except for cases where your data is tied directly to a specific CSS selector, e.g. `<span class = ”price”> $10,000 </span>`, your code will be very specific to how the data is laid out. For example, `<h1>2017 Cadillac Escalade in 4339 Hempstead Turnpike, Farmingdale, New York 11735</h1>`. If your parsing data elements from that string in it's particular position in the document, and the site makes any changes to the format, your code will likely break. If your broken code causes an exception, at least you’ll know when it’s broken. If it fails silently, you may not know that you’re not getting the all data you are counting on getting. If you’re performing metrics on incomplete data, you’ll have wrong answers. On the plus side, scraping is a way to get a large amount of real-world data into your program for other purposes (such as POC, class design, etc.) that would be impractical to enter by hand.

### *Namespaces*
My project didn’t seem to lend itself to needing its own namespace, but I thought it would be a good thing to do as I was going to package it as a gem. So I went and added `Classified::` to all my class definitions. This was a simpler notation then putting `module Classified` verbiage in each class file and indenting the contents.

At that point, Ruby starting requiring that I qualify all my class references with that namespace name, even though all my classes were in the same namespace. That was a real nuisance. Later, I discovered that wouldn’t have been necessary if I had used the `module Classified` method instead.

### *The CLI*
Coding even a less-than-robust command line interface from scratch is not fun. A lot of time was spent formatting data elements for a decent looking display. (I gained a new appreciation for GUI-based apps.)

Using `#gets` to input a string was a little odd. If I typed 123, but meant to type 456, and therefore backspaced, this is what the line looked like: 123\321/456. I searched online and found a way to prevent that, but didn’t have the will to implement it.

### *Ruby Coding*
This project had enough functionality that I was able to get a lot of practice with Ruby. I also did quite a bit of web browsing finding better ways to do things that I had been doing in a very non-Ruby way. Even little things, like using the step enumerator and the ‘…’ range operator, were an important learning experience.

* Replaced

```ruby
        index = 0
        while (index < dl_tag.size)
           index += 2
        end
```
* with

```ruby
        (0...dl_tag.size).step(2) { |index| }
```

I also found that I need to learn how to use a Ruby debugger that I can step through code with. Pry is nice, but to keep moving the pry call while not being able to step through the code is a real time sink. Also, calling pry inside a major loop is not a great idea. It might get hit a hundred times, when all I wanted was once or twice. Ruby -rdebug looks too crude to be a lot of help. Perhaps there's something better than what was around in 1983 (sdb). I'll have to investigate.

