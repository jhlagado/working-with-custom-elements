# Chapter 4 - Attributes and Properties

While slots are a very useful way to pass markup-based data to Custom Elements, we have two other ways at our disposal: passing values as attribute values through the DOM and setting properties on the DOM elements directly.

## Observing Attributes

HTML attributes are string values that are accessible from JavaScript using the methods `hasAttribute`, `getAttribute`, `setAttribute` and `removeAttribute` on the component’s DOM element.

Custom Elements also provide a way to monitor the state of specific attributes and to notify the component with a callback when an attribute changes. To observe one of more attributes we need to add a getter method called `observedAttributes` to the component’s class (i.e. a static method). This method needs to return an array of attribute names to monitor.

For example, to monitor the `disabled` attribute on a custom element

    static get observedAttributes() {
        return ['disabled'];
    }

Now our component will get notified every time its disabled attribute is changed by being called back via its `attributeChangedCallback` which will inform us of the name of the attribute that changed, it’s old value and it’s new value.

This is usually enough information but in the case of boolean attributes we might decide to check the value of `hasAttribute` as well.

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

Attributes are very limited in what they can pass to a component. They can communicate their presence or absence from an element and they can also communicate a simple string value. Often though, this is not enough and there are cases in which we need to pass more complex data types to our components. Data types such as dates, arrays, sets, maps and dictionaries etc. To achieve this we need to make use of properties.

Properties are simply the values that we can get and set on the DOM element itself (i.e. not via attributes) and we can do this by getting access to the DOM element and setting one of its properties. What the element chooses to do with this property value is up to it.

    const input = document.getElementById('first-name');

    input.value = 'John';

As mentioned earlier, properties can also be assigned via LitHtml by using the dot prefix to distinguish this from an attribute value.

    const lastName = 'Hardy';

    const literal = html`
      <input id="last-name" type="text" .value=${lastName}>
    `;

While Custom Elements provide us with an `attributeChangedCallback` when the value of an attribute changes, with properties we are pretty much on our own. That said, JavaScript does already give us some powerful ways of intercepting property accesses with [getters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get) and [setters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set).

Take a look at this utility function `observeProperties` which will help us add observability to our properties.

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

When you hand this function an element and an array of property names to observe, it iterates through the array creates getters and setters for each property. These properties become in effect “virtual” properties because the public name of the property differs from where it is actually stored. For example, a property with the name `value` is actually stored in a property called `_value` but its getters and setters hide this fact.

When an observed property is assigned to, its setter function is called. If the new value is different from its old value (as determined by a shallow comparison) then a method named `propertyChangedCallback` is called to inform the component that an observed property has in fact changed.

The decision to use a shallow comparison rather than a deep (recursive) one was made for reasons of efficiency. As long as the values of the properties are treated as though they were [immutable](https://spapas.github.io/2018/04/05/easy-immutable-objects/) and that their values get replaced rather that modified then we can use this lightweight and computationally inexpensive approach to observing changes.

To see observed properties in action, let’s create a new component called `my-clock`.

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

In this code the `observeProperties` function is called in the constructor which goes through and adds getters and setters to each observed property. In this case only `time` is an observed property.

When the element is added to the document, `connectedCallback` is called and the component starts off an interval timer using `setInterval`. The component also saves the timer’s `intervalID` for cleanup later on if the component is ever removed from the document. The CustomElement life-cycle callback `disconnectedCallback` is called whenever a component is removed from the document.

In our previous examples, `connectedCallback` was where we first called the `render` method but now that we have at least one observed property, the render will be called whenever it is changed which is something that happens repeatedly with an interval one second. The `render` method is also called when the `time` property is first initialized.

Now that we have an easy way to react to changes in properties, let’s return now to our earlier example of the `my-counter` component with its `up` and `down` buttons. You may recall that each click event handler that modified the state of the counter needed to call the `render` method if the changes made were to be reflected visually. With observed properties, this is no longer necessary.

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
              <button @click=${
                e => this.upClick()}>up</button>
              <button @click=${
                e => this.downClick()}>down</button>
              <button @click=${
                e => this.resetClick()}>reset</button>
            </div>
          `, 
          this.shadowRoot 
        );
      }
    }
      
    customElements.define('my-counter', MyCounter);

[See a working version here (use Chrome)](https://codepen.io/jhlagado/pen/gdNgKM?editors=1001)

