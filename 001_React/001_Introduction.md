# React.js Notes

## Module 1: Introduction to React.js

### What is React?
- A JavaScript library for building user interfaces.
- Developed by Facebook.
- Focuses on the **view layer** (MVC).
- Component-based architecture.
- Uses a virtual DOM for efficient rendering.

### Why Use React?
- **Reusable Components:** Write once, reuse anywhere.
- **Fast Rendering:** Virtual DOM ensures optimal updates.
- **Rich Ecosystem:** Large community and extensive libraries.
- **Declarative Syntax:** Easy to read and understand.
- **Cross-Platform:** React Native for mobile development.

### When to Use React?
- Single Page Applications (SPAs).
- Dynamic web apps with frequent UI updates.
- Applications requiring high performance.

---

## JavaScript Refreshment

### `let`, `var`, and `const`
- **`let`**: Block-scoped, allows reassignment.
- **`var`**: Function-scoped, avoid using due to hoisting issues.
- **`const`**: Block-scoped, immutable reference (cannot reassign).

### Import and Export Syntax
#### Named Exports
```javascript
// file.js
export const myFunction = () => {};
export const myVariable = 42;
```
```javascript
// main.js
import { myFunction, myVariable } from './file';
```

#### Default Exports
```javascript
// file.js
const myDefault = () => {};
export default myDefault;
```
```javascript
// main.js
import myDefault from './file';
```

---

### JS Classes and Inheritance
#### Defining a Class
```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    console.log(`${this.name} makes a noise.`);
  }
}
```

#### Extending a Class
```javascript
class Dog extends Animal {
  speak() {
    console.log(`${this.name} barks.`);
  }
}
const dog = new Dog('Rex');
dog.speak(); // Rex barks.
```
**Return Type:**
- `console.log` returns `undefined`.
- Classes themselves do not return values directly.
- Modifies no external state.

---

### Arrays and Common Methods
#### Common Methods
- **`map()`**: Creates a new array with the results of calling a provided function on every element.
  - **Return Type:** New array.
  - **Modification:** Does not modify the original array.
  
- **`filter()`**: Creates a new array with elements that pass the test implemented by the provided function.
  - **Return Type:** New array.
  - **Modification:** Does not modify the original array.

- **`reduce()`**: Executes a reducer function on each array element, resulting in a single output value.
  - **Return Type:** Single value of any type.
  - **Modification:** Does not modify the original array.

- **`forEach()`**: Executes a provided function once for each array element.
  - **Return Type:** `undefined`.
  - **Modification:** Does not create a new array, but may cause side effects.

- **`find()`**: Returns the first element that satisfies the provided testing function.
  - **Return Type:** Single element or `undefined`.
  - **Modification:** Does not modify the original array.

- **`includes()`**: Determines whether an array includes a certain value.
  - **Return Type:** Boolean (`true` or `false`).
  - **Modification:** Does not modify the original array.

- **`some()`**: Tests whether at least one element passes the provided function.
  - **Return Type:** Boolean.
  - **Modification:** Does not modify the original array.

- **`every()`**: Tests whether all elements pass the provided function.
  - **Return Type:** Boolean.
  - **Modification:** Does not modify the original array.

- **`push()`**: Adds one or more elements to the end of an array.
  - **Return Type:** New length of the array (number).
  - **Modification:** Modifies the original array.

- **`pop()`**: Removes the last element of an array.
  - **Return Type:** Removed element.
  - **Modification:** Modifies the original array.

- **`shift()`**: Removes the first element of an array.
  - **Return Type:** Removed element.
  - **Modification:** Modifies the original array.

- **`unshift()`**: Adds one or more elements to the beginning of an array.
  - **Return Type:** New length of the array (number).
  - **Modification:** Modifies the original array.

- **`slice()`**: Returns a shallow copy of a portion of an array into a new array.
  - **Return Type:** New array.
  - **Modification:** Does not modify the original array.

- **`splice()`**: Changes the contents of an array by removing or replacing existing elements and/or adding new elements.
  - **Return Type:** Array of removed elements.
  - **Modification:** Modifies the original array.

