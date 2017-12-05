---
layout: post
title:      "Copying JavaScript Objects"
date:       2017-12-05 16:28:17 +0000
permalink:  copying_javascript_objects
---

My latest portfolio project centered around the use of React and Redux.
Maintaining state using these libraries involves plenty of object copying.
The state provided to the app is meant to be read-only.
To make a change to the state, the app presents a new version of the state to the libraries, which take care of updating the state.
## The Problem
Making changes to the state outside of React/Redux invalidates the whole purpose of using these libraries.
Making changes directly to the state is not allowed and seems to be caught at runtime.
Making changes to objects that the state references should not be allowed either, but such changes seem to go undetected.
My state object contains objects as well, so a shallow copy will not protect the entire state.
This may cause problems with React’s virtual DOM diffing, display refreshing and refresh optimization, not to mention other hard to diagnose side-effects in other parts of the app.
I stumbled onto this problem when I created classes for my primary data items, rather than just relying on Plain Old JavaScript Objects.
I had some getters and setters defined on the class.
Using the Object.assign() method, the class methods existed in the class instance but did not exist in the copied object, thereby breaking my code.
## Rejected Solutions
Everywhere you look online about copying JavaScript objects tends to have the caveat that the copy methods shown in the examples are shallow copies only and that deep copies may be necessary.
However, when one is focused on the bigger challenges of crafting an app, it’s easy to overlook this warning.
### *Typical Example*
•	The typical example will use this shallow-copy code:
```
        var copy = Object.assign({}, obj);
```
        or
```
        var copy = {…obj};
```
### *Class-aware Example*
Researching further on the internet, produced this example as a solution:
```
        var copy = Object.assign(Object.create(obj), obj);
```
This preserved the class designation and class methods, so my code was no longer broken.
However, this is still a shallow copy.
### *Mozilla Recommendation*
Researching even further, produced this warning from [Mozilla](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign), in a section called “Warning for Deep Clone”:
```
“For deep cloning, we need to use other alternatives…”
```
Followed by this example:
```
let obj3 = JSON.parse(JSON.stringify(obj1));
```
I haven’t tested this one, but I suspect it’s not performant, and probably does not preserve class and class methods.
Further down on the same page, they offer this as a way to get class methods copied as well:
```
var copy = Object.assign({}, obj); 
function completeAssign(target, ...sources) {
  sources.forEach(source => {
    let descriptors = Object.keys(source).reduce((descriptors, key) => {
      descriptors[key] = Object.getOwnPropertyDescriptor(source, key);
      return descriptors;
    }, {});
    // by default, Object.assign copies enumerable Symbols too
    Object.getOwnPropertySymbols(source).forEach(sym => {
      let descriptor = Object.getOwnPropertyDescriptor(source, sym);
      if (descriptor.enumerable) {
        descriptors[sym] = descriptor;
      }
    });
    Object.defineProperties(target, descriptors);
  });
  return target;
}
var copy = completeAssign({}, obj);
```
This didn’t look too good to me.
I want to code my app’s logic, not take a deep dive into JavaScript Object architecture.
## Working Solutions
### *First Attempt*
To get a deep copy, I resorted to explicitly creating and initializing a new instance.
```
	const newPosition = new Position(position.portfolio_id, position.id, instrument, editedPosition.quantity, editedPosition.cost, editedPosition.date_acquired);
```
This works, but looks like a maintenance chore.
If the class gets new members, we’ll have to track down everywhere we’ve hardcoded the members, and update the code.
### *Accepted Solution*
I mentioned this issue to Cernan, a Flatiron School instructor.
He suggested taking a look at lodash. Sure enough, there it was:
```
   const newPortfolio = _.cloneDeep(portfolio);
```
Without considering performance, this method was exactly what I was looking for.
It’s as concise and as readable as you can get.
There seems to be a popular opinion favoring **not** adding third party libraries to your app whenever possible (particularly evident when it comes to jQuery).
I’m not sure if that’s always a reasonable opinion considering how many libraries are already included with the app.
Additionally, it seems you can import just the deepClone method without taking on all of lodash.
## Conclusion
I think I didn’t stumble on the lodash solution originally due to search terminology.
‘Deep cloning’ just wasn’t in my vocabulary.
It seems most languages define cloning as copying the structure of an object (no data), and define copying as making a copy of an object (with data).
JavaScript doesn’t seem to make this distinction;
Cloning is copying.
Searching for copying methods, I got swamped by thousands of homebrew solutions and speculations. There are even 100 npm packages for deep copying!
There might be more exotic cases where lodash deep cloning doesn’t work.
For example, copying a class method that has a closure may or may not yield the expected results.
I didn’t test for that.

