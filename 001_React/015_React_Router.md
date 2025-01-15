# Complete React Router Documentation

## Table of Contents
1. Core Concepts
2. Installation and Setup
3. Routing Fundamentals
4. Navigation Methods
5. Dynamic Routing
6. Data Management
7. Advanced Features
8. Error Handling
9. Performance Optimization
10. Best Practices

## 1. Core Concepts

### Understanding Client-Side Routing
React Router enables client-side routing in React applications, allowing navigation between different views without full page reloads. This creates a smoother user experience by maintaining application state and reducing server load.

Key benefits include:
- Seamless user experience with instant page transitions
- Reduced server load through partial content updates
- Maintained application state during navigation
- Support for browser history and bookmarking
- Deep linking capabilities

## 2. Installation and Setup

First, let's install React Router in your project:

```bash
npm install react-router-dom
```

### Basic Application Setup
Here's how to set up routing in your React application:

```jsx
// src/index.jsx
import { createRoot } from 'react-dom/client';
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import App from './App';

// Router configuration function
function createAppRouter() {
  const router = createBrowserRouter([
    {
      path: '/',
      element: <App />,
      children: [
        {
          path: '/',
          element: <HomePage />,
          // Data loader for this route
          loader: async () => {
            return { message: 'Welcome to our app!' };
          }
        },
        {
          path: 'about',
          element: <AboutPage />
        }
      ]
    }
  ]);
  
  return router;
}

// Application initialization function
function initializeApp() {
  const container = document.getElementById('root');
  const root = createRoot(container);
  
  root.render(
    <RouterProvider router={createAppRouter()} />
  );
}

initializeApp();
```

## 3. Routing Fundamentals

### Route Definition Methods

#### Object-based Routes
The modern approach using `createBrowserRouter`:

```jsx
// src/routes/index.jsx
import { createBrowserRouter } from 'react-router-dom';

function defineRoutes() {
  return createBrowserRouter([
    {
      path: '/',
      element: <RootLayout />,
      errorElement: <ErrorPage />,
      children: [
        {
          index: true,
          element: <HomePage />,
          loader: homeLoader
        },
        {
          path: 'products',
          element: <ProductsLayout />,
          children: [
            {
              index: true,
              element: <ProductsList />,
              loader: productsLoader
            },
            {
              path: ':id',
              element: <ProductDetail />,
              loader: productDetailLoader
            }
          ]
        }
      ]
    }
  ]);
}
```

#### JSX-based Routes
Alternative approach using `createRoutesFromElements`:

```jsx
// src/routes/jsx-routes.jsx
import { createRoutesFromElements, Route } from 'react-router-dom';

function createJsxRoutes() {
  return createBrowserRouter(
    createRoutesFromElements(
      <Route path="/" element={<RootLayout />}>
        <Route index element={<HomePage />} loader={homeLoader} />
        <Route path="products" element={<ProductsLayout />}>
          <Route 
            index 
            element={<ProductsList />} 
            loader={productsLoader}
          />
          <Route 
            path=":id" 
            element={<ProductDetail />} 
            loader={productDetailLoader}
          />
        </Route>
      </Route>
    )
  );
}
```

## 4. Navigation Methods

### Link Component
The basic navigation component:

```jsx
// src/components/Navigation.jsx
import { Link } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      <Link 
        to="/" 
        className="nav-link"
      >
        Home
      </Link>
      <Link 
        to="/products" 
        className="nav-link"
      >
        Products
      </Link>
    </nav>
  );
}
```

### NavLink Component
Enhanced link component with active state awareness:

```jsx
// src/components/MainNavigation.jsx
import { NavLink } from 'react-router-dom';

function MainNavigation() {
  // Custom style function for active links
  function getNavLinkStyle({ isActive }) {
    return {
      fontWeight: isActive ? 'bold' : 'normal',
      color: isActive ? '#0066cc' : '#333333'
    };
  }

  return (
    <nav>
      <NavLink 
        to="/" 
        style={getNavLinkStyle}
        end
      >
        Home
      </NavLink>
      <NavLink 
        to="/products" 
        style={getNavLinkStyle}
      >
        Products
      </NavLink>
    </nav>
  );
}
```

### Programmatic Navigation
Using the `useNavigate` hook for code-based navigation:

