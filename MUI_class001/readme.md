# Lecture 1 — Material UI Practical: Navbar and Hero Section
---

## Section 1 — Install

```bash
npm install @mui/material @emotion/react @emotion/styled
npm install @mui/icons-material
```

`@mui/material` = components. `@emotion/*` = styling engine MUI needs internally. `@mui/icons-material` = icons, separate package.

```bash
npm run dev
```

---

## Section 2 — Typography + Button

Type this in `App.jsx`:

```jsx
import { Typography, Button } from "@mui/material";

function App() {
  return (
    <div>
      <Typography variant="h4">Hello Material UI</Typography>
      <Button variant="contained">Get Started</Button>
    </div>
  );
}

export default App;
```

Run it. `variant` controls the look — try `h1`, `body1` on the Typography, and `outlined`, `text` on the Button. Same component, different style, that's the whole idea.

---

## Section 3 — The `sx` Prop

`sx` = styling, written as a JS object. Build it up:

```jsx
import { Box } from "@mui/material";

<Box sx={{ p: 4 }}>Hello</Box>
```

`p` = padding. 1 unit = `8px`. So `p: 4` = `32px`.

```jsx
<Box sx={{ p: 4, mt: 2 }}>Hello</Box>
```

`mt` = margin-top. Same pattern for `m`/`p` + `t`/`b`/`l`/`r`/`x`/`y`.

```jsx
<Box
  sx={{
    p: 4,
    mt: 2,
    backgroundColor: "primary.main",
    color: "white",
    borderRadius: 2,
  }}
>
  Hello
</Box>
```

`backgroundColor: "primary.main"` — MUI's theme color, no hex needed. `borderRadius` — bigger number, rounder corners.

```jsx
<Box sx={{ fontSize: 20, boxShadow: 3, textAlign: "center" }}>
  Hello
</Box>
```

`boxShadow` goes `0` (flat) to `24` (heavy).

**Practice:** make your own box, change padding, margin, background color, border radius, text align. Try it before checking mine:

```jsx
<Box
  sx={{
    p: 3,
    m: 2,
    backgroundColor: "secondary.main",
    color: "white",
    borderRadius: 3,
    textAlign: "center",
  }}
>
  This is my styled card
</Box>
```

---

## Section 4 — Container

```jsx
import { Container } from "@mui/material";

<Container maxWidth="lg">
  ...your content...
</Container>
```

Centers content, stops it stretching edge-to-edge on wide screens. `maxWidth`: `xs` to `xl`. We'll use `"lg"` everywhere today.

---

## Section 5 — Navbar

```
src/
  components/
    Navbar.jsx
```

**Step 1 — bar + logo**

```jsx
import { AppBar, Toolbar, Typography } from "@mui/material";

function Navbar() {
  return (
    <AppBar position="static">
      <Toolbar>
        <Typography variant="h6">MyBrand</Typography>
      </Toolbar>
    </AppBar>
  );
}

export default Navbar;
```

Add `<Navbar />` in `App.jsx`, run it.

**Step 2 — nav links**

```jsx
import { AppBar, Toolbar, Typography, Button, Box } from "@mui/material";

function Navbar() {
  return (
    <AppBar position="static">
      <Toolbar>
        <Typography variant="h6" sx={{ flexGrow: 1 }}>
          MyBrand
        </Typography>

        <Box>
          <Button color="inherit">Home</Button>
          <Button color="inherit">About</Button>
          <Button color="inherit">Services</Button>
          <Button color="inherit">Contact</Button>
        </Box>
      </Toolbar>
    </AppBar>
  );
}
```

`flexGrow: 1` on the logo pushes everything else to the right. Run it, then delete that line for a second to see why.

**Step 3 — CTA button**

```jsx
<Button variant="contained" sx={{ ml: 2 }}>
  Get Started
</Button>
```

Add right after the `Box`, inside `Toolbar`.

**Step 4 — wrap in Container**

```jsx
import { AppBar, Toolbar, Typography, Button, Box, Container } from "@mui/material";

function Navbar() {
  return (
    <AppBar position="static">
      <Container maxWidth="lg">
        <Toolbar>
          <Typography variant="h6" sx={{ flexGrow: 1 }}>
            MyBrand
          </Typography>

          <Box>
            <Button color="inherit">Home</Button>
            <Button color="inherit">About</Button>
            <Button color="inherit">Services</Button>
            <Button color="inherit">Contact</Button>
          </Box>

          <Button variant="contained" sx={{ ml: 2 }}>
            Get Started
          </Button>
        </Toolbar>
      </Container>
    </AppBar>
  );
}

export default Navbar;
```

