
## Working with Custom Elements

![](https://cdn-images-1.medium.com/max/2000/1*hj_oFpaaV6tfhsv_Jt1P_A.png)

[Custom Elements](https://html.spec.whatwg.org/dev/custom-elements.html) are a feature of modern browsers which allow you to modularise and install your JavaScript components into the browser itself in order to extend it in new and powerful ways. Custom Elements are HTML components which have their own self-contained markup, styling and behaviour.

For example, say we wanted to create an element that formatted a person’s name

    <name-card first-name="John" last-name="Hardy"></name-card>

If the name-card tag was registered as a custom element component then the browser could be made to render it as

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

## Building Custom Elements

The examples so far have concentrated on selecting some existing DOM element in the HTML page and then replacing its contents with some JavaScript-generated content. This is fine as it goes but what we’re really after is a way to build reusable web components that exist autonomously in the browser. When they appear in the HTML of the page, they become active, know how and when to render themselves and respond to browser events with their own behaviours.

Let’s start with a simple example

    <my-element></my-element>

Note that custom elements must have at least one hyphen in their name. Also note that custom elements always must have a closing tag.

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

[See a working version here (use Chrome).](https://codepen.io/jhlagado/pen/QVPQWb?editors=1101)

Custom elements are made using a JavaScript class definition (1) which is registered with the browser (2).

The class definition must extend one of the built in classes of the browser which implements an element. While it is possible to extend and inherit the behaviours of built-in element types such as HTMLButtonElement, this is currently still a poorly supported feature in browsers. Therefore all of the examples here will extend from the generic HTMLElement.

In this basic example, I have provided a constructor (3) which currently does nothing except call super(). In this simple example it could have been omitted.

The class definition also provides an implementation a Custom Element life-cycle hook called connectedCallback() (4) which is called when the element is first added to the document. This callback is a good place to initially update the DOM with new content. In our example it does this by calling its own method render() (4) which in turn calls the LitHtml render method passing a LitHtml literal and the element’s own DOM to render to.

Note: unlike React components the render() method has no special meaning. Deciding when to update the DOM is completely left up to the custom element to decide.

The resulting HTML in the browser looks like this.

![](https://cdn-images-1.medium.com/max/2000/1*7QFHXjMufZYy9_yZJflVtw.png)

While this is already pretty good, it has the downside of the custom element replacing its own body content. This makes it difficult to pass additional information in the body of the custom element.

We can overcome this problem and at the same time unlock even more powerful features of Custom Elements by using another feature of the modern browser platform called the [Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM).

## Shadow DOM

Any HTML element in the browser act like a tiny universe of its own by having its own Shadow DOM. An element can acquire a shadowRoot during its construction phase and this is what it will render to the browser instead of the elements in its body. The body content of the element is invisible therefore and may be used for other purposes such as passing data to the component.

The HTML elements inside the element’s Shadow DOM are isolated from the elements outside and can be styled and controlled independently of everything else on the page.

Given the following custom element

    <my-shady-element>
      <i>Hello</i>
    </my-shady-element>

You can see that it contains HTML children elements in its body. Because this component will be defined to have a Shadow DOM these children elements won’t get displayed directly.

Let’s now turn to the component’s definition and see how we can use a Shadow DOM.

    class MyShadyElement extends HTMLElement {
      
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
            <h1>
              <slot></slot> world!
            </h1>
          `, 
          this.shadowRoot
        );
      }
    }
      
    customElements.define('my-shady-element', MyShadyElement);

[See a working version here (use Chrome).](https://codepen.io/jhlagado/pen/GXLxGm?editors=1101)

When this component gets constructed, it calls its inherited method attachShadow() and this creates a Shadow DOM for this element and puts the root of this structure in a property called shadowRoot. When the render function is invoked, unlike in the previous example, it renders to the element’s shadowRoot rather than to the element itself.

The resulting HTML in the browser looks a bit different to the previous example

![](https://cdn-images-1.medium.com/max/2000/1*ok8m9Y35gSS-tPZuAZOX6A.png)

The first thing to notice is that the element has a child marked #shadow-root which contains all the DOM that will be rendered. The actual body of the custom element is not rendered directly but it gets referenced by the Shadow DOM using a special tag called slot.

You can think about slot as a kind of symbolic link. You can use slots to link to content in the body of the custom element and render it right in the middle of the Shadow DOM. Nothing actually is moved, these are references but the effect is the same as it the DOM elements had been moved.

If you have several items that you want to reference from the body you can use **named slots**. In the following custom element we are passing information through three slots: the default one, a named one called first-name and a second one called last-name.

    <my-shadier-element>
      <i>Hello</i>
      <span slot="first-name">John</span>
      <span slot="last-name">Hardy</span>
    </my-shadier-element>

In the render method we use slot elements to reference these items of passed in data.

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

[See a working version here (use Chrome).](https://codepen.io/jhlagado/pen/WgWJNa?editors=1101)

![](https://cdn-images-1.medium.com/max/2060/1*TZS6YDK01FqqSS_JsvlF0g.png)

## Refactoring Custom Elements

As the complexity of a custom element grows, the amount of HTML to be rendered also tends to increase. This in turn makes our code longer and harder to read and maintain. It may come to a point where it makes sense to decompose our component into smaller components. Decomposition can aid us by improving readability and code reusability.

For example, consider this my-comment component which could be used to represent a comment on a blog or social media site.

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
          <div>
            <div>
              <img className="Avatar"
                src=${this.author.avatar}
                alt=${this.author.name}
              />
              <div>
                ${this.author.name}
              </div>
            </div>
            <div>
              <slot></slot>
            </div>
            <div>
              ${this.date}
            </div>
          </div>
          `, 
          this.shadowRoot 
        );
      }
    }
      
    customElements.define('my-comment', MyComment);

which we will render directly using LitHtml

    const author = {
      name: 'John Hardy',
      avatar: '[https://bit.ly/2OHRT9v](https://bit.ly/2OHRT9v)'
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

    render(
      literal,
      document.getElementById('root')
    );

[See a working version here (use Chrome)](https://codepen.io/jhlagado/pen/QVPxqK?editors=1101)

You can see that the render method of the custom element is rather long and hard to read. We can do better by decomposing this unwieldy structure into smaller and more reusable components.

Let’s start by extracting the my-avatar component

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

my-avatar doesn’t need to know that it is being rendered inside a my-comment component. In fact the less it knows about the surrounding context in which it is used the better.

Next, we will extract a my-user-info component that renders an my-avatar component next to the user’s name

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
            <div>
              <my-avatar .user=${this.user}></my-avatar> 
              <div>
                ${this.user.name}
              </div>
            </div>
          `, 
          this
        );
      }
    }
      
    customElements.define('my-user-info', MyUserInfo);

Now let’s us simplify my-comment

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
          <div>           
            <my-user-info .user=${this.author}></my-user-info>
            <div>
              <slot></slot>
            </div>
            <div>
              ${date}
            </div>
          </div>
          `, 
          this.shadowRoot
        );
      }
    }
      
    customElements.define('my-comment', MyComment);

[See a working version here (use Chrome)](https://codepen.io/jhlagado/pen/LJvBjY?editors=1101)

## Handling Events

LitHtml provides is with a way to add event handlers to your markup. Events can be built-in ones such as click and focus or they can be custom events. An event handler is usually a function that is called when the element receives an event of a certain type.

To demonstrate event handling, we’ll create a my-counter component which has three buttons up and down and reset.

    class MyCounter extends HTMLElement {
      
      constructor() { 
        super();
        this.attachShadow({mode: 'open'});
        this.counter = 0; 
      }
      
      connectedCallback() {  
        this.render();
      } 
      
      upClick() {
        this.counter++;
        this.render();
      }
      
      downClick() {
        this.counter--;
        this.render();
      }

    resetClick() {
        this.counter = 0;
        this.render();
      }
      
      render() { 
        render( 
          html`
            <h1>Counter:</h1>
            <h2>${this.counter}</h2>           
            <div>
              <button [@click](http://twitter.com/click)=${e => this.upClick()}>up</button>
              <button [@click](http://twitter.com/click)=${e => this.downClick()}>down</button>           
              <button [@click](http://twitter.com/click)=${e => this.resetClick()}>reset</button>        
            </div>
          `, 
          this.shadowRoot 
        );
      }
    }
      
    customElements.define('my-counter', MyCounter);

[See a working version here (use Chrome)](https://codepen.io/jhlagado/pen/RYzKwm?editors=1101)

One thing you will notice is that every event handler that alters the counter property needs to make a call to render it. We can reduce the need for adding render calls everywhere by observing when things change. We will revisit this example a bit later.

## Attributes and Properties

While slots are a very useful way to pass markup-based data to Custom Elements, we have two other ways at our disposal: passing values as attribute values through the DOM and setting properties on the DOM elements directly.

## Observing Attributes

HTML attributes are string values that are accessible from JavaScript using the methods hasAttribute(), getAttribute(), setAttribute() and removeAttribute() on the component’s DOM element.

Custom Elements also provide a way to monitor the state of specific attributes and to notify the component with a callback when an attribute changes. To observe one of more attributes we need to add a getter method called observedAttributes() to the component’s class (i.e. a static method). This method needs to return an array of attribute names to monitor.

For example, to monitor the disabled attribute on a custom element

    static get observedAttributes() {
        return ['disabled'];
    }

Now our component will get notified every time its disabled attribute is changed by being called back via its attributeChangedCallback() which will inform us of the name of the attribute that changed, it’s old value and it’s new value.

This is usually enough information but in the case of boolean attributes we might decide to check the value of hasAttribute() as well.

    attributeChangedCallback(name, oldValue, newValue) {
      
      if (this.hasAttribute('disabled')) {
        this.setAttribute('tabindex', '-1');
        this.setAttribute('aria-disabled', 'true');
      } 
      else {
        this.setAttribute('tabindex', '0');
        this.setAttribute('aria-disabled', 'false');
      }
    }

## Observing Properties

Attributes are very limited in what they can pass to a custom component, they can communicate their presence or absence from an element, they can also communicate a simple string value which the the element is free to interpret as it likes. Often though, this is not enough and often we need to pass more complex data types to our components such as date objects, arrays, sets, maps and dictionaries. To achieve this we need to make use of properties.

Properties are simply the values that we set and get on the DOM element itself (not via attributes) and we can do this by getting access to the element and setting one of it’s properties. What the element chooses to do with the property is up to it.

    const input = document.getElementById('first-name');

    input.value = 'John';

As mentioned earlier, properties can be assigned via LitHtml by using the dot prefix to distinguish them from attributes.

    const lastName = 'Hardy';

    const literal = html`
      <input id="last-name" type="text" .value=${lastName}>
    `;

While Custom Elements provide us with a callback when an attribute changes, with properties we are pretty much on our own. At the same time JavaScript already gives us a powerful way of intercepting property access by using [getters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get) and [setters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set).

Let’s just start by talking about this utility function observeProperties() which can help us add observability to properties.

    function observeProperties(object, props) {
      for (let prop of props) {

        // if the component already has a property of this 
        // name then save it for later

        const hasProp = object.hasOwnProperty(prop)
        let initValue; 
        if (hasProp) {
          initValue = object[prop];
          delete object[prop]; 
        }
        
        // define getters and setters for this property name

        const key = `_${prop}`;
        Object.defineProperty(object, prop, {
        
          get() {
            return object[key]
          },
        
          set(value) {
            const oldValue = object[key];
            object[key] = value;
            if (oldValue !== value) {
              object.propertyChangedCallback(
                prop, value, oldValue
              );
            }
          }
        });
        
        // if we saved an old property value earlier 
        // reassign it to the component

        if (hasProp) {
          object[prop] = initValue;
        }
      }
    }

This function iterates through our array of property names and creates getters and setters for each one. The properties are in effect virtual properties because their actual values are stored in a “private” property with the same name but prefixed with an underscore. For example, a property called value is actually stored in a property called _value but has getters and setter to hide this fact from the outside.

When a property is assigned to, its setter function is called. If the new value is different from its old value (by using a shallow comparison) then the propertyChangedCallback() is called informing the component that the property has in fact changed.

The choice of using a shallow comparison rather than a deep one was made for efficiency. As long as the values of properties are treated as [immutable](https://spapas.github.io/2018/04/05/easy-immutable-objects/) and replaced rather that modified, then we can use this computationally in expensive approach to observing properties.

To see observed properties in action create a new component called my-clock.

    class MyClock extends HTMLElement {
      
      constructor() { 
        super();
        this.attachShadow({mode: 'open'});
        observeProperties(this, ['time']);
        this.updateTime();
      }
      
      connectedCallback() {  
        this.intervalID = setInterval(
          () => this.updateTime(), 
          1000 
        );
      }
      
      disconnectedCallback() {
        clearInterval(this.intervalID);
      }
      
      updateTime() {
        this.time = new Date().toLocaleTimeString();
      }
      
      propertyChangedCallback(name, value, oldValue) {
        this.render();
      }
      
      render() { 
        render( 
          html`
            <h1>The current time is:</h1>
            <h2>${this.time}</h2>
          `, 
          this.shadowRoot
        );
      }
    }
      
    customElements.define('my-clock', MyClock);

[See a working version here (use Chrome)](https://codepen.io/jhlagado/pen/oPrXQP?editors=1101)

In this code the observeProperties method is called in the constructor which goes through and adds getters and setters to each property, in this case only time is an observed property.

When the element is added to the document, the component starts an interval and saves its intervalID for cleanup later if the component is ever removed from the document. The CustomElement life-cycle callback disconnectedCallback() is called whenever a component is removed.

In our previous examples, the connectedCallback() method was where we first called the render() method. But now that we have at least one observed property, the render will be called whenever it is changed which is something that happens repeatedly with an interval one second. The render method will be called when the time property is first initialised.

Let’s return now to our earlier example of the my-counter component. As I mentioned, each click event handler would need to call the render method if the changes it made were to be reflected. With observed properties, that is no longer needed.

Furthermore, because we can rely on the counter being initialised after we start observing it, we can dispense with the connectedCallback() method altogether.

    class MyCounter extends HTMLElement {
      
      constructor() { 
        super();
        this.attachShadow({mode: 'open'});
        observeProperties(this, ['counter']);  
        this.counter = 0; 
      }
      
      propertyChangedCallback(name, value, oldValue) { 
        this.render(); 
      }
      
      upClick() {
        this.counter++;  
      }
      
      downClick() {
        this.counter--;
      }

      resetClick() {
        this.counter = 0;
      }
      
      render() { 
        render( 
          html`
            <h1>Counter:</h1>
            <h2>${this.counter}</h2>           
            <div>
              <button [@click](http://twitter.com/click)=${e => this.upClick()}>up</button>
              <button [@click](http://twitter.com/click)=${e => this.downClick()}>down</button>           
              <button [@click](http://twitter.com/click)=${e => this.resetClick()}>reset</button>        
            </div>
          `, 
          this.shadowRoot 
        );
      }
    }
      
    customElements.define('my-counter', MyCounter);

[See a working version here (use Chrome)](https://codepen.io/jhlagado/pen/gdNgKM?editors=1001)

To be Continued…

*PLEASE NOTE: This is still a working draft. Stay tuned for more updates.*
