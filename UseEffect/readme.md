# useEffect and API Calls — The Simple Version

## What is useEffect?

Think about opening Instagram. First the app screen shows up, then the posts load in a second later.

`useEffect` is that "then load the posts" part. It means: once this component is on screen, go do something — like asking an API for data.

## Why not just call the API directly in the component?

A component re-runs every time something on screen changes. If you put `axios.get(...)` straight in there with nothing around it, it would fire off request after request, non-stop, forever.

`useEffect` fixes that. It tells React: "only run this once — not every single time."

## The Basic Shape

```jsx
useEffect(() => {
  // code to run
}, []);
```

Two parts:
- The function — the code you want to run
- `[]` at the end — tells React "just run this one time, right when the page loads"

That's it. Ignore anything more complicated for now — running once is 90% of what you need for a GET call.

---

## Calling an API — Step by Step

**Step 1 — install axios**
```
npm install axios
```

**Step 2 — import what you need**
```jsx
import { useState, useEffect } from "react";
import axios from "axios";
```

**Step 3 — make a place to store the data**
```jsx
const [posts, setPosts] = useState([]);
```

**Step 4 — call the API inside useEffect**
```jsx
useEffect(() => {
  axios.get("https://jsonplaceholder.typicode.com/posts")
    .then((response) => {
      setPosts(response.data);
    });
}, []);
```

**Step 5 — show it on screen**
```jsx
{posts.map((post) => (
  <p key={post.id}>{post.title}</p>
))}
```

---

## Full Example, All Together

```jsx
import { useState, useEffect } from "react";
import axios from "axios";

function Posts() {
  const [posts, setPosts] = useState([]);

  useEffect(() => {
    axios.get("https://jsonplaceholder.typicode.com/posts")
      .then((response) => {
        setPosts(response.data);
      });
  }, []);

  return (
    <div>
      {posts.map((post) => (
        <p key={post.id}>{post.title}</p>
      ))}
    </div>
  );
}

export default Posts;
```

Run it. That's the whole flow:

Component loads → useEffect fires once → axios asks the API for data → data comes back → `setPosts` saves it → the component shows it on screen.

---

## In One Line Each

- `useEffect` = "once this shows up, go do this"
- `axios.get(url)` = "go get the data from this address"
- `setPosts(response.data)` = "now save it, so the screen updates"
---
