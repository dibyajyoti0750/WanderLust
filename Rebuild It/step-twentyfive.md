### **`router.route`**

#### **Overview**

`router.route` provides a functionality that allows us to group multiple HTTP methods (GET, POST, PUT, DELETE) under the same route path. This helps in reducing code duplication and making the code more readable and maintainable.

#### **Why Use `router.route`?**

- **Reduces Code Duplication** – No need to repeatedly write the same route path for different HTTP methods.
- **Improves Readability** – All related HTTP methods are defined in one place.
- **More Compact & Organized Code** – Helps keep the route definitions structured and neat.

---

### **Implementation**

#### **Basic Example**

If we have multiple HTTP methods (GET, POST) for the same path, we can use `router.route()` to group them together.

```javascript
router
  .route("/")
  .get(wrapAsync(listingController.index))
  .post(
    isLoggedIn,
    validateListing,
    wrapAsync(listingController.createListing)
  );
```

---

### **Handling `/:id` Routes**

```javascript
router
  .route("/:id")
  .get(wrapAsync(listingController.showListing))
  .put(
    isLoggedIn,
    isOwner,
    validateListing,
    wrapAsync(listingController.updateListing)
  )
  .delete(isLoggedIn, isOwner, wrapAsync(listingController.destroyListing));
```

---

### **User Authentication Routes**

#### **Signup Route**

```javascript
router
  .route("/signup")
  .get(userController.renderSignupForm)
  .post(wrapAsync(userController.signup));
```

- `GET /signup` – Renders the signup form.
- `POST /signup` – Handles form submission and user registration.

#### **Login Route**

```javascript
router
  .route("/login")
  .get(userController.renderLoginForm)
  .post(
    saveRedirectUrl,
    passport.authenticate("local", {
      failureRedirect: "/login",
      failureFlash: true,
    }),
    userController.login
  );
```

- `GET /login` – Renders the login form.
- `POST /login` –
  - `saveRedirectUrl` saves the URL to redirect the user after login.
  - `passport.authenticate("local", {...})` authenticates the user.
  - `userController.login` processes the login and manages the session.

---
