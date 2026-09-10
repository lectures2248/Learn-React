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

## 5. `useReducer()`

`useReducer` is useful when state logic becomes more complicated.

Consider a login form. We may have:

- `username`
- `password`
- `loading`
- `error`
- `success`

Managing every change separately can become difficult. `useReducer` lets us organize the state logic around actions.

Basic structure:

```jsx
const [state, dispatch] =
    useReducer(reducer, initialState);
```

The reducer receives current **state** and **action**, and returns the new **state**.

### Example 1: Counter

```jsx
import { useReducer } from "react";

const initialState = {
    count: 0
};

function reducer(state, action) {

    switch (action.type) {

        case "increment":
            return {
                count: state.count + 1
            };

        case "decrement":
            return {
                count: state.count - 1
            };

        default:
            return state;
    }
}

function Counter() {

    const [state, dispatch] =
        useReducer(reducer, initialState);

    return (
        <div>
            <h1>{state.count}</h1>

            <button
                onClick={() => dispatch({ type: "increment" })}
            >
                +
            </button>

            <button
                onClick={() => dispatch({ type: "decrement" })}
            >
                -
            </button>
        </div>
    );
}

export default Counter;
```

When the `+` button is clicked:

```jsx
dispatch({
    type: "increment"
});
```

The reducer receives this action, then `state.count + 1` is returned.

### Example 2: Login Form

Now let's use `useReducer` for something closer to a real application.

Initial state:

```jsx
const initialState = {
    email: "",
    password: "",
    loading: false,
    error: ""
};
```

Reducer:

```jsx
function reducer(state, action) {

    switch (action.type) {

        case "SET_EMAIL":
            return {
                ...state,
                email: action.payload
            };

        case "SET_PASSWORD":
            return {
                ...state,
                password: action.payload
            };

        case "LOGIN_START":
            return {
                ...state,
                loading: true,
                error: ""
            };

        case "LOGIN_ERROR":
            return {
                ...state,
                loading: false,
                error: action.payload
            };

        case "LOGIN_SUCCESS":
            return {
                ...state,
                loading: false,
                error: ""
            };

        default:
            return state;
    }
}
```

Inside component:

```jsx
const [state, dispatch] =
    useReducer(reducer, initialState);
```

Email input:

```jsx
<input
    value={state.email}
    onChange={(e) =>
        dispatch({
            type: "SET_EMAIL",
            payload: e.target.value
        })
    }
/>
```

Password:

```jsx
<input
    type="password"
    value={state.password}
    onChange={(e) =>
        dispatch({
            type: "SET_PASSWORD",
            payload: e.target.value
        })
    }
/>
```

When login starts:

```jsx
dispatch({ type: "LOGIN_START" });
```

If login fails:

```jsx
dispatch({
    type: "LOGIN_ERROR",
    payload: "Invalid email or password"
});
```

This gives us a clean structure for complicated forms.

### Example 3: Shopping Cart

This is one of the best practical examples of `useReducer`. Cart can have many actions:

- `ADD_ITEM`
- `REMOVE_ITEM`
- `INCREASE_QUANTITY`
- `DECREASE_QUANTITY`
- `CLEAR_CART`

Reducer:

```jsx
function cartReducer(state, action) {

    switch (action.type) {

        case "ADD_ITEM":
            return [
                ...state,
                action.payload
            ];

        case "REMOVE_ITEM":
            return state.filter(
                item => item.id !== action.payload
            );

        case "CLEAR_CART":
            return [];

        default:
            return state;
    }
}
```

Add product:

```jsx
dispatch({
    type: "ADD_ITEM",
    payload: product
});
```

Remove product:

```jsx
dispatch({
    type: "REMOVE_ITEM",
    payload: product.id
});
```

Clear cart:

```jsx
dispatch({ type: "CLEAR_CART" });
```

This type of structure is much easier to manage when the application has many state-changing actions.

---

## 6. `useId()`

`useId` is mainly useful when creating unique IDs for HTML elements. This becomes particularly useful in reusable form components — for example connecting a `label` to the correct `input`.

### Example 1: Login Input

```jsx
import { useId } from "react";

function EmailInput() {

    const emailId = useId();

    return (
        <div>
            <label htmlFor={emailId}>
                Email
            </label>

            <input
                id={emailId}
                type="email"
            />
        </div>
    );
}
```

The same ID connects `label` and `input`.

### Example 2: Reusable Input Component

This is more practical.

```jsx
function Input({ label, type = "text" }) {

    const id = useId();

    return (
        <div>
            <label htmlFor={id}>
                {label}
            </label>

            <input
                id={id}
                type={type}
            />
        </div>
    );
}
```

Now we can reuse it:

```jsx
<Input label="Name" />
<Input label="Email" type="email" />
<Input label="Password" type="password" />
```

Each component instance gets its own unique ID.

### Example 3: Multiple Form Fields

```jsx
function RegisterForm() {

    const usernameId = useId();
    const emailId = useId();

    return (
        <form>
            <label htmlFor={usernameId}>
                Username
            </label>
            <input id={usernameId} />

            <label htmlFor={emailId}>
                Email
            </label>
            <input
                id={emailId}
                type="email"
            />
        </form>
    );
}
```

