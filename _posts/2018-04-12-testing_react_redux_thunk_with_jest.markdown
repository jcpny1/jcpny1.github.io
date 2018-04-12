---
layout: post
title:      "Testing React/Redux/Thunk with Jest"
date:       2018-04-12 17:27:59 +0000
permalink:  testing_react_redux_thunk_with_jest
---

### *Introduction*

The application is a Rails API back end with a JavaScript front end, in a single GitHub repository.
All the Rails testing is done with RSpec.
Some feature tests will start a headless browser and exercise the JavaScript front end.
With RSpec, other than providing mock responses for third-party API's, the testing was a simple as navigating to a page and issuing the command 'click'.
It is incredibly readable.
```
      visit 'http://localhost:3000'
      page.first('#portfolioEdit').click
      fill_in "Name", :with => '<do-not-use>'
      click_button 'Submit'
      expect(page).to have_text('<do-not-use>')
```
Unfortunately, since RSpec only instruments Ruby code, there is no code coverage information for the JavaScript code.

I added Jest as the test suite for the JavaScript code in order to get coverage statistics.
Additionally, it's probably not good practice to expect a Rails API app to properly test a JavaScript front end.
I immediately ran into configuration problems.
It appears that Jest testing, when you are using Redux, Router, and Thunk, requires quite a bit of mocking, stubbing, and workarounds.
From what I could find on the internet, these workarounds create a lot of overhead, are not obvious, and look very fragile (a maintenance nightmare).
As I wade through the issues faced, I'll update this blog to show the methods I found to be most useful.

### *First Pass*

My first pass at Jest testing was to just render each page of the app and to test that all actions and corresponding reducer logic exists, without any functionality testing.
```
expect(PositionAction.positionAdd(myPosition, myMock)).toBeDefined();
expect(ActionRequest.positionAdd(myDispatch, myPosition, mySort)).toBeDefined();
```
This brought code coverage up to 40%.

### ***Conclusion***

