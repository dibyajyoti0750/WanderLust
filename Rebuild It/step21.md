# **Setting Authorization for Listings**

## **1. Need for Authorization in Listings**

By default, any logged-in user can edit or delete any listing. We need to restrict these actions so that only the listing's owner can modify or delete it.

---

## **2. Updating the Update Route for Authorization**

### **Current Issue**:

- The existing update route directly calls `findByIdAndUpdate`, allowing any logged-in user to update a listing, even if they don’t own it.

### **Solution**:

- First, fetch the listing using `findById` to check ownership.
- If the user is not the owner, deny access and redirect with an error message.
- If the user is the owner, proceed with updating the listing.

### **Updated Route:**

```javascript
router.put(
  "/:id",
  isLoggedIn,
  validateListing,
  wrapAsync(async (req, res) => {
    let { id } = req.params;
    let listing = await Listing.findById(id);

    if (!listing.owner.equals(res.locals.currUser._id)) {
      req.flash("error", "You don't have permission to edit");
      return res.redirect(`/listings/${id}`);
    }

    await Listing.findByIdAndUpdate(id, { ...req.body.listing });
    req.flash("success", "Listing Updated!");
    res.redirect(`/listings/${id}`);
  })
);
```

---

## **3. Avoiding Repetitive Code Using Middleware**

Rather than manually checking ownership in multiple routes, we create a middleware function.

### **Why Middleware?**

- The authorization check must be implemented for multiple routes (edit, delete, etc.).
- Instead of repeating the logic in each route, we create a reusable function in `middleware.js`.

### **Middleware for Checking Ownership (`isOwner`):**

```javascript
module.exports.isOwner = async (req, res, next) => {
  let { id } = req.params;
  let listing = await Listing.findById(id);

  if (!listing.owner.equals(res.locals.currUser._id)) {
    req.flash("error", "You are not the owner of this listing");
    return res.redirect(`/listings/${id}`);
  }

  next();
};
```

### **Require Listing Model in `middleware.js`:**

Since `isOwner` needs access to the `Listing` model, we must require it at the top:

```javascript
const Listing = require("./models/listing.js");
```

---

## **4. Updating Routes to Use Middleware**

Now, instead of manually checking ownership in each route, we just add `isOwner` middleware:

### **Updated Routes in `routes/listings.js`**

```javascript
const { isLoggedIn, isOwner } = require("../middleware.js");

// Update Route
router.put(
  "/:id",
  isLoggedIn,
  isOwner,
  validateListing,
  wrapAsync(async (req, res) => {
    let { id } = req.params;
    await Listing.findByIdAndUpdate(id, { ...req.body.listing });
    req.flash("success", "Listing Updated!");
    res.redirect(`/listings/${id}`);
  })
);

// Edit Route
router.get(
  "/:id/edit",
  isLoggedIn,
  isOwner,
...);

// Delete Route
router.delete(
  "/:id",
  isLoggedIn,
  isOwner,
...);
```

---

## **5. Moving `validateListing` to Middleware**

`validateListing` checks if the submitted listing data follows the correct schema. Instead of defining it in `routes/listings.js`, we move it to `middleware.js`:

### **Middleware for Validating Listings (`validateListing`):**

```javascript
module.exports.validateListing = (req, res, next) => {
  let { error } = listingSchema.validate(req.body);

  if (error) {
    let errMsg = error.details.map((el) => el.message).join(",");
    throw new ExpressError(400, errMsg);
  } else {
    next();
  }
};
```

### **Require Dependencies in `middleware.js`:**

```javascript
const ExpressError = require("./utils/ExpressError.js");
const { listingSchema } = require("./schema.js");
```

### **Update `routes/listing.js` to Use Middleware:**

Remove the `ExpressError` and `listingSchema` lines, and require `validateListing` from `middleware.js`.

```javascript
const { isLoggedIn, isOwner, validateListing } = require("../middleware.js");
```

---

## **6. Moving `validateReview` to Middleware**

Similar to `validateListing`, we move `validateReview` from `routes/review.js` to `middleware.js`:

### **Middleware for Validating Reviews (`validateReview`):**

```javascript
module.exports.validateReview = (req, res, next) => {
  let { error } = reviewSchema.validate(req.body);

  if (error) {
    let errMsg = error.details.map((el) => el.message).join(",");
    throw new ExpressError(400, errMsg);
  } else {
    next();
  }
};
```

### **Require Dependencies in `middleware.js`:**

```javascript
const { listingSchema, reviewSchema } = require("./schema.js");
```

### **Update `routes/review.js` to Use Middleware:**

Remove the `ExpressError` and `reviewSchema` lines, and require `validateReview` instead:

```javascript
const { validateReview } = require("../middleware.js");
```

---

## **Key Takeaways**

✅ Implemented ownership checks before modifying listings using `isOwner` middleware.  
✅ Moved validation logic (`validateListing`, `validateReview`) into `middleware.js` for reusability.  
✅ Updated routes to use middleware functions instead of repeating code.  
✅ Ensured all necessary models and schemas are correctly required in respective files.

---
