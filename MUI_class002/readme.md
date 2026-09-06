# Lecture 2 — Material UI: Layout, Navigation, Feedback
---

# PART 1 — DRAWER

## Section 1 — Drawer Basics

```jsx
import { useState } from "react";
import { Drawer, Button } from "@mui/material";

function App() {
  const [open, setOpen] = useState(false);

  return (
    <div>
      <Button variant="contained" onClick={() => setOpen(true)}>
        Open Drawer
      </Button>

      <Drawer open={open} onClose={() => setOpen(false)}>
        <p>Content inside the Drawer</p>
      </Drawer>
    </div>
  );
}

export default App;
```

- `open` — shows/hides the panel
- `onClose` — fires on outside click or Escape. Skip it and the drawer never closes.
- `anchor="left" | "right" | "top" | "bottom"` — which side it slides from (default `left`)

---

## Section 2 — Drawer Width

```jsx
import { Box } from "@mui/material";

<Drawer anchor="right" open={open} onClose={() => setOpen(false)}>
  <Box sx={{ width: 260, p: 2 }}>
    <p>Menu</p>
  </Box>
</Drawer>
```

Drawer has no width of its own — wrap the content in a `Box` to fix it. Inside `sx`, a plain number = px.

---

## Section 3 — List (Menu Inside the Drawer)

```jsx
import {
  List,
  ListItem,
  ListItemButton,
  ListItemIcon,
  ListItemText,
} from "@mui/material";
import HomeIcon from "@mui/icons-material/Home";
import InfoIcon from "@mui/icons-material/Info";
import BuildIcon from "@mui/icons-material/Build";
import MailIcon from "@mui/icons-material/Mail";

const menuItems = [
  { text: "Home", icon: <HomeIcon /> },
  { text: "About", icon: <InfoIcon /> },
  { text: "Services", icon: <BuildIcon /> },
  { text: "Contact", icon: <MailIcon /> },
];

<List>
  {menuItems.map((item) => (
    <ListItem key={item.text} disablePadding>
      <ListItemButton>
        <ListItemIcon>{item.icon}</ListItemIcon>
        <ListItemText primary={item.text} />
      </ListItemButton>
    </ListItem>
  ))}
</List>
```

- `List` — wrapper, `ListItem` — one row, `ListItemButton` — makes it clickable/hoverable, `ListItemIcon`/`ListItemText` — icon and label
- `disablePadding` on `ListItem` because `ListItemButton` supplies its own padding
- Drive it from an array, not hand-written items — one line to add a new link

---

## Section 4 — Adding the Drawer to the Navbar

```jsx
// src/components/Navbar.jsx
import { useState } from "react";
import {
  AppBar, Toolbar, Typography, Button, Box, Container, IconButton,
  Drawer, List, ListItem, ListItemButton, ListItemIcon, ListItemText, Divider,
} from "@mui/material";
import MenuIcon from "@mui/icons-material/Menu";
import CloseIcon from "@mui/icons-material/Close";
import HomeIcon from "@mui/icons-material/Home";
import InfoIcon from "@mui/icons-material/Info";
import BuildIcon from "@mui/icons-material/Build";
import MailIcon from "@mui/icons-material/Mail";

const menuItems = [
  { text: "Home", icon: <HomeIcon /> },
  { text: "About", icon: <InfoIcon /> },
  { text: "Services", icon: <BuildIcon /> },
  { text: "Contact", icon: <MailIcon /> },
];

function Navbar() {
  const [open, setOpen] = useState(false);

  return (
    <>
      <AppBar position="static" sx={{ backgroundColor: "#1a1a2e", py: 1 }}>
        <Container maxWidth="lg">
          <Toolbar>
            <Typography variant="h6" sx={{ flexGrow: 1 }}>MyBrand</Typography>

            <Box sx={{ display: { xs: "none", md: "flex" }, gap: 2 }}>
              {menuItems.map((item) => (
                <Button key={item.text} color="inherit">{item.text}</Button>
              ))}
            </Box>

            <Button
              variant="contained"
              sx={{ ml: 2, borderRadius: 3, textTransform: "none",
                backgroundColor: "#e94560", display: { xs: "none", md: "flex" } }}
            >
              Get Started
            </Button>

            <IconButton
              color="inherit"
              onClick={() => setOpen(true)}
              sx={{ display: { xs: "flex", md: "none" } }}
            >
              <MenuIcon />
            </IconButton>
          </Toolbar>
        </Container>
      </AppBar>

      <Drawer anchor="right" open={open} onClose={() => setOpen(false)}>
        <Box sx={{ width: 260 }}>
          <Box sx={{ display: "flex", alignItems: "center", justifyContent: "space-between", p: 2 }}>
            <Typography variant="h6">MyBrand</Typography>
            <IconButton onClick={() => setOpen(false)}><CloseIcon /></IconButton>
          </Box>

          <Divider />

          <List>
            {menuItems.map((item) => (
              <ListItem key={item.text} disablePadding>
                <ListItemButton onClick={() => setOpen(false)}>
                  <ListItemIcon>{item.icon}</ListItemIcon>
                  <ListItemText primary={item.text} />
                </ListItemButton>
              </ListItem>
            ))}
          </List>
        </Box>
      </Drawer>
    </>
  );
}

export default Navbar;
```

