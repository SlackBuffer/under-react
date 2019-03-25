- `setState` ***enqueues*** changes to the component state and tells React that this **component and its children** need to be re-rendered with the updated state
- Think of `setState` as a ***request*** rather than an immediate command to update the component
    - For better perceived performance, React may **delay** it, and then update several components in a single pass
    - React does not guarantee that the state changes are applied immediately
- `setState` does not immediately update the component. It may **batch** or **defer** the update until later
- This makes reading `this.state` right after calling `setState` a potential pitfall
- Use `componentDidUpdate` or a `setState` callback (`setState(updater, callback)`), either of which are **guaranteed to fire after the (previous, first in queue) update has been applied**
- `setState` will always lead to a re-render unless `shouldComponentUpdate` returns `false`
    - If mutable objects are being used and conditional rendering logic cannot be implemented in `shouldComponentUpdate`, calling `setState` only when the new state differs from the previous state will avoid unnecessary re-renders
- `setState((state, props) => stateChange[, callback])`
    - Both `state` and `props` received by the updater function are **guaranteed to be up-to-date**
    - `state` should not be directly mutated
    - Changes should be represented by building a new object based on the input from `state` and `props`
    - The output of the updater is **shallowly merged** with `state`
    - > The second parameter to `setState` is an optional callback function that will be executed once `setState` is **completed** and the component is **re-rendered**. Generally we recommend using `componentDidUpdate` for such logic instead
- `setState(object)`
    - This form is also asynchronous. Multiple call during the same circle may be batched together
    - If you attempt to increment to increment an item quantity more than once in the same circle, that will result in the equivalent of

        ```js
        // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign
        Object.assign(
            previousState,
            {quantity: state.quantity + 1},
            {quantity: state.quantity + 1}
            // ...
        )
        ```

        - Subsequent calls will override value from the previous calls in the same circle, so the quantity will only be incremented once
    - If the next state depends on the current state, we recommend using the updater function form
- https://stackoverflow.com/questions/48563650/does-react-keep-the-order-for-state-updates/48610973#48610973
- [ ] https://github.com/facebook/react/issues/11527#issuecomment-360199710