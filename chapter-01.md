## Working with the DOM

Before we get too deeply into Custom Elements themselves, I’d like to first talk a little about updating the DOM using JavaScript. The browser gives us an entire API for doing this. For example, say we want to attach an h1 element containing the text “Hello, world!” to an element with an id of ”root”.

    // create a new h1 element
    const newHeading = document.createElement("h1");

    // and give it some content 
    const newContent = document.createTextNode("Hello, world!");

    // add the text node to the newly created h1
    newHeading.appendChild(newContent);

    // add the newly created element and its content into the DOM 
    const root = document.getElementById('root');

    // replace the content of the root element the new heading 
    root.innerHTML = '';
    root.appendChild(newHeading);

Running this code will display a heading with the words “Hello, world!” on the page but, to be frank, this does seem like an awful lot of work for such a simple task! A more straightforward and declarative way of doing the same thing would be to construct a string containing your markup and replace the root element’s content by assigning the string to the [innerHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML) property.

    const string = '<h1>Hello, world!</h1>';

    document.getElementById('root').innerHTML = string;

Even better than a conventional JavaScript string is a [template literal](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) which allows you to express text that can run over multiple lines. Note the use of `back-ticks` rather that ‘single’ or “double” quotes to delimit the template literal.

    const template = `
      <h1>
        Hello, world!
      </h1>
    `;

    document.getElementById('root').innerHTML = template;

Template literals evaluate as conventional strings but they have the additional feature of letting you easily interpolate data with text by using **dollar brace** notation ${}.

    const greeting = 'Hello';

    const template = `
      <h1>
        ${greeting}, world!
      </h1>
    `;

    document.getElementById('root').innerHTML = template;

