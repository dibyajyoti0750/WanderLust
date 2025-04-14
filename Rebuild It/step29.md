## **Saving the Link in MongoDB**

The next step is to store the image link in the MongoDB database. To do this, we need to modify our listing schema slightly.

Previously, the `image` field in our schema was just a simple string. However, we now plan to store two values for each image:

1. The **URL** (path) of the image.
2. The **filename** of the image.

Although storing the filename may not be useful immediately, it will be helpful in the future if we want to implement features such as **deleting or updating** images.

---

### **Modifying the Schema**

To accommodate these changes, we modify our `models/listing.js` file:

```js
image: {
    url: String,
    filename: String,
},
```

This change will temporarily **break certain functionalities** on our website, so we will need to make additional modifications gradually.

---

### **Updating the Routes**

Next, we update our `routes/listing.js` file to handle image uploads properly. When creating a new listing, we need to ensure that our middleware processes the uploaded image before saving the listing.

```js
router
  .route("/")
  .get(wrapAsync(listingController.index))
  .post(
    isLoggedIn,
    upload.single("listing[image]"),
    validateListing,
    wrapAsync(listingController.createListing)
  );
```

Here, `upload.single("listing[image]")` runs **before** `validateListing`, ensuring that the image is properly handled before validation.

---

### **Saving the Image URL and Filename**

Now, we modify `controllers/listings.js` to store both the **image URL and filename** in the database when a new listing is created.

```js
const cloudinary = require("../cloudConfig.js");

module.exports.createListing = async (req, res, next) => {
  const file = req.file;

  // Convert buffer to base64
  const b64 = Buffer.from(file.buffer).toString("base64");
  const dataURI = `data:${file.mimetype};base64,${b64}`;

  // Upload to Cloudinary
  const result = await cloudinary.uploader.upload(dataURI, {
    folder: "wanderlust_DEV",
  });

  const url = result.secure_url; // This is the image URL
  const filename = result.public_id; // This is used for deletion

  const newListing = new Listing(req.body.listing);
  newListing.owner = req.user._id;
  newListing.image = { url, filename };

  await newListing.save();

  req.flash("success", "New Listing Created!");
  res.redirect("/listings");
};
```

With this change, every new listing will now include an image object containing both:

- The `url` where the image is stored.
- The `filename` of the uploaded image.

---

### **Verifying in the Database**

To confirm that everything is working, we can check our database by running:

```sh
db.listings.find({ title: "newVilla" });
```

The new listing should now be saved correctly, and the image object should include both:

- The URL of the uploaded image.
- The filename of the uploaded file.

---

### **Updating Our Views**

Right now, the images being displayed are from our **initial dataset** (`init/data.js`). However, these were saved in the old format, while our new schema requires an **image object** with `url` and `filename`.

To fix this:

1. **Update the data initialization script (`init/data.js`)** to use the new format.
2. **Reinitialize the database** with the updated data.

We can either generate new sample data or modify the existing data:  
[New Init Data](https://drive.google.com/drive/folders/10uB6zyVQIOfa1ngmgsPDpHSND8IDi5do)

Then, reinitialize the database:

```sh
cd init/
node index.js
```

Once we restart the server and refresh our platform, all images from the old dataset will disappear. This is because our views (`index.ejs` and `show.ejs`) are still expecting a simple image string.

---

### **Fixing Image Display in EJS Templates**

To properly display images in our EJS templates, we update `index.ejs` and `show.ejs` to use the new image object:

```html
src="<%= listing.image.url %>"
```

Now, if we create a **new listing with an uploaded image**, the image should appear correctly on both:

- The **all listings** page.
- The **individual listing** page.

---

### **Final Summary**

1. **Modified the schema** to store an image object (`url` and `filename`).
2. **Updated the listing creation route** to process uploaded images.
3. **Changed the controller** to save both image URL and filename.
4. **Reordered middleware** to avoid Joi validation errors.
5. **Verified the changes** by checking MongoDB.
6. **Reinitialized the database** with new image data format.
7. **Updated EJS templates** to use `listing.image.url` instead of a simple string.
