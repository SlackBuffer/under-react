# Before using context
- If you only want to passing some props through many levels, [component composition](https://reactjs.org/docs/context.html#before-you-use-context) is often a simpler solution than context
- It means **Passing components down**
    - This patterns is sufficient for many cases when you need to **decouple a child from its immediate parents**
    - [x] Apply render props if the child needs to communicate with the parent before rendering
- Applying context makes component reuse more difficult
## Scenarios
- A `Page` component that passes a `user` and `avatarSize` prop several levels down so that deeply nested `Link` and `Avatar` components can read it

    ```jsx
    <Page user={user} avatarSize={avatarSize} />
    // ... which renders ...
    <PageLayout user={user} avatarSize={avatarSize} />
    // ... which renders ...
    <NavigationBar user={user} avatarSize={avatarSize} />
    // ... which renders ...
    <Link href={user.permalink}>
        <Avatar user={user} size={avatarSize} />
    </Link>
    ```

    - Composition solution - passing down the `Avatar` component (`userLink`)

        ```jsx
        function Page(props) {
            const user = props.user
            const userLink = (
                <Link href={user.permalink}>
                    <Avatar user={user} size={props.avatarSize} />
                </Link>
            )
            return <PageLayout userLink={userLink} />
        }

        // Now, we have:
        <Page user={user} />
        // ... which renders ...
        <PageLayout userLink={...} />
        // ... which renders ...
        <NavigationBar userLink={...} />
        // ... which renders ...
        {props.userLink}
        ```

        - With this change, only the top-most `Page` component needs to know about the `Link` and `Avatar` components' use of `user` and `avatarSize` (**inversion of control**)
        - The inversion of control can make code cleaner by **reducing the amount** of props you need to pass through and **giving more control to the root component**
        - This **isn't the right choice in every case**: moving more complexity higher in the tree makes higher-level components more complex and force the lower-level components to be more flexible than you may want
# [Context](https://reactjs.org/docs/context.html)
- Sometimes the same data needs to be accessible by many components at different nesting levels in the tree
- Context provides a way to pass ("broadcast") data and **changes** to it required by many components through the component tree without having to pass props manually at every level
- Context is designed to share data that can be considered **"global"** for a tree of React components (such as the current authenticated user, theme, or preferred language)

    ```jsx
    // 1. create a context for the current theme (with 'light' as the default)
    const ThemeContext = React.createContext('light')

    class App extends React.Component {
        render() {
            // 2. use a Provider to pass the theme prop with value "dark" to the tree below
            // any component however deep can read this value
            return (
                <ThemeContext.Provider value="dark">
                    <Toolbar />
                </ThemeContext.Provider>
            )
        }
    }

    // A component in the middle doesn't have to pass the theme props explicitly
    function Toolbar(props) {
        return <div><ThemeButton /></div>
    }

    // 3. assign a contextType to read the current theme context
    // React will find the closest theme Provider above and use its value
    class ThemeButton extends React.Component {
        static contextType = ThemeContext
        render() {
            return <Button theme={this.context} />
        }
    }
    ```

    - **Common examples** where using context might be simpler than the alternatives include managing the current locale, theme, or a data cache
## `React.createContext`
- `const MyContext = React.createContext(defaultValue)`
    - Creates a **Context object**. When React renders a component that subscribes to this Context, it will read the current context value from the **closest matching `Provider` above it** in the tree
    - The `defaultValue` argument is **only** used when a component does not have a matching Provider (`<MyContext.Provider></MyContext.Provider>`) above it in the tree
        - This can be helpful for **testing components in isolation without wrapping them**
    - Passing `undefined` as a Provider value does not cause consuming components to use `defaultValue`
## `Context.Provider`
- `<MyContext.Provider value={/* some value */}>`
    - Accept a `value` prop to be passed to consuming components that are **descendants** of this Provider
    - Every Context object comes with a **Provider React Component** that allows consuming components to **subscribe to context changes**
    - One Provider can be connected to many consumers
    - Providers can be **nested** to override values deeper within the tree
    - All consumers that are descendants of a Provider will re-render **whenever the Provider's `value` prop changes**
        - The propagation from Provider to its descendant consumer is **not subject to the `shouldComponentUpdate` method**, so the consumer is updated even when an ancestor component bails out of the update
        - Changes are determined by comparing the new and old values using the same algorithm as **`Object.is`**
            - [Issues](#Caveats) when passing objects as `value`
## `Class.contextType` (consumer for class components)
- `MyClass.contextType = MyContext`
    - The `contextType` property on a ***class*** can be assigned a Context object created by `React.createContext()`
    - This lets you consume the nearest current value of that Context type using **`this.context`**
    - `this.context` can be referenced in **any of the lifecycle methods including the render function**

    ```jsx
    class MyClass extends React.Component {
        componentDidMount() { let value = this.context }
        componentDidUpdate() { let value = this.context }
        componentWillUnmount() { let value = this.context }
        render() { let value = this.context }
    }
    MyClass.ContextType = MyContext
    ```

    - You can only **subscribe to a single context** using this API
        - [Consuming multiple contexts](#Consuming-multiple-contexts)
- `static contextType = MyContext ` (public class fields syntax)

    ```jsx
    class MyClass extends React.Component {
        static contextType = MyContext
        render() {
            let value = this.context
        }
    }
    ```

## `Context.Consumer` (consumer for function components)

```jsx
<MyContext.Consumer>
    {value => /* render sth based on the context value */}
</MyContext.Consumer>
```

- A React component that subscribes to context changes
    - Allows you to subscribe to a context within a ***function component***
- Requires a [function as a child](https://reactjs.org/docs/render-props.html)
    - The function receives the current context value and returns a React node
    - The `value` argument passed to the function will be equal to the `value` prop of **the closest Provider for this context** above
    - If there is no Provider for this context above, the `value` argument will be equal to the `defaultValue` that was passed to `createContext()`
## Dynamic context

```jsx
// toggleTheme invokes setState
<ThemeContext.Provider value={this.state.theme}>
    <Toolbar changeTheme={this.toggleTheme} />
</ThemeContext.Provider>
```

## Updating context from a nested component
- Pass a function (updating context using dynamic context routine) down through the context to allow consumers to update the context

## Consuming multiple contexts
- To keep context re-rendering fast, React needs to **make each context consumer a separate node** in the tree

    ```jsx
    const ThemeContext = React.createContext('light')
    const UserContext = React.createContext({ name: 'Guest' })
    class App extends React.Component {
        render() {
            const {signedInUser, theme} = this.props
            return (
                <ThemeContext.Provider value={theme}>
                    <UserContext.Provider value={signedInUser}>
                        <Layout />
                    </UserContext.Provider>
                </ThemeContext.Provider>
            )
        }
    }
    function Layout() {
        return (
            <div>
                <Sidebar />
                <Content />
            </div>
        )
    }
    // A component may consume multiple contexts
    function Content() {
        return (
            <ThemeContext.Consumer>
                {theme => (
                    <UserContext.Consumer>
                    {user => (
                        <ProfilePage user={user} theme={theme} />
                    )}
                    </UserContext.Consumer>
                )}
            </ThemeContext.Consumer>
        )
    }
    ```

    - If two of more context values are often used together, you might want ot consider creating your own render prop component that provides both
# Caveats
- Context use **reference identity** to determine when to re-render
- The code below will re-render all consumers every time the Provider re-renders because **a new object** is always created for `value`

    ```jsx
    class App extends React.Component {
        render() {
            return (
                <Provider value={{something: 'something'}}>
                    <Toolbar />
                </Provider>
            )
        }
    }
    ```

    - To get around this, lift the value into the parent's state

        ```jsx
        class App extends React.Component {
            constructor(props) {
                super(props)
                this.state = {
                    value: {something: 'something'}
                }
            }
            render() {
                return (
                    <Provider value={this.state.value}>
                        <Toolbar />
                    </Provider>
                )
            }
        }
        ```