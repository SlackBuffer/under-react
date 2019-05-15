- Portals provide a first-class way to render children into a  DOM node that exists outside the DOM hierarchy of the parent component
    - `ReactDOM.createPortal(child, container)`
    - The first argument is any renderable React child
    - The second is a DOM element
- A typical use case for portals is when a parent component has an `overflow: hidden` or `z-index` style, but you need the child to visually break out of its container
- An event fired from inside a portal will propagate to ancestors in the containing React tree, even if those elements are not ancestors in the DOM tree

    ```html
    <html>
        <body>
            <div id="app-root"></div>
            <div id="modal-root"></div>
        </body>
    </html>
    ```

    - A `Parent` component is `#app-root` would be able to catch an uncaught, bubbling event from the sibling node `#modal-root`

        ```jsx
        // https://codepen.io/gaearon/pen/jGBWpE
        const appRoot = document.getElementById('app-root')
        const modalRoot = document.getElementById('modal-root')
        class Modal extends React.Component {
            constructor(props) {
                super(props)
                this.el = document.createElement('div')
            }
            componentDidMount() {
                modalRoot.appendChild(this.el)
            }
            componentWillUnmount() {
                modalRoot.removeChild(this.el)
            }
            render() {
                return ReactDOM.createPortal(
                    this.props.children,
                    this.el
                )
            }
        }
        class Parent extends React.Component {
            constructor(props) {
                super(props)
                this.state = { clicks: 0 }
                this.handleClick = this.handleClick.bind(this)
            }
            // This will fire when the button in `Child` is clicked,
            // updating Parent's state, even though that button
            // is not direct descendant in the DOM
            handleClick() {
                this.setState(state => {
                    clicks: state.clicks + 1
                })
            }
            render() {
                return (
                    <div onClick={this.handleClick}>
                        <p>Number of clicks: { this.state.clicks }</p>
                        <Modal>
                            <Child />
                        </Modal>
                    </div>
                )
            }
        }
        function Child() {
            // the click event on this button will bubble up to parent
            // because there's no `onClick` attribute defined
            return (
                <div class="modal">
                    <button>Click</button>
                </div> 
            )
        }
        ReactDOM.render(<Parent />, appRoot)
        ```