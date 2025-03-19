## Edit Listing Image

#### **1. Understanding the Problem**

When editing a listing, the current setup only allows updating the listing details but not replacing the existing image. We need to modify it so that users can upload a new image, which will replace the old one.

---

#### **2. Updating the Edit Form (`edit.ejs`)**

First, we need to update our `edit.ejs` file to allow file uploads.

- Add `enctype="multipart/form-data"` to the `<form>` tag:

  ```sh
  enctype="multipart/form-data"
  >
  ```

- Modify the form to include a file input for a new image upload:
  ```html
  <div class="mb-3">
    <label for="image" class="form-label">Upload New Image</label>
    <input name="listing[image]" type="file" class="form-control" />
  </div>
  ```
  This allows users to upload a new image when editing a listing.

---

#### **3. Updating the Route (`routes/listing.js`)**

Now that the form can send files, we need to modify our `PUT` route to handle the uploaded image.

- Add the same `multer` middleware used in the `POST` route to handle image uploads:

  ```js
  .put(
      isLoggedIn,
      isOwner,
      upload.single("listing[image]"),
      validateListing,
      wrapAsync(listingController.updateListing)
  )
  ```

  This ensures that when updating a listing, a new image (if uploaded) is processed correctly.

---

#### **4. Updating the Controller (`controllers/listings.js`)**

Now, we modify the `updateListing` function to handle the new image:

1. **Find and update the listing**:

   ```js
   module.exports.updateListing = async (req, res) => {
       let { id } = req.params;
       let listing = await Listing.findByIdAndUpdate(id, { ...req.body.listing });
   ```

   Here, we retrieve the listing from the database and update it with the new details (except the image, which we'll handle separately).

2. **Check if a new image was uploaded**:

   ```js
   if (req.file) {
     let url = req.file.path;
     let filename = req.file.filename;
     listing.image = { url, filename };
     await listing.save();
   }
   ```

   - If a new image is uploaded, we replace the old one with the new one and save the listing.

---

#### **5. Handling Edge Cases**

What if a user does **not** upload a new image?

- If no new image is uploaded, `req.file` will be `undefined`, and attempting to access `req.file.path` will cause an error.
- We prevent this by checking if `req.file` exists before trying to update the image.

  Alternative approach using `typeof`:

  ```js
  if (typeof req.file !== "undefined") {
    let url = req.file.path;
    let filename = req.file.filename;
    listing.image = { url, filename };
    await listing.save();
  }
  ```

  This ensures that the image update logic runs **only** when a new image is uploaded.

---

### **Final Summary**

✅ Updated `edit.ejs` to include a file input and `multipart/form-data`.  
✅ Modified the `PUT` route in `routes/listing.js` to accept image uploads.  
✅ Updated `updateListing` in `controllers/listings.js` to replace the old image only if a new one is uploaded.  
✅ Handled edge cases to avoid errors when no new image is provided.
