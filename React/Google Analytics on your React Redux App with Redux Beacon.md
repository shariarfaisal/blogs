# Google Analytics on your React Redux App with Redux Beacon

```React```

Redux Beacon is an analytics middleware for Redux and ngrx that enables you to track actions as they are dispatched and send them to any number of analytics providers. Supported providers include Google Analytics, Segment, and Amplitude.


In this article, we’ll use Redux Beacon, Google Analytics, and track pageviews and events in a single-page React app. Let’s get started!


# Setup


Note that you’ll first need a Google Analytics account and Analytics.js added to your site. Be sure that your Analytics snippet is placed before any other scripts, and then update the tracking ID on the snippet with your property ID.


To install Redux Beacon for Google Analytics, use npm or Yarn and install redux-beacon and @redux-beacon/google-analytics.


```
$ npm install —save redux-beacon @redux-beacon/google-analytics

# or, using Yarn:
$ yarn add redux-beacon @redux-beacon/google-analytics

```



When working and testing with Google Analytics, whitelist your site on any ad/tracking blockers you may have installed. Learn from my pain!

# Defining What to Track


We’ll be covering pageview and event tracking, but the Google Analytics plugin for Redux Beacon also supports tracking timing, e-commerce, social interaction, and errors. You can find the full documentation on that here.



Redux Beacon works by inspecting every action that you dispatch and comparing its type to the action types in your events map. If the action is found, it will call the tracking function that you define there.


```
const eventsMap = {
  MY_ACTION_ONE: someTrackingFn(…),
  MY_ACTION_TWO: someOtherTrackingFn(…)
  …
}

```


When MY_ACTION_ONE is dispatched, someTrackingFn will be called.


## Pageviews


Tracking pageviews with Redux Beacon means mapping your action type for changing routes to the pageview function. In this example, we’ll be using the LOCATION_CHANGE event from react-redux-router.


```
import { LOCATION_CHANGE } from ‘react-router-redux’;

const eventsMap = {
  [LOCATION_CHANGE]: ourPageViewFn()
};

```


Each time LOCATION_CHANGE is dispatched, our pageview function will be called. We can use this function to pull the data we want to track out of the action.


What does the pageview function look like? The full example:


```
import { LOCATION_CHANGE } from ‘react-router-redux’;
import { trackPageView } from '@redux-beacon/google-analytics';

const eventsMap = {
  [LOCATION_CHANGE]: trackPageView(action => ({
    page: action.payload.pathname,
    title: "My amazing page title"
  }))
};

```


Optionally, the arrow function passed to the trackPageView function may also take two additional parameters, prevState and nextState.


```
trackPageView((action, prevState, nextState) => { … })

```


## Events


In Google Analytics land, events are trackable user interactions on your site/app. Clicking buttons, downloading content, and clicking an ad are all trackable events.


Let’s see what this tracking definition looks like:


```
const eventsMap = {
  "DOWNLOAD_CLICKED": trackEvent(action => ({
    category: "VideoFile",
    action: "download",
    value: action.videoId
  }))
};


```


In this case, when the user clicks a download button, the videoId is tracked, along with a category and action.


Note that value must be an integer.


# Final Steps


Before Redux Beacon can read your actions, the event definitions must be converted into middleware with the createMiddleware function and passed into the applyMiddleware function when creating your redux store:


```
import { createStore, applyMiddleware } from 'redux';
import { createMiddleware } from ‘redux-beacon’;
import { LOCATION_CHANGE } from ‘react-router-redux’;
import GoogleAnalytics, { trackPageView, trackEvent }
  from '@redux-beacon/google-analytics';

const ga = GoogleAnalytics();
const eventsMap = {
    [LOCATION_CHANGE]: trackPageView(/* ... */),
    "DOWNLOAD_CLICKED": trackEvent(/* ... */)
};
const middleware = createMiddleware(eventsMap, ga)

const myReducer = (state, action) => { ... }
const myInitialState = {}
const store = createStore(myReducer, myInitialState,
  applyMiddleware(middleware));

```


Now our redux actions can be tracked by Google Analytics! You can open the network tab on your browser’s inspection tools to see the magic as it unfolds.


To recap:


- Install Redux Beacon and Google Analytics on your site, and remember to disable ad/tracking blockers on your browser
- Define your event map, which maps redux actions to tracking functions
- Convert this event map to middleware and pass it to your redux store

If a tree falls in the woods and Google doesn’t track it, does it make a sound? 🕵


