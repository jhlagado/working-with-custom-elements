## Refactoring Custom Elements

As the complexity of a custom element grows, the amount of HTML to be rendered also tends to increase. This in turn makes our code longer and harder to read and maintain. It may come to a point where it makes sense to decompose our component into smaller components. Decomposition can aid us by improving readability and code reusability.

For example, consider this `my-comment` component which could be used to represent a comment on a blog or social media site.

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
              >
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

Let’s start by extracting the `my-avatar` component

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
            >
          `, 
          this
        );
      }
    }
      
    customElements.define('my-avatar', MyAvatar);

`my-avatar` doesn’t need to know that it is being rendered inside a `my-comment` component. In fact the less it knows about the surrounding context in which it is used the better.

Next, we will extract a `my-user-info` component that renders an `my-avatar` component next to the user’s name

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

Now let’s us simplify `my-comment`

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

