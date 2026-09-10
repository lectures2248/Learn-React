# React Practice Task — Users App

## What You Will Build

A small multi-page app that shows users from an API, lets you click on a user to see their details, and shows a search box to filter users.

You will use:

- React Router (multiple pages)
- `useState` and `useEffect`
- Axios (GET requests only)
- Material UI or Bootstrap (your choice)

**API you will use:** `https://jsonplaceholder.typicode.com`

---

## Page 1: Home Page (`/`)

- Fetch all users from `https://jsonplaceholder.typicode.com/users`
- Show each user as a card: **name**, **email**, **city**
- Show a "Loading..." message while the data is coming
- Clicking a user's card takes you to that user's detail page

---

## Page 2: User Detail Page (`/users/:id`)

- Get the `id` from the URL using `useParams()`
- Fetch that one user: `https://jsonplaceholder.typicode.com/users/:id`
- Show: name, email, phone, city, company name
- Add a button **"View Posts"** that goes to the posts page for this user

---

## Page 3: User Posts Page (`/users/:id/posts`)

- Fetch posts for this user: `https://jsonplaceholder.typicode.com/posts?userId=:id`
- Show a list of post titles
- Clicking a post shows its full body (you can just show/hide it, no need for a popup)

---

## Page 4: Search Page (`/search`)

- One input box
- User types a name
- Filter the users list (the one you already fetched) using `.filter()`
- Show only matching users
- This does NOT need a new API call — just filter data you already have

---

## Page 5: 404 Page

- If someone visits a route that doesn't exist, show "Page Not Found"

---

## Extra Requirements

- Add a Navbar with links to Home and Search, visible on every page
- Use `useState` for: loading, users list, search text
- Use `useEffect` for: fetching data when a page loads

---

- Share the repo link
- App should run with `npm install` and `npm run dev` (or `npm start`)
