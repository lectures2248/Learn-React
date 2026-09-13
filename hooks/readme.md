# React Hooks — Practical Lecture

In the previous lectures, we already covered:

- `useState()`
- `useEffect()`

In this lecture, we will work with other important React Hooks.

The focus of this lecture is **not** memorizing hooks. The focus is:

- What problem does this hook solve?
- Where would we use it in a real application?
- How do we actually implement it?

**Hooks covered in this lecture:**

1. `useContext()`
2. `useRef()`
3. `useMemo()`
4. `useCallback()`
5. `useReducer()`
6. `useId()`
7. `useLayoutEffect()`
8. `useTransition()`
9. `useDeferredValue()`

---

## 1. `useContext()`

Let's start with a common problem.

Suppose the logged-in user's information is stored in the top-level `App` component, but we need that data inside `Navbar`, `Dashboard`, and `Profile`.

One way is to pass the user as a prop from `App` → `Dashboard` → `Profile`. But when the application becomes large, passing the same data through many components becomes difficult. This is called **prop drilling**.

`useContext` gives us a simple way to share data between components.

### Example 1: User Authentication

Imagine we have a dashboard application. After login, we have:

- `name`
- `email`
- `role`

We want these values available in different components.

Create a file: `UserContext.jsx`

```jsx
import { createContext, useContext } from "react";

export const UserContext = createContext();

export function UserProvider({ children }) {

    const user = {
        name: "John",
        email: "john@gmail.com",
        role: "Admin"
    };

    return (
        <UserContext.Provider value={user}>
            {children}
        </UserContext.Provider>
    );
}
```

Now wrap our application. `main.jsx`

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import { UserProvider } from "./UserContext";

ReactDOM.createRoot(document.getElementById("root")).render(
    <UserProvider>
        <App />
    </UserProvider>
);
```

Now any component inside `UserProvider` can access the user. `Navbar.jsx`

```jsx
import { useContext } from "react";
import { UserContext } from "./UserContext";

function Navbar() {

    const user = useContext(UserContext);

    return (
        <nav>
            Welcome, {user.name}
        </nav>
    );
}

export default Navbar;
```

The important line is:

```jsx
const user = useContext(UserContext);
```

We are getting the value directly from the context. We do not need to pass `user` as a prop from `App` to `Navbar`.

### Example 2: Theme Management

Another common use case is application theme, for example:

- Light Mode
- Dark Mode

Create: `ThemeContext.jsx`

```jsx
import { createContext, useState } from "react";

export const ThemeContext = createContext();

export function ThemeProvider({ children }) {

    const [darkMode, setDarkMode] = useState(false);

    return (
        <ThemeContext.Provider
            value={{
                darkMode,
                setDarkMode
            }}
        >
            {children}
        </ThemeContext.Provider>
    );
}
```

Wrap the application:

```jsx
<ThemeProvider>
    <App />
</ThemeProvider>
```

Now create a button: `ThemeButton.jsx`

```jsx
import { useContext } from "react";
import { ThemeContext } from "./ThemeContext";

function ThemeButton() {

    const {
        darkMode,
        setDarkMode
    } = useContext(ThemeContext);

    return (
        <button
            onClick={() => setDarkMode(!darkMode)}
        >
            {darkMode ? "Light Mode" : "Dark Mode"}
        </button>
    );
}

export default ThemeButton;
```

The same context can be accessed by:

- `Navbar`
- `Sidebar`
- `Settings`
- `Dashboard`
- `Footer`

This is why Context becomes useful in larger applications.

### Example 3: Shopping Cart

Consider an e-commerce application. The cart may be needed in:

- `Navbar`
- `ProductCard`
- `ProductPage`
- `CartPage`
- `CheckoutPage`

Instead of passing cart data through many components, we can keep it inside a `CartContext`.

```jsx
import { createContext } from "react";

export const CartContext = createContext();
```

Provider:

```jsx
function CartProvider({ children }) {

    const cart = [];

    return (
        <CartContext.Provider value={{ cart }}>
            {children}
        </CartContext.Provider>
    );
}
```

Then inside `Cart.jsx`:

```jsx
import { useContext } from "react";
import { CartContext } from "./CartContext";

function Cart() {

    const { cart } = useContext(CartContext);

    return (
        <div>
            Cart Items: {cart.length}
        </div>
    );
}
```

This is a very common real-world use of Context.

---

## 2. `useRef()`

`useRef` is useful when we need to keep a value between renders without causing a new render. It is also commonly used to directly access a DOM element.

The basic syntax is:

```jsx
const myRef = useRef();
```

The actual value is stored in `myRef.current`.

Let's use it in real examples.

### Example 1: Automatically Focus an Input

Suppose we have a login page. When the page opens, we want the email field to receive focus.

```jsx
import { useEffect, useRef } from "react";

function Login() {

    const emailRef = useRef();

    useEffect(() => {
        emailRef.current.focus();
    }, []);

    return (
        <div>
            <h2>Login</h2>

            <input
                ref={emailRef}
                type="email"
                placeholder="Enter email"
            />

            <input
                type="password"
                placeholder="Enter password"
            />
        </div>
    );
}

export default Login;
```

Here, `ref={emailRef}` connects the input with our ref. Then `emailRef.current.focus();` directly focuses the input.

### Example 2: Focus Search Box

Consider an admin dashboard. We have a search button. When the user clicks the button, we want to focus the search input.

```jsx
import { useRef } from "react";

