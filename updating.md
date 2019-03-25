- React elements are immutable. **Once you create an element, you can't change its children or attributes**
    - An element is like a single **frame of a movie**: it **represents the UI at a certain point in time**
- React DOM compares the **element and its children** to the previous one, and only applies the DOM updates **necessary** to bring the DOM to the desire state
- Thinking about how the UI should look at any given moment rather than how to change it over time eliminates a whole class of bugs (declarative)
- The `render` method (class component) will be called each time an update happens
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