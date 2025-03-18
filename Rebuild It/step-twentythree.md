### **MVC (Model-View-Controller) Pattern Implementation for Listings**

The MVC pattern helps in organizing the code by separating concerns. The controller manages logic, the model interacts with the database, and the view handles UI rendering.

---

### **1. Setting Up the Controller**

Instead of writing callbacks directly in the routes, we move them to a separate file inside a `controllers` folder.

1. **Create a new folder named `controllers`.**
2. **Inside it, create a file for listings:**
   ```
   controllers/listings.js
   ```

---

### **2. Moving Callbacks to the Controller**

#### **Index Route (Displaying All Listings)**

Previously, the index route callback was inside `routes/listing.js`. We now move it to `controllers/listings.js`:

```javascript
module.exports.index = async (req, res) => {
  const allListings = await Listing.find({});
  res.render("listings/index", { allListings });
};
```

- This function fetches all listings from the database and renders them in `index.ejs`.

#### **Require the Listing Model**

Since we are moving this logic to a separate file, we need to require the Listing model:

```javascript
const Listing = require("../models/listing");
```

#### **Require the Controller in Routes**

Now, in `routes/listing.js`, require the controller:

```javascript
const listingController = require("../controllers/listings.js");
```

#### **Update the Route**

Modify the index route to use the controller:

```javascript
router.get("/", wrapAsync(listingController.index));
```

---

### **3. Moving Other Routes to the Controller**

#### **New Route (Render New Listing Form)**

Move the callback from `routes/listing.js` to `controllers/listings.js`:

```javascript
module.exports.renderNewForm = (req, res) => {
  res.render("listings/new.ejs");
};
```

Update the route in `routes/listing.js`:

```javascript
router.get("/new", isLoggedIn, listingController.renderNewForm);
```

---

#### **Show Route (Displaying a Single Listing)**

Move the callback to `controllers/listings.js`:

```javascript
module.exports.showListing = async (req, res) => {
  let { id } = req.params;
  const listing = await Listing.findById(id)
    .populate({
      path: "reviews",
      populate: {
        path: "author",
      },
    })
    .populate("owner");

  if (!listing) {
    req.flash("error", "Listing you requested for does not exist!");
    res.redirect("/listings");
  }

  console.log(listing);
  res.render("listings/show", { listing });
};
```

- This function finds a listing by ID and populates its reviews and owner.

Update the route:

```javascript
router.get("/:id", wrapAsync(listingController.showListing));
```

---

#### **Create Route**

Move the callback to `controllers/listings.js`:

```javascript
module.exports.createListing = async (req, res, next) => {
  const newListing = new Listing(req.body.listing);
  console.log(req.user);
  newListing.owner = req.user._id;
  await newListing.save();
  req.flash("success", "New Listing Created!");
  res.redirect("/listings");
};
```

Update the route:

```javascript
router.post(
  "/",
  isLoggedIn,
  validateListing,
  wrapAsync(listingController.createListing)
);
```

---

#### **Edit Route (Rendering Edit Form)**

Move the callback to `controllers/listings.js`:

```javascript
module.exports.renderEditForm = async (req, res) => {
  let { id } = req.params;
  const listing = await Listing.findById(id);

  if (!listing) {
    req.flash("error", "Listing you requested for does not exist!");
    res.redirect("/listings");
  }

  res.render("listings/edit", { listing });
};
```

Update the route:

```javascript
router.get(
  "/:id/edit",
  isLoggedIn,
  isOwner,
  wrapAsync(listingController.renderEditForm)
);
```

---

#### **Update Route**

Move the callback to `controllers/listings.js`:

```javascript
module.exports.updateListing = async (req, res) => {
  let { id } = req.params;
  await Listing.findByIdAndUpdate(id, { ...req.body.listing });
  req.flash("success", "Listing Updated!");
  res.redirect(`/listings/${id}`);
};
```

Update the route:

```javascript
router.put(
  "/:id",
  isLoggedIn,
  isOwner,
  validateListing,
  wrapAsync(listingController.updateListing)
);
```

---

#### **Delete Route**

Move the callback to `controllers/listings.js`:

```javascript
module.exports.destroyListing = async (req, res) => {
  let { id } = req.params;
  let deletedListing = await Listing.findByIdAndDelete(id);
  console.log(deletedListing);
  req.flash("success", "Listing Deleted!");
  res.redirect("/listings");
};
```

Update the route:

```javascript
router.delete(
  "/:id",
  isLoggedIn,
  isOwner,
  wrapAsync(listingController.destroyListing)
);
```

---

### **Final Thoughts**

- Now, all routes in `routes/listing.js` only reference functions from `controllers/listings.js`.
- This makes the code cleaner, more maintainable, and follows the MVC pattern correctly.
- The controller now handles all business logic, keeping routes minimal and readable.

This completes the MVC implementation for Listings! 🚀