Three things to notice:
- `<>...</>` — a Fragment, needed since `AppBar` and `Drawer` are siblings but need one parent, without adding an extra DOM tag
- `onClick={() => setOpen(false)}` on every link — closes the drawer after navigating
- `display: { xs: "none", md: "flex" }` on the CTA — desktop navbar only; mobile version lives in the drawer

---

# PART 2 — LAYOUT

## Section 5 — Spacing System

**1 = 8px.** `p: 1` → 8px, `p: 2` → 16px, `p: 3` → 24px, `p: 4` → 32px.

`m`/`p` = all sides, `mt/mb/ml/mr` = one side, `mx` = left+right, `my` = top+bottom. Same numbers apply to `Stack` and `Grid` `spacing`. Keeps spacing consistent instead of random `13px`, `17px`, `22px`.

## Section 6 — Box

`Box` = `div` with `sx`. Mainly used for flexbox:

```jsx
<Box sx={{ display: "flex", justifyContent: "space-between", alignItems: "center" }}>
  <Typography variant="h6">Products</Typography>
  <Button variant="contained">Add New</Button>
</Box>
```

This header pattern repeats on almost every page. Can also become any tag: `<Box component="img" ... />`.

## Section 7 — Stack

Box + flex + gap in one, for a single row or column.

```jsx
<Stack direction={{ xs: "column", sm: "row" }} spacing={2}>
  <Button variant="contained">One</Button>
  <Button variant="contained">Two</Button>
</Stack>
```

Default direction is `column`. Use `Box` instead when you need more control (justify, align, wrap, background).

## Section 8 — Paper

A raised surface — simple version of Card.

```jsx
<Paper elevation={3} sx={{ p: 3 }}>Content</Paper>
<Paper variant="outlined" sx={{ p: 3 }}>Border instead of shadow</Paper>
```

`elevation` 0–24. Paper follows the theme (adapts to dark mode); `Box` doesn't unless you remove hardcoded colors.

## Section 9 — Card

Paper + structure for image/text/buttons content:

```jsx
<Card sx={{ maxWidth: 320 }}>
  <CardMedia component="img" height="180" image="..." alt="Product" />
  <CardContent>
    <Typography variant="h6">Wireless Headphones</Typography>
    <Typography variant="h6" sx={{ color: "primary.main" }}>Rs 12,500</Typography>
  </CardContent>
  <CardActions>
    <Button size="small">Add to Cart</Button>
  </CardActions>
</Card>
```

`CardMedia` (image), `CardContent` (text, own padding), `CardActions` (buttons, own gap). Wrap in `CardActionArea` to make the whole card clickable with a ripple.

## Section 10 — Divider

```jsx
<Divider />
<Divider orientation="vertical" flexItem />
<Divider>OR</Divider>
```

The "OR" divider on login pages is built this way.

## Section 11 — Grid

12 columns per row.

```jsx
<Grid container spacing={3}>
  {products.map((p) => (
    <Grid key={p.id} size={{ xs: 12, sm: 6, md: 4 }}>
      <Card>...</Card>
    </Grid>
  ))}
</Grid>
```

`container` = parent, `size` = columns taken (`12` full, `6` half, `4` a third). Read `{ xs: 12, sm: 6, md: 4 }` as: full width on phone, half on tablet, a third on desktop. `spacing` only goes on the container. Newer MUI uses `size={{ xs: 12 }}`, not `item`/`xs={12}`.

