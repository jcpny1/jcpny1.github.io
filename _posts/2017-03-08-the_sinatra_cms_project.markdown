---
layout: post
title:  "The Sinatra CMS Project"
date:   2017-03-08 18:56:54 +0000
---

This is the first comprehensive Sinatra coding project. It brings together Ruby, Sinatra, ActiveRecord, and a little HTML. The assignment is to design and implement a basic CRUD application. It needs to include a one-to-many relationship between two of the data models.

Once I picked a domain and created a rough design (modeling merchandise purchases), the database setup was complete in short order. I created some seed data so I'd have some data to work with quickly while setting up views and routes. Where things started to get bogged down, was when I decided to spruce up the html a bit. That's a process that I find seems to have no end - You eventually have to stop fiddling with the code and convince yourself that good enough is good enough!

Besides the html editing, I seemd to have run across a glitch in Sinatra. A route with more than one part causes the css stylesheet to be partly ignored. For example, if my route was '/purchases', my web page displayed properly. If my route was '/purchases/:id', it would would not render the proper containers and their positioning. I tried a variety of changes to isolate the problem, but wasn't able to see any difference that would cause the corruption, other than have a multi-part path. I could have left the bad formatting, but it bothered me too much. So, where any get route had multiple parts (e.g., /purchases/:id/edit), I packed the appropriate data in the session variable and redirected to a single part path (i.e., /purchases) for the proper processing. Further diagnosing will have to wait for another day - the project must move forward.

Another thing that surprised me, was how vulnerable to hacking protected data could be by what would seem to be things easy to overlook, such as forgetting to check, at every route, that the user is logged in and that the user is authorized to view the data being requested.

I found tux and pry to be indispensable during development. Tux to review the proper data model setups, and pry to check on parameter formats being passed back from the html forms.

