# Redux Toolkit: Deep Dive into Slices and Thunks

## Understanding Redux Slices

Redux Slices are a way to organize related state, reducers, and actions together. Think of a slice as a self-contained module for a specific feature in your application. For example, you might have separate slices for authentication, shopping cart, and user preferences.

### Basic Slice Structure

Let's create multiple slices for a typical e-commerce application:

```javascript
// features/auth/authSlice.js
import { createSlice } from '@reduxjs/toolkit';

const authSlice = createSlice({
    name: 'auth',
    initialState: {
        user: null,
        isAuthenticated: false,
        error: null
    },
    reducers: {
        login(state, action) {
            state.user = action.payload;
            state.isAuthenticated = true;
            state.error = null;
        },
        logout(state) {
            state.user = null;
            state.isAuthenticated = false;
        },
        setError(state, action) {
            state.error = action.payload;
        }
    }
});

export const { login, logout, setError } = authSlice.actions;
export default authSlice.reducer;

// features/cart/cartSlice.js
const cartSlice = createSlice({
    name: 'cart',
    initialState: {
        items: [],
        totalAmount: 0,
        isLoading: false
    },
    reducers: {
        addItem(state, action) {
            const newItem = action.payload;
            const existingItem = state.items.find(item => item.id === newItem.id);
            
            if (existingItem) {
                existingItem.quantity++;
                existingItem.totalPrice += newItem.price;
            } else {
                state.items.push({
                    ...newItem,
                    quantity: 1,
                    totalPrice: newItem.price
                });
            }
            state.totalAmount += newItem.price;
        },
        removeItem(state, action) {
            const id = action.payload;
            const existingItem = state.items.find(item => item.id === id);
            
            if (existingItem.quantity === 1) {
                state.items = state.items.filter(item => item.id !== id);
            } else {
                existingItem.quantity--;
                existingItem.totalPrice -= existingItem.price;
            }
            state.totalAmount -= existingItem.price;
        }
    }
});

// features/products/productsSlice.js
const productsSlice = createSlice({
    name: 'products',
    initialState: {
        items: [],
        filteredItems: [],
        filters: {
            category: 'all',
            priceRange: { min: 0, max: Infinity }
        },
        isLoading: false,
        error: null
    },
    reducers: {
        setProducts(state, action) {
            state.items = action.payload;
            state.filteredItems = action.payload;
        },
        applyFilters(state, action) {
            const { category, priceRange } = action.payload;
            state.filters = { category, priceRange };
            
            state.filteredItems = state.items.filter(product => {
                const categoryMatch = category === 'all' || product.category === category;
                const priceMatch = product.price >= priceRange.min && 
                                 product.price <= priceRange.max;
                return categoryMatch && priceMatch;
            });
        }
    }
});
```

### Combining Multiple Slices

```javascript
// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import authReducer from '../features/auth/authSlice';
import cartReducer from '../features/cart/cartSlice';
import productsReducer from '../features/products/productsSlice';

const store = configureStore({
    reducer: {
        auth: authReducer,
        cart: cartReducer,
        products: productsReducer
    }
});

export default store;
```

## Understanding Thunks

Thunks are a pattern that allows you to write async logic that interacts with the store. They're particularly useful for API calls and other side effects.

### Basic Thunk Structure

Let's implement thunks for our e-commerce slices:

```javascript
// features/auth/authThunks.js
import { createAsyncThunk } from '@reduxjs/toolkit';
import { login, logout, setError } from './authSlice';

export const loginUser = createAsyncThunk(
    'auth/loginUser',
    async ({ email, password }, { dispatch }) => {
        try {
            const response = await fetch('api/login', {
                method: 'POST',
                body: JSON.stringify({ email, password }),
                headers: {
                    'Content-Type': 'application/json'
                }
            });
            
            if (!response.ok) throw new Error('Login failed');
            
            const user = await response.json();
            dispatch(login(user));
            return user;
        } catch (error) {
            dispatch(setError(error.message));
            throw error;
        }
    }
);

// features/products/productsThunks.js
import { createAsyncThunk } from '@reduxjs/toolkit';

export const fetchProducts = createAsyncThunk(
    'products/fetchProducts',
    async (_, { getState }) => {
        const { auth } = getState();
        const response = await fetch('api/products', {
            headers: {
                'Authorization': `Bearer ${auth.user?.token}`
            }
        });
        return response.json();
    }
);

// Enhanced productsSlice with async handling
const productsSlice = createSlice({
    name: 'products',
    initialState: {
        items: [],
        filteredItems: [],
        isLoading: false,
        error: null
    },
    reducers: {
        // ... previous reducers
    },
    extraReducers: (builder) => {
        builder
            .addCase(fetchProducts.pending, (state) => {
                state.isLoading = true;
                state.error = null;
            })
            .addCase(fetchProducts.fulfilled, (state, action) => {
                state.isLoading = false;
                state.items = action.payload;
                state.filteredItems = action.payload;
            })
            .addCase(fetchProducts.rejected, (state, action) => {
                state.isLoading = false;
                state.error = action.error.message;
            });
    }
});
```

