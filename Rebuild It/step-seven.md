## **1. Setting Up an Error View (error.ejs)**

In the `views/listings` folder, create an `error.ejs` file to display error messages using Bootstrap alerts.

### **Code for `error.ejs`**

```ejs
<% layout("/layouts/boilerplate") %>
<div class="row">
  <div class="alert alert-danger col-6 offset-3" role="alert">
    A simple danger alert—check it out!
  </div>
</div>
```

### **Middleware for Error Handling in `app.js`**

Modify the global error-handling middleware to render the `error.ejs` page:

```js
app.use((err, req, res, next) => {
  let { statusCode = 500, message = "Something went wrong!" } = err;
  res.render("listings/error.ejs");
});
```

At this point, the error alert will always display a default Bootstrap message. We can improve this by passing a custom error message.

### **Passing a Custom Error Message**

Modify the error-handling middleware to pass a message:

```js
res.status(statusCode).render("listings/error.ejs", { message });
```

### **Update `error.ejs` to Display Custom Messages**

```ejs
<% layout("/layouts/boilerplate") %>
<div class="row mt-3">
  <div class="alert alert-danger col-6 offset-3" role="alert">
    <p class="alert-heading"><%= message %></p>
  </div>
</div>
```

Now, error messages will be displayed in a better format.

---

## **2. Validations for Listing Schema**

The `listingSchema` should ensure that every listing contains the required fields: `title, description, image, price, location,` and `country`.

### **Current Validation in POST Route**

Initially, we only checked if `req.body.listing` exists:

```js
app.post(
  "/listings",
  wrapAsync(async (req, res, next) => {
    if (!req.body.listing) {
      throw new ExpressError(400, "Send valid data for listing");
    }
    const newListing = new Listing(req.body.listing);
    await newListing.save();
    res.redirect("/listings");
  })
);
```

#### **Issue with This Approach:**

- If some required fields are missing but `listing` exists, the listing will still be saved.
- For example, if `location` or `description` is missing, it won't throw an error, but an incomplete listing will be saved.

### **Solution: Manual Validation Using `if-else`**

```js
app.post(
  "/listings",
  wrapAsync(async (req, res, next) => {
    if (!req.body.listing) {
      throw new ExpressError(400, "Send valid data for listing");
    }

    const newListing = new Listing(req.body.listing);
    if (!newListing.description) {
      throw new ExpressError(400, "Description is missing!");
    }
    if (!newListing.title) {
      throw new ExpressError(400, "Title is missing!");
    }
    if (!newListing.location) {
      throw new ExpressError(400, "Location is missing!");
    }

    await newListing.save();
    res.redirect("/listings");
  })
);
```

#### **Problem with This Approach:**

- Requires writing multiple `if-else` statements.
- If the model changes, we have to update validations manually.
- Becomes tedious when handling multiple models.

---

## **3. Using Joi for Schema Validation**

Joi is a schema validation library that simplifies server-side validation.

### **Installing Joi**

```sh
npm i joi
```

### **Defining a Joi Schema (`schema.js`)**

Create a new file `schema.js`:

```js
const Joi = require("joi");

module.exports.listingSchema = Joi.object({
  listing: Joi.object({
    title: Joi.string().required(),
    description: Joi.string().required(),
    location: Joi.string().required(),
    country: Joi.string().required(),
    price: Joi.number().required().min(0),
    image: Joi.string().allow("", null),
  }).required(),
});
```

- `required()`: Ensures the field is provided.
- `Joi.number().min(0)`: Ensures the price is at least 0.
- `Joi.string().allow("", null)`: Allows an empty string or `null` value for the image.

### **Import the Schema in `app.js`**

```js
const { listingSchema } = require("./schema.js");
```

### **Using Joi in the POST Route**

Now, instead of writing multiple `if-else` statements, we can validate the request in a single step:

```js
app.post(
  "/listings",
  wrapAsync(async (req, res, next) => {
    let result = listingSchema.validate(req.body);
    console.log(result);
    if (result.error) {
      throw new ExpressError(400, result.error);
    }
    const newListing = new Listing(req.body.listing);
    await newListing.save();
    res.redirect("/listings");
  })
);
```

This method automatically checks each field and throws an error if validation fails.

---

## **4. Converting Validation into Middleware**

Instead of manually validating in every route, we can create a middleware function.

### **Creating the Validation Middleware**

```js
const validateListing = (req, res, next) => {
  let result = listingSchema.validate(req.body);
  console.log(result);
  if (result.error) {
    throw new ExpressError(400, result.error);
  } else {
    next();
  }
};
```

This middleware:

- Validates the `req.body` using Joi.
- Throws an error if validation fails.
- Calls `next()` if validation passes.

### **Using Middleware in Routes**

Modify the POST route:

```js
app.post(
  "/listings",
  validateListing,
  wrapAsync(async (req, res, next) => {
    const newListing = new Listing(req.body.listing);
    await newListing.save();
    res.redirect("/listings");
  })
);
```

Modify the UPDATE route:

```js
app.put(
  "/listings/:id",
  validateListing,
  wrapAsync(async (req, res) => {
    let { id } = req.params;
    await Listing.findByIdAndUpdate(id, { ...req.body.listing });
    res.redirect(`/listings/${id}`);
  })
);
```

### **Testing Error Handling**

We can test the middleware by sending invalid data and checking if errors are correctly handled.

---

## **5. Improving Error Messages**

Joi provides detailed error messages, which can be extracted and formatted.

### **Refining the Middleware**

```js
const validateListing = (req, res, next) => {
  let { error } = listingSchema.validate(req.body);
  if (error) {
    let errMsg = error.details.map((el) => el.message).join(", ");
    throw new ExpressError(400, errMsg);
  } else {
    next();
  }
};
```

### **How This Works:**

- `error.details` contains multiple error messages.
- `map(el => el.message)`: Extracts each error message.
- `.join(", ")`: Combines messages into a single string.

Now, when multiple fields are missing, Joi will return a single error message listing all issues.

---

## **Summary**

**1. Created an `error.ejs` page to display Bootstrap alerts.**
**2. Modified the global error-handling middleware.**
**3. Initially used manual `if-else` validation but found it inefficient.**
**4. Installed and configured Joi for schema validation.**
**5. Created a validation schema in `schema.js`.**
**6. Used Joi for validation in routes.**
**7. Converted validation into a reusable middleware.**
**8. Improved error messages using Joi’s `error.details`.**
