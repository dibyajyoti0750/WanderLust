## **Styling the Index Page**

We transformed the individual listings on the index page into **Bootstrap cards** to enhance the UI.

### **Updating `index.ejs`**

We replaced the existing listing structure with Bootstrap cards:

```ejs
<% layout("/layouts/boilerplate") %>
  <h3>All listings</h3>
  <!-- <form method="get" action="/listings/new">
    <button>Create New Listing</button>
  </form> -->
  <div class="row row-cols-lg-3 row-cols-md-2 row-cols-sm-1">
    <% for(let listing of allListings) { %>
    <div class="card col">
      <img
        src="<%= listing.image %>"
        class="card-img-top"
        style="height: 20rem"
        alt="listing_image"
      />
      <div class="card-body">
        <p class="card-text">
          <b><%= listing.title %></b> <br />
          &#8377;<%= listing.price.toLocaleString("en-IN") %> / night
        </p>
      </div>
    </div>
    <% } %>
  </div>
```

### **Adding Custom Styling in `styles.css`**

We customized Bootstrap’s default card styles to improve the design:

```css
/* Cards */
.card {
  border: none !important;
  margin-bottom: 2rem;
}

.card-img-top {
  border-radius: 1rem !important;
  width: 100% !important;
  object-fit: cover !important;
}

.card-body {
  padding: 0 !important;
}

.card-text p {
  font-weight: 400 !important;
}
```

### **Adding Google Font - Jakarta Sans**

We improved the typography by adding the **Plus Jakarta Sans** font in `boilerplate.ejs`:

```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link
  href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:ital,wght@0,200..800;1,200..800&display=swap"
  rel="stylesheet"
/>
```

And applied the font globally in `styles.css`:

```css
body {
  font-family: "Plus Jakarta Sans", sans-serif !important;
}
```

### **Making Cards Clickable**

To ensure users can **click on a card to visit its show page**, we wrapped each card inside an `<a>` tag:

```ejs
<div class="row row-cols-lg-3 row-cols-md-2 row-cols-sm-1">
    <% for(let listing of allListings) { %>
    <a href="/listings/<%= listing._id %>" class="listing-link">
      <div class="card col">
        <img
          src="<%= listing.image %>"
          class="card-img-top"
          style="height: 20rem"
          alt="listing_image"
        />
        <div class="card-img-overlay"></div>
        <div class="card-body">
          <p class="card-text">
            <b><%= listing.title %></b> <br />
            &#8377;<%= listing.price.toLocaleString("en-IN") %> / night
          </p>
        </div>
      </div>
    </a>
    <% } %>
</div>
```

### **Additional Styling in `styles.css`**

- **Removing link underline**:

```css
.listing-link {
  text-decoration: none;
}
```

- **Adding hover effect on cards**:

```css
.card-img-overlay {
  opacity: 0;
}

.card-img-overlay:hover {
  opacity: 0.2;
  background-color: white;
}
```

---

## **Styling the "Add New Listing" Page**

We redesigned the **form for adding a new listing**.

### **Updating `new.ejs`**

```ejs
<% layout("/layouts/boilerplate") %>
<div class="row mt-3">
  <div class="col-8 offset-2">
    <h3>Create a New Listing</h3>
    <form method="post" action="/listings">
      <div class="mb-3">
        <label for="title" class="form-label">Title</label>
        <input
          name="listing[title]"
          placeholder="Add a catchy title"
          type="text"
          class="form-control"
        />
      </div>
      <div class="mb-3">
        <label for="description" class="form-label">Description</label>
        <textarea name="listing[description]" class="form-control"></textarea>
      </div>
      <div class="mb-3">
        <label for="image" class="form-label">Image Link</label>
        <input
          name="listing[image]"
          placeholder="enter image URL/Link"
          type="text"
          class="form-control"
        />
      </div>
      <div class="row">
        <div class="mb-3 col-md-4">
          <label for="price" class="form-label">Price</label>
          <input
            name="listing[price]"
            placeholder="1200"
            type="number"
            class="form-control"
          />
        </div>
        <div class="mb-3 col-md-8">
          <label for="country" class="form-label">Country</label>
          <input
            name="listing[country]"
            placeholder="India"
            type="text"
            class="form-control"
          />
        </div>
      </div>
      <div class="mb-3">
        <label for="location" class="form-label">Location</label>
        <input
          name="listing[location]"
          placeholder="Jaipur, Rajasthan"
          type="text"
          class="form-control"
        />
      </div>
      <button class="btn btn-dark add-btn mt-3 mb-3">Add</button>
    </form>
  </div>
</div>
```