[See a working version here.](https://codepen.io/jhlagado/pen/oPOGeQ?editors=1100)

So working with the innerHtml property and template literals is great but unfortunately any solution employing innerHTML doesn’t actually scale very well. The innerHTML property is inefficient for working with large amounts of HTML and it is destructive because it completely replaces the contents of the DOM element every time it gets assigned to. Worse, if there were any event listeners attached to child elements they might too get lost.

Imagine for a second that we had a way to update the DOM that wasn’t so destructive. *What if we had a way to only update the parts that actually changed?* This is usually the point in most discussions on DOM manipulation where [Virtual DOMs](https://www.codecademy.com/articles/react-virtual-dom) are raised as an efficient way to update the browser DOM. While this is certainly true and a good idea (and forms the main justification of many frameworks, React, Vue etc.), this solution doesn’t come without its own costs in terms of memory and computation. Virtual DOMs are certainly powerful and useful but they aren’t the *only* solution to this problem of efficiently updating the DOM.

Instead I’m going to talk about a lesser known Virtual DOM alternative that uses [Tagged Template Literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#Tagged_templates) to only update the parts of the DOM that changed. Tagged template literals differ only from the other ordinary template literals in that they are paired with a “tag” function which can preprocess it and give it specialised behaviours.

In these examples we’ll be using a very lightweight library from Google called [LitHtml](https://github.com/Polymer/lit-html). The great thing about LitHtml is that we can use it update the DOM very efficiently and, when applied repeatedly, *only modify the elements that need changing*. If there are any event listeners attached, LitHtml will leave them unharmed.

Remarkably LitHtml really isn’t any more difficult to use than innerHTML.

    import {html, render} from 'lit-html';

    const greeting = 'Hello';

    const literal = html`
      <h1>
        ${greeting}, world!
      </h1>
    `

    render(
      literal,
      document.getElementById('root')
    );

[See a working version here.](https://codepen.io/jhlagado/pen/jvRbOY)

This code imports two functions from the lit-html module: html and render. The html function is used to “tag” its associated template literal and make it usable by the render function. The render function takes this literal and uses it to replace the content of the DOM element which was specified in the second argument.

The main difference here to using innerHTML is that if nothing changes, no work gets done. For example, if the value of the greeting variable has not been altered, then even multiple calls to the render function will not change the DOM. If the greeting variable does change then only the parts of the DOM that are affected by it will be altered.

This gives us all the benefits of clarity and declarative style of using innerHTML but without the destructiveness and inefficiency that normally comes with it. We can achieve everything we need and we can do it without bringing in the overhead of a Virtual DOM system.

To demonstrate the last point, let’s look at an example that calls render function multiple times. For the sake of brevity, I’ll leave out the import statement from the previous example.

To show that the render function is being called multiple times, let’s add a timestamp.

    function tick() {

    const literal = html`
        <h1>Hello, world!</h1>
        <h2>It is ${new Date().toLocaleTimeString()}.</h2>
      `;
      
      render(
        literal, 
        document.getElementById('root')
      );
    }

    setInterval(tick, 1000);

[See a working version here](https://codepen.io/jhlagado/pen/BOEwZN?editors=1100).

You can see that only the DOM associated with the expression ${new Date().toLocaleTimeString()} is being updated.

![](https://cdn-images-1.medium.com/max/2000/1*ZazICqefkknioGVUCo4NJg.gif)

## Expressions in Literals

The dollar brace ${} syntax allows you to put any valid [JavaScript expression](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#Expressions) inside a template literal. For example, 2 + 3, user.firstName, or formatName(user) are all valid JavaScript expressions which can be used for [expression interpolation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#Expression_interpolation).

In the example below, we embed the result of calling a JavaScript function, formatName(user) into an h1 element.

    function formatName(user) {
      return user.firstName + ' ' + user.lastName;
    }

    const user = {
      firstName: 'John',
      lastName: 'Hardy'
    };

    const literal = html`
      <h1>
        Hello, ${formatName(user)}!
      </h1>
    `;

    render(
      literal,
      document.getElementById('root')
    );

[See a working version here.](https://codepen.io/jhlagado/pen/dqLJOR?editors=1100)

Tagged Template Literals which are processed by LitHtml evaluate to ordinary JavaScript objects which can in turn be used as expressions in other LitHtml literals.

This means that you can use them inside if statements and for loops, assign them to variables, accept them as arguments, and return them from functions

    function getGreeting(user) {
      if (user) {
        return html`<h1>Hello, ${formatName(user)}!</h1>`;
      }
      return html`<h1>Hello, Stranger.</h1>`;
    }

LitHtml literals can be nested within other LitHtml literals.

    const literal = html`
      ${getGreeting(user)}      
      ${getGreeting()}      
    `;

    render(
      literal,
      document.getElementById('root')
    );

You can also use dollar brace syntax to embed JavaScript expressions inside HTML attributes

    const literal = html`<img src=${user.avatarUrl}>`;

If the attribute is a **boolean attribute**, for example the disabled attribute on a button element, LitHtml has a special syntax to allow you to assign from a JavaScript boolean expression. The ? attribute prefix enforces the following behaviour: if the expression is true then the attribute gets added to the element, if the expression is false then the attribute gets removed.

    const literal = html`<input type="checkbox" ?checked=${checked}>`

LitHtml enables you to assign expressions not only to attributes but also **properties** on the DOM element itself. For this LitHtml uses another special syntax. The . attribute prefix assigns the value of the expression not to an attribute but to a property on the DOM element itself.

    const literal = html`<input .value=${value}>`;

LitHtml also allows you to attach error handlers using yet another attribute prefix. The @ attribute prefix attaches a function expression as an event handler. The type of the event (e.g. click) is the name of the event handler attribute.

    const literal = html`
      <button @click=${(e) => console.log('clicked')}>
        Click Me
      </button>
    `;

By default, LitHtml [escapes](http://stackoverflow.com/questions/7381974/which-characters-need-to-be-escaped-on-html) any values embedded in the literal before rendering them. Thus it ensures that you can never inject anything that’s not explicitly written in your application. Everything is converted to a string before being rendered. This helps prevent [XSS (cross-site-scripting)](https://en.wikipedia.org/wiki/Cross-site_scripting) attacks.

For example, it is perfectly safe to embed user input inside LitHtml literal:

    const title = response.potentiallyMaliciousInput;

    // This is safe:
    const literal = html`<h1>${title}</h1>`;

