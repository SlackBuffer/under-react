- https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce
- The term "render prop" refers to a technique for sharing code between React components using a **prop** whose value is a **function**
- A component with a `render` prop takes a function that **returns a React element** and **calls it instead of implementing its own render logic**

    ```jsx
    <DataProvider render={data => (
        <h1>Hello {data.target}</h1>
    )}>
    ```

# Use render props for cross-cutting concerns
- It's not always obvious how to share the **state** or **behavior** that one component encapsulates to other components that need that same state
- Track the cursor position example

    ```jsx
    class MouseTracker extends React.Component {
        constructor(props) {
            super(props)
            this.handleMouseMOve = this.handleMouseMove.bind(this)
            this.state = { x: 0, y: 0 }
        }
        handleMouseMove(e) {
            this.setState({
                x: e.clientX,
                y: e.clientY
            })
        }
        render() {
            return (
                <div onMouseMove={this.handleMouseMove}>
                    <h1>Move the mouse around!</h1>
                    <p>The current mouse position is ({this.state.x}, {this.state.y})</p>
                </div>
            )
        }
    }
    ```

- Use a `<Mouse>` component that encapsulates the behavior

    ```jsx
    class Mouse extends React.Component {
        constructor(props) {
            super(props)
            this.handleMouseMOve = this.handleMouseMove.bind(this)
            this.state = { x: 0, y: 0 }
        }
        handleMouseMove(e) {
            this.setState({
                x: e.clientX,
                y: e.clientY
            })
        }
        render() {
            return (
                <div onMouseMove={this.handleMouseMove}>
                    <p>The current mouse position is ({this.state.x}, {this.state.y})</p>
                </div>
            )
        }
    }

    class MouseTracker extends React.Component {
        render() {
            return (
                <div>
                    <h1>Move the mouse around!</h1>
                    <Mouse />
                </div>
            )
        }
    }
    ```

    - It's not truly reusable
    - A `<Cat>` component renders the image of a cat chasing the mouse around the screen. We might use a `<Cat mouse={{ x, y }}>` prop to tell the component the coordinates of the mouse so it knows where to position the image on the screen

        ```jsx
        class Cat extends React.Component {
            render() {
                const mouse = this.props.mouse
                return (
                    <img src="/cat.jpg" style={{ position: "absolute", left: mouse.x, top: mouse.y }} />
                )
            }
        }

        class MouseWithCat extends React.Component {
            constructor(props) {
                super(props)
                this.handleMouseMOve = this.handleMouseMove.bind(this)
                this.state = { x: 0, y: 0 }
            }
            handleMouseMove(e) {
                this.setState({
                    x: e.clientX,
                    y: e.clientY
                })
            }
            render() {
                return (
                    <div onMouseMove={this.handleMouseMove}>
                        <Cat mouse={this.state} />
                    </div>
                )
            }
        }

        class MouseTracker extends React.Component {
            render() {
                return (
                    <div>
                        <h1>Move the mouse around!</h1>
                        <Mouse />
                    </div>
                )
            }
        }
        ```

        - Every time we want the mouse position for a different use case, we have to create a new component (i.e. essentially another `<MouseWithCat>`) that renders something specifically for that use case
