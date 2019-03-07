- A higher-order component (HOC) is a technique in React for ***reusing component logic***
- An HOC is a **function** that takes a component and returns a new component
- HOCs are a pattern that emerges from React's compositional nature
# Use HOCs For Cross-Cutting Concerns
- Define the logic in a single place and share it across many components

    ```jsx
    const CommentListWithSubscription = withSubscription(
        CommentList,
        DataSource => DataSource.getComments()
    )
    const BlogPostWithSubscription = withSubscription(
    BlogPost,
    (DataSource, props) => DataSource.getBlogPost(props.id)
    )

    function withSubscription(WrappedComponent, selectData)
        return class extends React.Component {
            constructor(props) {
                super(props)
                this.handleChange = this.handleChange.bind(this)
                this.state = {
                    data: selectData(DataSource, props)
                }
            }
            componentDidMount() {
                DataSource.addChanegListener(this.handleChange)
            }
            componentWillUnmount() {
                DataSource.removeChangeListener(this.handleChange)
            }
            // DataSource here is some global data source
            // props from HOC's parent component (e.g., specifies `id`)
            handleChange() {
                this.setState({
                    data: selectData(DataSource, this.props)
                })
            }
            render() {
                return <WrappedComponent data={this.state.data} {...this.props} />
            }
        }
    }
    ```

- A HOC doesn't modify the input component, nor does it use inheritance to copy its behavior
- A HOC composes the original component by wrapping it in a container component
- A HOC is a pure function with zero side-effects
- The HOC isn't concerned with how or why the data is used, and the wrapped component isn't concerned with where the data came from
- `withSubscription` is a normal function, you can add as many or as few arguments as you like. HOC has **full control** over how the component is defined
    - You may want to make the name of the data prop configurable, to further isolate the HOC from the wrapped component
    - You could accept an argument that configures `shouldComponentUpdate`, or one that configures the data source
- Like components, the contract between `withSubscription` and the wrapped component is entirely **props-based**. This makes it easy to swap one HOC for a different one, as long as they provide the same props to the wrapped component
    - This may be useful if you change data-fetching libraries, for example