> **Important:** `useId` is for UI IDs. Do not use it for database IDs, product IDs, user IDs, or order IDs. For those, the ID should normally come from your backend/database or another appropriate ID-generation strategy.

---

## 7. `useLayoutEffect()`

`useLayoutEffect` is similar to `useEffect`, but it runs at a different point in the browser rendering process.

The main practical reason to use it is when we need to measure or adjust the DOM **before** the browser paints the updated screen.

Typical situations:

- Measure an element
- Get width or height
- Calculate position
- Position a tooltip
- Adjust scrolling

Most of the time, `useEffect` is enough. Use `useLayoutEffect` when the timing really matters.

### Example 1: Get Element Width

```jsx
import { useLayoutEffect, useRef } from "react";

function Box() {

    const boxRef = useRef();

    useLayoutEffect(() => {
        const width =
            boxRef.current.getBoundingClientRect().width;

        console.log("Width:", width);
    }, []);

    return (
        <div ref={boxRef}>
            Hello React
        </div>
    );
}
```

Here we access the real DOM element and measure its width.

### Example 2: Tooltip Position

Suppose we have a `Button` and a `Tooltip`. When the tooltip opens, we need to calculate where it should appear.

We can get the button position:

```jsx
const buttonPosition =
    buttonRef.current.getBoundingClientRect();
```

Then we can use `top`, `left`, `width`, `height` to calculate the tooltip position.

A real UI component may look like:

```jsx
function TooltipButton() {

    const buttonRef = useRef();
    const tooltipRef = useRef();

    useLayoutEffect(() => {

        if (!tooltipRef.current) {
            return;
        }

        const button =
            buttonRef.current.getBoundingClientRect();

        tooltipRef.current.style.left = `${button.left}px`;
        tooltipRef.current.style.top = `${button.bottom}px`;
    });

    return (
        <>
            <button ref={buttonRef}>
                Help
            </button>

            <div ref={tooltipRef}>
                More information
            </div>
        </>
    );
}
```

The important part is not memorizing the code — it's understanding why layout measurement is needed.

### Example 3: Chat Scroll

Imagine a chat application. New messages are added. We want the chat container to move to the latest message.

```jsx
const chatRef = useRef();

useLayoutEffect(() => {
    chatRef.current.scrollTop =
        chatRef.current.scrollHeight;
}, [messages]);
```

This is a practical example where DOM layout information matters.

---

## 8. `useTransition()`

Now we move to performance-related hooks.

Imagine an application with thousands of products. The user selects a category, and React now has to render a very large list. If that update becomes expensive, we may want the important user interaction to remain responsive.

`useTransition` lets us mark an update as non-urgent.

Basic syntax:

```jsx
const [isPending, startTransition] = useTransition();
```

### Example 1: Large Product List

```jsx
function ProductPage() {

    const [category, setCategory] = useState("");

    const [isPending, startTransition] = useTransition();

    function changeCategory(value) {
        startTransition(() => {
            setCategory(value);
        });
    }

    return (
        <div>
            <button
                onClick={() => changeCategory("Electronics")}
            >
                Electronics
            </button>

            {isPending && (
                <p>Updating products...</p>
            )}
        </div>
    );
}
```

The update inside `startTransition` is treated as non-urgent.

### Example 2: Large Dashboard Tabs

Imagine a dashboard: `Overview`, `Analytics`, `Reports`, `Students`.

Analytics may contain many charts and large tables. We can mark changing the heavy tab as a transition.

```jsx
function Dashboard() {

    const [tab, setTab] = useState("overview");

    const [isPending, startTransition] = useTransition();

    function changeTab(newTab) {
        startTransition(() => {
            setTab(newTab);
        });
    }
}
```

The UI can show:

```jsx
{isPending && <p>Loading...</p>}
```

while React processes the transition.

### Example 3: Large Search Result UI

Suppose searching causes a very large result component to update. We can separate the urgent interaction from the expensive UI update.

```jsx
function SearchPage() {

    const [search, setSearch] = useState("");

    const [isPending, startTransition] = useTransition();

    function handleSearch(value) {

        setSearch(value);

        startTransition(() => {
            // update expensive result UI
        });
    }
}
```

The important idea: typing and direct user interaction should remain responsive, while expensive rendering can be treated as lower priority.

---

## 9. `useDeferredValue()`

`useDeferredValue` solves a similar type of performance problem, but the approach is different.

Suppose we have `search` and `large search results`. We want the input to update immediately, but the large results do not necessarily need to update at exactly the same time.

We can create a deferred version of the search value.

### Example 1: Student Search

Suppose we have 20,000 students.

```jsx
const [search, setSearch] = useState("");

const deferredSearch = useDeferredValue(search);
```

Input:

```jsx
<input
    value={search}
    onChange={(e) => setSearch(e.target.value)}
/>
```

Student list:

```jsx
<StudentList
    search={deferredSearch}
/>
```

