- `componentDidMount` runs **after** the component output has been rendered to the **DOM**
- ***For components defined as classes***
- http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/
# Mounting
- These method are called in the following order when an **instance** of a component is being created and inserted into the DOM
    1. `constructor()`
    2. `static getDerivedStateFromProps()`
    3. `render()`
    4. `componentDidMount()`
# Updating
- An updating can be caused by changes to **props** or **state**
- These methods are called in the following order when a component is being **re-rendered**
    1. `static getDerivedStateFromProps()`
    2. `shouldComponentUpdate()`
    3. `render()`
    4. `getSnapshotBeforeUpdate()`
    5. `componentDidUpdate()`
# Unmouting
- This method is called when a component is being removed from the DOM
    - `componentWillUnmount`
# Error handling
- These methods are called when there's an error during **rendering**, in a **lifecycle method**, or in the **constructor of any child component**
# Reference
## `render()`
- The only required method in a class component
- When called, it should examine `this.props` and `this.state` and return one of the following types
    1. React elements
        - Typically created by JSX
        - `<div />` and `<MyComponent />` are React elements that instruct React to render a DOM node, or another user-defined component, respectively
    2. Arrays and fragments
    3. Portals
        - Let you render children into a different DOM subtree
    4. String and numbers
        - Rendered as text nodes in the DOM
    5. Booleans or null
        - Render nothing 
        - Mostly exists to support `return test && <Child />` pattern, where `test` is boolean
- `render()` should be **pure**, meaning that it does not modify component state, it returns the same result each time it's invoked, and it does not directly interact with the browser
    - > If you need to interact with the browser, perform your work in `componentDidMount` or other lifecycle methods
- `render()` will not be invoked if `shouldComponentUpdate()` returns `false`
## `constructor(props)`
- If you don't **initialize state** and you don't **bind methods**, you don't need to implement a constructor for the React component
    - Typically, in React constructors are only used for initializing local state by **assigning an object** to `this.state` and binding event handler methods **to an instance**
- When implementing the constructor for a `React.Component` subclass, call `super(props)` before any other statement. Otherwise, `this.props` will be undefined in the constructor, which can lead to bugs
- Shouldn't call `setState` in the `constructor()`. Instead, assign the initial state to `this.state` directly in the constructor
    - Constructor is the only place where you should assign `this.state` directly. In all other methods, use `this.setState()` instead
- Avoiding introducing any side-effects or **subscriptions** in the constructor. For those use cases, use `componentDidMount()`
- **Avoid copying props into state**

    ```js
    constructor(props) {
        super(props)
        this.state = { color: props.color }
    }
    ```

    - It's both unnecessary (use `this.props.color` directly instead), and create bugs (updates to the `color` prop won't be reflected in the state)
    - Only use this pattern if you intentionally want to ignore prop updates
        - In that case, it makes sense to rename the prop to be called `initialColor` or `defaultColor`. You can then force a component to reset its internal state by [changing its key](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-uncontrolled-component-with-a-key) when necessary
## `componentDidMount()`
- `componentDidMount()` is invoked immediately after a component is mounted (inserted into DOM tree)
    - Initialization that requires DOM nodes should go there
    - If you need to load data from a remote endpoint, this is a good place to instantiate the network request
    - It's a good place to set up any subscriptions and remember to unsubscribe in `componentWillUnmount()`
- Calling `setState()` immediately in `componentDidMount()` will **trigger an extra rendering**, but it will **happen before the browser updates the screen**
    - This guarantees that even though `render()` will be called twice, the user won't see the intermediate state
    - Use this pattern with caution because it often causes performance issues
        - In most cases, you should be able to assign the initial state in the `constructor()` instead
    - It can be necessary for cases like modals and tooltips when you need to **measure a DOM node** before rendering something that **depends on its size or position**
## `componentDidUpdate(prevProps, prevState, snapshot)`
- `componentDidUpdate()` is invoked immediately after updating occurs
- Not called for the initial render
- Use this as an opportunity to operate on the DOM when the component ahs been updated
- This is also a good place to do network requests **as long as you compare** the current props to previous props

    ```js
    componentDidUpdate(prevProps) {
        if (this.props.userID !== prevProps.userID) {
            this.fetchData(this.props.userID)
        }
    }
    ```

- You may call `setState()` immediately in `componentDidUpdate()` but it must **be wrapped in a condition**, or you'll get an infinite loop
    - It would also cause an extra re-rendering, which, while not visible to the user, can affect the component performance
- If your component implements the `getSnapShotBeforeUpdate()` lifecycle, the value it returns will be passed as a third snapshot parameter to `componentDidUpdate()`. Otherwise this parameter will be undefined
- `componentWillUpdate()` will not be invoked if `shouldComponentUpdate` returns false
## `componentWillUnmount()`
- `componentWillUnmount()` is invoked immediately before a component is unmounted and destroyed
- Perform any necessary cleanup in this method, such as invalidating timers, canceling network requests, or cleaning up any subscriptions created in `componentDidMount()`
- Shouldn't call `setState()` in `componentWillUnmount` because the component will never be re-rendered
- Once a component instance is unmounted, it will never be mounted again
## `shouldComponentUpdate(nextProps, nextState)`
- Use `shouldComponentUpdate()` to let React know if a component's output is not affected by the current in state or props
- The default behavior (for class component without implementing `shouldComponentUpdate()`) is to re-render on every state change
    - In the vast majority of cases you should rely on the default behavior
    - [x] What about prop change?
- `shouldComponentUpdate()` is invoked before rendering **when** new props or state are being received
    - [x] Define new props or state
- Defaults to `true` (意思是不手动实现的话会自动根据状态变化来做更新)
- It's not called for the **initial render** or when **`forceUpdate()`** is used
- This method only exists as a performance optimization
    - Don't rely on it to prevent a rendering, as this can lead to bugs
    - Consider using the built-in `PureComponent` instead of writing `shouldComponentUpdate()` by hand
        - `PureComponent()` performs a shallow comparison of props and state, and reduces the chance that you'll skip a necessary update
- If you're confident you want to write it by hand, you may compare `this.props` with `nextProps` and `this.state` with `nextState` and return `false` to tell React the update can be skipped
- Returning `false` ***doesn't* prevent child components from re-rendering when their state changes**
- Doing deep equality checks or using `JSON.stringify()` in `shouldComponentUpdate()` is not recommended as it's very inefficient and will harm performance
- **Currently**, if `shouldComponentUpdate()` returns `false`, then `UNSAFE_componentWillUpdate()`, `render()`, and `componentDidUpdate()` wil not be invoked
    - In the future React may treat `shouldComponentUpdate()` as a hint rather than a strict directive, and returning `false` may still result in a re-rendering of the component


- `state` 更新但 `render()` 方法里并未用到更新的字段，`render` 方法仍会被调用，但组件的输出并未发生变化
- 父组件的 `render()` 方法被调用，子组件的 `render()` 必被调用，不论父组件传给子组件的 `props` 是否改变
- `connect()` 进来的 `props` 发生变化，但 `render()` 并未用到发生变化的 `props` 字段，被包裹组件的 `render()` 也会被调用
    - 被包裹组件能接收到变化的 `props` （前提是要将对应的 model `connect` 进去，不然未注入的话对应的 props 字段都不在 props 里，props 的更新无从谈起），表明 HOC 组件的 `render()` 必被调用；父组件 `render()` 被调用，子组件的 `render()` 必被调用