# React Side Effects and useEffect Hook - Complete Guide

## 1. What is a Side Effect?

### Definition
A side effect is any operation in a React component that:
- Interacts with the outside world
- Cannot be done during rendering
- Affects something outside the scope of the current function

### Examples of Side Effects
1. Data fetching (API calls)
2. Manually modifying the DOM
3. Setting up subscriptions or timers
4. Writing to local storage
5. Integrating with third-party libraries

### Why Side Effects Need Special Handling
1. They can cause unexpected behavior if not managed properly
2. May lead to infinite re-render loops
3. Can create memory leaks if not cleaned up
4. Might execute at wrong times during the component lifecycle

## 2. The useEffect Hook

### Definition
`useEffect` is a built-in React Hook that lets you synchronize a component with external systems and manage side effects in functional components.

### Basic Syntax
```jsx
useEffect(() => {
  // Effect code here
  
  return () => {
    // Optional cleanup code
  };
}, [dependencies]);
```

### Key Characteristics
1. Runs after every render by default
2. Can specify dependencies to control when it runs
3. Can include a cleanup function
4. Executes asynchronously after the render is committed to the screen

## 3. When to Use useEffect

### Appropriate Use Cases

1. **Data Fetching**
```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    // Basic fetch example
    async function fetchUser() {
      const response = await fetch(`/api/users/${userId}`);
      const data = await response.json();
      setUser(data);
    }
    fetchUser();
  }, [userId]);

  // Usage
  return <div>{user?.name}</div>;
}
```

2. **DOM Manipulation**
```jsx
function PageTitle({ title }) {
  useEffect(() => {
    document.title = title;
  }, [title]);

  return <h1>{title}</h1>;
}
```

3. **Subscriptions**
```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const chat = new ChatService(roomId);
    chat.subscribe(newMessage => {
      setMessages(prev => [...prev, newMessage]);
    });

    return () => chat.unsubscribe();
  }, [roomId]);

  return <MessageList messages={messages} />;
}
```

## 4. When NOT to Use useEffect

### Scenarios Where useEffect is Unnecessary

1. **Calculations from Props or State**
```jsx
// ❌ Bad: Using useEffect for derived state
function ProductList({ products }) {
  const [filteredProducts, setFilteredProducts] = useState([]);
  
  useEffect(() => {
    setFilteredProducts(
      products.filter(p => p.inStock)
    );
  }, [products]);

  // ✅ Good: Direct calculation
  const filteredProducts = products.filter(p => p.inStock);
}
```

2. **Synchronous Side Effects**
```jsx
// ❌ Bad: Unnecessary useEffect
function Form() {
  const [value, setValue] = useState('');
  
  useEffect(() => {
    localStorage.setItem('formValue', value);
  }, [value]);

  // ✅ Good: Direct implementation
  function handleChange(e) {
    const newValue = e.target.value;
    setValue(newValue);
    localStorage.setItem('formValue', newValue);
  }
}
```

## 5. Effect Dependencies

### Empty Dependency Array ([])
Runs only once after initial render:
```jsx
function App() {
  useEffect(() => {
    // Initialization code
    console.log('App mounted');
  }, []); // Empty array = run once
}
```

### With Dependencies
Runs when specified values change:
```jsx
function Counter({ initialCount }) {
  const [count, setCount] = useState(initialCount);

  useEffect(() => {
    console.log(`Count changed to: ${count}`);
  }, [count]); // Runs when count changes
}
```

### No Dependency Array
Runs after every render:
```jsx
useEffect(() => {
  // Runs after every render
  console.log('Component updated');
}); // No dependency array
```

## 6. Cleanup Functions

### Definition
A cleanup function is returned by an effect to clean up any side effects (subscriptions, timers, etc.) before the component unmounts or before the effect runs again.

### When Cleanup Runs
1. Before the component unmounts
2. Before the effect runs again (if dependencies change)

### Examples

1. **Timer Cleanup**
```jsx
function Timer() {
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('Timer tick');
    }, 1000);

    return () => {
      clearInterval(timer); // Cleanup
    };
  }, []);
}
```

2. **Event Listener Cleanup**
```jsx
function WindowTracker() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    function handleResize() {
      setWidth(window.innerWidth);
    }
    
    window.addEventListener('resize', handleResize);
    
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);
}
```

## 7. Browser API Integration

### Example: Geolocation API
```jsx
function LocationTracker() {
  const [location, setLocation] = useState(null);

  useEffect(() => {
    if (!navigator.geolocation) {
      return;
    }

    const watchId = navigator.geolocation.watchPosition(
      position => {
        setLocation({
          latitude: position.coords.latitude,
          longitude: position.coords.longitude
        });
      },
      error => {
        console.error('Error:', error);
      }
    );

    return () => {
      navigator.geolocation.clearWatch(watchId);
    };
  }, []);

  return location ? (
    <div>
      Latitude: {location.latitude}
      Longitude: {location.longitude}
    </div>
  ) : (
    <div>Loading location...</div>
  );
}
```

## 8. The useCallback Hook

### Definition
`useCallback` is a Hook that returns a memoized version of a callback that only changes if one of the dependencies has changed.

### Basic Syntax
```jsx
const memoizedCallback = useCallback(
  () => {
    // Function body
  },
  [dependencies]
);
```

### Example with useEffect
```jsx
function SearchComponent({ query, onResult }) {
  // Memoize the search function
  const performSearch = useCallback(async () => {
    const results = await fetch(`/api/search?q=${query}`);
    const data = await results.json();
    onResult(data);
  }, [query, onResult]);

  // Use the memoized function in effect
  useEffect(() => {
    performSearch();
  }, [performSearch]);
}
```

## 9. Common Problems and Solutions

### Problem 1: Infinite Loops
```jsx
// ❌ Bad: Creates infinite loop
function BadComponent() {
  const [data, setData] = useState([]);
  
  useEffect(() => {
    setData([...data, 'new item']); // Triggers re-render
  }); // Missing dependency array

  // ✅ Good: Add proper dependencies
  useEffect(() => {
    // Only run once on mount
    setData(['initial item']);
  }, []); // Empty dependency array
}
```

### Problem 2: Race Conditions
```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);

  useEffect(() => {
    let isCurrent = true; // Flag to track component mount state

    async function fetchResults() {
      const response = await fetch(`/api/search?q=${query}`);
      const data = await response.json();
      
      if (isCurrent) { // Only update if component is still mounted
        setResults(data);
      }
    }

    fetchResults();

    return () => {
      isCurrent = false; // Cleanup to prevent setting state on unmounted component
    };
  }, [query]);
}
```

## 10. Best Practices

1. **Always specify dependencies**
2. **Keep effects focused on one responsibility**
3. **Use cleanup functions when needed**
4. **Avoid object and function dependencies unless memoized**
5. **Use ESLint rules for hooks**
6. **Consider extracting complex effects into custom hooks**
```jsx
// Custom hook example
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return width;
}

// Usage
function ResponsiveComponent() {
  const width = useWindowWidth();
  return <div>Window width is: {width}</div>;
}
```
