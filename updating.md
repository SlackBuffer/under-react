- React elements are immutable. **Once you create an element, you can't change its children or attributes**
    - An element is like a single **frame of a movie**: it **represents the UI at a certain point in time**
- React DOM compares the **element and its children** to the previous one, and only applies the DOM updates **necessary** to bring the DOM to the desire state
- Thinking about how the UI should look at any given moment rather than how to change it over time eliminates a whole class of bugs (declarative)
- The `render` method (class component) will be called each time an update happens
    - > https://stackoverflow.com/questions/48563650/does-react-keep-the-order-for-state-updates/48610973#48610973
    - Currently (React 16 and earlier), **only updates inside React event handlers** are batched by default (`unstable_batchedUpdates` forces batching outside of event handlers for rare cases when you need it)
    - No matter how many `setState()` calls in how many components you do inside a React event handler, they will produce only a single re-render at the end of the event. This is crucial for good performance in large applications because if `Child` and `Parent` each call `setState()` when handling a click event, you don't want to re-render the `Child` twice
- As long as a component is rendered into the same DOM node, only a single instance of that component will be used
- React may batch multiple `setState` calls into a single update for performance
- `this.props` and `this.state` may be updated asynchronously. Shouldn't rely on their values for calculating the next state

    ```js
    // wrong
    this.setState({
        counter: this.state.counter + this.props.increment
    })
    // correct
    this.setState((state, props) => ({
        counter: state.counter + props.increment
    }))
    ```

- State updates are merged

    ```jsx
    constructor(props) {
        super(props)
        this.state = {
            posts: [],
            comments: []
        }
    }
    componentDidMount() {
        fetchPosts().then(response => {
            this.setState({
                posts: response.posts
            })
        })
        fetchComments().then(response => {
            this.setState({
                comments: response.comments
            })
        })
    }
    ```

    - The merging is shallow. `this.setState({comments})` leaves `this.state.posts` intact, but completely replaces `this.state.comments`
> # [Why is `setState` asynchronous](https://github.com/facebook/react/issues/11527#issuecomment-360199710)
- First, I think we agree that delaying reconciliation in order to batch updates is beneficial
    - That is, we agree that `setState()` re-rendering synchronously would be inefficient in many cases, and it is better to batch updates if we know we’ll likely get several ones
    - For example, if we’re inside a browser `click` handler, and both `Child` and `Parent` call `setState`, we don’t want to re-render the `Child` twice, and instead prefer to mark them as dirty, and re-render them together before exiting the browser event
## Reasons of waiting for the end of reconciliation instead of writing `setState` updates immediately to `this.state`
### Guaranteeing internal consistency
- Even if `state` is updated synchronously, `props` are not
- You cannot know `props` until you re-render the parent component
- Right now the objects provided by React (`state`, `props`, `refs`) are internally consistent with each other
- This means that if you only use those objects, they are guaranteed to refer to a fully reconciled tree (even if it's an older version of that tree)
- In React, both `this.state` and `this.props` update only after the reconciliation and flushing
- the React model doesn’t always lead to the most concise code, but it is internally consistent and ensures lifting state up is safe
### Enabling Concurrent Updates
- React could assign different priorities to setState() calls depending on where they’re coming from: an event handler, a network response, an animation, etc
    - If you are typing a message, `setState()` calls in the `TextBox` component need to be flushed immediately
    - If you receive a new message while you’re typing, it is probably better to delay rendering of the new `MessageBubble` up to a certain threshold (e.g. a second) than to let the typing stutter due to blocking the thread
    - If we let certain updates have lower priority, we could split their rendering into small chunks of a few milliseconds so they wouldn't be noticeable to the user
- For example, consider the case where you’re navigating from one screen to another
    - Typically you’d show a spinner while the new screen is rendering
    - However, if the navigation is fast enough (within a second or so), flashing and immediately hiding a spinner causes a degraded user experience. Worse, if you have multiple levels of components with different async dependencies (data, code, images), you end up with a cascade of spinners that briefly flash one by one. This is both visually unpleasant and makes your app slower in practice because of all the DOM reflows. It is also the source of much boilerplate code
    - Wouldn’t it be nice if when you do a simple `setState()` that renders a different view, we could “start” rendering the updated view “in background”? Imagine that without any writing any coordination code yourself, you could choose to show a spinner if the update took more than a certain threshold (e.g. a second), and otherwise let React perform a seamless transition when the async dependencies of the whole new subtree are satisfied
    - Moreover, while we’re “waiting”, the “old screen” stays interactive (e.g. so you can choose a different item (screen) to transition to), and React enforces that if it takes too long, you have to show a spinner
- Note that this is only possible because `this.state` is not flushed immediately. If it were flushed immediately, we’d have no way to start rendering a “new version” of the view in background while the “old version” is still visible and interactive. Their independent state updates would clash