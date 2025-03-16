## **Connecting Login Route**

Currently, any user can create a listing without authentication. We need to ensure that users must be logged in before they can create a listing.

### **Checking User Authentication**

To verify whether a user is logged in, we can either write a custom function from scratch or use **Passport.js**, which provides a built-in method:

```js
req.isAuthenticated();
```

This method checks if the user’s information is stored in the session.

### **Restricting Access to Create Listing Form**

Modify the `new` route inside `routes/listings.js`:

```js
router.get("/new", (req, res) => {
  if (!req.isAuthenticated()) {
    req.flash("error", "You must be logged in to create a listing!");
    return res.redirect("/login");
  }
  res.render("listings/new.ejs");
});
```

Now, if a user is not authenticated, they will be redirected to the login page instead of being allowed to create a listing.

To debug, we can log the `req.user`:

```js
console.log(req.user);
```

This shows that once a user logs in, their information is stored in `req.user`, which `isAuthenticated()` uses to determine authentication status.

### **Restricting Access to Edit, Update, and Delete Routes**

Instead of copying the same logic into every route, we extract it into a **middleware function**.

1. **Create a new file `middleware.js`**

```js
module.exports.isLoggedIn = (req, res, next) => {
  console.log(req.user);
  if (!req.isAuthenticated()) {
    req.flash("error", "You must be logged in!");
    return res.redirect("/login");
  }
  next();
};
```

2. **Import Middleware in `routes/listing.js`**

```js
const { isLoggedIn } = require("../middleware");
```

3. **Apply Middleware to Relevant Routes**

```js
router.get("/new", isLoggedIn, (req, res) => {
  res.render("listings/new.ejs");
});

router.get(
  "/:id/edit",
  isLoggedIn, {...});

router.put(
  "/:id",
  isLoggedIn, {...});

router.delete(
  "/:id",
  isLoggedIn, {...});

router.post(
  "/",
  isLoggedIn, {...});
```

Now, all these routes require authentication.

---

## **Logout Functionality**

A `GET` request to `/logout` should log out the user. Passport provides a built-in method for this.

### **Implementing Logout Route**

Modify `routes/user.js`:

```js
router.get("/logout", (req, res, next) => {
  req.logOut((err) => {
    if (err) {
      return next(err);
    }
    req.flash("success", "You are logged out!");
    res.redirect("/listings");
  });
});
```

- `req.logOut()` clears the user’s session.
- If an error occurs, it is passed to `next(err)`.
- If logout is successful, we flash a success message and redirect the user.

---

## **Adding Authentication Status to Navbar**

Now, we dynamically show/hide authentication-related buttons in the navbar.

### **Modify the collapse class in `navbar.ejs`**

```html
<div class="navbar-nav ms-auto">
  <% if (!currUser) { %>
  <a class="nav-link" href="/signup">Sign up</a>
  <a class="nav-link" href="/login">Log in</a>
  <% } %> <% if (currUser) { %>
  <a class="nav-link" href="/logout">Log out</a>
  <% } %>
</div>
```

- If `currUser` exists (i.e., the user is logged in), only the **Log out** button is shown.
- If `currUser` doesn’t exist, **Sign up** and **Log in** buttons are displayed.

### **Passing `currUser` to EJS Templates**

Modify `app.js`:

```js
app.use((req, res, next) => {
  res.locals.success = req.flash("success");
  res.locals.error = req.flash("error");
  res.locals.currUser = req.user;
  next();
});
```

- `req.user` (set by Passport) is stored in `res.locals.currUser`, making it accessible in all EJS templates.

---

## **Final Outcome**

- **Login Route Connected**: Users must be logged in to create, edit, update, or delete listings.
- **Middleware Implemented**: `isLoggedIn` middleware ensures authentication across routes.
- **Logout Functionality Added**: Users can log out via `/logout`, clearing their session.
- **Navbar Updated**: Shows authentication buttons dynamically based on login status.
