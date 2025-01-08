# Module 3: React Essential Deep Dive

## JSX vs `React.createElement`

### JSX
- JSX is a syntax extension for JavaScript that looks similar to HTML but is processed by React to create elements.
- Easier to write and understand compared to `React.createElement`.

#### Example:
```jsx
const element = <h1>Hello, world!</h1>;
```

### `React.createElement`
- A function that creates React elements.
- JSX is transformed into `React.createElement` calls under the hood.

#### Example:
```javascript
const element = React.createElement('h1', null, 'Hello, world!');
```

### Why JSX?
- Cleaner and more intuitive.
- Helps visualize component structure.
- Easier to debug with better error messages.

---

## React Fragments
### What are React Fragments?
- A way to group multiple elements without adding extra nodes to the DOM.

### Syntax
#### Old Syntax:
```jsx
<React.Fragment>
  <p>First line</p>
  <p>Second line</p>
</React.Fragment>
```

#### Short Syntax:
```jsx
<>
  <p>First line</p>
  <p>Second line</p>
</>
```

### Why Use Fragments?
- Prevent unnecessary wrapper `<div>` elements.
- Improve performance by avoiding extra DOM nodes.

---

## When Should We Split Components?
- **When components become too large or complex.**
  - Easier to test, debug, and reuse.
- **When a section of the UI is reused in multiple places.**
- **When a component has distinct responsibilities.**
  - Example: A form split into `FormInput`, `FormButton`, and `FormContainer` components.

---

## Forwarding Props to Wrapped Elements
- Forwarding props is passing down props from a parent to a child component without directly using them in the middle component.

#### Example:
```jsx
const Wrapper = (props) => {
  return <div {...props}>{props.children}</div>;
};

const App = () => {
  return <Wrapper className="custom">Hello!</Wrapper>;
};
```

---

## Multiple JSX Slots
- Multiple JSX slots allow you to pass JSX as children to components in a flexible way.

#### Why?
- To make components more dynamic and reusable.

#### Example:
```jsx
const Layout = ({ header, footer, children }) => {
  return (
    <div>
      <header>{header}</header>
      <main>{children}</main>
      <footer>{footer}</footer>
    </div>
  );
};

const App = () => {
  return (
    <Layout 
      header={<h1>Header</h1>} 
      footer={<p>Footer</p>}>
      <p>Content goes here</p>
    </Layout>
  );
};
```

---

## Setting Components Dynamically in React
- Dynamically render components by passing their names as strings and mapping them to actual components.

#### Example:
```jsx
const components = {
  Header: () => <h1>Header</h1>,
  Footer: () => <p>Footer</p>,
};

const DynamicComponent = ({ componentName }) => {
  const Component = components[componentName];
  return Component ? <Component /> : <p>Component not found</p>;
};

const App = () => {
  return <DynamicComponent componentName="Header" />;
};
```

---

## Default Props
- Provide default values for props when none are specified.

#### Example:
```jsx
const Button = ({ label = 'Click me' }) => <button>{label}</button>;
```

---

## Public vs Asset Folder
### Key Differences:
1. **Public Folder:**
   - Files in `public` are not processed by Webpack.
   - Accessible directly via `/filename`.
   - Best for static files like `index.html` or robots.txt.

2. **Asset Folder:**
   - Files in `src/assets` are processed and bundled.
   - Use for images and files imported into components.

### Which Gets Built?
- Only the `src` folder gets built and optimized.

---

## Updating React States Correctly Using Old State
- Always use a callback to access the previous state when updating.

#### Example:
```jsx
const [count, setCount] = useState(0);

const increment = () => {
  setCount((prevCount) => prevCount + 1);
};
```

---

## Correct Way to Update State with Arrays/Objects
### Why Not Directly Modify State?
- Directly modifying state can lead to bugs because React relies on immutability to detect changes.

#### Example with Spread Operator:
```jsx
const [items, setItems] = useState([1, 2, 3]);

const addItem = () => {
  setItems((prevItems) => [...prevItems, 4]);
};
```

---

## Lifting State Up
- **What:** Move state to the closest common ancestor component when multiple components need to share it.

#### When to Lift State
- When state needs to be shared between components.
- Example: A parent component managing form inputs.

#### When Not to Lift State
- When the state is specific to a single component.
- Over-lifting can make components less reusable and harder to maintain.

### Alternatives to Lifting State
- **Context API:** For global state.
- **State Management Libraries:** Redux, MobX, etc.

---

## Avoid Intersecting States
- **What:** Ensure states in different components don’t depend on each other unnecessarily.
- **Why:** Prevent unintended re-renders and complex debugging.

#### Example:
Instead of managing `isLoggedIn` and `userRole` separately, manage a single `user` object.

---

## Why Immutability Matters
- **What:** Ensures previous states remain unchanged.
- **Why:**
  - Allows React to detect state changes efficiently.
  - Prevents bugs caused by accidental state mutations.

#### Example:
```jsx
const [user, setUser] = useState({ name: 'John', age: 30 });

const updateName = () => {
  setUser((prevUser) => ({ ...prevUser, name: 'Jane' }));
};
```

---

## Prefer Computed Values
- Avoid storing values in state that can be derived from other state or props.

#### Example:
```jsx
const [firstName, setFirstName] = useState('John');
const [lastName, setLastName] = useState('Doe');

const fullName = `${firstName} ${lastName}`; // Computed value
```