function SearchBox() {

    const searchRef = useRef();

    function focusSearch() {
        searchRef.current.focus();
    }

    return (
        <div>
            <input
                ref={searchRef}
                placeholder="Search students..."
            />

            <button onClick={focusSearch}>
                Search
            </button>
        </div>
    );
}

export default SearchBox;
```

This pattern is useful in:

- Search forms
- Login forms
- Registration forms
- Admin panels
- Modal forms

### Example 3: Store Timer ID

Suppose we create a timer. We need to store the interval ID so we can stop it later.

```jsx
import { useRef } from "react";

function Timer() {

    const timerRef = useRef();

    function startTimer() {
        timerRef.current = setInterval(() => {
            console.log("Timer running");
        }, 1000);
    }

    function stopTimer() {
        clearInterval(timerRef.current);
    }

    return (
        <div>
            <button onClick={startTimer}>
                Start
            </button>

            <button onClick={stopTimer}>
                Stop
            </button>
        </div>
    );
}

export default Timer;
```

We do not need a re-render when the interval ID changes. That is why `useRef` is suitable here.

---

## 3. `useMemo()`

Now let's look at a performance problem.

Imagine an e-commerce application with thousands of products. We want to filter products by category. Filtering a small array is not a big problem, but when the calculation becomes expensive, we do not want to perform the same calculation again and again when it is not necessary.

This is where `useMemo` can help.

`useMemo` stores the result of a calculation.

Basic syntax:

```jsx
const result = useMemo(() => {
    return someCalculation();
}, [dependencies]);
```

The important thing to remember: **`useMemo` is about a VALUE, not a function.**

### Example 1: Filter Products

```jsx
const products = [
    { id: 1, name: "Laptop", category: "Electronics" },
    { id: 2, name: "Chair", category: "Furniture" },
    { id: 3, name: "Phone", category: "Electronics" }
];
```

We want to display products according to the selected category.

```jsx
import { useMemo } from "react";

const filteredProducts = useMemo(() => {
    return products.filter(product => {
        return product.category === selectedCategory;
    });
}, [products, selectedCategory]);
```

Now React can reuse the previous result when the dependencies have not changed.

### Example 2: Calculate Cart Total

```jsx
const cart = [
    { name: "Laptop", price: 1000, quantity: 2 },
    { name: "Mouse", price: 50, quantity: 3 }
];
```

We need the total.

```jsx
const total = useMemo(() => {
    return cart.reduce((sum, item) => {
        return sum + item.price * item.quantity;
    }, 0);
}, [cart]);
```

Then:

```jsx
<h2>Total: ${total}</h2>
```

Whenever `cart` changes, the total is recalculated. If `cart` does not change, React can reuse the previous result.

### Example 3: Search Large Student List

Suppose an admin dashboard contains thousands of students.

```jsx
const filteredStudents = useMemo(() => {
    return students.filter(student =>
        student.name
            .toLowerCase()
            .includes(search.toLowerCase())
    );
}, [students, search]);
```

Now the filtering calculation depends on `students` and `search`. If neither changes, there is no reason to perform the filtering calculation again.

> **IMPORTANT:** Do not use `useMemo` for every calculation. For example:
> ```jsx
> const total = price * quantity;
> ```
> There is no need for `useMemo` here. Use it when a calculation is expensive enough that avoiding repeated work is useful.

---

## 4. `useCallback()`

`useCallback` is closely related to `useMemo`. But instead of remembering a calculated value, `useCallback` remembers a function reference.

Easy way to remember:

- `useMemo` → remembers a **value**
- `useCallback` → remembers a **function**

This becomes especially useful when passing functions to child components.

### Example 1: Product Card

Suppose `Products` renders a child component `ProductCard`.

Parent:

```jsx
function Products() {

    const addToCart = useCallback((product) => {
        console.log("Added to cart:", product);
    }, []);

    return (
        <ProductCard
            onAddToCart={addToCart}
        />
    );
}
```

Child:

```jsx
import React from "react";

const ProductCard = React.memo(function ProductCard({ onAddToCart }) {

    console.log("ProductCard rendered");

    return (
        <button
            onClick={() => onAddToCart("Laptop")}
        >
            Add to Cart
        </button>
    );
});
```

**Why `useCallback`?** If the child is memoized using `React.memo`, maintaining the same function reference can help prevent unnecessary child renders.

### Example 2: Delete User

Admin dashboard:

```jsx
function Users() {

    const deleteUser = useCallback((id) => {
        console.log("Deleting user:", id);
    }, []);

    return (
        <UserList
            onDelete={deleteUser}
        />
    );
}
```

`UserList`:

```jsx
function UserList({ onDelete }) {

    return (
        <button onClick={() => onDelete(10)}>
            Delete
        </button>
    );
}
```

This pattern is common when the parent provides action functions to reusable child components.

### Example 3: Add Product to Cart

A real product component may receive: `product`, `onAdd`, `onRemove`.

Parent:

```jsx
const handleAddToCart = useCallback((product) => {
    console.log("Adding product:", product);
}, []);
```

Then:

```jsx
<ProductCard
    product={product}
    onAdd={handleAddToCart}
/>
```

Again, `useCallback` is mainly useful when function identity matters, especially with memoized child components.

---

