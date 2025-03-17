## **Associating an Owner with a Listing**

### **Why Associate an Owner?**

Before implementing authorization, we need to associate each listing with an owner. Authorization ensures that only the owner of a listing can perform sensitive operations like editing or deleting it.

### **Updating the Listing Schema**

To store the owner's information, we modify the listing schema (`models/listing.js`) and add a new field:

```js
owner: {
    type: Schema.Types.ObjectId,
    ref: "User",
}
```

- `Schema.Types.ObjectId` ensures the owner references a registered user in our system.
- The `ref: "User"` establishes a relationship with the `User` model.

### **Re-initializing the Database**

Since existing listings lack an `owner` field, we must update them. Instead of manually modifying `data.js`, we dynamically insert an `owner` property when seeding data.

#### **Modifying `index.js`**

Locate the function that inserts listings into the database and modify it to add an owner automatically.

```js
const initDB = async () => {
  await Listing.deleteMany({});
  initData.data = initData.data.map((obj) => ({
    ...obj,
    owner: "67d69032f60d44aba9a6a193", // Example user ID
  }));
  await Listing.insertMany(initData.data);
  console.log("DB was initialized");
  mongoose.connection.close();
};
```

- The `map` function iterates over `initData.data`, adding an `owner` field to each listing.
- The modified `initData.data` is then inserted into the database.

### **Re-initializing Data**

Run the following commands:

```sh
cd init/
node index.js
```

Check the database to confirm that every listing now has an owner.

---

## **Populating the Owner in Listings**

Now that each listing has an associated owner, we need to fetch and display that information.

### **Updating the Show Route (`routes/listing.js`)**

Modify the `Show Route` to include the owner’s details:

```js
const listing = await Listing.findById(id)
  .populate("reviews")
  .populate("owner");
```

- `populate("owner")` retrieves the complete user object associated with `owner`.

#### **Verifying the Data**

```js
console.log(listing);
res.render("listings/show.ejs", { listing });
```

When clicking on a listing, the owner's details should now be included in the retrieved data.

---

## **Displaying the Owner in the UI**

Modify `show.ejs` to display the owner's username:

```html
<i><%= listing.owner.username %></i>
```

For better readability, wrap it in a structured layout:

```html
<div class="card-body">
  <p class="card-text">Owned by <i><%= listing.owner.username %></i></p>
  <p class="card-text"><%= listing.description %></p>
  <p class="card-text">&#8377;<%= listing.price.toLocaleString("en-IN") %></p>
  <p class="card-text"><%= listing.location %></p>
  <p class="card-text"><%= listing.country %></p>
</div>
```

---

## **Fixing "Cannot Read Properties of Undefined" Error**

If an error occurs while viewing a newly created listing, it means the `owner` field is missing.

### **Solution: Assign Owner When Creating Listings**

Modify the `POST` route in `routes/listing.js`:

```js
newListing.owner = req.user._id;
await newListing.save();
```

- `req.user._id` contains the current logged-in user’s ID, ensuring the newly created listing is assigned to them.

---

### **Summary**

✅ **Added an `owner` field** to the `Listing` schema.  
✅ **Updated the database** to include an owner for all listings.  
✅ **Populated owner details** in the show route.  
✅ **Displayed owner information** in the UI.  
✅ **Fixed owner-related errors** when creating new listings.
