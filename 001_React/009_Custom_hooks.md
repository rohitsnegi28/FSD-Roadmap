# React Custom Hooks: A Comprehensive Guide

## Introduction

React custom hooks represent a powerful pattern in React development, enabling developers to extract and reuse stateful logic across components. This guide explores the fundamentals, best practices, and advanced techniques for creating and utilizing custom hooks effectively.

## Understanding Custom Hooks

Custom hooks are specialized JavaScript functions that begin with the prefix "use" and can leverage other React hooks. They serve as a mechanism to share logic between components without duplicating code. Unlike regular functions, custom hooks can maintain their own state and tap into React's lifecycle.

### Core Rules

When working with custom hooks, two fundamental rules must be followed:

```javascript
// The naming convention is crucial for React to recognize hooks
const useCustomHook = () => {
  // Hook implementation
};

// Hooks must be called at the component's top level
function Component() {
  // ✅ Correct: Called at top level
  const value = useCustomHook();

  if (condition) {
    // ❌ Incorrect: Called conditionally
    const value = useCustomHook();
  }
}
```

## Creating Custom Hooks

Let's explore how to create custom hooks through practical examples.

### Basic Hook Structure

```javascript
const useCustomHook = (initialValue) => {
  // State declarations come first
  const [state, setState] = useState(initialValue);

  // Effects follow state declarations
  useEffect(() => {
    // Effect implementation
    return () => {
      // Cleanup logic
    };
  }, [/* dependencies */]);

  // Helper functions and additional logic
  const updateState = (newValue) => {
    setState(newValue);
  };

  // Return values and functions
  return {
    state,
    updateState
  };
};
```

### Practical Example: Window Size Hook

The following example demonstrates a custom hook that tracks window dimensions:

```javascript
const useWindowSize = () => {
  const [dimensions, setDimensions] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });

  useEffect(() => {
    // Debouncing the resize handler improves performance
    let timeoutId = null;
    
    const handleResize = () => {
      clearTimeout(timeoutId);
      
      timeoutId = setTimeout(() => {
        setDimensions({
          width: window.innerWidth,
          height: window.innerHeight
        });
      }, 150);
    };

    window.addEventListener('resize', handleResize);
    
    // Cleanup prevents memory leaks
    return () => {
      window.removeEventListener('resize', handleResize);
      clearTimeout(timeoutId);
    };
  }, []);

  return dimensions;
};
```

## Advanced Patterns

### Custom Hook Composition

Complex functionality can be built by combining simpler hooks:

```javascript
const useEnhancedData = (resourceId) => {
  // Combine multiple hooks to create enhanced functionality
  const { data, loading } = useFetch(`/api/resource/${resourceId}`);
  const { width } = useWindowSize();
  const [preferences, setPreferences] = useLocalStorage(
    `preferences-${resourceId}`,
    defaultPreferences
  );

  // Derive additional values from hook data
  const isMobileView = width < 768;
  const enrichedData = useMemo(() => {
    if (!data) return null;
    return {
      ...data,
      formattedDate: new Date(data.timestamp).toLocaleDateString(),
      isFavorite: preferences.favorites.includes(resourceId)
    };
  }, [data, preferences]);

  return {
    data: enrichedData,
    loading,
    isMobileView,
    preferences,
    setPreferences
  };
};
```

### Error Handling and Loading States

Robust error handling is crucial for production-ready hooks:

```javascript
const useFetchWithRetry = (url, options = {}) => {
  const [state, setState] = useState({
    data: null,
    error: null,
    loading: true,
    retries: 0
  });

  const maxRetries = options.maxRetries || 3;
  const retryDelay = options.retryDelay || 1000;

  const fetchWithRetry = async (retryCount = 0) => {
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
      const data = await response.json();
      
      setState({
        data,
        error: null,
        loading: false,
        retries: retryCount
      });
    } catch (error) {
      if (retryCount < maxRetries) {
        setTimeout(() => {
          fetchWithRetry(retryCount + 1);
        }, retryDelay * Math.pow(2, retryCount)); // Exponential backoff
      } else {
        setState({
          data: null,
          error,
          loading: false,
          retries: retryCount
        });
      }
    }
  };

  useEffect(() => {
    fetchWithRetry();
  }, [url]);

  return state;
};
```

## Testing Custom Hooks

Testing custom hooks requires special consideration. Here's how to approach it:

```javascript
import { renderHook, act } from '@testing-library/react-hooks';

describe('useCounter', () => {
  it('should initialize with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  it('should increment counter', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });

  it('should handle multiple updates correctly', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
      result.current.increment();
      result.current.decrement();
    });

    expect(result.current.count).toBe(1);
  });
});
```

## Best Practices and Guidelines

1. **Isolation**: Each custom hook should have a single responsibility and be independent of component-specific logic.

2. **Naming**: Use clear, descriptive names that indicate the hook's purpose:
   - `useFormValidation` instead of `useValidation`
   - `useAuthenticationState` instead of `useAuth`

3. **Documentation**: Include comprehensive documentation for complex hooks:

```javascript
/**
 * Hook for managing paginated data fetching
 * @param {string} endpoint - API endpoint to fetch data from
 * @param {Object} options - Configuration options
 * @param {number} options.pageSize - Number of items per page
 * @param {number} options.initialPage - Starting page number
 * @returns {Object} Pagination state and control functions
 */
const usePagination = (endpoint, options = {}) => {
  // Implementation...
};
```

## Conclusion

