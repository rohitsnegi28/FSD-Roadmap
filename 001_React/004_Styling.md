# Styling React Components

## Breaking CSS and Importing in JSX
In React, CSS can be written in separate files and imported into components to style them. This approach provides a clear separation of concerns and reusability.

### Steps:
1. Create a `.css` file (e.g., `App.css`).
2. Add your styles to the file.
3. Import the CSS file in your React component:

```jsx
import './App.css';

function App() {
    return (
        <div className="container">
            <h1>Hello, World!</h1>
        </div>
    );
}

export default App;
```

### Pros and Cons of Vanilla CSS in JSX
**Pros:**
- Easy to write and import.
- Decouples styles from components.
- No need to learn a JSX-specific syntax.

**Cons:**
- Styles are not scoped to individual components, which can lead to naming collisions.
- Hard to maintain in large projects.

## Dynamic and Conditional Inline CSS
React allows you to define styles directly in JSX using the `style` attribute, which takes an object.

### Example of Inline Styles:
```jsx
function Button({ isActive }) {
    const buttonStyle = {
        backgroundColor: isActive ? 'green' : 'red',
        color: 'white',
        padding: '10px 20px',
        border: 'none',
        borderRadius: '5px',
    };

    return <button style={buttonStyle}>Click Me</button>;
}
```

## Dynamic Class Names
To apply different classes dynamically, you can use JavaScript expressions inside JSX.

### Example:
```jsx
function Alert({ type }) {
    const className = `alert ${type === 'success' ? 'alert-success' : 'alert-danger'}`;

    return <div className={className}>This is an alert</div>;
}
```

### Libraries for Dynamic Class Management
Using libraries like `clsx` or `classnames` makes dynamic class management cleaner.

```jsx
import clsx from 'clsx';

function Alert({ type, isVisible }) {
    const className = clsx('alert', {
        'alert-success': type === 'success',
        'alert-danger': type === 'danger',
        hidden: !isVisible,
    });

    return <div className={className}>This is an alert</div>;
}
```

## Scoping CSS Rules with CSS Modules
CSS Modules provide scoped styles to React components by generating unique class names at build time.

### Example:
1. Create a CSS module file: `Button.module.css`:
   ```css
   .primary {
       background-color: blue;
       color: white;
       padding: 10px;
   }
   ```
2. Import and use it in your component:
   ```jsx
   import styles from './Button.module.css';

   function Button() {
       return <button className={styles.primary}>Click Me</button>;
   }
   ```

## Styled Components
Styled Components is a library for styling React components using tagged template literals.

### Installation:
```bash
npm install styled-components
```

### Example:
```jsx
import styled from 'styled-components';

const Button = styled.button`
    background-color: ${(props) => (props.$primary ? 'blue' : 'gray')};
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 5px;

    &:hover {
        background-color: ${(props) => (props.$primary ? 'darkblue' : 'darkgray')};
    }
`;

function App() {
    return <Button $primary>Click Me</Button>;
}

export default App;
```

### Features:
1. **Dynamic Conditions:**
   Styled Components accept props to dynamically adjust styles.
2. **Avoiding Prop Forwarding:**
   Use a `$` prefix for props to ensure they are not forwarded to the DOM. For example:
   ```jsx
   <Button $primary>Click Me</Button>
   ```
3. **If-Else in Props:**
   Use ternary operators inside styled templates for conditional logic.
4. **Nested Selectors and Pseudo Selectors:**
   Easily define nested or pseudo styles, such as `:hover` and `:active`.

## Tailwind CSS
Tailwind CSS is a utility-first CSS framework providing predefined classes to style elements directly in JSX.

### Installation:
1. Install Tailwind CSS:
   ```bash
   npm install -D tailwindcss postcss autoprefixer
   npx tailwindcss init
   ```
2. Configure `tailwind.config.js` and add Tailwind to your CSS:
   ```css
   @tailwind base;
   @tailwind components;
   @tailwind utilities;
   ```
3. Import it in your `index.css` or main CSS file.

### Example:
```jsx
function Button() {
    return <button className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">Click Me</button>;
}
```

### Pros:
- Fast to use with predefined utility classes.
- Encourages consistent design systems.
- Great for rapid prototyping.

### Cons:
- Can clutter JSX with too many classes.
- Learning curve for understanding class names.

## Summary
- **Vanilla CSS:** Simple and decoupled but lacks scoping.
- **CSS Modules:** Scoped styles with unique class names.
- **Inline Styles:** Best for dynamic styles but limited in functionality.
- **Styled Components:** Fully dynamic, supports theming, and scoped.
- **Tailwind CSS:** Utility-first and rapid, but JSX can become verbose.
