## **Restructuring Listings and Reviews in Express.js**

### **1. Setting Up the Routes Folder**

To organize and clean up the `app.js` file, we first need to **create a `routes` folder** inside the `MajorProject` directory. Inside this folder, we will separate different route functionalities into different files.

---

## **Restructuring Listings**

### **Step 1: Create a `listing.js` File**

Inside the `routes` folder, create a new file called `listing.js` and define the router:

```js
const express = require("express");
const router = express.Router();
```

### **Step 2: Move Listing Routes from `app.js`**

- Cut all the `/listings` routes from `app.js` and paste them into `listing.js`.
- Then, replace all instances of `app` with `router`.

### **Step 3: Export the Router**

At the end of `listing.js`, add:

```js
module.exports = router;
```

### **Step 4: Connect `listing.js` to `app.js`**

In `app.js`, import the newly created `listing.js`:

```js
const listings = require("./routes/listing.js");
```

Then, register the route:

```js
app.use("/listings", listings);
```

### **Step 5: Fix the Server Crash**

After moving the routes, the server will crash because:

1. `listing.js` is using middleware functions that were originally in `app.js`.
2. It requires certain dependencies that are not yet imported.

### **Step 6: Remove Common `/listings` Parts from `listing.js`**

Since `app.use("/listings", listings);` is now in `app.js`, we remove the `/listings` prefix from each route in `listing.js`:

```js
// Index Route
router.get("/", ...);

// New Route
router.get("/new", ...);

// Create Route
router.post("/", ...);

// Show Route
router.get("/:id", ...);

// Edit Route
router.get("/:id/edit", ...);

// Update Route
router.put("/:id", ...);

// Delete Route
router.delete("/:id", ...);
```

### **Step 7: Import Missing Dependencies**

Since `listing.js` is inside the `routes` folder, we need to adjust relative paths while importing required modules:

```js
const wrapAsync = require("../utils/wrapAsync.js");
const ExpressError = require("../utils/ExpressError.js");
const { listingSchema } = require("../schema.js");
const Listing = require("../models/listing.js");
```

### **Step 8: Move `validateListing` to `listing.js`**

Cut the `validateListing` middleware function from `app.js` and paste it inside `listing.js`.

### **Final Result**

- The server now works properly.
- `app.js` is much cleaner because all listing-related logic is now inside `routes/listing.js`.
- `app.js` only contains:

```js
app.use("/listings", listings);
```

---

## **Restructuring Reviews**

### **Step 1: Create `review.js`**

Inside the `routes` folder, create a new file called `review.js` and define the router:

```js
const express = require("express");
const router = express.Router();
```

### **Step 2: Move Review Routes from `app.js`**

- Cut all `/listings/:id/reviews` routes from `app.js` and paste them into `review.js`.
- Replace `app` with `router`.

Since all review routes share the common path `/listings/:id/reviews`, we remove it:

```js
// Post Review Route
router.post("/", ...);

//Delete Review Route
router.delete("/:reviewId", ...)
```

### **Step 3: Export the Router**

At the end of `review.js`, add:

```js
module.exports = router;
```

### **Step 4: Connect `review.js` to `app.js`**

In `app.js`, import the newly created `review.js`:

```js
const reviews = require("./routes/review.js");
```

Then, register the route:

```js
app.use("/listings/:id/reviews", reviews);
```

### **Step 5: Import Missing Dependencies**

Just like we did for `listing.js`, import the required dependencies in `review.js`:

```js
const wrapAsync = require("../utils/wrapAsync.js");
const ExpressError = require("../utils/ExpressError.js");
const { reviewSchema } = require("../schema.js");
const Review = require("../models/review.js");
const Listing = require("../models/listing.js"); // Needed for adding reviews
```

### **Step 6: Move `validateReview` to `review.js`**

Cut the `validateReview` middleware function from `app.js` and paste it inside `review.js`.

### **Step 7: Remove Unused Imports from `app.js`**

After moving routes, `app.js` will highlight some unused imports. Remove them:

```js
// const Listing = require("./models/listing.js");  ❌ Remove
// const wrapAsync = require("./utils/wrapAsync.js"); ❌ Remove
// const { listingSchema, reviewSchema } = require("./schema.js"); ❌ Remove
// const Review = require("./models/review.js"); ❌ Remove
```

---

## **Fixing the Review Route Bug**

Now, when trying to add a review, we get an error:

```
Cannot read properties of null (reading 'reviews')
```

This happens because:

- We try to **find a listing** using `findById(req.params.id)`.
- But `req.params.id` is **not available** inside `review.js` due to Express’s routing behavior.

### **Step 1: Understanding the Issue**

When Express routes the request to:

```js
app.use("/listings/:id/reviews", reviews);
```

- The `:id` parameter stays **inside `app.js`**.
- It **does not** automatically transfer to `review.js`.

### **Step 2: Using `mergeParams` to Fix It**

To pass the `id` parameter to `review.js`, modify the router initialization:

```js
const router = express.Router({ mergeParams: true });
```

### **Final Check**

Now, when we **post a review**, `req.params.id` is correctly passed to `review.js`, and everything works perfectly.

---

## **Final App.js Structure**

- `app.js` is now **clean and minimal**.
- It handles **major operations** like:
  - Connecting to **MongoDB**.
  - Starting the **server**.
  - Managing **error handling**.
- All **route logic** is now modularized in `routes/`.

---
