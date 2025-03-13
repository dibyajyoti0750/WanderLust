## **New Model: Reviews**

- We have introduced a `Review` model that will store three key pieces of information:

  - `comment` (Text for the review)
  - `rating` (A number between 1 to 5)
  - `createdAt` (Date of creation, stored but not used yet)

- The `createdAt` field is stored in the database but will not be used immediately. It will be utilized later when we work with dates.

- At this stage, reviews do **not** store information about who wrote them, since we haven't created a `User` model yet. This means:

  - Any user can add or delete reviews for any listing.

- **Removing Sample Listings**: Before proceeding, delete any sample listings from the "All Listings" page.

---

## **Step 1: Creating the Review Model**

Create a new file in the `models` folder:  
`models/review.js`

```javascript
const mongoose = require("mongoose");
const { Schema } = mongoose;

const reviewSchema = new Schema({
  comment: String,
  rating: {
    type: Number,
    min: 1,
    max: 5,
  },
  createdAt: {
    type: Date,
    default: Date.now(),
  },
});

module.exports = mongoose.model("Review", reviewSchema);
```

- The `Review` schema consists of:
  - `comment`: A string storing user feedback.
  - `rating`: A number restricted between 1 and 5.
  - `createdAt`: A timestamp that defaults to the current time when a review is created.

---

## **Step 2: Associating Reviews with Listings**

- **One-to-Many Relationship**:
  - Each listing can have multiple reviews.
  - Example: A hotel might have 100 reviews attached to it.
  - This is implemented using an array of `ObjectId` references in the `Listing` model.

Modify the `listingSchema` in `models/listing.js`
After country:

```javascript
  reviews: [
    {
      type: Schema.Types.ObjectId,
      ref: "Review",
    },
  ],
```

- The `reviews` field is an array of `ObjectId`s referencing `Review` documents.
- This ensures that reviews are tied to a specific listing.

---

## **Step 3: Creating the Review Form in UI**

- The review form is added **inside** the `show.ejs` page (single listing view).
- Placed **after** the `btns` div.

`views/listings/show.ejs`

```html
<div class="col-8 offset-3 mb-3">
  <hr />
  <h4>Leave a review</h4>
  <form method="post" action="/listings/<%= listing.id %>/reviews">
    <div class="mb-3 mt-3">
      <label for="rating" class="form-label">Rating</label>
      <input
        type="range"
        min="1"
        max="5"
        id="rating"
        name="review[rating]"
        class="form-range"
      />
    </div>

    <div class="mb-3 mt-3">
      <label for="comment" class="form-label">Comment</label>
      <textarea
        name="review[comment]"
        id="comment"
        cols="30"
        rows="5"
        class="form-control"
      ></textarea>
    </div>
    <button class="btn btn-outline-dark">Submit</button>
  </form>
</div>
```

- This form:
  - Accepts a `rating` (1-5) using a range slider.
  - Accepts a `comment` in a textarea.
  - Submits data via a `POST` request to `/listings/:id/reviews`.

---

## **Step 4: Handling Review Submission**

- We need to create a **route** to handle form submission and save reviews to the database.

### **Modify `app.js` to include the `Review` model**

`app.js`

```javascript
const Review = require("./models/review.js");
```

### **Create the Route for Submitting Reviews**

Add the following code **after** the delete route:

```javascript
// Post Route for Reviews
app.post("/listings/:id/reviews", async (req, res) => {
  let listing = await Listing.findById(req.params.id);
  let newReview = new Review(req.body.review);
  listing.reviews.push(newReview);
  await newReview.save();
  await listing.save();
  res.redirect(`/listings/${listing._id}`);
});
```

### **Why `/listings/:id/reviews` Instead of `/reviews`?**

- Reviews are always **linked to a listing**.
- We do **not** need separate index/show routes for reviews because:
  - They will always be displayed as part of a listing.
  - There's no need to access reviews independently.

---

## **Step 5: Verify Data in MongoDB**

### **Check if a review is added to a listing**

In MongoDB shell:

```bash
db.listings.find({title: "Cozy Beachfront Cottage"})
```

- You should see a `reviews` array inside the listing document containing an `ObjectId`.

### **Check all stored reviews**

```bash
db.reviews.find()
```

- This command will display all review documents, each containing:
  - `comment`
  - `rating`
  - `createdAt`
  - `_id`

---

## **Step 6: Next Steps**

1. **Validation**:
   - Currently, users can submit **empty** reviews.
   - We need to add **client-side and server-side validation**.
2. **Displaying Reviews on the Page**:
   - Right now, reviews are stored in the database but are not visible on the listing page.
   - We need to fetch and display them properly.

---

## **Summary**

### **What We Have Implemented So Far**

1. Created a `Review` model with `comment`, `rating`, and `createdAt`.
2. Linked `Review` to `Listing` using a **one-to-many relationship**.
3. Added a **review form** in the UI.
4. Created a **POST route** to handle review submissions.
5. Verified data is being stored in MongoDB.

### **What We Will Do Next**

⟶ Implement **validation** to prevent empty reviews.

⟶ Display stored reviews on the listing page.

---
