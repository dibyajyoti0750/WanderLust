## **Validations for Reviews**

Validations are performed in two steps:

1. **Client-Side Validation** – Ensuring form fields have valid input before submission.
2. **Server-Side Validation** – Ensuring no invalid data reaches the database, even if bypassed from the client.

---

## **Client-Side Validation**

We add client-side validation inside **`show.ejs`** to prevent users from submitting empty or incorrect reviews.

### **Requiring Necessary Fields**

- The **rating** field (range input) does not need explicit validation because it always has a default minimum value.
- The **textarea (comment)** field must be required to ensure users provide feedback.

### **Adding Bootstrap Validation**

1. Add `novalidate` and `class="needs-validation"` to the form.

   ```html
   <form
     method="post"
     action="/listings/<%= listing.id %>/reviews"
     novalidate
     class="needs-validation"
   ></form>
   ```

2. Since we already included Bootstrap validation logic inside `script.js`, this will automatically apply to all forms using `needs-validation`.

3. Add custom error messages for better user experience:

   ```html
   <textarea
     name="review[comment]"
     id="comment"
     cols="30"
     rows="5"
     class="form-control"
     required
   ></textarea>
   <div class="invalid-feedback">Please add some comments for the review</div>
   ```

### **Why Client-Side Validation?**

- Prevents normal users from submitting empty reviews.
- Enhances user experience with real-time feedback.

---

## **Server-Side Validation**

Even with client-side validation, a user can bypass it using tools like **Hoppscotch/Postman** to send invalid requests.

To prevent this, we implement **server-side validation** using `Joi`.

### **1. Creating a Joi Schema**

Define the schema inside **`schema.js`**:

```js
module.exports.reviewSchema = Joi.object({
  review: Joi.object({
    rating: Joi.number().required().min(1).max(5),
    comment: Joi.string().required(),
  }).required(),
});
```

This schema ensures:

- `rating` is a required number between 1 and 5.
- `comment` is a required string.

---

### **2. Validating Requests in `app.js`**

#### **Import the Schema**

Modify `app.js` to include `reviewSchema`:

```js
const { listingSchema, reviewSchema } = require("./schema.js");
```

#### **Create a Middleware for Validation**

Define a `validateReview` function:

```js
const validateReview = (req, res, next) => {
  let { error } = reviewSchema.validate(req.body);
  if (error) {
    let errMsg = error.details.map((el) => el.message).join(",");
    throw new ExpressError(400, errMsg);
  } else {
    next();
  }
};
```

- This function checks if the review data matches the `reviewSchema`.
- If validation fails, an error message is generated and thrown.

---

### **3. Applying Validation to the Reviews Route**

Modify the **POST route for reviews** inside `app.js`:

```js
app.post("/listings/:id/reviews", validateReview, async (req, res) => {
```

- `validateReview` is used as middleware before handling the request.
- If validation fails, the request is stopped before it reaches the database.

To improve error handling, we use `wrapAsync`:

```js
app.post(
  "/listings/:id/reviews",
  validateReview,
  wrapAsync(async (req, res) => {
```

- `wrapAsync` ensures unhandled errors are passed to the central error handler.

---

## **Clearing Reviews Collection**

Before displaying reviews, it's useful to clear any existing data:

```js
db.reviews.deleteMany({});
```

This removes all existing reviews from the database.

---

## **Summary of What We Implemented**

1. **Client-side validation** using Bootstrap
2. **Server-side validation** using `Joi`
3. **Middleware for validation** in `app.js`
4. **Applied validation in the POST route**
5. **Ensured error handling with `wrapAsync`**
6. **Cleared the reviews collection for a fresh start**

This ensures reviews are validated both on the front-end and back-end, preventing invalid data from entering the database.

---