### **Styling in `styles.css`**

```css
/* New Page */
.add-btn {
  background-color: #fe424d !important;
  border: none !important;
}
```

---

## **Making the Footer Stick to the Bottom**

We ensured that the **footer always stays at the bottom** of the page:

```css
body {
  font-family: "Plus Jakarta Sans", sans-serif !important;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.container {
  flex: 1;
}
```

---

## **Styling the "Edit Listing" Page**

We applied the same styling changes to the **edit listing page**.

### **Updating `edit.ejs`**

```ejs
<% layout("/layouts/boilerplate") %>
<div class="row mt-3">
  <div class="col-8 offset-2">
    <h3>Edit your Listing</h3>
    <form method="post" action="/listings/<%= listing._id %>?_method=PUT">
      <div class="mb-3">
        <label for="title" class="form-label">Title</label>
        <input
          name="listing[title]"
          value="<%= listing.title %>"
          type="text"
          class="form-control"
        />
      </div>
      <div class="mb-3">
        <label for="description" class="form-label">Description</label>
        <textarea name="listing[description]" class="form-control">
<%= listing.description %></textarea
        >
      </div>
      <div class="mb-3">
        <label for="image" class="form-label">Image Link</label>
        <input
          name="listing[image]"
          value="<%= listing.image %>"
          type="text"
          class="form-control"
        />
      </div>
      <div class="row">
        <div class="mb-3 col-md-4">
          <label for="price" class="form-label">Price</label>
          <input
            name="listing[price]"
            value="<%= listing.price %>"
            type="number"
            class="form-control"
          />
        </div>
        <div class="mb-3 col-md-8">
          <label for="country" class="form-label">Country</label>
          <input
            name="listing[country]"
            value="<%= listing.country %>"
            type="text"
            class="form-control"
          />
        </div>
      </div>
      <div class="mb-3">
        <label for="location" class="form-label">Location</label>
        <input
          name="listing[location]"
          value="<%= listing.location %>"
          type="text"
          class="form-control"
        />
      </div>
      <button class="btn btn-dark edit-btn mt-3 mb-3">Edit</button>
    </form>
  </div>
</div>
```

### **Styling in `styles.css`**

```css
/* Edit Page */
.edit-btn {
  background-color: #fe424d !important;
  border: none !important;
}
```

---

## **Styling the "Show Listing" Page**

We designed the **show page** to display listing details.

### **Updating `show.ejs`**

```ejs
<% layout("/layouts/boilerplate") %>
<div class="row mt-3">
  <div class="col-8 offset-3">
    <h3>Listing Details</h3>
  </div>
  <div class="card col-6 offset-3 show-card">
    <img
      src="<%= listing.image %>"
      class="card-img-top show-img"
      alt="listing_image"
    />
    <div class="card-body">
      <p class="card-text">
        <b> <%= listing.title %> </b> <br />
        <%= listing.description %> <br />
        &#8377;<%= listing.price.toLocaleString("en-IN") %> <br />
        <%= listing.location %> <br />
        <%= listing.country %>
      </p>
    </div>
  </div>
  <br />
  <div class="btns">
    <a
      href="/listings/<%= listing._id %>/edit"
      class="btn btn-dark col-1 offset-3 edit-btn"
      >Edit</a
    >
    <form method="post" action="/listings/<%= listing._id %>?_method=DELETE">
      <button class="btn btn-dark offset-5">Delete</button>
    </form>
  </div>
</div>
```

### **Styling in `styles.css`**

```css
/* Show Page */
.show-img {
  height: 30vh;
}

.btns {
  display: flex;
}

.show-card {
  padding-left: 0;
  padding-right: 0;
}
```

---
