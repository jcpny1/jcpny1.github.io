---
layout: post
title:      "Testing React/Redux/Thunk with Jest"
date:       2018-04-12 13:28:00 -0400
permalink:  testing_react_redux_thunk_with_jest
---

In my portfolio-analyzer repository, containing a Rails API back end and a JavaScript front end, all of the Rails testing is done with RSpec.
RSpec testing is as simple as navigating to a page and issuing the command 'click'.
It is incredibly readable.
```
      visit 'http://localhost:3000'
      page.first('#portfolioEdit').click
      fill_in "Name", :with => '<do-not-use>'
      click_button 'Submit'
      expect(page).to have_text('<do-not-use>')
```
There is one feature test that starts a headless browser and exercises a part of the JavaScript front end.
Unfortunately, RSpec only instruments Ruby code.
There is no code coverage information for the JavaScript code.

I added Jest as the test suite for the JavaScript code in order to get coverage statistics.
(Additionally, it's probably not good practice to expect a Rails API back end to capture front end functionality.)
I immediately ran into configuration problems.
It appears that Jest testing, when you are using Redux, Router, and Thunk, requires quite a bit of mocking, stubbing, and workarounds.
From what I could find on the internet, these workarounds create a lot of overhead, are not obvious, and look very fragile (a maintenance nightmare).
As I wade through the issues faced, I'll update this post to show the methods I found to be most useful.

### *First Pass*
My first pass at Jest testing was to
1. render each page of the app,
2. test that all actions and corresponding reducer logic exists.

There is no functionality testing.

```
expect(PositionAction.positionAdd(myPosition, myMock)).toBeDefined();
expect(ActionRequest.positionAdd(myDispatch, myPosition, mySort)).toBeDefined();
```
This brought code coverage up to 40%.

### ***Conclusion***
>TBD