Run it. Navbar shape done.

---

## Section 6 — Style the Navbar

```jsx
<AppBar position="static" sx={{ backgroundColor: "#1a1a2e", py: 1 }}>
```

```jsx
<Box sx={{ display: "flex", gap: 2 }}>
```

Hover on one button first:

```jsx
<Button
  color="inherit"
  sx={{ "&:hover": { backgroundColor: "rgba(255,255,255,0.1)" } }}
>
  Home
</Button>
```

Run it, hover over "Home", then copy that `sx` to the other three buttons.

CTA button:

```jsx
<Button
  variant="contained"
  sx={{
    ml: 2,
    borderRadius: 3,
    textTransform: "none",
    backgroundColor: "#e94560",
    "&:hover": { backgroundColor: "#c73650" },
  }}
>
  Get Started
</Button>
```

`textTransform: "none"` stops the all-caps default.

---

## Section 7 — Responsive Navbar

Breakpoints: `xs`, `sm`, `md`, `lg`, `xl` (phone → big monitor).

```jsx
<Box sx={{ display: { xs: "none", md: "flex" }, gap: 2 }}>
```

Hidden on small screens, flex row from `md` up. Resize your browser and watch it.

Mobile menu icon:

```jsx
import MenuIcon from "@mui/icons-material/Menu";
import { IconButton } from "@mui/material";

<IconButton color="inherit" sx={{ display: { xs: "flex", md: "none" } }}>
  <MenuIcon />
</IconButton>
```

Opposite rule — shows on `xs`, hides on `md` up.

---

## Section 8 — Hero: Skeleton First

```
src/
  components/
    Hero.jsx
```

```jsx
import { Container, Box } from "@mui/material";
import Grid from "@mui/material/Grid";

function Hero() {
  return (
    <Box sx={{ py: 10 }}>
      <Container maxWidth="lg">
        <Grid container spacing={4} alignItems="center">
          <Grid size={{ xs: 12, md: 6 }} sx={{ backgroundColor: "#eee", height: 200 }} />
          <Grid size={{ xs: 12, md: 6 }} sx={{ backgroundColor: "#ccc", height: 200 }} />
        </Grid>
      </Container>
    </Box>
  );
}

export default Hero;
```

Add `<Hero />` in `App.jsx`, run it, resize the browser. Two boxes stack on mobile, sit side by side on desktop.

`size={{ xs: 12, md: 6 }}` — full width on small screens, half width (`6` of `12`) from `md` up. No `item` prop in current MUI, `Grid` inside `container` is already treated as an item.

---

## Section 9 — Hero: Left Side (Text)

Replace the first gray box:

```jsx
<Grid size={{ xs: 12, md: 6 }}>
  <Typography variant="overline" color="primary">
    Build Something Amazing
  </Typography>

  <Typography variant="h2" sx={{ fontWeight: 700, mt: 1 }}>
    Create Better Digital Experiences
  </Typography>

  <Typography variant="body1" sx={{ mt: 2, color: "text.secondary" }}>
    Build modern websites using React and Material UI.
  </Typography>
</Grid>
```

Import `Typography`. Run it.

---

## Section 10 — Buttons with Stack

Add after the description, still inside the left `Grid`:

```jsx
<Stack direction="row" spacing={2} sx={{ mt: 4 }}>
  <Button variant="contained">Get Started</Button>
  <Button variant="outlined">Learn More</Button>
</Stack>
```

Import `Stack`, `Button`. `direction="row"` = side by side, `spacing={2}` = gap between them.

Responsive version:

```jsx
<Stack direction={{ xs: "column", sm: "row" }} spacing={2} sx={{ mt: 4 }}>
```

Column on phone, row from `sm` up.

---

## Section 11 — Hero: Right Side (Image)

Replace the second gray box:

```jsx
<Grid size={{ xs: 12, md: 6 }}>
  <Box
    component="img"
    src="https://picsum.photos/600/450"
    alt="Landing page hero illustration"
    sx={{
      width: "100%",
      maxWidth: 500,
      borderRadius: 3,
      display: "block",
      mx: "auto",
    }}
  />
</Grid>
```

`component="img"` — `Box` renders as a real `<img>` tag instead of a `div`, still keeps `sx`.

---

## Section 12 — Combine

```
src/
  components/
    Navbar.jsx
    Hero.jsx
  App.jsx
```

```jsx
import Navbar from "./components/Navbar";
import Hero from "./components/Hero";

function App() {
  return (
    <div>
      <Navbar />
      <Hero />
    </div>
  );
}

export default App;
```

Run it. Navbar + Hero, done, responsive.

---
