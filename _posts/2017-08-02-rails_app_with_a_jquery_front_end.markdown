---
layout: post
title:  "Rails App with a jQuery Front End"
date:   2017-08-02 19:27:11 +0000
---


For this project, we were to take our pre-existing Rails app (written without using JSON or Ajax), and rewrite all or part of it using JSON and Ajax. My previous Rails project was Recipe Cat, a recipe catalog application. It makes extensive use of Embedded Ruby (ERB) pages and is pretty tightly coupled with the backend controllers. We were also asked to reuse the same GitHub project instead of starting a new one.

I created a branch off of master in which to perform the modifications, so as to have the working master branch available for reference. Since this is also a learning exercise, I felt that completely rewriting the back and front ends to support the new model would incur a lot of time and a lot of coding without really knowing if I was going down the right design path. So instead, I opted to select portions of the functionality at a time to convert.

Some lessons learned:

No JavaScript syntax warnings equals time consuming debugging; Especially if lots of line changes were made in multiple places. Missing a parenthesis or something that another language would easily pick up and warn about caused a lot of trouble. I wound up finding an online code checker tool for JavaScript, and for HTML, that also allowed for Handlebars and ERB syntaxes, that helped to detect some hard to find errors.

After the conversion, I had quite a few leftover forms, routes, policies, etc. It was time consuming, and a bit error-prone, to go through the code to validate that pieces are no longer in use and can be safely deleted. In a professional environment, test-drive development would have picked up if I deleted a still required route or form, rather than me having to exercise the code manually and hoping to hit all the right places. I added some controller and feature tests, just to get some experience. But, I did not have the time to create a complete test suite. This experience highlighted the importance of creating tests as functionality is added, if not beforehand.

The project was made harder by not having a fully formed design plan in place. As this was a learning experience, I didn’t know what the design should look like until I tried out various approaches. A more professional approach would have been to have fleshed out a fully RESTful back end API, implement it, then cleanly re-implement the front end to use it. I felt going with that method would have had me spend too much time coding potentially unworkable ideas that would have taken a long time to back out and do over. The project is now in a hybrid state. Some functionality is using Ajax and some is still using ERB, (where Ajax could be used). I’m not happy with that, but the lessons objectives have been met.

I found the need to replicate validations on the front-end that were already done on the back end (and could be used in ERB code) troubling. For example, the ability of a user to edit a record only if that user had permission to edit the record. The logic to determine if the user had that permission was already coded in the back end, and now needed to be coded in the front end as well. I needed to add data to the back end JSON response so that the front end could determine if the user had editing permission. When using ERB, I could just call a Rails helper function from within the form. I suspect this problem has been solved, and that we will cover the solution in future lessons.

I wanted to preserve the state of the display when a user navigated away from the page then back again. For example, if a list of ingredients for a recipe was expanded on the recipe show page, I wanted that state to be preserved as the user navigated to the ‘next’ and ‘previous’ recipes. My first thought was to create a status flag and hide it somewhere on the page (on a part of the page that doesn’t change from one recipe to the next). Then, I would examine that value as each recipe is displayed, and adjust the display as required, keeping it in the same state. As it stands now, if you wanted to bounce through the recipes looking at their ingredients list, you would have click in each recipe’s ingredients link to expand the ingredients list. (The default is to not show the ingredients list.)

I wanted to add functionality such that if any data was changed, and the user attempted to navigate away before saving, a warning would be displayed. This also seems like functionality that would be included in some other package and not need to be hand-coded.

Adding object-orientation at the end of a project is not a good idea. I added a Recipe class to the app, but it was hard to integrate it in a meaningful way. Object orientation needs to be designed-in, not added on.

In a similar fashion, good error handling needs to be built-in, not added-on. I tried to catch and display errors wherever they could occur. No effort was made to logically handle the error though. For example, what should happen when a record is displayed for editing, but not all the record’s data was loaded to be edited? Probably, the edit should not be allowed to proceed.

I found Active Model Serialization to be a bit fragile. If I modified the serializer class to include additional data (by using a has_many attribute, for example), it seemed to shift around how the serialized values were located in the resulting object, breaking the frontend code. Also, I added seven serializers without any problems. However, when I went to add an eighth one, I couldn’t configure it to serialize more than the record’s id. If I deleted the serializer altogether, I got all the data. So, for now, I just left it out and let Active Model Serializer provide the default serialization.

A recipe ingredient record consists of an ingredient_id, a quantity, and a unit_id. Converting the ingredient and unit id’s to names seems to require that the recipe ingredient serializer pass along the corresponding ingredient and unit records as separate objects, requiring me to load them in the front end and perform the id-to-name lookups myself. That seemed awkward. There may be a better way, but I didn’t find one in the time allotted.

Since this is intended to be a multi-user app, I was thinking of adding record locking, but decided against it as it would add more DB accesses and obfuscate the core logic somewhat. I did research some of the methods available, and feel it wouldn’t be too much trouble to add it on in the future.

I converted some of the HTML to Handlebars templates. This cleaned up the HTML quite a bit, making it easier to comprehend. This improvement in HTML clarity is somewhat offset by pushing more logic to JavaScript. But, overall, I think it’s an improvement.

To keep from re-requesting records from the server that I already had, I started creating a bunch of ‘xyz_is_loaded’ flags. For example, when the user would toggle a recipe’s ingredients list on and off, I would check the appropriate flag to see if I already had the ingredient records loaded. A simpler method would be to load all a recipe’s data when the recipe was selected. Then, just show it on demand. I thought by only requesting the needed data when and if it was needed, the responsiveness of the app would be better. Given the fairly small size of the records, any miniscule improvement in responsiveness may not be worth the added code complexity.

All the above issues aside, I found the portions of the app that were rewritten to use Ajax, instead of page refreshes, were much more responsive and provided a much better user experience. The ability to show and hide extra content on user request made the display much cleaner.
.
