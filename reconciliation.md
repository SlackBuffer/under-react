# Motivation
- When you use React, at a single point in time you can think of the `render()` function as creating a tree of React elements. On the next state or props update, that `render()` function will return a different tree of React elements
- React then needs to figure out how to efficiently update the UI to match the most recent tree
- There are some generic solutions to this algorithmic problem of generating the minimum number of operations to transform one tree into another
    - However, the [state of the art algorithms](https://grfia.dlsi.ua.es/ml/algorithms/references/editsurvey_bille.pdf) have a complexity in the order of O(n3) where n is the number of elements in the tree
    - If this is used in React, displaying 1000 elements would require in the order of one billion comparisons
- React implements a heuristic O(n) algorithm based on 2 assumptions
   1. 2 elements with different **types** will produce different trees
   2. The developer can hint at which child elements may be stable across different renders with a `key` prop
    - In practice, these assumptions are valid for almost all practical use cases
# The diffing algorithm
- When diffing 2 trees, React first compares the 2 **root element**
- The behavior is different depending on the types of the root elements
## 1. Elements of different types
- When the root elements have different types, React will tear down the old tree and build the new tree from scratch
    - Go from `<a>` to `<img>`, or from `<Article>` to `<Comment>`, or from `<Button>` to `<div>` - any of those will lead to a full rebuild
- When tearing down a tree, old DOM nodes are destroyed. Component instances receive `componentWillUnmount`. Any state associated with the old tree is lost. Any components below the root will also get unmounted and have their state destroyed
- When building up a new tree, new DOM nodes are inserted into the DOM. Component instances receive `componentWillMount` and `componentDidMount`
## 2. DOM element of the same type
- When comparing 2 React DOM elements of the same type, React looks at the **attributes** of both, **keeps the same underlying DOM node**, and **only updates the changed attributes**
- After handling the DOM node, React then **recurses on the children**
## 3. Component elements of the same type
- When a component updates, the instance stays the same, so that state is maintained across renders
- React updates the props of the underlying component instance to match the new element, and calls `componentWillReceiveProps` and `componentWillUpdate` on the underlying instance
- Next, the `render` method is called and the diff algorithm **recurses on the previous and the new result**
## 4. Recursing on children
- By default, when recursing on **the children of a DOM node**, React just iterates over both lists of children at the same time and generates a mutation whenever there's a difference
- If you implement it naively, inserting an element at the beginning has worse performance

    ```html
    <ul>
        <li>Duke</li>
        <li>Villanova</li>
    </ul>

    <ul>
        <li>Connecticut</li>
        <li>Duke</li>
        <li>Villanova</li>
    </ul>
    ```

    - React will mutate every child instead of realizing it can keep the `<li>Duke</li>` and `<li>Villanova</li>` subtrees intact. This inefficiency can be a problem
## Keys
- React supports a `key` attribute
- When children have keys, React uses the key to match children in the original tree with children in the subsequent tree
- In practice, finding a key is usually not hard. The element you are going to display may already have a unique ID, so the key can just come from your data
- When that’s not the case, you can add a new `ID` property to your model or **hash some parts of the content** to generate a key
- As a last resort, you can pass an item’s index in the array as a key
    - This can work well if the items are never reordered, but reorders will be slow
- Reorders can also cause issues with component state when indexes are used as keys
    - Component instances are updated and reused based on their key
    - If the key is an index, moving an item changes it
    - As a result, component state for things like uncontrolled inputs can get mixed up and updated in unexpected ways
# Tradeoffs
- It is important to remember that the reconciliation algorithm is an implementation detail
- React are regularly refining the heuristics in order to make common use cases faster
    - In the current implementation, you can express the fact that a subtree has been moved amongst its siblings, but you cannot tell that it has moved somewhere else
- Because React rely on heuristics, if the assumptions behind them are not met, performance will suffer
   1. The algorithm will not try to match subtrees of different types
        - If you see yourself alternating between two component types with very similar output, you may want to make it the same type
   2. Keys should be stable, predictable, and unique
        - Unstable keys (like those produced by `Math.random()`) will cause many component instances and DOM nodes to be unnecessarily recreated, which can cause performance degradation and lost state in child components