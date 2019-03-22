- React DOM is React's way of representation of the web page

- React elements (created by `React.createElement()` which is compiled from JSX) are plain objects that can be passed as props like any other data
    - Think of React element as descriptions of what you want to see on the screen
    - React reads these objects and uses them to construct the DOM and keep it up to date
    - **React DOM takes care of updating the DOM to match the React elements**

- Components are like **JavaScript functions**. They **accept arbitrary inputs** (called **`props`**) and return React elements describing what should appear on the screen
    - Function components and class components
- When React sees an element representing a user-defined component (**capitalized**), it passes JSX attributes to this component as **a single object called `props`**
    - React treats components starting with lowercase letters as DOM tags
- Typically, new React apps have a single `App` component at the very top
    - If you integrate React into an existing app, you might start bottom-up with a small component like `Button` and gradually work your way up to the top of the view hierarchy
- A good rule of thumb of **extracting components** is that if a part of UI is used several times, or is complex enough on its own, it's a good candidate to be a reusable component

- `props` are read-only
- Never modify a component's own props
- All React components must act like [pure functions](https://en.wikipedia.org/wiki/Pure_function) with respect to their props

- `<div id="root"></div>` is the root DOM node. Everything inside it will be managed by React DOM

    ```jsx
    const element = <h1>hello</h1>
    ReactDOM.render(element, document.getElementById('root'))
    ```

    - > Applications built with just React usually have a single root DOM node
    - > If you're integrating React into an existing app, you may have as many isolated root DOM nodes as you like