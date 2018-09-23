### Writing with Custom Elements

![](https://cdn-images-1.medium.com/max/1600/1*hj_oFpaaV6tfhsv_Jt1P_A.png)

[Custom Elements](https://html.spec.whatwg.org/dev/custom-elements.html) are a
feature of modern browsers which allow you to package and install your
JavaScript code into the browser itself to extend it in new and powerful ways.
Custom Elements do this allowing you to add your own HTML elements with their
own markup, styling and behaviours.

For example, say we wanted to create an element that formatted a person’s name

    <name-card first-name="John" last-name="Hardy"></name-card>

If the `name-card` tag was registered with the browser as a custom element then
the browser could be made to render this as

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

This code imports two functions from the lit-html module: `html` and `render`.
The `html` function is used to “tag” its associated template literal and make it
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


LitHtml literals can be nested within other LitHtml literals.

For example, we can create an `App` component that renders `Welcome` many times:




You can also use dollar brace syntax to embed JavaScript expressions inside HTML
attributes:


By default, LitHtml
[escapes](http://stackoverflow.com/questions/7381974/which-characters-need-to-be-escaped-on-html)
any values embedded in the literal before rendering them. Thus it ensures that
you can never inject anything that’s not explicitly written in your application.
Everything is converted to a string before being rendered. This helps prevent
[XSS (cross-site-scripting)](https://en.wikipedia.org/wiki/Cross-site_scripting)
attacks.

For example, it is perfectly safe to embed user input inside LitHtml literal:



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

The first thing to notice is that the element has a child marked `#shadow-root`
which has all the DOM that will be rendered. The actual body of the custom
element is not rendered directly but it is referenced by the Shadow DOM using a
special tag called `<slot>`.

You can think about slot as a kind of symbolic link. You can use slots to import
content from the body of the custom element. If you have several items that you
want to import from the body you can use **named slots**.

In the following custom element we are passing information through three slots:
the default one, a named one called first-name and a second one called
last-name.

    <my-shadier-element>
      <i>Hello</i>
      <span slot="first-name">John</span>
      <span slot="last-name">Hardy</span>
    </my-shadier-element>

In the render method we slot elements to reference these items of passed in
data.

    class MyShadierElement extends HTMLElement {
      
      constructor() { 
        super();
        this.attachShadow({mode: 'open'}); // 1    
      }
      
      connectedCallback() {  
        this.render();
      } 
      
      render() { 
        render( 
          html`
          <h1>
            <slot></slot> 
            <slot name="first-name"></slot> 
            <slot name="last-name"></slot> 
          </h1>
          `, 
          this.shadowRoot
        );
      }
    }
      
    customElements.define('my-shadier-element', MyShadierElement);

[See a working version here (use
Chrome).](https://codepen.io/jhlagado/pen/WgWJNa?editors=1101)

![](https://cdn-images-1.medium.com/max/1600/1*TZS6YDK01FqqSS_JsvlF0g.png)

### Attributes and Properties

While slots are a very useful way to pass data to Custom Elements, we two other
ways at our disposal: passing values as attribute values through the DOM and
setting properties on the DOM element objects directly.

### Observing Attributes

### Observing Properties

### Handling Events

### Extracting Custom Elements

Custom elements can be decomposed into smaller components. This aids readability
and reuse.

For example, given this `Comment` component rendered from a LitHtml literal:

    const author = {
      name: 'John Hardy',
      avatar: '
    '
    };

    const date = new Date();
    const text = html`
        <p>
          Hello, this is my comment.
        </p>
    `;
      
    const literal = html`
      <my-comment .author=${author} .date=${date}>
        ${text}
      </my-comment>
    `;


And this definition

    class MyComment extends HTMLElement {
      
      constructor() { 
        super();
        this.attachShadow({mode: 'open'});
      }
      
      connectedCallback() {  
        this.render();
      } 
      
      render() { 
        render( 
          html`
          <div className="Comment">
            <div className="UserInfo">
              <img className="Avatar"
                src=${this.author.avatar}
                alt=${this.author.name}
              />
              <div className="UserInfo-name">
                ${this.author.name}
              </div>
            </div>
            <div className="Comment-text">
              <slot></slot>
            </div>
            <div className="Comment-date">
              ${this.date}
            </div>
          </div>
          `, 
          this.shadowRoot 
        );
      }
    }
      
    customElements.define('my-comment', MyComment);

[See a working version here (use
Chrome)](https://codepen.io/jhlagado/pen/QVPxqK?editors=1101)

This custom element accepts `author` (an object), `date` (a date) as props and
text is its content body.

The render method of this custom element is rather long and hard to read. It
probably should be decomposed into smaller more reusable components.

First, we will extract `MyAvatar`:

    class MyAvatar extends HTMLElement {
      
      constructor() { 
        super();
      }
      
      connectedCallback() {  
        this.render();
      } 
      
      render() { 
        render( 
          html`
            <img 
              src=${this.user.avatar}
              alt=${this.user.name}
            />
          `, 
          this
        );
      }
    }
      
    customElements.define('my-avatar', MyAvatar);

The `MyAvatar` component doesn’t need to know that it is being rendered inside a
`Comment`. This is why we have given its prop a more generic name of `user`
rather than `author`.

Next, we will extract a `UserInfo` component that renders an `Avatar` next to
the user’s name:

    class MyUserInfo extends HTMLElement {
      
      constructor() { 
        super();
      }
      
      connectedCallback() {  
        this.render();
      } 
      
      render() { 
        render( 
          html`
              <my-avatar .user=${this.user}></my-avatar> 
              <div>
                ${this.user.name}
              </div>
          `, 
          this
        );
      }
    }
      
    customElements.define('my-user-info', MyUserInfo);

This lets us simplify `Comment` 

    class MyComment extends HTMLElement {
      
      constructor() { 
        super();
        this.attachShadow({mode: 'open'});
      }
      
      connectedCallback() {  
        this.render();
      } 
      
      render() { 
        render( 
          html`
               
          `, 
          this.shadowRoot
        );
      }
    }
      
    customElements.define('my-comment', MyComment);

[See a working version here (use
Chrome)](https://codepen.io/jhlagado/pen/LJvBjY?editors=1101)

*This is still a working draft. Stay tuned for more updates.*
