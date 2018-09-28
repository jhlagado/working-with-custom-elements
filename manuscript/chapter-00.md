# Introduction

## Frontend development: a quick history

In September 2018, GitHub [announced](https://githubengineering.com/removing-jquery-from-github-frontend/) that it had finally removed the last vestige of jQuery from its codebase. This was a gradual process.

![JQuery gradually being removed from GitHub](images/ch0-github-jquery.png)

This event seems like an appropriate moment to pause and take a look back at the history of front-end development over the past decade of so.

Ten years doesn't seem like a terribly long passage of time to consider but, from the perspective of the web and especially of front-end development, a decade can seem like an eternity.

Consider what the development landscape of ten years ago looked like. Back then we had

* no Google Chrome
* no standard way to query DOM elements via a CSS selectors
* no standard way to animate the visual styles of an element
* no standard way to make a backend request

Since then, browsers have improved dramatically and more importantly have converged upon a [single living standard](https://html.spec.whatwg.org/dev/). Internet Explorer remains a bit of an outlier with regard to web standards but it is important to note that 

1) Internet Explorer 11 at least is not a terrible browser and can be polyfilled and transpiled for 

2) worldwide usage of all versions of Internet Explorer on the desktop have plummeted in recent years to less that 10% of all users. Usage continues to drop significantly every year.

As a consequence, an overwhelming majority of desktop users today use what we call “evergreen” browsers, i.e. browsers that get regularly and automatically updated. More importantly, web use is moving increasingly toward mobile devices (where Internet Explorer barely exists) and nearly all mobile users browse with an evergreen browser.

This means that developers would be wise to take these changes into account when thinking about their approach and attitude toward newer features of the web platform. In the past, web developers were forced to ignore newer features of browsers for years after their initial release. Instead they had to develop for the lowest common denominator of browser (this usually meant some version of Internet Explorer). 

This is now no longer the case and developers should really track more closely with the changes that are happening to their platform. This doesn't mean abandoning users of minority browsers but it does open up other ways of dealing with them. It is now becoming increasingly common for developers to create two sets of bundles, ones that target evergreen browsers and the other for non-evergreen browsers. This separation means that code sizes shrink for the modern browsers and not all features need be fully supported for non-evergreens. 

Just because we as developers have a responsibility to not cut off any community of our users, at the same time this doesn't mean that we have to guarantee exactly the same experience. On older browsers some features of our software may be limited or missing altogether. Supporting the weakest browsers **should no longer mean foregoing the benefits of the browsers most people actually use!** We can feature detect the user's browser and provide the experience that is best suited to that browser.

Over the past decade, the browser has become a universal application environment. Instead of being content to simply display HTML pages loaded from a server, most modern web applications now have adopted a "single-page application" architecture in which the browser downloads its static resources from an asset server (possibly a CDN) and fetches its data from a backend server in the form of AJAX requests and JSON payloads (rather than HTML). Modern versions of this architecture include local caching of the application's assets with a service worker and this offers all kinds of benefits, not least to enable the web application to operate even without an internet connection.  

A timeline of the significant events in frontend development history actually stretch back a little bit further than a decade. I'm going to start it in 2004 with the release of GMail, the arguably first modern web application.

* 2004 Google GMail limited beta
* 2004 Firefox released
* 2005 [Jesse James Garrett coins the term “AJAX”](http://adaptivepath.org/ideas/ajax-new-approach-web-applications/) 
* 2007 GitHub adopts jQuery
* 2008 Google releases the Chrome browser
* 2010 Angular JS released
* 2011 React released
* 2013 Custom Elements v0 
* 2014 Vue.js released 
* 2015 Babel released
* 2015 Redux released 
* 2016 Custom Elements v1 
* 2016 Angular 2 released
* 2018 React dominates

Looking back, I might argue that frontend development passed through three epoch which defined the way we did things. Each epoch while only loosely defined is best exemplified by looking at the influence on developer practice of the most dominant library or framework of that era.

### The jQuery era

This era was when JavaScript started to be looked upon as a viable language for development. Thanks to the popularising and educational work of Douglas Crockford and others, JavaScript stopped being viewed as some kind of toy language and a variant of Java and began to be seen as as a language with first class functions and closures. 

This was also the time when much work was put into overcoming browser differences and establishing new norms which would later go on to become web standards. The biggest incompatibilities that needed ironing out were in the areas of Cascading Style Sheets and Document Object Model.

This was, of course, when JQuery, an awesome library written by John Resig, helped programmers achieve finer-grained control of the DOM than ever before and to allow them to add dynamism to every page. People talked about the possibility of "graceful degradation" of software and accessible and "unobtrusive JavaScript".

This sparked the JavaScript revolution.

### The Angular JS era

* declarative interfaces using templates and directives
* data binding of the state to the view
* MVC app structure
* app routing
* testing framework
* transformative

### The React era (present)

* functional representation UI = f(state)
* virtual DOM as an optimisation
* unidirectional data flow
* “dumb” components
* state management (Flux, Redux, MobX etc)

Meanwhile, the browser platform itself has also matured

* promises — functional way to deal with asynchronicity
* fetch — ability to load resources without changing the URL
* querySelector — ability to select elements using CSS selection rules
* history API — change URL without navigating the browser
* web workers
* local storage
* service workers
* custom elements — extending the DOM with components

## What are Custom Elements?

[Custom Elements](https://html.spec.whatwg.org/dev/custom-elements.html) are a feature of modern browsers which allow you to modularise and install your JavaScript components into the browser itself in order to extend it in new and powerful ways. Custom Elements are HTML components which have their own self-contained markup, styling and behaviour.

For example, say we wanted to create an element that formatted a person’s name

    <name-card first-name="John" last-name="Hardy"></name-card>

If the `name-card` tag was registered as a custom element component then the browser could be made to render it as

    <table>
      <tr>
        <th>First Name</th>
        <th>Last Name</th>
      </tr>
      <tr>
        <td>John</td>
        <td>Hardy</td>
      <tr>
    </table>

Custom Elements are useful for defining reusable components such as buttons and dropdown menus but they are expressive enough that they could even constitute entire web pages.

Because they are registered with the browser itself, Custom Elements do not compete with existing libraries and frameworks. In fact they can be used to facilitate greater interoperability and code sharing between them. A custom element designed for, say, an Angular application could be reused in a React application. The work that went into developing the component in one framework could be reused without modification in another. This enables flexibility in web development and it also helps defeat the general tendency towards vendor and framework lock-in on the front-end.

You can think of Custom Elements as a sort of containerisation for web components. They have been designed to maximise flexibility and code reuse.


