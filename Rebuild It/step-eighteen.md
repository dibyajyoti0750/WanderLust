### **Automatic Login After Signup**

When a user signs up, they should be automatically logged in instead of manually logging in again. Passport provides a `req.login` method, which logs in the user after registration. This method requires two parameters:

1. The `registeredUser` object
2. A callback function to handle errors

#### **Implementation:**

```javascript
router.post(
  "/signup",
  wrapAsync(async (req, res, next) => {
    try {
      let { username, email, password } = req.body;
      const newUser = new User({ email, username });
      const registeredUser = await User.register(newUser, password);
      console.log(registeredUser);

      req.login(registeredUser, (err) => {
        if (err) {
          return next(err);
        }
        req.flash("success", "Welcome to WanderLust!");
        res.redirect("/listings");
      });
    } catch (e) {
      req.flash("error", e.message);
      res.redirect("/signup");
    }
  })
);
```

- `req.login(registeredUser, callback)`: Automatically logs in the newly registered user.
- If an error occurs during login, it is passed to the `next()` function for handling.
- If successful, the user is redirected to `/listings` with a success message.

---

### **Post-Login Redirect to the Intended Page**

Currently, after logging in, users are always redirected to `/listings`. However, if they were trying to access another page before logging in (e.g., adding a new listing), they should be redirected back to that specific page.

#### **Understanding Request Object**

- The `req` object contains `req.path` (relative path) and `req.originalUrl` (full URL).
- `req.originalUrl` is the URL that the user originally tried to access before being redirected to the login page.

#### **Storing the Original URL**

We only need to store `originalUrl` if the user is not already logged in. The best place to store this is in the session:

```javascript
module.exports.isLoggedIn = (req, res, next) => {
  if (!req.isAuthenticated()) {
    req.session.redirectUrl = req.originalUrl;
    ...
  }
};
```

- If the user is not authenticated, we store `req.originalUrl` in `req.session.redirectUrl`.

#### **Using the Stored URL After Login**

After logging in, within the POST `/login` route, we retrieve `req.session.redirectUrl` and redirect the user to that URL.

```javascript
async (req, res) => {
  req.flash("success", "Welcome back to WanderLust!");
  res.redirect(req.session.redirectUrl);
};
```

##### **Problem: Passport Automatically Resets `req.session`**

- Passport resets `req.session` upon authentication, which deletes `redirectUrl`.
- To prevent this, we store `redirectUrl` in `res.locals`, which is accessible across all routes but not affected by Passport.

#### **Solution: Saving Redirect URL in `res.locals`**

```javascript
module.exports.saveRedirectUrl = (req, res, next) => {
  if (req.session.redirectUrl) {
    res.locals.redirectUrl = req.session.redirectUrl;
  }
  next();
};
```

- This new middleware extracts `redirectUrl` from `req.session` and saves it in `res.locals`.

#### **Using `saveRedirectUrl` in the Login Route**

```javascript
router.post(
  "/login",
  saveRedirectUrl,
  ...);
```

- Before Passport authentication, `saveRedirectUrl` middleware saves `redirectUrl` into `res.locals`.

#### **Fixing "Page Not Found" Issue When Logging in from the Homepage**

- If a user logs in from the homepage, `req.session.redirectUrl` is **undefined**, causing an error.
- To fix this, provide a fallback to `/listings`:

```javascript
async (req, res) => {
  req.flash("success", "Welcome back to WanderLust!");
  let redirectUrl = res.locals.redirectUrl || "/listings";
  res.redirect(redirectUrl);
};
```

- If `redirectUrl` exists, the user is redirected there; otherwise, they are sent to `/listings`.

---

### **Summary of Fixes and Features Implemented**

✅ Automatic login after signup using `req.login()`.  
✅ Storing the originally requested URL in `req.session.redirectUrl`.  
✅ Redirecting users to their intended page after login.  
✅ Storing `redirectUrl` in `res.locals` to prevent Passport from deleting it.  
✅ Fixing the "Page Not Found" error by providing a fallback URL (`/listings`).
