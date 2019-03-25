- React events are named using camelCase, rather than lowercase 
- With JSX you pass a function as the event handler, rather than a string
- Cannot return false to prevent default behavior in React. Must call `e.preventDefault` explicitly
    - `e` is a synthetic event
    - React defines these synthetic events according to the [W3C spec](https://www.w3.org/TR/DOM-Level-3-Events/), so you don’t need to worry about cross-browser compatibility
- `this` in JSX

    ```jsx
    // 1. es6 class
    constructor(props) {
        super(props)
        // class methods are not bound by default
        this.handleClick = this.handleClick.bind(this)
    }

    // 2. public class fields syntax
    handleClick = () => {}

    // 3. arrow function in the callback
    <button onClick={e => this.handleClick(e)} />
    ```

    - In 3, a different callback is created each time the component renders
        - If this callback is passed as a prop to lower components, those components might do an extra re-rendering
        - Try to avoid this pattern
- Passing arguments to event handlers

    ```jsx
    <button onClick={e => this.deleteRow(id, e)}>Delete Row</button>
    <button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
    ````