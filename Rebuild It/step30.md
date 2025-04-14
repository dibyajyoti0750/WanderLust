## Edit Listing Image

#### **1. Understanding the Problem**

When editing a listing, the current setup only allows updating the listing details but not replacing the existing image. We need to modify it so that users can upload a new image, which will replace the old one.

---

#### **2. Updating the Edit Form (`edit.ejs`)**

Add `enctype="multipart/form-data"` to the `<form>` tag to allow file uploads:

```sh
enctype="multipart/form-data"
>
```

Modify the form to include a file input for a new image upload:

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

---

#### **4. Updating the Controller (`controllers/listings.js`)**

Now, we modify the `updateListing` function to handle the new image:

```js
module.exports.updateListing = async (req, res) => {
  const { id } = req.params;

  const listing = await Listing.findById(id);

  // Update fields from form
  Object.assign(listing, req.body.listing);

  // If a new image was uploaded
  if (req.file) {
    // Delete old image from Cloudinary
    if (listing.image && listing.image.filename) {
      await cloudinary.uploader.destroy(listing.image.filename);
    }

    // Convert buffer to base64 and upload new image
    const file = req.file;
    const b64 = Buffer.from(file.buffer).toString("base64");
    const dataURI = `data:${file.mimetype};base64,${b64}`;

    const result = await cloudinary.uploader.upload(dataURI, {
      folder: "wanderlust_DEV",
    });

    listing.image = {
      url: result.secure_url,
      filename: result.public_id,
    };
  }

  await listing.save();

  req.flash("success", "Listing Updated!");
  res.redirect(`/listings/${id}`);
};
```

---

### **Final Summary**

✅ Updated `edit.ejs` to include a file input and `enctype="multipart/form-data"` for image uploads.  
✅ Modified the `PUT` route in `routes/listing.js` to accept image uploads.  
✅ Updated `updateListing` in `controllers/listings.js`:

- Deletes the old image from Cloudinary if a new one is uploaded.
- Uploads the new image using buffer-to-base64 conversion (consistent with the create function).
- Only updates the image field if a new file is provided.
- Retains existing image if no new image is uploaded.