#### Examples
```javascript
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(num => num * 2); // [2, 4, 6, 8, 10]
const evens = numbers.filter(num => num % 2 === 0); // [2, 4]
const sum = numbers.reduce((acc, num) => acc + num, 0); // 15
const firstEven = numbers.find(num => num % 2 === 0); // 2
const hasThree = numbers.includes(3); // true
```

---

### Objects and Object Destructuring
#### Object Basics
```javascript
const person = {
  name: 'John',
  age: 30
};
```
#### Destructuring
```javascript
const { name, age } = person;
console.log(name, age); // John, 30
```
**Return Type:**
- Destructuring itself does not return but assigns variables.
- Does not modify the original object.

---

### Reference vs Primitives in JavaScript
- **Primitives**: String, Number, Boolean, Null, Undefined, Symbol.
  - Immutable.
  - Passed by value.

- **Reference Types**: Objects, Arrays, Functions.
  - Mutable.
  - Passed by reference.

#### Example
```javascript
let a = 10;
let b = a;
b = 20;
console.log(a); // 10

let obj1 = { value: 10 };
let obj2 = obj1;
obj2.value = 20;
console.log(obj1.value); // 20
```

---

## React Fiber, Reconciliation, and Diffing

### React Fiber
React Fiber is like a new engine in React that helps it work better and faster. Imagine React is building a Lego house, one brick at a time. Fiber breaks the work into small pieces, so it can pause and come back later if something important (like a user clicking a button) happens.

#### Benefits of React Fiber:
- **Smooth User Experience**: React can stop working on the Lego house to respond to user actions and then continue later.
- **Faster Updates**: It focuses on what’s most important first, like showing a button click immediately.
- **Efficient Rendering**: Breaks rendering into tiny tasks, so your app doesn’t freeze.

---

### Reconciliation
Reconciliation is React’s way of figuring out how to update the real DOM without doing too much work. It’s like comparing two Lego houses to see which bricks changed and only fixing those.

#### How It Works:
1. React looks at the new version of your components.
2. It compares the new virtual DOM with the old one.
3. React figures out the smallest number of changes to update the real DOM.

#### Benefits:
- **Saves Time**: Updates only what’s needed.
- **Keeps Apps Fast**: Doesn’t rebuild everything.

---

### Diffing Algorithm
The diffing algorithm is how React finds out what’s different between the old and new Lego houses (virtual DOMs). It’s like a super-smart checklist that skips unnecessary comparisons.

#### Key Steps:
1. **Same Type Check**: If two components have the same type, React reuses the DOM node.
2. **Different Types**: If the type changes, React replaces the node completely.
3. **Keys for Lists**: React uses keys to know which list items changed, moved, or stayed the same.

#### Benefits:
- **Smart Updates**: React minimizes changes to keep things fast.
- **Predictable Behavior**: With keys, it handles lists efficiently.

---

## Miscellaneous

### Best Practices
- **Code Organization**: Keep components and files modular.
- **Use Functional Components**: Prefer `useState` and `useEffect` over class components.
- **Prop Validation**: Use `PropTypes` or TypeScript for type checking.
- **Error Handling**: Use error boundaries.
- **Use Linting**: Enforce coding standards with ESLint and Prettier.
- **Optimize Performance**:
  - Use React.memo.
  - Avoid unnecessary re-renders.
  - Use `React.lazy` and `Suspense` for code splitting.

### Tips for Interviews
- Be clear about React lifecycle methods.
- Understand hooks deeply (`useState`, `useEffect`, `useContext`, etc.).
- Know state management libraries like Redux or Context API.
- Be comfortable with debugging and performance optimization.
- Have a solid understanding of JavaScript fundamentals.

---

## Additional Resources
- [React Official Documentation](https://reactjs.org/docs/getting-started.html)
- [JavaScript Info](https://javascript.info/)
- [MDN Web Docs](https://developer.mozilla.org/en-US/)

---