```jsx
// src/components/LoginForm.jsx
import { useNavigate } from 'react-router-dom';

function LoginForm() {
  const navigate = useNavigate();

  // Form submission handler
  async function handleSubmit(event) {
    event.preventDefault();
    const success = await submitLoginData(formData);
    
    if (success) {
      // Navigate and replace current history entry
      navigate('/dashboard', { 
        replace: true,
        state: { from: 'login' }
      });
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
    </form>
  );
}
```

## 5. Dynamic Routing

### Route Parameters
Handling dynamic segments in URLs:

```jsx
// src/routes/product-detail.jsx
import { useParams, useLoaderData } from 'react-router-dom';

// Data loader function
async function loadProductData({ params }) {
  const response = await fetch(`/api/products/${params.id}`);
  if (!response.ok) {
    throw new Response('Product not found', { status: 404 });
  }
  return response.json();
}

function ProductDetail() {
  const { id } = useParams();
  const product = useLoaderData();

  return (
    <div>
      <h1>{product.name}</h1>
      <p>Product ID: {id}</p>
      <div>{product.description}</div>
    </div>
  );
}
```

## 6. Data Management

### Data Loading
Using loader functions for data fetching:

```jsx
// src/routes/products.jsx
import { useLoaderData, defer, Await } from 'react-router-dom';
import { Suspense } from 'react';

// Data loader with deferred loading
async function productsLoader() {
  return defer({
    products: fetchProducts(),
    categories: fetchCategories()
  });
}

function ProductsPage() {
  const { products, categories } = useLoaderData();

  return (
    <div>
      <Suspense fallback={<ProductsSkeleton />}>
        <Await resolve={products}>
          {(resolvedProducts) => (
            <ProductList products={resolvedProducts} />
          )}
        </Await>
      </Suspense>
      
      <Suspense fallback={<CategoriesSkeleton />}>
        <Await resolve={categories}>
          {(resolvedCategories) => (
            <CategoryFilter categories={resolvedCategories} />
          )}
        </Await>
      </Suspense>
    </div>
  );
}
```

### Form Handling
Using actions for form processing:

```jsx
// src/routes/new-product.jsx
import { Form, useActionData, useNavigation } from 'react-router-dom';

// Form action handler
async function createProductAction({ request }) {
  const formData = await request.formData();
  const productData = Object.fromEntries(formData);
  
  const validationErrors = validateProductData(productData);
  if (Object.keys(validationErrors).length > 0) {
    return { errors: validationErrors };
  }
  
  const response = await fetch('/api/products', {
    method: 'POST',
    body: JSON.stringify(productData),
    headers: {
      'Content-Type': 'application/json'
    }
  });
  
  if (!response.ok) {
    throw new Response('Failed to create product', { 
      status: response.status 
    });
  }
  
  return { success: true };
}

function NewProductForm() {
  const actionData = useActionData();
  const navigation = useNavigation();
  const isSubmitting = navigation.state === 'submitting';

  return (
    <Form method="post">
      <div>
        <label htmlFor="name">Product Name:</label>
        <input 
          type="text" 
          id="name" 
          name="name" 
          required 
        />
        {actionData?.errors?.name && (
          <p className="error">{actionData.errors.name}</p>
        )}
      </div>
      
      <button 
        type="submit" 
        disabled={isSubmitting}
      >
        {isSubmitting ? 'Creating...' : 'Create Product'}
      </button>
    </Form>
  );
}
```

## 7. Advanced Features

### Lazy Loading
Implementing code splitting with lazy loading:

```jsx
// src/routes/lazy-routes.jsx
import { lazy, Suspense } from 'react';
import { createBrowserRouter } from 'react-router-dom';

// Lazy loaded components
const ProductsPage = lazy(() => 
  import(/* webpackChunkName: "products" */ './pages/Products')
);

const ProductDetail = lazy(() => 
  import(/* webpackChunkName: "product-detail" */ './pages/ProductDetail')
);

// Router configuration with lazy loading
function createLazyRouter() {
  return createBrowserRouter([
    {
      path: '/',
      element: <RootLayout />,
      children: [
        {
          path: 'products',
          element: (
            <Suspense fallback={<ProductsLoader />}>
              <ProductsPage />
            </Suspense>
          ),
          children: [
            {
              path: ':id',
              element: (
                <Suspense fallback={<ProductLoader />}>
                  <ProductDetail />
                </Suspense>
              )
            }
          ]
        }
      ]
    }
  ]);
}
```

