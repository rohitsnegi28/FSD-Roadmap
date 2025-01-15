# React Router

## Fundamentals of React Routing

### What is React Router?
React Router is a client-side routing library that enables navigation and view management in React applications without full page reloads. It maintains synchronization between the UI and URL, providing a seamless single-page application experience.

### Why Use React Router?
- Enables client-side routing without server requests
- Maintains clean, semantic URLs
- Improves user experience with faster page transitions
- Supports complex navigation patterns
- Facilitates state preservation during navigation

### How to Implement React Router?
First, install the package:
```bash
npm install react-router-dom
```

Basic setup in your application:
```jsx
// index.js
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    children: [
      { path: '/', element: <HomePage /> },
      { path: '/about', element: <AboutPage /> }
    ]
  }
]);

ReactDOM.render(
  <RouterProvider router={router} />,
  document.getElementById('root')
);
```

## Router Creation Methods

### createBrowserRouter
**What**: The recommended router for web applications, using clean URLs.

**When to use**: For most web applications where you want clean URLs without hash fragments.

```jsx
import { createBrowserRouter } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    errorElement: <ErrorPage />,
    children: [
      { 
        path: 'products',
        element: <ProductsPage />,
        loader: productsLoader // Data fetching
      }
    ]
  }
]);
```

### createRoutesFromElements
**What**: Enables JSX-style route definitions instead of objects.

**When to use**: When you prefer JSX syntax or are migrating from older versions of React Router.

```jsx
import { createRoutesFromElements, Route } from 'react-router-dom';

const router = createBrowserRouter(
  createRoutesFromElements(
    <Route path="/" element={<RootLayout />}>
      <Route index element={<HomePage />} />
      <Route path="products" element={<ProductsPage />} />
    </Route>
  )
);
```

## Navigation and Links

### Basic Navigation
**Using Link Component:**
```jsx
import { Link } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/products">Products</Link>
      {/* Relative path example */}
      <Link to="../">Back</Link>
    </nav>
  );
}
```

### NavLink Component
**What**: Enhanced version of Link with active state awareness.

**When to use**: For navigation menus where you need to highlight the active route.

```jsx
import { NavLink } from 'react-router-dom';

function MainNav() {
  return (
    <nav>
      <NavLink 
        to="/"
        className={({ isActive }) => 
          isActive ? 'nav-link active' : 'nav-link'
        }
        end // Prevents matching on child routes
      >
        Home
      </NavLink>
    </nav>
  );
}
```

### useNavigate Hook
**What**: Hook for programmatic navigation.

**When to use**: For navigation after form submissions, timeouts, or other events.

```jsx
import { useNavigate } from 'react-router-dom';

function LoginForm() {
  const navigate = useNavigate();

  const handleSubmit = async (event) => {
    event.preventDefault();
    const success = await submitForm();
    
    if (success) {
      // Navigate with replace to prevent back button to login
      navigate('/dashboard', { replace: true });
    }
  };

  return <form onSubmit={handleSubmit}>{/* form fields */}</form>;
}
```

## Layouts and Nested Routes

### Layout Routes
**What**: Parent routes that provide common UI elements for child routes.

```jsx
// layouts/RootLayout.jsx
import { Outlet } from 'react-router-dom';

function RootLayout() {
  return (
    <>
      <MainNavigation />
      <main>
        <Outlet /> {/* Child routes render here */}
      </main>
      <Footer />
    </>
  );
}

// Route configuration
const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    children: [
      { path: '', element: <HomePage /> },
      { 
        path: 'products',
        element: <ProductsLayout />,
        children: [
          { index: true, element: <ProductsList /> },
          { path: ':id', element: <ProductDetail /> }
        ]
      }
    ]
  }
]);
```

## Dynamic Routing and Parameters

### Dynamic Route Parameters
**What**: URL segments that capture values from the URL.

```jsx
// routes/product-detail.jsx
import { useParams } from 'react-router-dom';

function ProductDetail() {
  const { id } = useParams();
  
  return <div>Product ID: {id}</div>;
}

// Route configuration
{
  path: 'products/:id', // :id is the dynamic parameter
  element: <ProductDetail />
}
```

## Data Loading with loader()

### Defining Loaders
**What**: Functions that load data before rendering a route.

**When to use**: For fetching data needed by a route component.

