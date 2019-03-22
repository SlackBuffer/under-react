- React elements are immutable. **Once you create an element, you can't change its children or attributes**
    - An element is like a single **frame of a movie**: it **represents the UI at a certain point in time**
- React DOM compares the **element and its children** to the previous one, and only applies the DOM updates **necessary** to bring the DOM to the desire state
- Thinking about how the UI should look at any given moment rather than how to change it over time eliminates a whole class of bugs (declarative)