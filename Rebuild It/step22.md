# **Setting Authorization for Reviews**

To properly secure and manage reviews, we must ensure that:

1. Each review is associated with an author.
2. Only logged-in users can create reviews.
3. Only the author of a review can delete it.
4. Reviews should display the author’s name.

---

## **1. Associating Reviews with Authors**

Each review must be linked to the user who created it. Modify the `reviewSchema` in **models/review.js** by adding an `author` field:

```js
author: {
    type: Schema.Types.ObjectId,
    ref: "User",
},
```

This ensures every review has an associated author.

---

## **2. Restricting Review Creation to Logged-In Users**

A user must be logged in to submit a review. We'll ensure this in **show.ejs** where the review form is displayed.

Modify the template to only show the review form when a user is logged in:

```html
<% if(currUser) { %>
<h4>Leave a review</h4>
<form
...
</form>
<hr />
<% } %>
```

This prevents non-logged-in users from seeing the review submission form.

### **Backend Protection**

Even though we’ve hidden the form for non-logged-in users, they could still send a request via tools like Postman or Hoppscotch. We must protect the backend route as well.

Modify **routes/review.js** to include authentication middleware:

```js
router.post(
  "/",
  isLoggedIn, // Ensures only logged-in users can create reviews
  validateReview,
  wrapAsync(async (req, res) => {
    let listing = await Listing.findById(req.params.id);
    let newReview = new Review(req.body.review);
    newReview.author = req.user._id; // Storing the review author
    ...
  })
);
```

- The `isLoggedIn` middleware ensures that only authenticated users can post reviews.
- `newReview.author = req.user._id` ensures that the logged-in user becomes the author of the review.

---

## **3. Displaying Author Names in Reviews**

We want each review to display the name of the user who created it. For this, we need **nested population** in Mongoose.

Modify the **Show Route** in **routes/listing.js**:

```js
const listing = await Listing.findById(id)
  .populate({
    path: "reviews",
    populate: {
      path: "author",
    },
  })
  .populate("owner");
```

This ensures:

- `listing` fetches its reviews.
- Each `review` fetches its `author`.

Now, modify **show.ejs** to display the review author’s name:

```html
<h5 class="card-title">@<%= review.author.username %></h5>
```

This ensures the username of the review author appears next to their review.

---

## **4. Restricting Review Deletion to Authors**

Currently, any user can delete any review. We need to implement **authorization** so only the review’s author can delete it.

### **Middleware for Authorization**

Create a new middleware function in **middleware.js**:

```js
module.exports.isReviewAuthor = async (req, res, next) => {
  let { id, reviewId } = req.params;
  let review = await Review.findById(reviewId);
  if (!review.author.equals(res.locals.currUser._id)) {
    req.flash("error", "You are not the author of this review");
    return res.redirect(`/listings/${id}`);
  }
  next();
};
```

This checks if the logged-in user is the author of the review before allowing deletion.

Now, just like we required the Listing model, we will also require the Review model:

```js
const Review = require("./models/review.js");
```

### **Using the Middleware**

Modify the **Delete Review Route** in **routes/review.js**:

```js
const {
  validateReview,
  isLoggedIn,
  isReviewAuthor,
} = require("../middleware.js");

// Delete Review Route
router.delete(
  "/:reviewId",
  isLoggedIn,
  isReviewAuthor, // Only allow authors to delete their reviews
...);
```

- `isLoggedIn` ensures the user is authenticated.
- `isReviewAuthor` ensures only the review’s author can delete it.

---

### **Summary**

✔ Each review is linked to an author.  
✔ Only logged-in users can post reviews.  
✔ The author's name appears alongside the review.  
✔ Only the review's author can delete it.
