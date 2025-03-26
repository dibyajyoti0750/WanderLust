# **User Authentication (Signup & Login)**

## **1. Signup User**

### **Routes**

- **GET /signup** → Renders the signup form.
- **POST /signup** → Registers the user in the database.

---

### **Step 1: Creating `user.js` in the `routes` directory**

We need to create a separate route file for user authentication:

```js
const express = require("express");
const router = express.Router();

module.exports = router;
```

---

### **Step 2: Requiring `user.js` in `app.js`**

In `app.js`, we import the newly created `user.js` and also rename `listing.js` and `review.js` for clarity:

```js
const listingRouter = require("./routes/listing.js");
const reviewRouter = require("./routes/review.js");
const userRouter = require("./routes/user.js");

app.use("/listings", listingRouter);
app.use("/listings/:id/reviews", reviewRouter);
app.use("/", userRouter);
```

---

### **Step 3: Creating the Signup Form Route (`GET /signup`)**

In `user.js`, we define a route that renders a signup form:

```js
router.get("/signup", (req, res) => {
  res.render("users/signup.ejs");
});
```

---

### **Step 4: Creating `signup.ejs`**

We create a `users` folder inside `views` and add `signup.ejs`:

```html
<% layout("/layouts/boilerplate") %>
<div class="row mt-3">
  <h1 class="col-6 offset-3">SignUp on WanderLust</h1>
  <div class="col-6 offset-3">
    <form action="/signup" method="post" class="needs-validation" novalidate>
      <div class="mb-3">
        <label for="username" class="form-label">Username</label>
        <input
          name="username"
          id="username"
          type="text"
          class="form-control"
          required
        />
        <div class="valid-feedback">Looks good!</div>
      </div>

      <div class="mb-3">
        <label for="email" class="form-label">Email</label>
        <input
          name="email"
          id="email"
          type="email"
          class="form-control"
          required
        />
      </div>

      <div class="mb-3">
        <label for="password" class="form-label">Password</label>
        <input
          name="password"
          id="password"
          type="password"
          class="form-control"
          required
        />
      </div>

      <button class="btn btn-success">SignUp</button>
    </form>
  </div>
</div>
```

Now, when we visit `/signup`, this form will be rendered.

---

### **Step 5: Implementing Signup Logic (`POST /signup`)**

Once the user submits the form, we need to store their details in the database. In `user.js`, we extract `username`, `email`, and `password` from `req.body`:

```js
const User = require("../models/user.js");

router.post("/signup", async (req, res) => {
  let { username, email, password } = req.body;
  const newUser = new User({ email, username });
  const registeredUser = await User.register(newUser, password);
  console.log(registeredUser);
  req.flash("success", "Welcome to WanderLust!");
  res.redirect("/listings");
});
```

- We create a new `User` instance.
- `User.register(newUser, password)` hashes the password and stores the user in the database.
- A success message is flashed, and the user is redirected.

---

### **Step 6: Improving Error Handling**

If a username is already taken, an error occurs. Instead of letting the server crash, we use `try-catch` to handle it:

```js
const wrapAsync = require("../utils/wrapAsync.js");

router.post(
  "/signup",
  wrapAsync(async (req, res) => {
    try {
      let { username, email, password } = req.body;
      const newUser = new User({ email, username });
      const registeredUser = await User.register(newUser, password);
      console.log(registeredUser);
      req.flash("success", "Welcome to WanderLust!");
      res.redirect("/listings");
    } catch (e) {
      req.flash("error", e.message);
      res.redirect("/signup");
    }
  })
);
```

- **wrapAsync** is used to handle asynchronous errors.
- **try-catch** prevents the server from crashing by catching errors like duplicate usernames and displaying them to the user.

---

## **2. Login User**

### **Routes**

- **GET /login** → Renders the login form.
- **POST /login** → Authenticates the user.

---

### **Step 1: Creating the Login Form Route (`GET /login`)**

In `user.js`, we define a route that renders a login form:

```js
router.get("/login", (req, res) => {
  res.render("users/login.ejs");
});
```

---

### **Step 2: Creating `login.ejs`**

Inside `views/users`, we create `login.ejs`:

```html
<% layout("/layouts/boilerplate") %>
<div class="row mt-3">
  <h1 class="col-6 offset-3">Login</h1>
  <div class="col-6 offset-3">
    <form action="/login" method="post" class="needs-validation" novalidate>
      <div class="mb-3">
        <label for="username" class="form-label">Username</label>
        <input
          name="username"
          id="username"
          type="text"
          class="form-control"
          required
        />
      </div>

      <div class="mb-3">
        <label for="password" class="form-label">Password</label>
        <input
          name="password"
          id="password"
          type="password"
          class="form-control"
          required
        />
      </div>

      <button class="btn btn-success">Login</button>
    </form>
  </div>
</div>
```

This form allows users to enter their credentials.

---

### **Step 3: Implementing Login Logic (`POST /login`)**

In `user.js`, we define the POST route:

```js
const passport = require("passport");

router.post(
  "/login",
  passport.authenticate("local", {
    failureRedirect: "/login",
    failureFlash: true,
  }),
  async (req, res) => {
    req.flash("success", "Welcome back to WanderLust!");
    res.redirect("/listings");
  }
);
```

#### **How It Works:**

- We use `passport.authenticate("local")` to verify user credentials.
- If authentication fails, `failureRedirect` sends the user back to `/login`.
- `failureFlash: true` shows an error message if login fails.
- If successful, a success message is flashed, and the user is redirected.

---

#### **Homework:**

Go through the documentation of Passport, Passport-Local, and Passport-Local-Mongoose.
