# Why
- React embraces the fact that **rendering logic** is inherently coupled with other **UI logic**: how events are handled, how the state changes over time, and how the data is prepared for display
- Instead of artificially separating technologies by putting **markup** and **logic** in separate files, React separates concerns with loosely coupled units called “components” that contain both
- > https://www.youtube.com/watch?v=x7cQ3mrcKaY
# JSX
- JSX is a syntax extension to JavaScript. It comes with the full power of **JavaScript**

    ```jsx
    // tag syntax below is JSX
    const element = <h1>Hello, world!</h1>
    ```

    - Some kind of a template language
- **JSX produces *React "elements"***

- Any valid JavaScript expression can be put inside the ***curly braces*** in JSX
- Splitting JSX over multiple lines is good for readability
- Since JSX is closer to JavaScript than to HTML, React DOM uses **`camelCase`** property naming convention instead of HTML attribute names

- **JSX is an expression**
- After compilation, JSX expressions become **regular JavaScript function calls** and **evaluate to plain JavaScript objects**
    - This means that you can use JSX inside of `if` statements and `for` loops, assign it to variables, accept it as arguments, and return it from function
    1. Babel compiles JSX down to **`React.createElement()`** calls

        ```jsx
        // tag syntax below is JSX
        const element = <h1 className="greeting">Hello, world!</h1>
        
        const element = React.createElement(
            'h1',
            {className: 'greeting'},
            'Hello, world!'
        )
        ```

    2. `React.createElement()` performs some checks to help you write bug-free code but essentially it creates an object (***React elements***) like this:

        ```JSX
        // this structure is simplified
        const element = {
            type: 'h1',
            props: {
                className: 'greeting',
                children: 'Hello, world!'
            }
        }
        ```

    3. React reads these objects and uses them to construct the DOM and keep it up to date
- Wrap JSX over multiple lines in **parentheses** to avoid pitfalls of [automatic semicolon insertion](https://stackoverflow.com/questions/2846283/what-are-the-rules-for-javascripts-automatic-semicolon-insertion-asi)

- By default, React DOM escapes any values embedded in JSX before rendering them
    - **Everything is converted to a string before being rendered**. Thus it ensures that you can never inject anything that’s not explicitly written in your application. This helps prevent XSS (cross-site-scripting) attacks
- Use ["Babel" language definition](https://babeljs.io/docs/en/editors/) for your editor of choice so that both ES6 and JSX code is properly highlighted
- React doesn’t require using JSX
    - But most people find it helpful as a visual aid when working with UI inside the JavaScript code
    - It also allows React to show more useful error and warning messages
# JSX in depth
- Fundamentally, JSX provides **syntactic sugar** for the `React.createElement(component, props, ...children)` function

    ```jsx
    <MyButton color="blue" shadowSize={2}>
        Click Me
    </MyButton>
    // compile to
    React.createElement(
        MyButton,
        {color: 'blue', shadowSize:2},
        'Click Me'
    )
    <div className="sidebar" />
    // compile to
    React.createElement(
        'div',
        {className: 'sidebar'},
        null
    )
    ```

    - Online Babel compiler
- The first part of a JSX tag determines the type of React element
    - Capitalized types indicates that the JSX tag is referring to a React component
    - These tags get compiled into a direct reference to the **named variable**. So if you use the JSX `<Foo />`, `Foo` must be in scope
- Since JSX compiles into calls to `React.createElement`, the `React` library must also always be in scope from your JSX code
    - If you don’t use a JavaScript bundler and loaded React from a `<script>` tag, it is already in scope as the `React` global
- You can also refer to a React component using dot-notation from within JSX

    ```jsx
    const MyComponents = {
        DatePicker: function DatePicker(props) {
            return <div>Imagine a {props.color} datepicker here.</div>
        }
    }
    function BlueDatePicker() {
        return <MyComponents.DatePicker color="blue" />
    }
    ```

    - This is convenient if you have a single module that exports many React components
- When an element type starts with a lowercase letter, it refers to a **built-in** component like `<div>` or `<span>` and results in a ***string*** `'div'` or `'span'` passed to `React.createElement`
- Types that start with a capital letter like `<Foo />` compile to `React.createElement(Foo)` and correspond to a component defined or imported in your JavaScript file
    - Naming components with a capital letter is recommended
    - If you do have a component that starts with a lowercase letter, assign it to a capitalized variable before using it in JSX
- You cannot use a general JS expression as the React element type. If you do want to use a general expression to indicate the type of the element, just assign it to a capitalized variable first
- When you pass a string literal, its value is HTML-unescaped

    ```jsx
    // equivalent
    <MyComponent message="&lt;3" />
    <MyComponent message={'<3'} />
    ```

- If you pass no value for a prop, it defaults to `true`
    - In general, we don’t recommend using this because it can be confused with the ES6 object shorthand `{foo}` which is short for `{foo: foo}` rather than `{foo: true}`
    - This behavior is just there so that it **matches the behavior of HTML**
- Pass the whole props object using `...`

    ```jsx
    function App1() {
        return <Greeting firstName="Ben" lastName="Hector" />
    }
    function App2() {
        const props = {firstName: 'Ben', lastName: 'Hector'}
        return <Greeting {...props} />
    }
    ```

- Pick specific props

    ```jsx
    const Button = props => {
        const { kind, ...other } = props
        const className = kind === "primary" ? "PrimaryButton" : "SecondaryButton"
        // pick `onClick` and `children`
        return <button className={className} {...other} />
    }
    const App = () => {
        return (
            <div>
                <Button kind="primary" onClick={() => console.log("clicked!")}>
                Hello World!
                </Button>
            </div>
        )
    }
    ```

- In JSX expressions that contain both an opening tag and a closing tag, the content between those tags is passed as a special prop `props.children`
- JSX removes whitespace at the beginning and ending of a line. It also removes blank lines. New lines adjacent to tags are removed; new lines that occur in the middle of string literals are condensed into a single space
- A React component can also return an array of elements

    ```jsx
    render() {
        return [
            // Don't forget the keys
            <li key="A">First item</li>,
            <li key="B">Second item</li>,
            <li key="C">Third item</li>
        ]
    }
    ```

- **Function as children** 

    ```jsx
    function Repeat(props) {
        let items = []
        for (let i = 0; i < props.numTimes; i++) {
            items.push(props.children(i))
        }
        return <div>{items}</div>
    }
    function ListOfThing() {
        return (
            <Repeat numTimes={10}>
                {index => <div key={index}>This is item {index} in the list</div>}
            </Repeat>
        )
    }
    ```

- Children passed to a custom component can be anything, as long as that component transforms them into something React can understand before rendering
- `false`, `null`, `undefined`, and `true` are valid children. They simply **don't render**
    - This can be useful to conditionally render React elements
    - Some falsy values such as `0` are still rendered by React
    - If you want a value like `false`, `true`, `null`, or `undefined` to appear in the output, you have to convert it to a string first (using `String()`)
- Conditional rendering - In JavaScript, `true && expression` always evaluates to `expression`, and `false && expression` always evaluates to `false`
    - Whenever conditions become too complex, it might be a good time to extract a component