### Using Multiple Thunks Together

Here's an example of how to coordinate multiple thunks for a checkout process:

```javascript
// features/checkout/checkoutThunks.js
import { createAsyncThunk } from '@reduxjs/toolkit';

const validateCart = createAsyncThunk(
    'checkout/validateCart',
    async (_, { getState }) => {
        const { cart } = getState();
        const response = await fetch('api/cart/validate', {
            method: 'POST',
            body: JSON.stringify(cart.items)
        });
        return response.json();
    }
);

const processPayment = createAsyncThunk(
    'checkout/processPayment',
    async (paymentDetails, { getState }) => {
        const { cart } = getState();
        const response = await fetch('api/payment', {
            method: 'POST',
            body: JSON.stringify({
                amount: cart.totalAmount,
                ...paymentDetails
            })
        });
        return response.json();
    }
);

const createOrder = createAsyncThunk(
    'checkout/createOrder',
    async (_, { getState, dispatch }) => {
        const { cart, auth } = getState();
        const response = await fetch('api/orders', {
            method: 'POST',
            body: JSON.stringify({
                items: cart.items,
                userId: auth.user.id
            })
        });
        return response.json();
    }
);

// Coordinating multiple thunks
export const processCheckout = createAsyncThunk(
    'checkout/process',
    async (paymentDetails, { dispatch }) => {
        // First validate the cart
        await dispatch(validateCart()).unwrap();
        
        // Then process the payment
        const paymentResult = await dispatch(processPayment(paymentDetails)).unwrap();
        
        // Finally create the order
        const order = await dispatch(createOrder()).unwrap();
        
        return {
            paymentResult,
            order
        };
    }
);
```

### Using Thunks in Components

```javascript
// components/Checkout.js
import { useDispatch, useSelector } from 'react-redux';
import { processCheckout } from '../features/checkout/checkoutThunks';

function Checkout() {
    const dispatch = useDispatch();
    const cart = useSelector(state => state.cart);
    const [isProcessing, setIsProcessing] = useState(false);
    const [error, setError] = useState(null);

    const handleCheckout = async () => {
        setIsProcessing(true);
        setError(null);
        
        try {
            const paymentDetails = {
                cardNumber: '4242424242424242',
                expiryMonth: '12',
                expiryYear: '2024',
                cvc: '123'
            };
            
            const result = await dispatch(processCheckout(paymentDetails)).unwrap();
            
            // Handle successful checkout
            console.log('Checkout completed:', result);
        } catch (err) {
            setError(err.message);
        } finally {
            setIsProcessing(false);
        }
    };

    return (
        <div>
            {/* Checkout UI */}
            {isProcessing && <p>Processing your order...</p>}
            {error && <p className="error">{error}</p>}
            <button 
                onClick={handleCheckout}
                disabled={isProcessing || cart.items.length === 0}
            >
                Complete Checkout
            </button>
        </div>
    );
}
```

## Best Practices for Slices and Thunks

1. Keep slices focused and cohesive - each slice should handle one specific feature or domain
2. Use meaningful action names that describe what's happening
3. Handle loading and error states in your reducers
4. Use the builder callback notation for `extraReducers` for better type safety
5. Use `unwrap()` when dispatching thunks to properly handle errors
6. Consider using RTK Query for API calls instead of thunks if you're dealing with CRUD operations
7. Use the `prepare` callback in reducers when you need to normalize or transform action payload data
8. Keep thunks focused on single responsibilities and compose them for complex operations

This structured approach to using slices and thunks helps maintain a clean and maintainable Redux codebase, especially as your application grows in complexity.
