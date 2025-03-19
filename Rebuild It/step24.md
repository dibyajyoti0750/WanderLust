## **MVC Implementation for Reviews and Users**

After implementing MVC for Listings, we now apply the same design pattern to **Reviews** and **Users** to make our codebase cleaner and more structured.

---

## **1. Implementing MVC for Reviews**

We begin by moving all review-related logic from `routes/review.js` to a dedicated `controllers/reviews.js` file.

### **Step 1: Create a Review Controller**

Create a new file:

```
controllers/reviews.js
```

---

### **Step 2: Move the "Create Review" Logic**

Previously, the logic for adding a review was inside `routes/reviews.js`. Now, we move it to `controllers/reviews.js`:

```javascript
module.exports.createReview = async (req, res) => {
  let listing = await Listing.findById(req.params.id);
  let newReview = new Review(req.body.review);
  newReview.author = req.user._id;
  listing.reviews.push(newReview);

  await newReview.save();
  await listing.save();
  req.flash("success", "New Review Created!");
  res.redirect(`/listings/${listing._id}`);
};
```

- This function:
  - Finds the listing by ID.
  - Creates a new review and assigns it to the current user.
  - Saves the review and updates the listing.
  - Shows a success message and redirects to the listing page.

#### **Step 3: Update the Route to Use the Controller**

In `routes/reviews.js`, require the new controller:

```javascript
const reviewController = require("../controllers/reviews.js");
```

Modify the **POST** route to use the controller:

```javascript
router.post(
  "/",
  isLoggedIn,
  validateReview,
  wrapAsync(reviewController.createReview)
);
```

---

### **Step 4: Move the "Delete Review" Logic**

Move the delete review logic from `routes/review.js` to `controllers/reviews.js`:

```javascript
module.exports.destroyReview = async (req, res) => {
  let { id, reviewId } = req.params;

  await Listing.findByIdAndUpdate(id, { $pull: { reviews: reviewId } });
  await Review.findByIdAndDelete(reviewId);

  req.flash("success", "Review Deleted!");
  res.redirect(`/listings/${id}`);
};
```

- This function:
  - Finds the listing and removes the review reference.
  - Deletes the actual review from the database.
  - Shows a success message and redirects.

#### **Step 5: Update the Delete Route**

Modify the **DELETE** route in `routes/review.js`:

```javascript
router.delete(
  "/:reviewId",
  isLoggedIn,
  isReviewAuthor,
  wrapAsync(reviewController.destroyReview)
);
```

#### **Step 6: Require Necessary Models in the Controller**

Since the controller uses `Listing` and `Review`, we add:

```javascript
const Listing = require("../models/listing");
const Review = require("../models/review");
```

---

## **2. Implementing MVC for Users**

We now structure user-related logic into a `controllers/users.js` file.

### **Step 1: Create a User Controller**

Create a new file:

```
controllers/users.js
```

---

### **Step 2: Move "Render Signup Form" Logic**

Move the logic for rendering the signup form from `routes/user.js` to `controllers/users.js`:

```javascript
module.exports.renderSignupForm = (req, res) => {
  res.render("users/signup.ejs");
};
```

#### **Step 3: Update Signup Route**

In `routes/user.js`, require the controller:

```javascript
const userController = require("../controllers/users.js");
```

Modify the **GET** route:

```javascript
router.get("/signup", userController.renderSignupForm);
```

---

### **Step 4: Move "Signup" Logic**

Move the signup logic from `routes/user.js` to `controllers/users.js`:

```javascript
module.exports.signup = async (req, res, next) => {
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
};
```

- This function:
  - Extracts user details from the request body.
  - Registers the user using `passport-local-mongoose`.
  - Logs the user in automatically after registration.
  - Redirects to the listings page with a success message.

#### **Step 5: Update the Signup Route**

Modify the **POST** route in `routes/user.js`:

```javascript
router.post("/signup", wrapAsync(userController.signup));
```

---

### **Step 6: Move "Render Login Form" Logic**

Move the logic for rendering the login form:

```javascript
module.exports.renderLoginForm = (req, res) => {
  res.render("users/login.ejs");
};
```

#### **Step 7: Update the Login Route**

Modify the **GET** route in `routes/user.js`:

```javascript
router.get("/login", userController.renderLoginForm);
```

---

### **Step 8: Move "Login" Logic**

Move the login logic from `routes/user.js` to `controllers/users.js`:

```javascript
module.exports.login = async (req, res) => {
  req.flash("success", "Welcome back to WanderLust!");
  let redirectUrl = res.locals.redirectUrl || "/listings";
  res.redirect(redirectUrl);
};
```

- This function:
  - Displays a success message after successful login.
  - Redirects to the last visited page (if saved) or `/listings`.

#### **Step 9: Update the Login Route**

Modify the **POST** route in `routes/user.js`:

```javascript
router.post(
  "/login",
  saveRedirectUrl,
  passport.authenticate("local", {
    failureRedirect: "/login",
    failureFlash: true,
  }),
  userController.login
);
```

> **Note:** Passport handles actual authentication. This function runs **after** login is successful.

---

### **Step 10: Move "Logout" Logic**

Move the logout logic from `routes/user.js` to `controllers/users.js`:

```javascript
module.exports.logout = (req, res, next) => {
  req.logOut((err) => {
    if (err) {
      return next(err);
    }
    req.flash("success", "You are logged out!");
    res.redirect("/listings");
  });
};
```

#### **Step 11: Update the Logout Route**

Modify the **GET** route in `routes/user.js`:

```javascript
router.get("/logout", userController.logout);
```

---

### **Step 12: Require User Model in Controller**

Since we use the `User` model in `controllers/users.js`, we need:

```javascript
const User = require("../models/user");
```

---

## **Final Thoughts**

- Now, `routes/reviews.js` and `routes/user.js` **only reference controller functions**, making them clean and readable.
- All business logic has been shifted to `controllers/reviews.js` and `controllers/users.js`, following **MVC principles**.

### **Project Documentation Tip**

When adding this project to your resume, mention **MVC framework** in the project description:

✅ **Example:**

> "Developed a full-stack web application using the MVC design pattern, implementing models, views, and controllers for user authentication, reviews, and listings management."

This highlights that you structured the project properly, making it clear that **you understand full-stack development beyond basic CRUD operations**. 🚀
