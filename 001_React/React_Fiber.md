**React Fiber**

---

### **Key Points / Summary**

1. **What is React Fiber?**
   - React Fiber is the re-implementation of React’s core algorithm (reconciler), introduced in React 16.
   - It enables asynchronous rendering and improves React's performance by addressing limitations in the previous synchronous algorithm.
   - Fiber splits rendering into smaller units of work, allowing React to pause, resume, and prioritize updates effectively.

2. **Problems Fiber Solves**:
   - Prevents blocking the browser’s main thread during large updates, reducing lag.
   - Prioritizes updates, ensuring smoother user interactions and animations.
   - Separates reconciliation and rendering processes for better control.

3. **Phases in Fiber**:
   - **Reconciliation Phase**: React determines changes and creates a list of updates (interruptible).
   - **Commit Phase**: Updates are applied to the DOM (non-interruptible).

4. **Benefits of Fiber**:
   - Enhanced rendering performance and smoother UIs.
   - Better error handling with error boundaries.
   - Support for fragments, strings, and multiple elements as render outputs.
   - Improved support for complex animations, layouts, and gestures.

5. **Key Features of Fiber**:
   - Incremental rendering with work prioritization.
   - Scheduling updates based on priority levels.
   - Compatibility with new React features like hooks and concurrent rendering.

---

### **Detailed Notes**

#### **What is React Fiber?**
React Fiber is a foundational update to React’s reconciliation algorithm. It replaces the older, synchronous rendering process with an asynchronous approach, enabling React to:
- Manage complex updates efficiently.
- Handle high-priority tasks like user inputs and animations promptly.
- Maintain a responsive interface even during heavy computation.

The term “Fiber” refers to the tree data structure React uses to represent each node in the DOM tree.

---

#### **Problems Fiber Solves**

1. **Blocking Main Thread**:
   - In the pre-Fiber era, React’s synchronous rendering blocked the browser’s main thread until updates were complete.
   - Fiber’s incremental rendering avoids such bottlenecks, allowing smoother updates.

2. **Prioritization of Updates**:
   - High-priority tasks, such as user interactions, were previously delayed by low-priority updates.
   - Fiber prioritizes critical tasks, improving responsiveness.

3. **Separation of Concerns**:
   - Reconciliation and rendering were previously intertwined, limiting flexibility.
   - Fiber separates these processes, enabling better scheduling and control.

---

#### **React Before Fiber**
1. **Initial Rendering**:
   - React built a tree of nodes representing UI elements.
   - A virtual DOM tree was cloned from the rendered tree.

2. **Updating DOM**:
   - React compared new and old trees, diffing nodes to determine changes.
   - The updates were applied to the DOM.

3. **Synchronous Execution**:
   - The entire reconciliation and rendering process was synchronous, blocking other tasks.

4. **Limitations**:
   - No interruption or prioritization during reconciliation.
   - Deep DOM trees or large updates caused noticeable lag.

---

#### **React After Fiber**
1. **Incremental Rendering**:
   - Fiber breaks rendering into smaller units (“work units”).
   - These units can be paused, resumed, or restarted as needed.

2. **Two Phases**:
   - **Reconciliation Phase**:
     - React compares trees to determine necessary changes.
     - Creates a list of updates (interruptible).
   - **Commit Phase**:
     - Updates are applied to the DOM (non-interruptible).

3. **Scheduling Updates**:
   - Fiber assigns priority levels to updates (e.g., animations > data fetching).
   - High-priority updates are processed first, improving user experience.

---

#### **Key Benefits of React Fiber**

1. **Improved Rendering Performance**:
   - Incremental rendering prevents main thread blocking.
   - Ensures faster updates, even for complex UIs.

2. **Error Handling**:
   - Fiber introduces error boundaries to catch runtime errors gracefully.
   - Example:
     ```javascript
     class ErrorBoundary extends React.Component {
       constructor(props) {
         super(props);
         this.state = { hasError: false };
       }

       static getDerivedStateFromError(error) {
         return { hasError: true };
       }

       render() {
         if (this.state.hasError) {
           return <h1>Something went wrong.</h1>;
         }
         return this.props.children;
       }
     }
     ```

3. **Support for New Render Types**:
   - Render fragments, strings, or multiple elements directly.
   - Example:
     ```javascript
     render() {
       return [<div key="1">Item 1</div>, <div key="2">Item 2</div>];
     }
     ```

4. **Enhanced User Interface Performance**:
   - Fiber’s ability to prioritize rendering ensures smoother animations and gestures.

5. **Advanced Graphics Rendering**:
   - Libraries like `react-three-fiber` utilize Fiber to create performant 3D models within React applications.

---

#### **Phases of Fiber**

1. **Reconciliation Phase**:
   - React compares the current and updated trees.
   - Creates a list of changes required for the DOM.
   - This phase is interruptible, allowing React to handle high-priority tasks.

2. **Commit Phase**:
   - React applies changes to the DOM.
   - This phase is synchronous and ensures consistent updates.

---

#### **Use Cases for React Fiber**

1. **Animations and Gestures**:
   - Fiber’s prioritization ensures smooth animations and responsiveness.

2. **Error Handling**:
   - Use error boundaries to catch and handle runtime errors.

3. **Complex Applications**:
   - Apps with deep component trees or frequent state updates benefit from Fiber’s performance enhancements.

4. **3D Rendering**:
   - Utilize `react-three-fiber` for high-performance 3D graphics.

5. **Concurrent Rendering**:
   - Fiber enables React’s concurrent mode, allowing multiple UI updates to be processed seamlessly.

---

#### **Conclusion**
React Fiber is a fundamental rewrite of the reconciliation algorithm, designed to address performance bottlenecks and improve user experience. By introducing incremental rendering, separating reconciliation from rendering, and enabling prioritization, Fiber empowers developers to build high-performing, responsive React applications. Its support for advanced features like concurrent rendering, error boundaries, and efficient animations makes it a crucial upgrade for modern web development. By understanding Fiber’s processes, developers can harness its full potential to deliver seamless and interactive user experiences.