# Don't Mutate the Original Component. Use Composition
- Resist the temptation to modify the **wrapped component's prototype** (or otherwise mutate it) inside a HOC

    ```jsx
    function logProps(InputComponent) {
        InputComponent.prototype.componentWillReceiveProps = function(nextProps) {
            console.log('Current props: ', this.props)
            console.log('Next props: ', nextProps)
        }
        // the fact that we're returning the original input is a hint that it has been mutated
        return InputComponent
    }
    // EnhancedComponent will log whenever props are received
    const EnhancedComponent = logProps(InputComponent)
    ```

    1. The input component cannot be reused separately from the enhanced component
    2. If you apply another HOC to `EnhancedComponent` that **also mutates** `componentWillReceiveProps, the first HOC's functionality will be overridden
    3. This HOC won't work with function components as they don't have lifecycle methods
- Mutating HOCs are a leaky abstraction—the consumer must know how they're implemented in order to avoid conflicts with other HOCs
- HOCs should use composition, by wrapping the input component in a container component

    ```jsx
    function logProps(WrapperComponent) {
        return class extends React.Component {
            componentWillReceiveProps(nextProps) {
                console.log('Current props: ', this.props)
                console.log('Next props: ', nextProps)
            }
        }
        return <WrappedComponent {...this.props} />
    }
    ```

    - Same functionality while avoiding the potential for clashes
    - Works for both class and function components
    - Pure function. Composable with other HOCs, or even with itself
- Similarities between HOCs and a pattern called **container components**
    - Container components are part of a strategy of **separating responsibility** between high-level and low-level concerns
        - Containers manage things like subscriptions and state, and pass props to components that handle things like rendering UI
    - HOCs use containers as part of their implementation
        - Think of HOCs as **parameterized** container component definitions
# Convention: pass unrelated props through to the wrapped component
- Filter props passed through

    ```jsx
    render() {
        // Filter out extra props that are specific to this HOC and shouldn't be passed through
        const { extraProp, ...passThroughProps } = this.props
        // Inject props into the wrapped component. These are usually state values or instance methods
        const injectedProp = someStateOrInstanceMethod
        // Pass props to wrapped component
        return (
            <WrappedComponent
                injectedProp={injectedProp}
                {...passThroughProps}
            />
        )
    }
    ```

# Convention: maximizing composability
- HOC example

    ```js
    const NavbarWithRouter = withRouter(Navbar)
    // a config object is used to specify a component's data dependencies
    const CommentWithRelay = Relay.createContainer(Comment, config)

    const ConnectedComment = connect(commentSelector, commentActions)(CommentList)

    // break part
    // connect is a function that returns another function
    const enhance = connect(commentSelector, commentActions)
    // the returned enhance is a HOC, which return a component that's connected to the Redux store
    const ConnectedComment = enhance(CommentList)
    // connect is a higher-order function that returns a higher-order component
    ```

- Single-argument HOCs like the one returned by the `connect` function have the signature `Component => Component`. Functions whose **output type** is the **same as** its **input type** are really easy to **compose together**


    ```jsx
    // Instead of doing this
    const EnhancedComponent = withRouter(connect(commentSelector)(WrappedComponent))

    // you can use a function composition utility
    // compose(f, g, h) is the same as (...args) => f(g(h(...args)))
    const enhance = compose(
        // These are both single-argument HOCs
        withRouter,
        connect(commentSelector)
    )
    const EnhancedComponent = enhance(WrappedComponent)
    ```

    - This same property also allows `connect` and other **enhancer-style HOCs** to be used as [ ] decorators, an experimental JavaScript proposal
# Convention: wrap the display name for easy debugging
- The container components created by HOCs show up in the React Developer Tools like any other component. To ease debugging, choose a display name that communicates that it's the result of a HOC
- The most common technique is to wrap the display name of the wrapped component 

    ```js
    function withSubscription(WrappedComponent) {
        class WithSubscription extends React.Component {/* ... */}
        WithSubscription.displayName = `WithSubscription(${getDisplayName(WrappedComponent)})`
        return WithSubscription
    }
    function getDisplayName(WrappedComponent) {
        return WrappedComponent.displayName || WrappedComponent.name || 'Component'
    }
    ```

    - If your higher-order component is named `withSubscription`, and the wrapped component's display name is `CommentList`, use the display name `WithSubscription(CommentList)`
# Caveats
## Don't use HOCs inside the render method
- React's diffing algorithm ([ ] *reconciliation*) uses **component identity** to determine whether it should update the existing subtree or throw it away and mount a new one
    - If the component returned from `render` is identical (`===`) to the component from the previous render, React recursively updates the subtree by diffing it with the new one
    - If they're not equal, the previous subtree is unmounted completely
- Can't apply a HOC to a component within the render method of a component

    ```jsx
    render() {
        // A new version of EnhancedComponent is created on every render
        // EnhancedComponent1 !== EnhancedComponent2
        const EnhancedComponent = enhance(MyComponent)
        // That causes the entire subtree to unmount/remount each time!
        return <EnhancedComponent />
    }
    ```

    - The problem isn't just about performance
    - Remounting a component causes the **state of that component and all of its children to be lost**
    - **Instead**, apply HOCs **outside the component definition** so that the resulting component is created only once. Then its identity will be consistent across renders
- In those rare cases where you need to **apply a HOC dynamically**, you can also do it inside a **component's lifecycle methods or its constructor**
## Static methods must be copied over
- When you apply a HOC to a component, the original component is wrapped with a container component. The new component does not have any of the static methods of the original component

    ```jsx
    // Define a static method
    WrappedComponent.staticMethod = function() {/*...*/}
    // Now apply a HOC
    const EnhancedComponent = enhance(WrappedComponent)
    // The enhanced component has no static method
    typeof EnhancedComponent.staticMethod === 'undefined' // true
    ```

- Copy the methods onto the container before returning it

    ```jsx
    function enhance(WrappedComponent) {
        class Enhance extends React.Component {/*...*/}
        // Must know exactly which method(s) to copy :(
        // You can use hoist-non-react-statics to automatically copy all non-React static methods
        Enhance.staticMethod = WrappedComponent.staticMethod
        return Enhance
    }
    ```

    - This requires you to know exactly which methods need to be copied
    - [hoist-non-react-statics](https://github.com/mridgway/hoist-non-react-statics) automatically copy all non-React static methods

        ```jsx
        import hoistNonReactStatic from 'hoist-non-react-statics'
        function enhance(WrappedComponent) {
            class Enhance extends React.Component {/*...*/}
            hoistNonReactStatic(Enhance, WrappedComponent)
            return Enhance
        }
        ```

- Another method is to export the static method separately from the component itself

    ```jsx
    // Instead of...
    MyComponent.someFunction = someFunction;
    export default MyComponent

    // ...export the method separately...
    export { someFunction }

    // ...and in the consuming module, import both
    import MyComponent, { someFunction } from './MyComponent.js'
    ```

## Refs aren't passed through
- While the convention for higher-order components is to pass through all props to the wrapped component, this does not work for refs
- `ref` is not really a prop - like `key`, it's handled specially be React
    - If you add a ref to an element whose component is the result of a HOC, the ref refers to an instance of the **outermost container component**, not the wrapped component
    - The solution is use the [ ] `React.forwardRef`
# `compose`
- `reduce`

    ```js
    // first round: accumulator is arr[0], currentValue is arr[1]
    // second round: accumulator is last round's return value, currentValue is arr[2]
    // ...
    arr.reduce(function(accumulator, currentValue) {})

    func compose(...funcs) {
        if (funcs.length === 0) { return comp => comp }
        if (funcs.lenght === 1) { return funcs[0] }
        return funcs.reduce((a, b) => (WrappedComp, ...restArgs) => a(b(WrappedComp, ...restArgs)))
    }

    const enhance0 = compose() = comp => comp
    const EnhancedComp = enhance0(Comp) = Comp

    const enhance1 = compose(hoc1) = hoc1
    const EnhancedComp1 = enhance1(Comp) = hoc1(Comp)
     
    const enhance2 = compose(hoc1, hoc2) = (WrappedComp, ...restArgs) => hoc1(hoc2(WrappedComp, ...restArgs))
    const EnhancedComp2 = enhance2(Comp) = hoc1(hoc2(Comp))
    
    const enhance3 = compose(hoc1, hoc2, hoc3) = (WrappedComp, ...restArgs) => hoc1(hoc2(hoc3(WrappedComp, ...restArgs)))
    const EnhancedComp3 = enhance3(Comp) = hoc1(hoc2(hoc3(Comp)))
    // Round 1
    // accumulator1: hoc1; currentValue1: hoc2
    // return value1: (WrappedComp, ...restArgs) => hoc1(hoc2(WrappedComp, ...restArgs))
    // Round 2
    // accumulator2:  (WrappedComp, ...restArgs) => hoc1(hoc2(WrappedComp, ...restArgs)); currentValue2: hoc3

    /* call accumulator2 with hoc3(WrappedComp, ...restArgs) as the single argument */
    // return value2: (WrappedComp, ...restArgs) => hoc1(hoc2(hoc3(WrappedComp, ...restArgs)))

    // 各层的 ...restArgs 都需传不同的内容时，compose 不适用
    ```

    - > https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce#How_reduce()_works