```jsx
// routes/products.jsx
export async function loader({ request, params }) {
  // Access URL parameters and search params
  const url = new URL(request.url);
  const searchTerm = url.searchParams.get('search');
  
  try {
    const response = await fetch(
      `/api/products?search=${searchTerm}`
    );
    
    if (!response.ok) {
      throw new Response(
        JSON.stringify({ message: 'Failed to fetch products' }), 
        { status: 500 }
      );
    }
    
    return response; // React Router will automatically call .json()
  } catch (error) {
    throw new Response(
      JSON.stringify({ message: 'Failed to fetch products' }), 
      { status: 500 }
    );
  }
}

// Using loader data in component
import { useLoaderData } from 'react-router-dom';

function ProductsPage() {
  const products = useLoaderData();
  
  return (
    <div>
      {products.map(product => (
        <ProductItem key={product.id} product={product} />
      ))}
    </div>
  );
}
```

## Form Handling and Actions

### Action Functions
**What**: Route-specific functions for handling form submissions and data mutations.

```jsx
// routes/new-product.jsx
export async function action({ request }) {
  const formData = await request.formData();
  const productData = Object.fromEntries(formData);
  
  // Validation
  const errors = {};
  if (!productData.title) {
    errors.title = 'Title is required';
  }
  
  if (Object.keys(errors).length > 0) {
    return { errors };
  }
  
  // Submit to backend
  const response = await fetch('/api/products', {
    method: 'POST',
    body: JSON.stringify(productData)
  });
  
  if (!response.ok) {
    throw new Response(
      JSON.stringify({ message: 'Failed to create product' }), 
      { status: 500 }
    );
  }
  
  return redirect('/products');
}

// Component with Form
import { Form, useActionData } from 'react-router-dom';

function NewProduct() {
  const actionData = useActionData();
  
  return (
    <Form method="post">
      <input 
        type="text" 
        name="title"
        className={actionData?.errors?.title ? 'error' : ''}
      />
      {actionData?.errors?.title && (
        <p className="error">{actionData.errors.title}</p>
      )}
      <button type="submit">Add Product</button>
    </Form>
  );
}
```

## Advanced Features

### Deferred Data Loading
**What**: Technique to load non-critical data after initial render.

```jsx
// routes/product-detail.jsx
import { Suspense } from 'react';
import { defer, useLoaderData, Await } from 'react-router-dom';

export async function loader({ params }) {
  const productPromise = fetch(`/api/products/${params.id}`);
  const reviewsPromise = fetch(`/api/products/${params.id}/reviews`);
  
  return defer({
    product: await productPromise.json(), // Critical data - wait
    reviews: reviewsPromise.json() // Non-critical - load after render
  });
}

function ProductDetail() {
  const { product, reviews } = useLoaderData();
  
  return (
    <div>
      <h1>{product.title}</h1>
      <Suspense fallback={<p>Loading reviews...</p>}>
        <Await 
          resolve={reviews}
          errorElement={<p>Error loading reviews!</p>}
        >
          {(resolvedReviews) => (
            <ReviewsList reviews={resolvedReviews} />
          )}
        </Await>
      </Suspense>
    </div>
  );
}
```

### useFetcher Hook
**What**: Hook for out-of-band data loading and mutations.

**When to use**: For data operations that shouldn't trigger navigation.

```jsx
import { useFetcher } from 'react-router-dom';

function ProductFavoriteButton({ productId }) {
  const fetcher = useFetcher();
  
  return (
    <fetcher.Form method="post" action={`/products/${productId}/favorite`}>
      <button 
        type="submit"
        disabled={fetcher.state !== 'idle'}
      >
        {fetcher.state === 'submitting' 
          ? 'Updating...' 
          : 'Add to Favorites'}
      </button>
    </fetcher.Form>
  );
}
```

## Error Handling

### Custom Error Elements
**What**: Components that render when routes throw errors.

```jsx
// error.jsx
import { useRouteError } from 'react-router-dom';

function ErrorPage() {
  const error = useRouteError();
  
  return (
    <div className="error-container">
      <h1>Oops! Something went wrong</h1>
      <p>
        {error.status === 404 
          ? 'Page not found!' 
          : 'An error occurred!'}
      </p>
      <pre>{error.data?.message || error.message}</pre>
    </div>
  );
}

// Route configuration
{
  path: '/',
  element: <RootLayout />,
  errorElement: <ErrorPage />,
  children: [/*...*/]
}
```

## Best Practices and Tips

1. Always use relative paths in nested routes
2. Implement proper error boundaries
3. Use loader functions for data fetching
4. Keep route configurations organized
5. Implement proper loading states
6. Handle errors gracefully
7. Use TypeScript for better type safety
8. Implement proper routing guards
9. Use proper HTTP status codes in errors
10. Implement proper data caching strategies

Remember that React Router is a powerful tool that can handle complex routing scenarios. The key is to understand these concepts thoroughly and apply them appropriately based on your specific use case.
