# React Essentials

## Module 2: Core Concepts of React

### React Components: Building Blocks of React
- **Definition**: Components are the smallest building blocks of a React application.
- **Reusable**: Promotes reusability of UI elements.
- **Separation of Concerns**: Groups related logic and UI together.
- **Types of Components**:
  - Functional Components: Simpler, use hooks for state and lifecycle.
  - Class Components: Use class syntax, older method, includes lifecycle methods.

### Understanding `index.html` and `App.jsx`
- **`index.html`**:
  - Contains a single `<div id="root"></div>` where the React app mounts.
  - Acts as the entry point for the React DOM.
- **`App.jsx`**:
  - The main component rendered inside the `root`.
  - Often serves as the parent component for other components in the app.

### JSX: What and Why?
- **Definition**: JavaScript XML, a syntax extension to write HTML in JavaScript.
- **Why JSX?**
  - Makes UI logic and HTML readable in the same file.
  - Easier to debug and manage.
  - Under the hood, converted to `React.createElement()` calls by React compiler.

### Components and Component Tree
- **Component Tree**:
  - Hierarchical structure showing parent-child relationships of components.
  - Helps visualize data flow and component relationships.
- **Example**:
```jsx
<App>
  <Header />
  <Main>
    <Sidebar />
    <Content />
  </Main>
</App>
```

### React Compiler
- Converts JSX code into JavaScript.
- Uses Babel and Webpack for bundling and transpiling.

### Why Component Names Should Be Capitalized
- React treats lowercase names as HTML tags and uppercase names as components.
- Convention helps React distinguish between custom and built-in elements.

### Dynamic Output in Components
- Use `{}` to embed dynamic values inside JSX.
- Example:
```jsx
const name = "John";
return <h1>Hello, {name}!</h1>;
```

### Setting HTML Attributes Dynamically
- Use JSX expressions to set attributes.
```jsx
const isActive = true;
return <button disabled={!isActive}>Click Me</button>;
```

### Making Components Reusable with Props
- **Props**:
  - Short for properties.
  - Pass data from parent to child components.
- **Example**:
```jsx
const Button = ({ label }) => <button>{label}</button>;
<Button label="Click Me" />;
```

### Best Practice: Storing Components in Files
- **Project Structure**:
```
src/
  components/
    Header.jsx
    Footer.jsx
  App.jsx
```
- Promotes clarity and separation of concerns.

### `children` Prop
- Allows components to pass nested elements.
- Example:
```jsx
const Card = ({ children }) => <div className="card">{children}</div>;
<Card>
  <h1>Title</h1>
  <p>Description</p>
</Card>
```

### Function Props and Naming Conventions
- **Naming Conventions**:
  - Use `onSomething` for prop names (e.g., `onChange`).
  - Use `handleSomething` for handler functions (e.g., `handleChange`).
- Example:
```jsx
const Input = ({ onChange }) => (
  <input type="text" onChange={onChange} />
);
const handleChange = (e) => console.log(e.target.value);
<Input onChange={handleChange} />;
```

### State and Re-rendering
- **State**:
  - Allows components to manage and update dynamic data.
  - Changes in state trigger a re-render.
- **Using `useState`**:
```jsx
const [count, setCount] = useState(0);
const increment = () => setCount(count + 1);
return <button onClick={increment}>Count: {count}</button>;
```

### Managing State with Hooks
- **`useState`**:
  - Returns an array: [state, updater function].
  - Example:
```jsx
const [state, setState] = useState(initialValue);
```

### Rendering Content Dynamically
- Use conditional rendering to display content based on state or props.
- Example:
```jsx
const isLoggedIn = true;
return isLoggedIn ? <h1>Welcome!</h1> : <h1>Please Log In</h1>;
```

### CSS Styling and Dynamic Styling
- **Inline Styling**:
```jsx
const style = { color: 'blue', fontSize: '20px' };
return <h1 style={style}>Hello</h1>;
```
- **Dynamic Styling**:
```jsx
const isActive = true;
const style = { backgroundColor: isActive ? 'green' : 'red' };
return <div style={style}>Status</div>;
