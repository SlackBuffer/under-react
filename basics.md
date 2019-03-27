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
- Class components should always call the base constructor with `props`
- `this.props` is set up by React itself and `this.state` has a special meaning
    - It's free to add additional fields to the class manually if you need to store something that doesn't participate in the data flow (like `this.timerID`)
- Do not modify state directly. Use `setState`
    - The only place where you can assign `this.state` is the constructor

- Returning `null` prevents component from rendering
- Returning `null` from a component’s `render` method does not affect the firing of the component’s lifecycle methods

- A "key" is a special **string attribute** you need to include when creating lists of elements
- Keys help React identify which items have changed, are added, or are removed
- Keys should be given to the elements inside the array to give the elements a ***stable identity***
    - If the key is the same as before React assumes that the DOM element **represents the same component**
        - > https://jsbin.com/wohima/edit?output
        - > https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318
    - If you choose not to assign an explicit key to list items then React will default to using indexes as keys
- Using indexes for keys is not recommended if the order of items may change. This can negatively impact performance and may cause issues with component state
- Keys used within arrays should be unique among their siblings
- Keys don't need to be globally unique
- Keys serve as a hint to React but they **don't get passed** to your components
- If the `map` body is too nested, it might be a good time to extract a component

- In HTML, form elements such as `<input>`, `<textarea>`, and `<select>` typically maintain their own state and update it based on user input
- In React, mutable state is typically kept in their state property of components, and only updated with `setState` (controlled component)
- In a controlled component, form data is handled by a React component
- **Specifying** the `value` prop on a controlled component **prevents** the user from changing the input unless you desire so
    - If you’ve specified a value but the input is still editable, you may have accidentally set value to `undefined` or `null`
- Fully-fledged solutions
    - [Formik](https://jaredpalmer.com/formik/): validation, keeping track of the visited fields, handling form submission (built on the same principles of controlled components and managing state )
- In an uncontrolled component, form data is handled by the DOM itself
    - Use a `ref` to get form values from the DOM

- Lift the shared state up to the closest common ancestor
- When you see something wrong in the UI, you can use React Developer Tools to inspect the props and move up the tree until you find the component responsible for updating the state
![](https://reactjs.org/react-devtools-state-ef94afc3447d75cdc245c77efb0d63be.gif)

- Single responsibility principle: ideally, a component should only do one thing
- Thinking in React
    1. Break The UI Into A Component Hierarchy
    2. Build A Static Version in React
        - Just use props. Don't use state at all. State is reversed for interactivity
        - In simpler examples, it’s usually easier to go top-down, and on larger projects, it’s easier to go bottom-up and write tests as you build
        - At the end of this step, the components will only have `render()` methods
    3. Identify The Minimal (but complete) Representation Of UI State
        - Think of the minimal set of mutable state that your app needs
            1. Is it passed in from a parent via props? If so, it probably isn’t state
            2. Does it remain unchanged over time? If so, it probably isn’t state
            3. Can you compute it based on any other state or props in your component? If so, it isn’t state
        - **Compute** everything else you need on-demand
    4. * Identify Where Your State Should Live
    5. Add Inverse Data Flow
        - The components **deep** in the hierarchy need to update the state of components top in the hierarchy

- React is one-way data flow (one-way binding)