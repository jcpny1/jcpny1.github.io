---
layout: post
title:  "The Rails Portfolio Project"
date:   2017-05-08 11:25:55 -0400
---

This is the first do-it-yourself from scratch Rails coding project. The previous projects were kind of paint-by-the-numbers, where you’re completing a part of an existing project or being guided as to what to create. This project had a brief description of what the project needed to accomplish [Rails Assessment](https://learn.co/lessons/rails-assessment). For my project, I chose to develop a recipe catalog application.

I first did a quick survey of several recipe websites to get an idea of the kind of data that is maintained for recipes. I then came up with the data model.

![E-R Diagram](https://raw.github.com/jcpny1/recipe-cat/master/doc/Recipe Cat E-R Diagram.jpg)

Implementing the data model was straightforward. Setting up the routes was a little more challenging in that I needed multiple routes for the same data. Something we hadn't covered much in past lessons. I needed one route to implement filtering by a record's user id and one route without. One level of permission would grant a user to be able to see their own data and no one else's, and another to allow an admin to see anyone's data. In some cases, a regular user to is allowed to see another user's data (without having admin privileges).

I incorporated Devise and Pundit into my solution to accomplish authentications and authorizations. They are a nice way of compartmentalizing a lot of logic that is not germane to the core functionality of a recipe app.

During this exercise, the Learn IDE servers became unstable, so I created an Ubuntu system and migrated my work over to it from my Windows PC. That was quite a learning experience in setup and configuration. To a certain degree, the interoperability of so many gems can be fragile. I found that having continuous integration, a solid test suite, and making small, incremental changes between tests was key to making solid progress and not backsliding into debugging mysterious environmental issues.
