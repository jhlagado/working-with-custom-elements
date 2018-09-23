### Writing with Custom Elements

[Custom Elements](https://html.spec.whatwg.org/dev/custom-elements.html) are a
feature of modern browsers which allow you to package and install your
JavaScript code into the browser itself to extend it in new and powerful ways.
Custom Elements do this allowing you to add your own HTML elements with their
own markup, styling and behaviours.

For example, say we wanted to create an element that formatted a person’s name

    <name-card first-name="John" last-name="Hardy"></name-card>

If the the `name-card` tag was registered with the browser as a custom element
then the browser could be made to render this as

    <table>
      <tr>
        <th>First Name</th>
        <th>Last Name</th>
      </tr>
      <tr>
        <td>
    </td>
        <td>
    </td>
      <tr>
    </table>

Custom Elements are useful for defining reusable components such as buttons and
dropdown menus but they are simple and expressive enough to form the basis of a
entire web pages!

Because they are registered with the browser itself, Custom Elements do not
compete with existing libraries and frameworks. In fact they can be used to
facilitate greater interoperability and code sharing between different
frameworks. A custom element designed for, say, an Angular application could be
reused in a React web application. The work that went into developing the
component in one framework can be used without modification in another. The
facilitates flexibility in web development and also helps defeat the general
tendency towards vendor and framework lock-in on the front-end.

You can think of Custom Elements are a sort of containerisation for web
components which have been designed to maximise flexibility and code reuse.

### Working with the DOM

Before we get too deeply into Custom Elements, I’d like to talk a little first
about updating the DOM using JavaScript. The browser gives us an entire API for
doing this. For example, say we want to attach an `h1` element containing the
text `“Hello, world!”` to an element with an `id` of `”root”`


    Hello, world!




Running this code will display a heading with the words “Hello, world!” on the
page but, to frank, this seems like an awful lot of work for such a simple task!

A more straightforward and declarative way to do the same thing would be to use
a string and to replace the root element’s content by using the
[innerHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML)
property.


Even better than a conventional JavaScript string is a [template
literal](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)**
**which allows you to express text that can run over multiple lines. Note the
use of ``back-ticks`` rather that `‘single’` or `“double”` quotes to delimit the
template literal.


Template literals evaluate as conventional strings but they have the additional
feature of letting you easily interpolate data with text by using **dollar
brace** notation ${}.

    const greeting = 'Hello';


[See a working version
here.](https://codepen.io/jhlagado/pen/oPOGeQ?editors=1100)

So working with the innerHtml property and template literals is great but
unfortunately any solution employing innerHTML doesn’t actually scale very well.
The innerHTML property is inefficient for working with large amounts of HTML and
it is destructive because it completely replaces the contents of the DOM element
every time it gets assigned to. Worse, if there were any event listeners
attached to these child elements they might too get lost.

Imagine for a second that we had a way to update the DOM that wasn’t so
destructive. *What if we had a way to only update the parts that actually
changed?* This is usually the point in most discussions where [Virtual
DOMs](https://www.codecademy.com/articles/react-virtual-dom) are raised as an
efficient way to update the browser DOM. While this is certainly a good idea and
is the basis of many frameworks, this solution doesn’t come without its own
costs in terms of memory and computation. Virtual DOMs are certainly powerful
and useful but they aren’t the *only* solution to this problem.

Instead I’m going to talk about a lesser known Virtual DOM alternative that uses
[Tagged Template
Literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#Tagged_templates)
to only update the parts that changed. These standard JavaScript literals differ
only from the other kind Template Literals in being paired with a “tag” function
which can preprocess it and give it specialised behaviours.

In these examples we’ll be using a very lightweight library from Google called
[LitHtml](https://github.com/Polymer/lit-html). The great thing about LitHtml is
that we can use it update the DOM very efficiently and, when applied repeatedly,
*only modify the elements that need changing*. If there are any event listeners
attached, it will leave them unharmed.

Remarkably LitHtml really isn’t any more difficult to use than innerHTML.





[See a working version here.](https://codepen.io/jhlagado/pen/jvRbOY)

This code imports two functions from the lit-html module: html and render. The
`html` function is used to “tag” its associated template literal and make it
usable by the `render` function. The `render` function takes this literal uses
it to replace the content of the DOM element specified as its second argument.

The main difference here is that if nothing changes, no work gets done. For
example, if the value of the greeting variable has not been altered, then
multiple calls to the render function will not change the DOM. If the greeting
variable does change then only the parts of the DOM that are affected by it will
be altered. 

This gives us all the benefits of clarity and declarative style of using
innerHTML to update the DOM but without the destructiveness and inefficiency
that normally comes with it. We can achieve everything we need and we can do it
without bringing in the overhead of a Virtual DOM system.

To demonstrate this last point, let’s set up an example that calls `render`
function multiple times (for the sake of brevity, I’ll leave out the `import`
statement from the previous example). 

To show that the render function is being called multiple times, let’s add a
timestamp.




[See a working version
here](https://codepen.io/jhlagado/pen/BOEwZN?editors=1100).

### Embedding Expressions in Literals

The dollar brace ${} syntax allows you to put any valid [JavaScript
expression](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#Expressions)
inside a template literal. For example, `2 + 3`, `user.firstName`, or
`formatName(user)` are all valid JavaScript expressions which can be used for
[expression
interpolation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#Expression_interpolation).

In the example below, we embed the result of calling a JavaScript function,
`formatName(user)` into an `<h1>` element.





[See a working version
here.](https://codepen.io/jhlagado/pen/dqLJOR?editors=1100)

Tagged Template Literals which are processed by LitHtml evaluate to ordinary
JavaScript objects which can in turn be used as expressions in other LitHtml
literals.

This means that you can use them inside `if` statements and `for` loops, assign
them to variables, accept them as arguments, and return them from functions:


You can also use dollar brace syntax to embed JavaScript expressions inside HTML
attributes:


By default, LitHtml
[escapes](http://stackoverflow.com/questions/7381974/which-characters-need-to-be-escaped-on-html)
any values embedded in the literal before rendering them. Thus it ensures that
you can never inject anything that’s not explicitly written in your application.
Everything is converted to a string before being rendered. This helps prevent
[XSS (cross-site-scripting)](https://en.wikipedia.org/wiki/Cross-site_scripting)
attacks.

For example, it is perfectly safe to embed user input inside  LitHtml literal:


     

### Back to Custom Elements

Returning to the topic of Custom Elements after that long detour into DOM
manipulation and rendering with literals, let’s talk again about what Custom
Elements actually bring to the party.

The examples so far have concentrated on selecting some existing DOM element in
the HTML page and then replacing its contents by executing some JavaScript. This
is fine as it goes but what we are really after is a way to build reusable web
components that exist autonomously on the page, know when to render themselves
and to respond to browser events with their own behaviours.

Let’s start with a simple example

    <my-element></my-element>

Note that custom elements must have one or more hyphens in their name. Also note
that custom elements always need a closing tag.

The basic definition of a custom element looks like this.

    class MyElement extends HTMLElement { // 1
      
      constructor() { // 3 
        super();        
      }
      
      connectedCallback() { // 4
        this.render();
      } 
      
      render() { // 5
        render( 
          html`<h1>Hello, world!</h1>`, 
          this
        );
      }
    }
    customElements.define('my-element', MyElement); // 2

[See a working version here (use
Chrome).](https://codepen.io/jhlagado/pen/QVPQWb?editors=1101) 

Custom elements are made using a JavaScript class definition (1) which is
registered with the browser (2). The class definition must extend one of the
built in classes of the browser which implements an element. While it is
possible to extend and inherit the behaviours of built-in element types such as
HTMLButtonElement, this is currently still a poorly supported feature in
browsers so all of the examples here will extend from the generic HTMLElement. 

In this basic example, I have provided a `constructor` (3) which currently does
nothing except call `super()` and attach something called a shadowRoot (4) which
we will discuss later. 

The class also provides an implementation a custom element life-cycle hook
called `connectedCallback()` (4) which is called when the element is first added
to the document. This callback is a good place to initially update the DOM with
new content. It does this by calling its own method `render()` (4) which in turn
calls the LitHtml render method passing a LitHtml literal and the element’s own
DOM to render to.

The resulting HTML in the browser looks like this.

![](https://cdn-images-1.medium.com/max/1600/1*7QFHXjMufZYy9_yZJflVtw.png)

While this is already pretty good, it has the downside of the custom element
replacing its own content which makes it difficult to pass additional
information in the body of the custom element.

We can overcome this problem and at the same time unlock even more powerful
features of Custom Elements by using an additional feature of modern browsers,
the Shadow DOM.

### Shadow DOM

Any HTML element in the browser can be like a tiny universe of its own by having
its own Shadow DOM. When an element gets a `shadowRoot` then the browser will
display that instead of its body. 

The HTML elements inside the element’s Shadow DOM are isolated from the elements
outside and can be styled and controlled independently. 

The body of the element is now invisible and may be used for other purposes.

For the following custom element which passes a piece of HTML in its body
content

    <my-shady-element>
      <i>Hello</i>
    </my-shady-element>

let’s take a look at a Custom Element definition that can use it. This example
uses a Shadow DOM.

    class MyShadyElement extends HTMLElement {
      
      constructor() { 
        super();
        this.attachShadow({mode: 'open'}); // 1    
      }
      
      connectedCallback() {  
        this.render();
      } 
      
      render() { 
        render( 
          html`<h1><slot></slot> world!</h1>`, 
          this.shadowRoot
        );
      }
    }
      
    customElements.define('my-shady-element', MyShadyElement);

[See a working version here (use
Chrome).](https://codepen.io/jhlagado/pen/GXLxGm?editors=1101)

The resulting HTML in the browser looks different to the previous example

![](https://cdn-images-1.medium.com/max/1600/1*ok8m9Y35gSS-tPZuAZOX6A.png)

<br> 

<br> 

<br> 

<br> 

<br> 

### Components and Props

Components let you split the UI into independent, reusable pieces, and think
about each piece in isolation. This page provides an introduction to the idea of
components. You can find a [detailed component API reference
here](https://reactjs.org/docs/react-component.html).

Conceptually, components are like JavaScript functions. They accept arbitrary
inputs (called “props”) and return React elements describing what should appear
on the screen.

<br> 

### Functional and Class Components

### <br> 

The simplest way to define a component is to write a JavaScript function:



This function is a valid React component because it accepts a single “props”
(which stands for properties) object argument with data and returns a React
element. We call such components “functional” because they are literally
JavaScript functions.

You can also use an [ES6
class](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes)
to define a component:



The above two components are equivalent from React’s point of view.

Classes have some additional features that we will discuss in the [next
sections](https://reactjs.org/docs/state-and-lifecycle.html). Until then, we
will use functional components for their conciseness.

### Rendering a Component

### <br> 

Previously, we only encountered React elements that represent DOM tags:



However, elements can also represent user-defined components:



When React sees an element representing a user-defined component, it passes JSX
attributes to this component as a single object. We call this object “props”.

For example, this code renders “Hello, Sara” on the page:




[Try it on
CodePen](https://reactjs.org/redirect-to-codepen/components-and-props/rendering-a-component)

Let’s recap what happens in this example:

1.  We call `ReactDOM.render()` with the `<Welcome name="Sara" />` element.
1.  React calls the `Welcome` component with `{name: 'Sara'}` as the props.
1.  Our `Welcome` component returns a `<h1>Hello, Sara</h1>` element as the result.
1.  React DOM efficiently updates the DOM to match `<h1>Hello, Sara</h1>`.

> **Note:*** Always start component names with a capital letter.*

> *React treats components starting with lowercase letters as DOM tags. For
> example, *`<div />`*represents an HTML div tag, but *`<Welcome />`* represents a
component and requires *`Welcome`* to be in scope.*

> *You can read more about the reasoning behind this convention
> *[here.](https://reactjs.org/docs/jsx-in-depth.html#user-defined-components-must-be-capitalized)

> <br> 

> <br> 

### Composing Components

### <br> 

Components can refer to other components in their output. This lets us use the
same component abstraction for any level of detail. A button, a form, a dialog,
a screen: in React apps, all those are commonly expressed as components.

For example, we can create an `App` component that renders `Welcome` many times:





[Try it on
CodePen](https://reactjs.org/redirect-to-codepen/components-and-props/composing-components)

Typically, new React apps have a single `App` component at the very top.
However, if you integrate React into an existing app, you might start bottom-up
with a small component like `Button` and gradually work your way to the top of
the view hierarchy.

### Extracting Components

### <br> 

Don’t be afraid to split components into smaller components.

For example, consider this `Comment` component:



[Try it on
CodePen](https://reactjs.org/redirect-to-codepen/components-and-props/extracting-components)

It accepts `author` (an object), `text` (a string), and `date` (a date) as
props, and describes a comment on a social media website.

This component can be tricky to change because of all the nesting, and it is
also hard to reuse individual parts of it. Let’s extract a few components from
it.

First, we will extract `Avatar`:



The `Avatar` doesn’t need to know that it is being rendered inside a `Comment`.
This is why we have given its prop a more generic name: `user` rather than
`author`.

We recommend naming props from the component’s own point of view rather than the
context in which it is being used.

We can now simplify `Comment` a tiny bit:



Next, we will extract a `UserInfo` component that renders an `Avatar` next to
the user’s name:



This lets us simplify `Comment` even further:



[Try it on
CodePen](https://reactjs.org/redirect-to-codepen/components-and-props/extracting-components-continued)

Extracting components might seem like grunt work at first, but having a palette
of reusable components pays off in larger apps. A good rule of thumb is that if
a part of your UI is used several times (`Button`, `Panel`, `Avatar`), or is
complex enough on its own (`App`, `FeedStory`, `Comment`), it is a good
candidate to be a reusable component.

### Props are Read-Only

### <br> 

Whether you declare a component [as a function or a
class](https://reactjs.org/docs/components-and-props.html#functional-and-class-components),
it must never modify its own props. Consider this `sum` function:



Such functions are called [“pure”](https://en.wikipedia.org/wiki/Pure_function)
because they do not attempt to change their inputs, and always return the same
result for the same inputs.

In contrast, this function is impure because it changes its own input:



React is pretty flexible but it has a single strict rule:

**All React components must act like pure functions with respect to their
props.**

Of course, application UIs are dynamic and change over time. In the [next
section](https://reactjs.org/docs/state-and-lifecycle.html), we will introduce a
new concept of “state”. State allows React components to change their output
over time in response to user actions, network responses, and anything else,
without violating this rule.

<br> 
