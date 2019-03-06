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
- JSX produces *React "elements"*

- Any valid JavaScript expression can be put inside the ***curly braces*** in JSX
- Splitting JSX over multiple lines is good for readability
    - While it isn’t required, when doing this, it's recommend to wrap it in parentheses to avoid the pitfalls of [ ] [automatic semicolon insertion](https://stackoverflow.com/questions/2846283/what-are-the-rules-for-javascripts-automatic-semicolon-insertion-asi)
- Since JSX is closer to JavaScript than to HTML, React DOM uses **`camelCase`** property naming convention instead of HTML attribute names

- **JSX is an expression**
- After compilation, JSX expressions become **regular JavaScript function calls** and **evaluate to plain JavaScript objects**
    - This means that you can use JSX inside of `if` statements and `for` loops, assign it to variables, accept it as arguments, and return it from function
    1. Babel compiles JSX down to **`React.createElement()`** calls

        ```jsx
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
- Use ["Babel" language definition](https://babeljs.io/docs/en/editors/) for your editor of choice so that both ES6 and JSX code is properly highlighted.
- React doesn’t require using JSX
    - But most people find it helpful as a visual aid when working with UI inside the JavaScript code
    - It also allows React to show more useful error and warning messages.