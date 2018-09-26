# Chapter 3 - Handling Events

LitHtml provides us with a way to add event handlers to your markup. Events can be the built-in ones such as `click` and `focus` or they can be custom events. An event handler is a function that is called when the element receives an event of a certain type.

To demonstrate event handling, we’ll create a `my-counter` component which contains three buttons `up`, `down` and `reset`. These buttons will change the value of a counter property.

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
              <button @click=${e => this.upClick()}>up</button>
              <button @click=${e => this.downClick()}>down</button>           
              <button @click=${e => this.resetClick()}>reset</button>        
            </div>
          `, 
          this.shadowRoot 
        );
      }
    }
      
    customElements.define('my-counter', MyCounter);

[See a working version here (use Chrome)](https://codepen.io/jhlagado/pen/RYzKwm?editors=1101)

One thing you will notice is that every event handler that alters the counter property needs to make a call to render it. We can reduce the need for adding render calls everywhere by observing when things change. We will revisit this example a bit later.
