## **Custom Error Handling in Express**

In our `app.js` server, there are multiple possible failure points where things can go wrong, especially when dealing with data validation, database operations, and async errors. Proper error handling ensures our application doesn't crash and provides meaningful feedback to users.

### **Handling Errors in Routes**

For example, our listing price should be a number, but if a user sends a string (e.g., via Postman or Hoppscotch), our app would throw an asynchronous error. These types of errors occur when the database rejects invalid data.

#### **Basic Error Handling Middleware**

We first define a global error handling middleware:

```js
app.use((err, req, res, next) => {
  res.send("Something went wrong!");
});
```

This middleware catches errors and prevents our app from crashing.

#### **Using Try-Catch Blocks**

Initially, in our **Create Route**, we fetch the listing object from `req.body` and try to save it directly. If there's an issue (like an invalid price format), MongoDB throws an async error. To catch this, we use a `try-catch` block:

```js
app.post("/listings", async (req, res, next) => {
  try {
    const newListing = new Listing(req.body.listing);
    await newListing.save();
    res.redirect("/listings");
  } catch (err) {
    next(err);
  }
});
```

Here, `next(err)` passes the error to our error-handling middleware.

Now, if we add an invalid price (e.g., `"abcd"` instead of a number) and submit, the error gets handled properly.

---

## **Custom `wrapAsync` Function**

Since handling async errors with `try-catch` in every route is repetitive, we create a reusable function called `wrapAsync` that automatically catches errors.

#### **Creating `wrapAsync.js`**

Inside a `utils` folder (which will store utilities like error handlers), we create a file `wrapAsync.js`:

```js
module.exports = (fn) => {
  return (req, res, next) => {
    fn(req, res, next).catch(next); // shorthand for `.catch(err => next(err))`
  };
};
```

#### **Using `wrapAsync` in Routes**

Now, in `app.js`, we require `wrapAsync`:

```js
const wrapAsync = require("./utils/wrapAsync.js");
```

We then modify our create route to use `wrapAsync`, eliminating the `try-catch` block:

```js
app.post(
  "/listings",
  wrapAsync(async (req, res, next) => {
    const newListing = new Listing(req.body.listing);
    await newListing.save();
    res.redirect("/listings");
  })
);
```

This ensures all async errors in the function are automatically passed to the error handler.

---

## **Custom `ExpressError` Class**

Instead of sending generic error messages, we want to send **custom error responses with meaningful status codes**.

#### **Creating `ExpressError.js`**

Inside `utils`, we create another file called `ExpressError.js`:

```js
class ExpressError extends Error {
  constructor(statusCode, message) {
    super();
    this.statusCode = statusCode;
    this.message = message;
  }
}

module.exports = ExpressError;
```

#### **Using `ExpressError` in Middleware**

In `app.js`, we require our new error class:

```js
const ExpressError = require("./utils/ExpressError.js");
```

Then, we modify our error-handling middleware to return status codes dynamically:

```js
app.use((err, req, res, next) => {
  let { statusCode, message } = err;
  res.status(statusCode).send(message);
});
```

#### **Handling 404 Errors (Page Not Found)**

If a user requests a **non-existent route**, we send a `404 Page Not Found` error:

```js
app.all("*", (req, res, next) => {
  next(new ExpressError(404, "Page Not Found!"));
});
```

Now, if we visit `/random`, we get a **404 Page Not Found** message.

---

## **Validating Requests and Handling Missing Data**

Another common issue occurs when users send **empty or malformed requests** via Hoppscotch directly to our API, potentially causing server errors.

For example, in our **Create Route**, if the request body is empty, we get a MongoDB validation error like:

```
Listing validation failed: title: Path `title` is required.
```

Since this is a **client error**, we should return a `400 Bad Request` response.

#### **Validating Data Before Processing**

We modify our create route to check for missing `listing` data:

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

Now, if we send an empty request via Hoppscotch, the response will be:

```
400 Bad Request - Send valid data for listing
```

---

## **Applying Error Handling to Update Routes**

Similarly, in the **Update Route**, we must check for missing data before updating:

```js
app.put(
  "/listings/:id",
  wrapAsync(async (req, res) => {
    if (!req.body.listing) {
      throw new ExpressError(400, "Send valid data for listing");
    }
    let { id } = req.params;
    await Listing.findByIdAndUpdate(id, { ...req.body.listing });
    res.redirect(`/listings/${id}`);
  })
);
```

This ensures we don’t update listings with invalid data.

---

## **Final Improvements**

To prevent generic errors from showing as `undefined`, we add **default values** in the error handler:

```js
app.use((err, req, res, next) => {
  let { statusCode = 500, message = "Something went wrong!" } = err;
  res.status(statusCode).send(message);
});
```

Now, any unhandled error will return:

```
500 Internal Server Error - Something went wrong!
```

---

## **Summary of What We Did**

1. **Implemented Global Error Handling**

   - Created an Express error handler using `app.use()`.
   - Used `next(err)` to pass errors to the handler.

2. **Created `wrapAsync` to Handle Async Errors**

   - Wrapped all async route handlers inside `wrapAsync` to automatically catch and handle errors.

3. **Developed a Custom `ExpressError` Class**

   - Used it to throw errors with custom messages and status codes.

4. **Handled 404 Errors for Invalid Routes**

   - Used `app.all("*")` to send a `404 Page Not Found` response.

5. **Validated Incoming Requests Before Processing**

   - Ensured `req.body.listing` exists before saving or updating.
   - Sent `400 Bad Request` errors for invalid data.

6. **Improved Default Error Handling**
   - Added a fallback error message for unexpected errors.

---
