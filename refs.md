# Refs
- React provide a way to access DOM nodes (elements) or React elements **created in the `render` method**
- > In the typical React dataflow, props are the only way that parent components interact with their children. To modify a child, you re-render it with new props
- There are a few cases where you need to **imperatively modify a child** outside of the typical dataflow
## When to use
- A few good use cases for refs
   1. Managing focus, text selection, or media playback
   2. Triggering imperative animations
   3. Integrating with third-party DOM libraries
- **Avoid using refs for anything that can be done declaratively**
    - For example, instead of exposing `open()` and `close()` methods on a `Dialog` component, pass an `isOpen` prop to it
- If your intention to use refs is to make things happen, think critically about where state should be owned in the component hierarchy
    - Often, it becomes clear that the proper place to own that state is at higher level in the hierarchy
## Usage
### Exposing child ***components*** (instances) to parent ***components*** 
- Refs are **created** using `React.createRef` and **attached** to React elements **via** the `ref` attribute

    ```jsx
    class MyComp extends React.Component {
        constructor(props) {
            super(props)
            this.myRef = React.createRef()
        }
        render() {
            return <div ref={this.myRef} />
        }
    }
    ```

    - Refs are commonly assigned to an **instance property** when a component is constructed so they can be referenced **throughout the component**
    - **Cannot** use the `ref` attribute on function **components** because they don’t have instances
- When a `ref` is passed to an element **in `render`**, a reference to that node becomes ***accessible* at the `current` attribute of the `ref`** variable

    ```js
    const node = this.myRef.current
    ```

- The value of the `ref` differs depending on the **type of the node**
    - When the `ref` attribute is used on an **HTML element**, the `ref` receives the **underlying DOM element** as its `current` property

        ```jsx
        // DOM element `ref` used for component that encapsulates it (not exposing to that component's parent component)
        class CustomTextInput extends React.Component {
            constructor(props) {
                super(props)
                this.textInput = React.createRef()
                this.focusTextInput = this.focusTextInput.bind(this)
            }
            focusTextInput() {
                // Explicitly focus the text input using the raw DOM API
                this.textInput.current.focus()
            }
            render() {
                return (
                    <div>
                        <input
                            type="text"
                            ref={this.textInput} />

                        <input
                            type="button"
                            value="Focus the text input"
                            onClick={this.focusTextInput}
                        />
                    </div>
                )
            }
        }
        ```

        - React will ***assign*** the `current` property with the DOM element the component mounts, and **assign it back to `null`** when it unmounts
        - `ref` updates happen ***before*** `componentDidMount` and `componentDidUpdate` lifecycle methods
    - When the `ref` attribute is used on a custom **class component**, the `ref` object receives the mounted ***instance*** of the component as its `current`

        ```jsx
        class AutoFocusTextInput extends React.Component {
            constructor(props) {
                super(props)
                this.textInput = React.createRef()
            }
            componentDidMount() {
                // has access to the mounted class instance's `focusTextInput` method
                this.textInput.current.focusTextInput()
            }
            render() {
                return (
                    <CustomTextInput ref={this.textInput} />
                )
            }
        }
        ```

        - The `<input>` element get focused immediately after mounting
- If you need a `ref` ***to*** a function component, **convert** it to a class component, just like when you do when you need lifecycle methods or state
- It's ok to use the `ref` attribute **inside** a function component as long as you **refer to** a DOM element or a class component
### Exposing **DOM node** refs to ***parent*** components
- In rare cases, you might want to have access to a child’s DOM node from a parent component
    - This is generally not recommended because it **breaks component encapsulation**
    - It can occasionally be useful for triggering focus or measuring the size or position of a child DOM node
- For such cases
   1. React 16.3 or higher
        - Use ref forwarding
   3. React 16.2 or lower, or needing more flexibility than provided by ref forwarding
        - [Explicitly pass a ref as a **differently named props**](https://gist.github.com/gaearon/1a018a023347fe1c2476073330cc5509)

            ```jsx
            function CustomTextInput(props) {
                // use `ref` on `<input>` element
                return (<div><input ref={props.inputRef} /></div>)
            }
            class Parent extends React.Component {
                constructor(props) {
                    super(props)
                    this.inputElement = React.createRef()
                }
                render() {
                    // expose a special prop (can be named anything other than `ref`) on the child
                    return (<CustomTextInput inputRef={this.inputElement} />)
                }
            }
            ```

            - `this.inputElement.current` in `Parent` will be set to the DOM node corresponding to the `<input>` element in the `CustomTextInput`
            - This approach works for **both** function components and class components
            - Another benefit of this pattern is that it works several components deep (just pass down)
- When possible, we advise against exposing DOM nodes, but it can be a useful escape hatch
- This approach requires you to add some code to the child component
- If you have absolutely no control over the child component implementation, the last option is to use [ ] [`findDOMNode`](https://reactjs.org/docs/react-dom.html#finddomnode) (discouraged and deprecated in `StrictMode`)
## Callback refs
- Callback refs gives more fine-grain control over when refs are set and unset
- Instead of passing a `ref` attribute created by `createRef`, **pass a function** (to `ref` attribute) that receives the React component instance or HTML DOM element as its **argument**, which can be stored and accessed elsewhere

    ```jsx
    class CustomTextInput extends React.Component {
        constructor(props) {
            super(props)
            this.textInput = null
            this.setTextInputRef = element => {
                this.textInput = element
            }
            this.focusTextInput = () => {
                // focus the text input using the raw DOM API
                if (this.textInput) this.textInput.focus()
            }
        }
        componentDidMount() {
            // autofocus the input on mount
            this.focusTextInput()
        }
        render() {
            return (
                <div>
                    <input
                        type="text"
                        ref={this.setTextInputRef}
                    />
                    <input
                        type="button"
                        value="Focus the text input"
                        onClick={this.focusTextInput}
                    />
                </div>
            )
        }
    }
    ```

    - React will ***call*** the `ref` callback ***with the DOM element*** when the component mounts, and call it with null when it unmounts
    - Refs are guaranteed to be up-to-date before `componentDidMount` or `componentDidUpdate` fires
- Ok to pass callback refs between components like with object refs created with `React.createRef`

    ```jsx
    function CustomTextInput(props) {
        return (<div><input ref={props.inputRef} /></div>)
    }
    class Parent extends React.Component {
        render() {
            return (<CustomTextInput inputRef={el => this.inputElement = el} />)
        }
    }
    ```

    - `this.inputElement` in `Parent`will be set to the DOM node corresponding to the `<input>` element in `CustomTextInput`
### Caveats with callback  refs
- If the `ref` callback is defined as an inline function, it will get **called twice during updates**, first with `null` and then again with the DOM element
    - This is because a new instance of the function is created with **each render**, so React needs to **clear the old ref and set up the new one**
- Avoid this by defining the `ref` callback as a bound method on the class, but note that it shouldn't matter in most cases
## Legacy string refs API
- [Some issues](https://github.com/facebook/react/pull/8333#issuecomment-271648615)
# Ref forwarding
[ ] https://reactjs.org/docs/forwarding-refs.html
[ ] https://reactjs.org/docs/uncontrolled-components.html#the-file-input-tag
