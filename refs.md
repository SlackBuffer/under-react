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
- If you have absolutely no control over the child component implementation, the last option is to use [`findDOMNode`](https://reactjs.org/docs/react-dom.html#finddomnode) (discouraged and deprecated in `StrictMode`)

    ```js
    ReactDOM.findDOMNode(component)
    ```

    - `findDOMNode` cannot be used on function component
    - `findDOMNode` only works on ***mounted components*** (that is, components that have been **placed in the DOM**). If you try to call this on a component that has not been mounted yet (like calling `findDOMNode()` in `render()` on a component that has yet to be created) an **exception** will be thrown
    - If this component has been mounted into the DOM, this returns the corresponding **browser DOM element**
       1. When a component renders to `null` or `false`, `findDOMNode` returns `null`
       2. When a component renders to a string, `findDOMNode` returns a text DOM node containing that value
       3. As of React 16, a component may return **a fragment with multiple children**, in which case `findDOMNode` will return the DOM node corresponding to **the first non-empty child**
    - Useful for **reading values out of the DOM**, such as form field values and performing DOM measurements
    - In most cases, it is discouraged because it pierces the component abstraction
    - In most cases, you can attach a ref to the DOM node and avoid using findDOMNode at all
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
- Ref forwarding is a technique for automatically passing a `ref` **through a component to one of its children**
    - Can be useful for reusable component libraries
## Forwarding refs to **DOM components**
- `FancyButton`

    ```jsx
    function FancyButton(props) {
        return (<button>{props.children}</button>)
    }
    ```

    - `FancyButton` component hide its implementation detail including its rendered output
    - Other components using `FancyButton` usually needn't obtain a ref to the inner `button` DOM element
    - This is good as it prevents components from relying on each other's DOM structure too much
- Such encapsulation is desirable for application-level components
- But it can be inconvenient for highly reusable "leaf" components like `FancyButton` or `MyTextInput`
    - These components tend to be used throughout the application **in a similar manner as a regular DOM `button` and `input`**, and accessing their DOM nodes may be unavoidable for managing focus, selection, or animation
- `React.forwardRef` accepts a render function that receives `props` and `ref` parameters and ***returns a React node***
- Ref forwarding is an opt-in feature that let some components **take a ref they receive** and **pass it further down** ("forward" it) to a child

    ```jsx
    // defines `FancyButton` component
    const FancyButton = React.forwardRef((props, ref) => (
        // `button` here is DOM component
        <button ref={ref}>{props.children}</button>
    ))

    // a component that uses `FancyButton` 
    const ref = React.createRef()
    <FancyButton ref={ref}>Click me!</FancyButton>
    ```

    - `FancyButton` use `React.forwardRef` to obtain the `ref` passed to it, and pass it down further ("forward" it) to the DOM `button` it renders
    - This way, components using `FancyButton` can get a ref to the underlying `button` DOM node and access it using `ref.current`
- The second `ref` argument only exists when you define a component with `React.forwardRef` call
- Regular function or class components don't receive the `ref` argument, and `ref` is not available in props either
- Ref forwarding is not limited to DOM components. Can forward refs to class component instances, too
## (Note for component library maintainers)[https://reactjs.org/docs/forwarding-refs.html#note-for-component-library-maintainers]
## Forwarding refs in higher-order components
- Log HOC

    ```jsx
    function logProps(WrappedComponent) {
        class LogProps extends React.Component {
            componentDidUpdate(prevProps) {
                console.log('old props:', prevProps)
                console.log('new props:', this.props)
            }
            render() {
                // refs will not get passed through
                return <WrappedComponent {...this.props} />
            }
        }
        return LogProps
    }
    ```

    - Refs will not get passed through even using `{...this.props}`
- `ref` is not a normal prop. Like `key`, it's **handled differently by React**
    - Use `React.forwardRef` to pass down refs
- If you add a ref to a HOC, the ref will refer to the **outermost container component**, not the wrapped component

    ```jsx
    // The FancyButton component imported is the `LogProps` HOC
    import FancyButton from './FancyButton'

    // a component that uses `FancyButton` 
    const ref = React.createRef()
    // ref will point to LogProps instead of the inner FancyButton component
    // This means we can't call e.g. ref.current.focus()
    <FancyButton ref={ref} />
    ```

- Explicitly forward refs to the inner `FancyButton` component

    ```jsx
    const FancyButton = React.forwardRef((props, ref) => (
        <button ref={ref}>{props.children}</button>
    ))
    function logProps(WrappedComponent) {
        class LogProps extends React.Component {
            componentDidUpdate(prevProps) {
                console.log('old props:', prevProps)
                console.log('new props:', this.props)
            }
            render() {
                const {forwardRef, ...ref} = this.props
                return <WrappedComponent ref={forwardedRef} {...rest} />
            }
        }
        return React.forwardRef((props, ref) => {
            return <LogProps {...props} forwardedRef={ref} />
        }
    }
    // a component that uses HOC `FancyButton` 
    const ref = React.createRef()
    <FancyButton ref={ref} />
    ```

## Displaying a custom name in DevTools
- `React.forwardRef` accepts a render function
- React DevTools uses this function to determine what to display for the ref forwarding component

    ```jsx
    // this component will appear as "ForwardRef" in the DevTools
    const WrappedComponent = React.forwardRef((props, ref) => {
        return <LogProps {...props} forwardedRef={ref} />
    })

    // If you name the render function, DevTools will also include its name (”ForwardRef(myFunction)”)
    const WrappedComponent = React.forwardRef(
        function myFunction(props, ref) {
            return <LogProps {...props} forwardedRef={ref} />
        }
    )

    // set the function’s displayName property to include the component you’re wrapping
    function logProps(Component) {
        class LogProps extends React.Component {
            // ...
        }
        function forwardRef(props, ref) {
            return <LogProps {...props} forwardedRef={ref} />
        }
        // Give this component a more helpful display name in DevTools
        // e.g. "ForwardRef(logProps(MyComponent))"
        const name = Component.displayName || Component.name
        forwardRef.displayName = `logProps(${name})`
        return React.forwardRef(forwardRef)
    }
    ```

# Uncontrolled components
- Use a ref to get form values from the DOM

    ```jsx
    class NameForm extends React.Component {
        constructor(props) {
            super(props);
            this.handleSubmit = this.handleSubmit.bind(this)
            this.input = React.createRef()
        }

        handleSubmit(event) {
            alert('A name was submitted: ' + this.input.current.value)
            event.preventDefault()
        }

        render() {
            return (
                <form onSubmit={this.handleSubmit}>
                    <label>
                        Name:
                        <input type="text" ref={this.input} />
                    </label>
                    <input type="submit" value="Submit" />
                </form>
            )
        }
    }
    ```

- Since an uncontrolled component keeps the source of truth in the DOM, it is sometimes easier to **integrate React and non-React code** when using uncontrolled components
## Default values
- In React rendering lifecycle, the **`value` attribute on form elements will override the value in the DOM**
- With an uncontrolled component, you often want React to specify the initial value, but leave subsequent updates uncontrolled
- To handle this case, you can specify a `defaultValue` attribute instead of `value`

    ```jsx
    render() {
        return (
            <form onSubmit={this.handleSubmit}>
                <label>
                    Name:
                    <input
                        defaultValue="Bob"
                        type="text"
                        ref={this.input} />
                </label>
                <input type="submit" value="Submit" />
            </form>
        )
    }
    ```

- `<input type="checkbox">` and `<input type="radio">` support **`defaultChecked`**, and `<select>` and `<textarea>` supports `defaultValue`
## The file input Tag
- In React, an `<input type="file" />` is always an uncontrolled component because its value can only be set by a user, and not programmatically
- Use the (File API)(https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications) to interact with files

    ```jsx
    class FileInput extends React.Component {
        constructor(props) {
            super(props)
            this.handleSubmit = this.handleSubmit.bind(this)
            this.fileInput = React.createRef()
        }
        handleSubmit(event) {
            event.preventDefault()
            alert(
                `Selected file - ${
                    this.fileInput.current.files[0].name
                }`
            )
        }
        render() {
            return (
                <form onSubmit={this.handleSubmit}>
                    <label>
                        Upload file:
                        <input type="file" ref={this.fileInput} />
                    </label>
                    <br />
                    <button type="submit">Submit</button>
                </form>
            )
        }
    }

    ReactDOM.render(
        <FileInput />,
        document.getElementById('root')
    )
    ```