Here, `search` is the immediate value, and `deferredSearch` is the value we allow React to update later when appropriate.

### Example 2: Product Search

```jsx
function ProductSearch() {

    const [search, setSearch] = useState("");

    const deferredSearch = useDeferredValue(search);

    return (
        <>
            <input
                value={search}
                onChange={(e) => setSearch(e.target.value)}
            />

            <ProductList
                search={deferredSearch}
            />
        </>
    );
}
```

The input stays connected to the immediate search value. The expensive product list uses the deferred value.

### Example 3: Large Table

Suppose an admin dashboard has a large table.

```jsx
const [search, setSearch] = useState("");

const deferredSearch = useDeferredValue(search);
```

Then:

```jsx
<StudentTable
    search={deferredSearch}
/>
```

This is useful when the result component is expensive to render.

---

## 10. `useTransition` vs `useDeferredValue`

These hooks solve related performance problems, but they are not the same thing.

**`useTransition`** — you control the state update.

```jsx
startTransition(() => {
    setTab("analytics");
});
```

You are saying: *"This state update is not urgent."*

**`useDeferredValue`** — you already have a value.

```jsx
const deferredSearch = useDeferredValue(search);
```

You are saying: *"Use a deferred version of this value."*

Simple way to remember:

| Hook | What it defers |
|---|---|
| `useTransition` | defers an **update** |
| `useDeferredValue` | defers a **value** |

---

## 11. Complete Practical Example

Now let's combine multiple hooks in one small application.

**Project:** Student Dashboard

**Features:**

- Search students
- Display students
- Add student
- Delete student
- Show logged-in user

We can use: `useContext`, `useRef`, `useMemo`, `useCallback`, `useReducer`.

### Step 1: User Context

`UserContext.jsx`

```jsx
import { createContext } from "react";

export const UserContext = createContext();

export function UserProvider({ children }) {

    const user = {
        name: "Admin",
        role: "Administrator"
    };

    return (
        <UserContext.Provider value={user}>
            {children}
        </UserContext.Provider>
    );
}
```

### Step 2: Student Reducer

```jsx
const initialStudents = [
    { id: 1, name: "Ali", course: "React" },
    { id: 2, name: "Ahmed", course: "Python" }
];

function studentReducer(state, action) {

    switch (action.type) {

        case "ADD_STUDENT":
            return [
                ...state,
                action.payload
            ];

        case "DELETE_STUDENT":
            return state.filter(
                student => student.id !== action.payload
            );

        default:
            return state;
    }
}
```

### Step 3: Student Dashboard

```jsx
function StudentDashboard() {

    const user = useContext(UserContext);

    const [students, dispatch] =
        useReducer(studentReducer, initialStudents);

    const searchRef = useRef();

    const [search, setSearch] = useState("");

    const filteredStudents = useMemo(() => {
        return students.filter(student =>
            student.name
                .toLowerCase()
                .includes(search.toLowerCase())
        );
    }, [students, search]);

    const deleteStudent = useCallback((id) => {
        dispatch({
            type: "DELETE_STUDENT",
            payload: id
        });
    }, []);

    return (
        <div>
            <h1>
                Welcome {user.name}
            </h1>

            <input
                ref={searchRef}
                value={search}
                onChange={(e) => setSearch(e.target.value)}
                placeholder="Search student"
            />

            {filteredStudents.map(student => (
                <div key={student.id}>
                    <h3>{student.name}</h3>
                    <p>{student.course}</p>

                    <button
                        onClick={() => deleteStudent(student.id)}
                    >
                        Delete
                    </button>
                </div>
            ))}
        </div>
    );
}
```

Now we are using several hooks for different jobs:

- **`useContext`** — Gives us the logged-in user.
- **`useReducer`** — Manages student data and student actions.
- **`useRef`** — Gives direct access to the search input.
- **`useMemo`** — Calculates the filtered student list.
- **`useCallback`** — Keeps the delete function stable.

This is the kind of situation where hooks become genuinely useful.

---

## 12. Which Hook Should I Use?

| Problem | Use |
|---|---|
| I need to share data between many components. | `useContext` |
| I need to access an input or DOM element. | `useRef` |
| I have an expensive calculation. | `useMemo` |
| I am passing a function to a memoized child component. | `useCallback` |
| My state logic has many actions. | `useReducer` |
| I need unique IDs for form elements. | `useId` |
| I need to measure or adjust DOM layout before paint. | `useLayoutEffect` |
| A UI update is expensive and should be treated as non-urgent. | `useTransition` |
| I want a deferred version of an existing value. | `useDeferredValue` |

---

## 13. Important Practical Rule

Do not add hooks to your project just because they exist.

For example, if you have:

```jsx
const total = price * quantity;
```

Do not write:

```jsx
const total = useMemo(() => {
    return price * quantity;
}, [price, quantity]);
```

There is no practical reason for this simple calculation.

Similarly:

- Do not use `useCallback` for every function.
- Do not use `useReducer` for a simple piece of state.
- Do not use Context for every variable in the application.

First identify the problem. Then choose the hook that actually solves that problem.

---