Single row or column → `Stack`. Needs to wrap into rows and columns → `Grid`.

---

# PART 3 — NAVIGATION

## Section 12 — Tabs

```jsx
const [value, setValue] = useState(0);

<Tabs value={value} onChange={(e, newValue) => setValue(newValue)}>
  <Tab label="Description" />
  <Tab label="Reviews" />
</Tabs>

{value === 0 && <Typography>Product details go here.</Typography>}
{value === 1 && <Typography>Customer reviews go here.</Typography>}
```

`onChange` gets **two** args — use the second (the new index), not the event. Tabs are 0-indexed. Lots of tabs → add `variant="scrollable" scrollButtons`.

## Section 13 — Menu

```jsx
const [anchorEl, setAnchorEl] = useState(null);
const open = Boolean(anchorEl);

<Button onClick={(e) => setAnchorEl(e.currentTarget)}>Account</Button>

<Menu anchorEl={anchorEl} open={open} onClose={() => setAnchorEl(null)}>
  <MenuItem onClick={() => setAnchorEl(null)}>Profile</MenuItem>
  <MenuItem onClick={() => setAnchorEl(null)}>Logout</MenuItem>
</Menu>
```

`anchorEl` = the clicked element, stored in state; `null` means closed, so `open` is just `Boolean(anchorEl)` — one state, two jobs. Every `MenuItem` needs `setAnchorEl(null)` or the menu stays open.

## Section 14 — Breadcrumbs

```jsx
<Breadcrumbs>
  <Link underline="hover" color="inherit" href="#">Home</Link>
  <Typography color="text.primary">Headphones</Typography>
</Breadcrumbs>
```

Last item is `Typography`, not `Link` — it's the current page, so it shouldn't be clickable.

## Section 15 — Pagination

```jsx
const [page, setPage] = useState(1);

<Pagination count={10} page={page} onChange={(e, value) => setPage(value)} color="primary" />
```

`onChange`'s second argument again. Pages start at 1 (Tabs start at 0).

---

# PART 4 — FEEDBACK

## Section 16 — Alert

```jsx
<Alert severity="success">Order placed successfully</Alert>
<Alert severity="warning" onClose={() => setShowAlert(false)}>
  Only 2 items left in stock.
</Alert>
```

`severity` sets color and icon. `onClose` adds a close (cross) icon automatically.

## Section 17 — Snackbar

```jsx
<Snackbar
  open={open}
  autoHideDuration={3000}
  onClose={() => setOpen(false)}
  anchorOrigin={{ vertical: "bottom", horizontal: "center" }}
>
  <Alert severity="success" onClose={() => setOpen(false)}>Added to cart</Alert>
</Snackbar>
```

`Snackbar` = position + timer, `Alert` = the colored box inside it. Auto-closes after `autoHideDuration`.

## Section 18 — Dialog

```jsx
<Dialog open={open} onClose={() => setOpen(false)}>
  <DialogTitle>Delete this item?</DialogTitle>
  <DialogContent>
    <DialogContentText>This action can't be undone.</DialogContentText>
  </DialogContent>
  <DialogActions>
    <Button onClick={() => setOpen(false)}>Cancel</Button>
    <Button color="error" variant="contained" onClick={() => setOpen(false)}>Delete</Button>
  </DialogActions>
</Dialog>
```

Same shape as Card — Title, Content, Actions. `fullWidth` + `maxWidth="sm"` or `fullScreen` for mobile.

- Dialog — blocks the user, needs a response
- Drawer — side navigation/filters
- Snackbar — just a notification

## Section 19 — Progress

```jsx
<Button variant="contained" disabled={loading}>
  {loading ? <CircularProgress size={24} color="inherit" /> : "Save"}
</Button>
```

`size={24}` keeps the spinner button-sized; `color="inherit"` matches the button's color. `variant="determinate" value={70}` when you know the percentage.

## Section 20 — Skeleton

```jsx
{loading ? (
  <Skeleton variant="rectangular" height={180} />
) : (
  <CardMedia component="img" height="180" image={product.image} />
)}
```

Grey placeholder shaped like the real content — no layout jump when data arrives. Keep skeleton height equal to the real content's height.

---