Custom hooks provide a powerful mechanism for code reuse in React applications. By following these patterns and best practices, developers can create maintainable, testable, and efficient applications while avoiding common pitfalls in state management and side effect handling.

Remember that custom hooks should be focused, well-documented, and thoroughly tested. They represent an investment in code quality that pays dividends through improved maintainability and reduced debugging time.





# React Hooks: Best Practices and Advanced Patterns

## Understanding useFetch Custom Hook

Let's explore a robust implementation of `useFetch` that handles common API interaction patterns. This hook manages loading states, error handling, and provides data fetching capabilities.

```javascript
import { useState, useEffect, useCallback } from 'react';

const useFetch = (url, options = {}) => {
  // Maintain state for our API call
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [isLoading, setIsLoading] = useState(false);

  // Configure default options with any custom overrides
  const defaultOptions = {
    headers: {
      'Content-Type': 'application/json',
      ...options.headers,
    },
    ...options,
  };

  // Create a memoized fetchData function to prevent unnecessary re-renders
  const fetchData = useCallback(async () => {
    setIsLoading(true);
    setError(null);

    try {
      const response = await fetch(url, defaultOptions);
      
      // Handle non-2xx responses
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err.message || 'An error occurred while fetching data');
    } finally {
      setIsLoading(false);
    }
  }, [url, JSON.stringify(defaultOptions)]); // Stringify options for stable dependency

  // Trigger the fetch when component mounts or url/options change
  useEffect(() => {
    fetchData();
  }, [fetchData]);

  // Expose a refetch method for manual data refresh
  const refetch = useCallback(() => {
    fetchData();
  }, [fetchData]);

  return {
    data,
    error,
    isLoading,
    refetch,
  };
};

export default useFetch;
```

### Usage Example

```javascript
function UserProfile({ userId }) {
  const {
    data: user,
    error,
    isLoading,
    refetch
  } = useFetch(`/api/users/${userId}`);

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <button onClick={refetch}>Refresh Data</button>
    </div>
  );
}
```

## When to Use Hooks vs. When Not to Use Hooks

Let's explore when hooks are appropriate and when other patterns might be better suited for your needs.

### Use Hooks When:

1. **Sharing Stateful Logic**
   ```javascript
   // Good: Reusable form validation logic
   const useFormValidation = (initialValues) => {
     const [values, setValues] = useState(initialValues);
     const [errors, setErrors] = useState({});

     // Validation logic that can be reused across different forms
     const validate = useCallback(() => {
       // Implementation
     }, [values]);

     return { values, errors, validate };
   };
   ```

2. **Managing Component Lifecycle**
   ```javascript
   // Good: Managing WebSocket connections
   const useWebSocket = (url) => {
     const [socket, setSocket] = useState(null);

     useEffect(() => {
       const ws = new WebSocket(url);
       setSocket(ws);

       return () => ws.close();
     }, [url]);

     return socket;
   };
   ```

3. **Abstracting Complex State Logic**
   ```javascript
   // Good: Managing pagination state
   const usePagination = (items, itemsPerPage) => {
     const [currentPage, setCurrentPage] = useState(1);
     const maxPage = Math.ceil(items.length / itemsPerPage);

     const currentItems = items.slice(
       (currentPage - 1) * itemsPerPage,
       currentPage * itemsPerPage
     );

     return {
       currentItems,
       currentPage,
       maxPage,
       setCurrentPage,
     };
   };
   ```

### Don't Use Hooks When:

1. **Simple Utility Functions**
   ```javascript
   // Bad: Using hooks for simple calculations
   const useCalculateTotal = (items) => {
     return items.reduce((sum, item) => sum + item.price, 0);
   };

   // Better: Use a regular function
   const calculateTotal = (items) => {
     return items.reduce((sum, item) => sum + item.price, 0);
   };
   ```

2. **Pure UI Rendering**
   ```javascript
   // Bad: Using hooks for formatting
   const useFormattedDate = (date) => {
     return new Date(date).toLocaleDateString();
   };

   // Better: Use a regular function or component
   const FormattedDate = ({ date }) => (
     <span>{new Date(date).toLocaleDateString()}</span>
   );
   ```

3. **Business Logic Without State**
   ```javascript
   // Bad: Using hooks for data transformation
   const useTransformData = (data) => {
     return data.map(item => ({
       ...item,
       fullName: `${item.firstName} ${item.lastName}`
     }));
   };

   // Better: Use a regular function
   const transformData = (data) => {
     return data.map(item => ({
       ...item,
       fullName: `${item.firstName} ${item.lastName}`
     }));
   };
   ```

### Consider Alternatives When:

1. **Managing Global State**
   ```javascript
   // Consider Context or State Management Libraries
   const GlobalStateProvider = ({ children }) => {
     const [state, dispatch] = useReducer(reducer, initialState);
     
     return (
       <StateContext.Provider value={{ state, dispatch }}>
         {children}
       </StateContext.Provider>
     );
   };
   ```

2. **Complex Data Operations**
   ```javascript
   // Consider using a dedicated state management solution
   // Rather than creating complex custom hooks
   import { createSlice } from '@reduxjs/toolkit';

   const dataSlice = createSlice({
     name: 'data',
     initialState: [],
     reducers: {
       // Complex data operations
     }
   });
   ```

The key is to remember that hooks are designed to manage state and side effects in React components. If your logic doesn't involve either of these, a regular function or component might be more appropriate. Always consider the maintainability and reusability of your code when deciding whether to use hooks.

This guide provides a foundation for making informed decisions about when to use hooks in your React applications. Remember that hooks are powerful tools, but they're not always the best solution for every problem.
