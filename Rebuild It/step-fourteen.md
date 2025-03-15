# **Implementing Flash Messages**

## **1. Installing and Setting Up Flash Messages**

To use flash messages in an Express application, we need to install the `connect-flash` package.

### **Installation**

Run the following command to install `connect-flash`:

```sh
npm i connect-flash
```

### **Require and Use Flash**

In your `app.js` or main server file, import `connect-flash` and use it after setting up sessions:

```js
const flash = require("connect-flash");

// Then, right after the line where we used session
app.use(flash());
```

⚠️ **Important:** Make sure `connect-flash` is used **before** defining routes because we will use flash messages inside route handlers.

---

## **2. Using Flash Messages in Routes**

We will now integrate flash messages when creating, updating, and deleting listings or reviews.

### **Create Route (Flash Message on Successful Creation)**

Inside `routes/listing.js`:

```js
router.post(
  "/",
  validateListing,
  wrapAsync(async (req, res, next) => {
    const newListing = new Listing(req.body.listing);
    await newListing.save();

    req.flash("success", "New Listing Created!");
    res.redirect("/listings");
  })
);
```

Here, after saving the listing, we store a flash message with the key `"success"`, which will be displayed on the redirected page.

### **Middleware to Pass Flash Messages to Views**

To make flash messages available in all templates, add this middleware in `app.js`, after session configuration:

```js
app.get("/", (req, res) => {
  res.send("Hi, I am root");
});

app.use(session(sessionOptions));
app.use(flash());

app.use((req, res, next) => {
  res.locals.success = req.flash("success");
  next();
});
```

This allows us to access `success` messages inside EJS files.

---

## **3. Displaying Flash Messages in Views**

Now, we display flash messages in the frontend using Bootstrap alerts.

### **Adding Flash Messages in `index.ejs`**

Instead of adding flash messages in each individual template, we create a reusable `flash.ejs` file.

1. **Create a new file `views/includes/flash.ejs`** and add:

   ```html
   <% if(success && success.length) { %>
   <div
     class="alert alert-success alert-dismissible fade show col-6 offset-3"
     role="alert"
   >
     <%= success %>
     <button
       type="button"
       class="btn-close"
       data-bs-dismiss="alert"
       aria-label="Close"
     ></button>
   </div>
   <% } %>
   ```

2. **Include it in `boilerplate.ejs`**:
   ```html
   <div class="container">
     <%- include("../includes/flash.ejs") %> <%- body %>
   </div>
   ```

This ensures all pages automatically display flash messages.

---

## **4. Adding Flash Messages for Other Operations**

### **Delete Route (Flash Message on Deletion)**

```js
router.delete(
  "/:id",
  wrapAsync(async (req, res) => {
    let { id } = req.params;
    await Listing.findByIdAndDelete(id);

    req.flash("success", "Listing Deleted!");
    res.redirect("/listings");
  })
);
```

### **Update Route (Flash Message on Update)**

```js
router.put(
  "/:id",
  validateListing,
  wrapAsync(async (req, res) => {
    let { id } = req.params;
    await Listing.findByIdAndUpdate(id, { ...req.body.listing });

    req.flash("success", "Listing Updated!");
    res.redirect(`/listings/${id}`);
  })
);
```

### **Adding Flash Messages for Reviews**

- **Create Review:**

  ```js
  router.post(
    "/",
    validateReview,
    wrapAsync(async (req, res) => {
      let listing = await Listing.findById(req.params.id);
      let newReview = new Review(req.body.review);

      listing.reviews.push(newReview);

      await newReview.save();
      await listing.save();

      req.flash("success", "New Review Created!");
      res.redirect(`/listings/${listing._id}`);
    })
  );
  ```

- **Delete Review:**

  ```js
  router.delete(
    "/:reviewId",
    wrapAsync(async (req, res) => {
      let { id, reviewId } = req.params;

      await Listing.findByIdAndUpdate(id, { $pull: { reviews: reviewId } });
      await Review.findByIdAndDelete(reviewId);

      req.flash("success", "Review Deleted!");
      res.redirect(`/listings/${id}`);
    })
  );
  ```

---

## **5. Implementing Error Flash Messages**

We also want to handle errors using flash messages.

### **Error When Listing Does Not Exist**

Modify the **Show Route** inside `routes/listing.js`:

```js
router.get(
  "/:id",
  wrapAsync(async (req, res) => {
    let { id } = req.params;
    const listing = await Listing.findById(id).populate("reviews");

    if (!listing) {
      req.flash("error", "Listing you requested for does not exist!");
      return res.redirect("/listings");
    }

    res.render("listings/show.ejs", { listing });
  })
);
```

Here, if a user tries to access a deleted listing, they will be redirected with an error message.

### **Pass Error Messages to Views**

Modify the middleware inside `app.js` to include error messages:

```js
app.use((req, res, next) => {
  res.locals.success = req.flash("success");
  res.locals.error = req.flash("error");
  next();
});
```

### **Displaying Error Flash Messages in `flash.ejs`**

Update `flash.ejs` to include an error message:

```html
<% if(error && error.length) { %>
<div
  class="alert alert-danger alert-dismissible fade show col-6 offset-3"
  role="alert"
>
  <%= error %>
  <button
    type="button"
    class="btn-close"
    data-bs-dismiss="alert"
    aria-label="Close"
  ></button>
</div>
<% } %>
```

### **Error When Editing a Deleted Listing**

Modify the **Edit Route** in `routes/listing.js`:

```js
router.get(
  "/:id/edit",
  wrapAsync(async (req, res) => {
    let { id } = req.params;
    const listing = await Listing.findById(id);

    if (!listing) {
      req.flash("error", "Listing you requested for does not exist!");
      return res.redirect("/listings");
    }

    res.render("listings/edit.ejs", { listing });
  })
);
```

---

## **6. Testing Flash Messages**

- Add a new listing → You should see a **success message**.
- Delete a listing → You should see a **success message**.
- Try accessing a deleted listing → You should see an **error message**.
- Edit a deleted listing → You should see an **error message**.
- Add or delete a review → You should see a **success message**.

---

## **Summary**

✔ Installed `connect-flash` and set it up with sessions.  
✔ Implemented success messages for creating, updating, and deleting listings and reviews.  
✔ Used Bootstrap alerts to style the messages.  
✔ Created a **flash.ejs** partial to display flash messages globally.  
✔ Implemented error messages when a listing is missing.  
✔ Successfully tested all flash messages.

This ensures a clean and user-friendly notification system in our application.

---