- Instead of **hard-coding** a `<Cat>` **inside** a `<Mouse>` component, and effectively changing its rendered output, we can provide `<Mouse>` with a function props that it uses to dynamically determine what to render - a render prop

    ```jsx
    class Cat extends React.Component {
        render() {
            const mouse = this.props.mouse
            return (
                <img src="/cat.jpg" style={{ position: "absolute", left: mouse.x, top: mouse.y }} />
            )
        }
    }
    class Mouse extends React.Component {
        constructor(props) {
            super(props)
            this.handleMouseMOve = this.handleMouseMove.bind(this)
            this.state = { x: 0, y: 0 }
        }
        handleMouseMove(e) {
            this.setState({
                x: e.clientX,
                y: e.clientY
            })
        }
        render() {
            return (
                <div onMouseMove={this.handleMouseMove}>
                    {/* 2. call the function with required props */}
                    {this.props.render(this.state)}
                </div>
            )
        }
    }
    class MouseTracker extends React.Component {
        render() {
            return (
                <div style={{ height: '100vh', width: '100vw' }}>
                    <h1>Move the mouse around!</h1>
                    {/* 1. pass a function */}
                    <Mouse render={mouse => (
                        <Cat mouse={mouse} />
                    )cod}>
                </div>
            )
        }
    }
    ```

    - Instead of effectively cloning the `<Mouse>` component and hard-coding something else in its `render` method to solve for a specific use case, a `render` prop that `<Mouse>` can **call** to dynamically determine what to render is provided
- To get the cursor track behavior, render a `<Mouse>` (behavior provider) with a prop that tells it what to render with the current (x, y) of the cursor
- You can ***implement most HOCs*** using **a regular component with a render prop**

    ```jsx
    class Cat extends React.Component {
        render() {
            const mouse = this.props.mouse
            return (
                <img src="/cat.jpg" style={{ position: "absolute", left: mouse.x, top: mouse.y }} />
            )
        }
    }
    class Mouse extends React.Component {
        constructor(props) {
            super(props)
            this.handleMouseMove = this.handleMouseMove.bind(this)
            this.state = { x: 0, y: 0 }
        }
        handleMouseMove(e) {
            this.setState({
                x: e.clientX,
                y: e.clientY
            })
        }
        render() {
            return (
                <div onMouseMove={this.handleMouseMove} style={{ height: '100vh', width: '100vw' }}>
                    {/* 2. call the function with required props */}
                    {this.props.render(this.state)}
                </div>
            )
        }
    }
    function withMouse(Component) {
        return class extends React.Component {
            render() {
                return (
                    <Mouse render={mouse => (
                        <Component {...this.props} mouse={mouse} />
                    )} />
                )
            }
        }
    }
    // embed Mouse behavior inside hoc withMouse1
    // extract it out is also fine
    function withMouse1(Component) {
        return class extends React.Component {
            constructor(props) {
                super(props)
                this.handleMouseMOve = this.handleMouseMove.bind(this)
                this.state = { x: 0, y: 0 }
            }
            handleMouseMove(e) {
                this.setState({
                    x: e.clientX,
                    y: e.clientY
                })
            }
            render() {
                return (
                    <div onMouseMove={this.handleMouseMove}>
                        <Component {...this.props} mouse={this.state} />
                    </div>
                )
            }
        }
    }
    const Mouse = withMouse1(Cat)
    class MouseTracker extends React.Component {
        render() {
            return (
                <div>
                    <h1>Move the mouse around!</h1>
                    <Mouse />
                </div>
            )
        }
    }
    ```

# Using props other than render
- The name of the props doesn't have to "render"
- Any prop that's a function that a component uses to know what to render is technically a "render prop"

    ```jsx
    <Mouse children={mouse => (
        <p>The mouse position is {mouse.x}, {mouse.y}</p>
    )} />

    <Mouse>
        {mouse => (
            <p>The mouse position is {mouse.x}, {mouse.y}</p>
        )} 
    />
    class Mouse extends React.Component {
        constructor(props) {
            super(props)
            this.handleMouseMOve = this.handleMouseMove.bind(this)
            this.state = { x: 0, y: 0 }
        }
        handleMouseMove(e) {
            this.setState({
                x: e.clientX,
                y: e.clientY
            })
        }
        render() {
            return (
                <div onMouseMove={this.handleMouseMove}>
                    {/* children is a function (unusual, better state this fact explicitly) */}
                    {this.props.children(this.state)}
                </div>
            )
        }
    }
    ```

    - The `children` prop doesn't have to be named in the list of "attributes" in the JSX element, it can be directly put inside the element
# Caveats
# Be careful when using render props with [ ] `React.PureComponent`