### Data Prefetching
Implementing prefetching for better performance:

```jsx
// src/components/ProductLink.jsx
import { Link, useFetcher } from 'react-router-dom';

function ProductLink({ product }) {
  const fetcher = useFetcher();

  // Prefetch product data on hover
  function handleMouseEnter() {
    fetcher.load(`/api/products/${product.id}`);
  }

  return (
    <Link 
      to={`/products/${product.id}`}
      onMouseEnter={handleMouseEnter}
      className="product-link"
    >
      {product.name}
    </Link>
  );
}
```

## 8. Error Handling

### Custom Error Boundaries
Implementing error handling for routes:

```jsx
// src/components/ErrorBoundary.jsx
import { useRouteError, isRouteErrorResponse } from 'react-router-dom';

function ErrorBoundary() {
  const error = useRouteError();

  // Handle specific error types
  function getErrorMessage(error) {
    if (isRouteErrorResponse(error)) {
      return error.status === 404
        ? 'Page not found'
        : `Error ${error.status}: ${error.statusText}`;
    }
    
    return error instanceof Error
      ? error.message
      : 'An unexpected error occurred';
  }

  return (
    <div className="error-container">
      <h1>Oops! Something went wrong</h1>
      <p>{getErrorMessage(error)}</p>
      <Link to="/" className="error-home-link">
        Return to Home
      </Link>
    </div>
  );
}
```

## 9. Performance Optimization

### Route-based Code Splitting
Implementing efficient code splitting:

```jsx
// src/routes/optimized-routes.jsx
import { lazy, Suspense } from 'react';

// Utility function for creating lazy components
function createLazyComponent(importFn, LoadingComponent) {
  const LazyComponent = lazy(importFn);
  
  return function LazyWrapper(props) {
    return (
      <Suspense fallback={<LoadingComponent />}>
        <LazyComponent {...props} />
      </Suspense>
    );
  };
}

// Create optimized route components
const OptimizedProducts = createLazyComponent(
  () => import('./pages/Products'),
  ProductsLoader
);

const OptimizedCheckout = createLazyComponent(
  () => import('./pages/Checkout'),
  CheckoutLoader
);

// Define optimized routes
function createOptimizedRouter() {
  return createBrowserRouter([
    {
      path: '/',
      element: <RootLayout />,
      children: [
        {
          path: 'products',
          element: <OptimizedProducts />,
          loader: productsLoader
        },
        {
          path: 'checkout',
          element: <OptimizedCheckout />,
          loader: checkoutLoader
        }
      ]
    }
  ]);
}
```

## 10. Best Practices

### Route Organization
Structuring routes for maintainability:

```jsx
// src/routes/root.jsx
import { lazy } from 'react';
import { createBrowserRouter } from 'react-router-dom';

// Import route modules
import { homeRoutes } from './home-routes';
import { productRoutes } from './product-routes';
import { checkoutRoutes } from './checkout-routes';

// Combine route configurations
function createAppRouter() {
  return createBrowserRouter([
    {
      path: '/',
      element: <RootLayout />,
      errorElement: <ErrorBoundary />,
      children: [
        ...homeRoutes,
        ...productRoutes,
        ...checkoutRoutes
      ]
    }
  ]);
}
```

### Authentication and Protection
Implementing protected routes:

```jsx
// src/components/ProtectedRoute.jsx
import { Navigate, useLocation } from 'react-router-dom';

function ProtectedRoute({ children }) {
  const location = useLocation();
  const isAuthenticated = useAuth();

  if (!isAuthenticated) {
    return <Navigate 
      to="/login" 
      state={{ from: location.pathname }}
      replace 
    />;
  }

  return children;
}

// Usage in route configuration
{
  path: 'admin',
  element: (
    <ProtectedRoute>
      <AdminDashboard />
    </ProtectedRoute>
  ),
  loader: adminLoader
}
```

### Performance Recommendations

1. Use lazy loading for large components and routes
2. Implement proper loading states with Suspense
3. Utilize data prefetching for anticipated user actions
4. Employ proper error boundaries
5. Structure routes hierarchically
6. Implement proper caching strategies
7. Use proper TypeScript types for better reliability
8. Monitor and optimize bundle sizes
9. Implement proper authentication guards
10. Use meaningful chunk names for lazy-loaded components

### Development Tips

1. Keep route configurations organized and modular
2. Use consistent naming conventions
3. Implement proper loading states
