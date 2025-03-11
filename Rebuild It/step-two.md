## 1. Setting Up EJS

- Install and set up EJS in `app.js`:
  ```js
  const path = require("path");
  app.set("view engine", "ejs");
  app.set("views", path.join(__dirname, "views"));
  ```
- Create a `views` folder and a `listings` subfolder inside it.

---

## 2. Index Route (Displaying Listings)

- Define `/listings` route in `app.js`:
  ```js
  app.get("/listings", async (req, res) => {
    const allListings = await Listing.find({});
    res.render("listings/index.ejs", { allListings });
  });
  ```
- Create `index.ejs` inside `views/listings`:
  ```html
  <h3>All Listings</h3>
  <ul>
    <% for(let listing of allListings) { %>
    <li><a href="/listings/<%= listing._id %>"><%= listing.title %></a></li>
    <% } %>
  </ul>
  ```

---

## 3. Show Route (Displaying a Single Listing)

- Enable `express.urlencoded` in `app.js`:
  ```js
  app.use(express.urlencoded({ extended: true }));
  ```
- Define show route in `app.js`:
  ```js
  app.get("/listings/:id", async (req, res) => {
    let { id } = req.params;
    const listing = await Listing.findById(id);
    res.render("listings/show.ejs", { listing });
  });
  ```
- Create `show.ejs`:
  ```html
  <h3>Listing Details</h3>
  <ul>
    <li><%= listing.title %></li>
    <li><%= listing.description %></li>
    <li>&#8377;<%= listing.price.toLocaleString("en-IN") %></li>
    <li><%= listing.location %></li>
    <li><%= listing.country %></li>
  </ul>
  ```

---

## 4. New & Create Route (Adding a Listing)

- Add "Create New Listing" button in `index.ejs`:
  ```html
  <form action="/listings/new" method="get">
    <button>Create New Listing</button>
  </form>
  ```
- Define New Route in `app.js`:
  ```js
  app.get("/listings/new", (req, res) => {
    res.render("listings/new.ejs");
  });
  ```
- Create `new.ejs` with a form:
  ```html
  <h3>Create a New Listing</h3>
  <form action="/listings" method="post">
    <input type="text" name="listing[title]" placeholder="Enter title" />
    <textarea
      name="listing[description]"
      placeholder="Enter description"
    ></textarea>
    <input type="text" name="listing[image]" placeholder="Enter image URL" />
    <input type="number" name="listing[price]" placeholder="Enter price" />
    <input type="text" name="listing[location]" placeholder="Enter location" />
    <input type="text" name="listing[country]" placeholder="Enter country" />
    <button>Add</button>
  </form>
  ```
- Define Create Route in `app.js`:
  ```js
  app.post("/listings", async (req, res) => {
    const newListing = new Listing(req.body.listing);
    await newListing.save();
    res.redirect("/listings");
  });
  ```

---

## 5. Edit & Update Route (Editing a Listing)

- Add "Edit" button in `show.ejs`:
  ```html
  <a href="/listings/<%= listing._id %>/edit">Edit this listing</a>
  ```
- Define Edit Route in `app.js`:
  ```js
  app.get("/listings/:id/edit", async (req, res) => {
    let { id } = req.params;
    const listing = await Listing.findById(id);
    res.render("listings/edit.ejs", { listing });
  });
  ```
- Create `edit.ejs`:
  ```html
  <h3>Edit your Listing</h3>
  <form method="post" action="/listings/<%= listing._id %>?_method=PUT">
    <input type="text" name="listing[title]" value="<%= listing.title %>" />
    <textarea name="listing[description]"><%= listing.description %></textarea>
    <input type="text" name="listing[image]" value="<%= listing.image %>" />
    <input type="number" name="listing[price]" value="<%= listing.price %>" />
    <input
      type="text"
      name="listing[location]"
      value="<%= listing.location %>"
    />
    <input type="text" name="listing[country]" value="<%= listing.country %>" />
    <button>Edit</button>
  </form>
  ```
- Install and set up `method-override`:
  ```sh
  npm i method-override
  ```
  ```js
  const methodOverride = require("method-override");
  app.use(methodOverride("_method"));
  ```
- Define Update Route in `app.js`:
  ```js
  app.put("/listings/:id", async (req, res) => {
    let { id } = req.params;
    await Listing.findByIdAndUpdate(id, { ...req.body.listing });
    res.redirect(`/listings/${id}`);
  });
  ```

---

## 6. Destroy Route (Deleting a Listing)

- Add "Delete" button in `show.ejs`:
  ```html
  <form method="post" action="/listings/<%= listing._id %>?_method=DELETE">
    <button>Delete this listing</button>
  </form>
  ```
- Define Delete Route in `app.js`:
  ```js
  app.delete("/listings/:id", async (req, res) => {
    let { id } = req.params;
    await Listing.findByIdAndDelete(id);
    res.redirect("/listings");
  });
  ```

---

### **Project Setup Completed**
