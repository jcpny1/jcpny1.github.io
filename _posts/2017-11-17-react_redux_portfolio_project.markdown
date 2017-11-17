---
layout: post
title:      "React Redux Portfolio Project"
date:       2017-11-17 15:09:31 -0500
permalink:  react_redux_portfolio_project
---

The goal of this project was to create an app using as much of what we have learned as possible.
It needed to utilize React and Redux, ES6 syntax, and communicate to a Rails backend.
I chose to write a stock portfolio valuation tool similar to what you’d see on Yahoo Finance, or at an online brokerage such as Schwab or Fidelity.
There’s a lot of functionality packed into those apps;
More than would be needed to satisfy this project’s requirements, and would take a long time to build from scratch.
So, I focused on getting and saving a user’s portfolio and applying current market prices to come up with current value and gain/loss amounts.

<img src='https://github.com/jcpny1/stock-analyzer/blob/master/Screenshot-2017-11-13%20StockAnalyzer.png?raw=true' alt='Stock-Analyzer Screenshot' title='Stock-Analyzer Screenshot' style='width:100%;'>

### *The Data Model*
The data model is pretty straightforward.
A user has one or more portolios, each of which contain positions.
Each position represents the holding of a single instrument.
Each instrument has market data associated with it.

<img src='https://github.com/jcpny1/stock-analyzer/blob/master/Stock%20Analyzer%20E-R%20Diagram.png?raw=true' alt='Stock-Analyzer E-R Diagram' title='Stock-Analyzer E-R Diagram' style='width:100%;'>

### *Market Data*
I needed a way to create realistic stock prices for the entered positions;
Preferably one that used a publish-subscribe model.
Unfortunately, I could only find data feeds that charged a hefty monthly subscription fee.
Not wanting to go that route, I thought of writing a faux price server.
But that seemed like a fair amount of coding and a poor substitute for the real thing.
After some poking around, and finding a lot of outdated information, I came across a Yahoo Finance api that offered a RESTful API for equity prices.
I spent about a week crafting the Rails methods.
After many years of continuous service, and just as I wrapped up the coding, the service began returning this message to any API request:
```
It has come to our attention that this service is being used in violation of the Yahoo Terms of
Service. As such, the service is being discontinued. For all future markets and equities data
research, please refer to finance.yahoo.com.
```
No more quotes!
Proving the old adage, you get what you pay for.
I then found another service that provided historical data that was updated in near real time, but was missing a few key pieces of data.
While I was coding for that service, I found a third that provided almost all the information I was looking for.
So, I switched providers again and created a third data feed handler.
Needless to say, this was a big time sink.
The app, as it stands now, uses Investors Exchange for market data, Alpha Vantage for the Dow Jones Industrial Average, and NewsAPI for headline news.
I would have preferred feeds that work off CUSIP or ISIN codes instead of symbols, but none were available (for free, that is).
Once I was effectively communicating with the market data provider’s API, I realized that my app wouldn't do anything usefule if the data feed was down.
### *Market Data*
I needed to cache the latest prices on my end in the event the data feed was not available, so the user would at least have some prices to work with, if not the latest ones.
Rather than create my own cache service, I used sqlite3 to maintain the latest prices for any equity in a user’s portfolio.
I then set out to create seed data for the database.
Again, to have something to work with if the live feed were not available.
The market data vendor provides information on about 9,000 instruments.
When I used this data for seeding, it took hours to load and created a database of about 200MB.
Not ideal.
So, I trimmed the seed data down to a handful of instruments and prices (at about 30KB) and provided an option in the app to fetch additional data, if desired.
### *Creating multi-page apps using Redux*
I created several container components, each having their own information in the application’s state.
When the user hits the browser’s refresh, the entire state is wiped out, unbeknownst to the components that are not presently displayed.
Like the disappearing data feed, this was another setback.
Fortunately, the nature of the app lent itself to having a single state with data that could be shared among the components.
Such that, if a refresh occurred, the currently active component would cause the reloading of the entire state, not just its own part.
### *Sqlite Performance*
Loading the full set of instruments and pricing data from the data feed was taking a long time.
I found that by creating my own transactions, instead of using the default single sql statement autocommits, sped things up dramatically.
For example, loading 9,000 instruments into the database took 30 minutes using autocommit, but only 30 seconds when I used just one transaction for all 9,000.
With autocommit, there are 9,000 begin transactions, 9,000 insert statements, and 9,000 commits.
With my own transactions, there is just one begin transaction and one commit.
This technique can’t be used in all situations, but in this case, it worked great.
A more advanced database manager would allow for bulk inserts.
That would surely improve performance even more.
### *Using a UI framework*
I used semantic-ui-react for this application.
It comes with a lot of functionality and styling.
The downside seems to be if you want certain things to behave a certain way, you’re in for overriding a lot of what they’ve done.
Also, the documentation is a bit sketchy in some areas.
Several of their examples used features not described in their documentation.
So, I’m left to wonder if there are other features that can do what I’m looking for that are not documented.
Things such as having a horizontal scroll bar, preventing a table from moving past it’s container boundary, etc.
I didn’t have the will to plow through 10’s of thousands of lines of CSS and JavaScript code to find out.
Since there were too many instruments to include in a dropdown picklist, I wanted to have an interactive selection list that would update the available selections after each character input.
Easy to do in Semantic, unless your input field is in a Modal window.
Then, maybe not possible with this framework without some serious rewriting.
### *Mixing JavaScript and Rails in the same repository*
I wonder if having the server code and the client code in the same repository makes sense.
They are independent areas of concern. It seems like they should be managed separately.
### *CI Service Feature Testing*
After creating a few feature tests, I couldn’t get them to execute on Travis CI’s system.
There’s some info on how to do headless browser testing floating around.
But, Travis CI’s website seems to indicate that you need a third-party’s paid subscription server service to execute on.
When I have time, I’ll look to enable the feature tests for executing on my system only and not on Travis CI’s side. Unfortunately, this take away some of the benefits of a CI service like Travis CI.
### *RuboCop*
I used CodeClimate to do test coverage analysis (via SimpleCov) and perform static code checking, using a variety of products.
One of those code checkers, RuboCop, spotted a few areas of poorly written code.
But it also threw thousands of warnings of dubious value.
Things like a line being longer than 80 characters, or a function have more than 25 lines in it.
I’ll have to spend some time tweaking the checker’s config file to alleviate having so many warnings.
### *Conclusion*
All in all, this project allowed me to really dig in to creating a full-fledged app such that one would be doing professionally.
Each obstacle encountered was a lesson learned for the future.

