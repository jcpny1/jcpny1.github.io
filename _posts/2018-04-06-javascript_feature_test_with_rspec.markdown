---
layout: post
title:      "Javascript Feature Test With RSpec"
date:       2018-04-06 11:35:26 -0400
permalink:  javascript_feature_test_with_rspec
---

### *Introduction*

The application is a Rails API backend with a Javascript frontend, in a single GitHub repository.
All the Rails testing for models and controllers is done with RSpec.
For expediency, I thought I'd try RSpec to perform feature tests on the javascript frontend rather than add a second testing framework.

Through a lot of trial and error, I was able to get features tests working locally, but had quite a bit of trouble getting them to work remotely on Travis CI.
There's a lot of information on the web about Travis configurations. Much of it is out of date, and the rest doesn't pertain to this situation. So I'll document what I found worked for me here 

Along the way, I realized that it's really not the perview of an API-only backend to know about the front end enough to create and run feature tests.
So, I wound up going to Jest for front end testing after all.
However, the difficulty in getting a browser running in Travis should be the same regardless of your front-end language.

I use Code Climate to measure test code coverage.
When Travis finishes testing a build, it forwards code coverage data to Code Climate.
Code Climate requires a special configuration to be able to consolidate code coverage from RSpec and Jest.
For now, Travis config just forwards the Ruby coverage data to Code Climate for reporting.
Getting a consolidated coverage report will be the topic of a future post.

### *.travis.yml*

RSpec is able to run what it needs to test Ruby code.
To test front-end functionality (with a browser), you need to have a web server running.
Locally, I was able to do an `npm start` before running RSpec so the feature test would have something with which to handle URL requests.
With Travis CI, the Travis config file must perform this task.

```
env:
  global:
    - CC_TEST_REPORTER_ID=95325b4141034c059f27f080ce9f4bb9ac4837204fe4fd73bb289060ed0c5b2a
    - MOZ_HEADLESS=1
addons:
  firefox: latest
language:
  - ruby
rvm:
  - 2.4.2
before_install:
  - wget https://github.com/mozilla/geckodriver/releases/download/v0.20.0/geckodriver-v0.20.0-linux64.tar.gz
  - mkdir geckodriver
  - tar -xzf geckodriver-v0.20.0-linux64.tar.gz -C geckodriver
  - export PATH=$PATH:$PWD/geckodriver
install:
  - . $HOME/.nvm/nvm.sh
  - cd client
  - nvm install stable
  - npm install <<<'Y'
  - cd ..
  - bundle install
before_script:
  - cd client
  - "PORT=3000 npm start&"
  - cd ..
  - sleep 5
  - bundle exec rake db:setup
  - curl -L https://codeclimate.com/downloads/test-reporter/test-reporter-latest-linux-amd64 > ../cc-test-reporter
  - chmod +x ../cc-test-reporter
script:
  - ../cc-test-reporter before-build
  - bundle exec rspec
  - cd client
  - npm test
  - cd ..
after_script:
  - ../cc-test-reporter after-build --exit-code $TRAVIS_TEST_RESULT
```
### ***Conclusion***

Through trial and error, I was able to come up with a `.travis.yml` file that provided feature testing when the front-end code is using Javascript.
Combing the test results from RSpec and from Jest into a consolidated Code Climate report requires additional changes to the Travis config file.
These changes will be covered in a future post.
