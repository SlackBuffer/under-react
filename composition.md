# [Composition](https://reactjs.org/docs/composition-vs-inheritance.html#containment)
- React has a powerful composition model, and using composition is recommended instead of inherence to **reuse code between components**
## Containment
- Some components **don't know their children ahead of time**
    - Especially common for components like `SideBar` or `Dialog` that represent **generic "boxes"**
- Use of the special `children` prop to pass children elements directly into such components' output
- Sometimes you might need multiple "holes" in a component. In such cases, you may come up with custom convention

    ```jsx
    function SplitPane(props) {
        return (
            <div className="SplitPane">
                <div className="SplitPane-left">
                    {props.left}
                </div>
                <div className="SplitPane-right">
                    {props.right}
                </div>
            </div>
        )
    }

    function App() {
        return (
            <SplitPane
            left={<Contacts />}
            right={<Chat />} />
        )
    }
    ```

## Specialization
- Sometimes we think about components as being **"special cases"** of other components
- In React this is also achieved by composition - a more specific component renders a more generic oen and configures with props

    ```jsx
    function Dialog(props) {
        return (
            <FancyBorder color="blue">
                <h1>{props.title}</p>
                <p>{props.message}</p>
            </FancyBorder>
        )
    }
    function WelcomeDialog() {
        return <Dialog title="welcome" message="hahaha" />
    }
    ```

    - Composition works equally well for components defined as classes
# Inheritance
- [ ] how to write - **`extends`**
- No use case according to Facebook
- Props and composition give you all the flexibility you need to customize a component's look and behavior in an explicit and safe way
- If you want to reuse non-UI functionality between components, we we suggest extracting it into a separate JavaScript module