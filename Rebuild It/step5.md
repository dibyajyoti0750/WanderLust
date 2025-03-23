## **Form Validations**

### **Client-Side Validation**

Client-side validation ensures that the data submitted from the frontend to the backend is correct before it even reaches the server. Without validation, a user can submit incomplete or incorrect data, which could lead to storing invalid information in the database.

For example, in the **Add New Listing** form, fields like **title, description, image, country, and location** can be left empty and still be submitted, which results in an incomplete record in the database.

To prevent this, we need to ensure:

1. The form data is useful and in the correct format before submission.
2. Server-side validation also enforces constraints based on the database schema and provides meaningful error messages when something goes wrong.

Client-side validation checks:

- Data should be in the correct **format** (e.g., numbers for price, text for titles).
- Constraints must be enforced (e.g., price should not be negative, title should not exceed a set length).
- Required fields should not be left empty.

---

## **Implementing Client-Side Validation**

The easiest way to enforce client-side validation is by marking fields as **required** in `new.ejs`. For example:

```html
<input
  name="listing[title]"
  placeholder="Add a catchy title"
  type="text"
  class="form-control"
  required
/>
```

Now, if the user tries to submit the form without filling this field, the browser will prevent submission and show a default validation message.

However, browser validation styles vary, and for a standardized experience across all browsers, we use **Bootstrap’s validation system**.

### **Using Bootstrap for Better Validation**

Bootstrap provides **custom form validation styles** that override the default browser messages.

#### **Step 1: Disable Default Browser Validation**

Add the `novalidate` attribute to the `<form>` tag:

```html
<form method="post" action="/listings" novalidate>...</form>
```

#### **Step 2: Enable Bootstrap Validation**

Bootstrap requires adding a `needs-validation` class to the form:

```html
<form method="post" action="/listings" novalidate class="needs-validation">
  ...
</form>
```

#### **Step 3: Add Custom JavaScript for Bootstrap Validation**

Copy the following script into `public/js/script.js`:

```js
(() => {
  "use strict";
  const forms = document.querySelectorAll(".needs-validation");
  Array.from(forms).forEach((form) => {
    form.addEventListener(
      "submit",
      (event) => {
        if (!form.checkValidity()) {
          event.preventDefault();
          event.stopPropagation();
        }
        form.classList.add("was-validated");
      },
      false
    );
  });
})();
```

Then, include this script in `boilerplate.ejs`:

```html
<script src="/js/script.js"></script>
```

Now, when a user submits invalid data, Bootstrap will handle the validation styling.

---

## **Displaying Success & Failure Messages**

When users enter incorrect data, it's useful to show messages explaining what went wrong.

Bootstrap provides:

- `valid-feedback` (for successful validation)
- `invalid-feedback` (for errors)

Example of a **success message** (when a title is correctly entered):

```html
<div class="mb-3">
  <label for="title" class="form-label">Title</label>
  <input
    name="listing[title]"
    placeholder="Add a catchy title"
    type="text"
    class="form-control"
    required
  />
  <div class="valid-feedback">Title looks good!</div>
</div>
```

Example of an **error message** (when price is incorrect):

```html
<div class="mb-3 col-md-4">
  <label for="price" class="form-label">Price</label>
  <input
    name="listing[price]"
    placeholder="1200"
    class="form-control"
    required
  />
  <div class="invalid-feedback">Price should be valid</div>
</div>
```

Similar messages can be added for:

- **Country:** `<div class="invalid-feedback">Country name should be valid</div>`
- **Location:** `<div class="invalid-feedback">Location should be valid</div>`
- **Description:**
  ```html
  <div class="mb-3">
    <label for="description" class="form-label">Description</label>
    <textarea
      name="listing[description]"
      class="form-control"
      required
    ></textarea>
    <div class="invalid-feedback">Please enter a short description</div>
  </div>
  ```

---

## **Implementing Validations in the Edit Form**

The same client-side validation should be implemented in `edit.ejs` to ensure data remains consistent.

1. **Mark Input Fields as Required**

- **Title, description, price, country, and location** should all be required, with appropriate validation messages.
- Similarly, we will make the image field required. If a listing is already added, it will have an existing image—either provided by the user or a default image from the database. If the user wants to replace the default image with a new link, that’s fine. However, we won’t allow them to remove the default image without providing a new one.

Then,

```html
<form
  method="post"
  action="/listings/<%= listing._id %>?_method=PUT"
  novalidate
  class="needs-validation"
></form>
```

2. **Add Error Messages for Missing Fields**
   For images:

```html
<div class="invalid-feedback">Please enter a valid Link</div>
```

---

## **Why Server-Side Validation is Still Needed**

Even though client-side validation prevents incorrect data from being entered through the frontend, **it is not enough** because:

1. **Users can bypass client-side validation** using tools like Postman or Hoppscotch.
2. **Direct API requests can still send incorrect data** to `/listings` using methods like POST, PATCH, or PUT.

### **Solution: Implement Server-Side Validation**

- The backend should **verify all incoming data** before saving it to the database.
- Implement proper **error handling** to provide meaningful feedback when validation fails.

---

## **Summary of What We Have Implemented So Far**

✅ **Client-Side Form Validation**

- Marked essential fields as `required`.
- Used Bootstrap for custom validation styles.
- Disabled default browser validation.
- Added JavaScript to apply Bootstrap validation.
- Displayed success (`valid-feedback`) and error (`invalid-feedback`) messages.

✅ **Applied Validations to Both New & Edit Forms**

- Ensured all fields are validated before submission.
- Required fields: **title, description, price, country, location, and image**.

✅ **Next Steps**

- Implement **Server-Side Validation** to prevent incorrect data from being sent directly to the API.
- Handle errors efficiently in the backend.
