### **Website Home Page Modifications**

#### **1. Changing the Homepage & Navbar**

- The "All Listings" page will now serve as the website's homepage.
- Remove the **Home** button from `views/includes/navbar.ejs`:
  ```html
  <!-- <a class="nav-link" href="/">Home</a> -->
  ```
- Rename **All Listings** to **Explore**:
  ```html
  <a class="nav-link" href="/listings">Explore</a>
  ```
- Make the website icon redirect to `/listings`:
  ```html
  <a class="navbar-brand" href="/listings"
    ><i class="fa-regular fa-compass"></i
  ></a>
  ```
- Move the **Add New Listing** button to the right and rename it to **Airbnb your home**:

  ```ejs
  <div class="navbar-nav">
    <a class="nav-link" href="/listings">Explore</a>
  </div>

  <div class="navbar-nav ms-auto">
    <a class="nav-link" href="/listings/new">Airbnb your home</a>
  ```

- Make **Sign up**, **Log in**, and **Log out** bold:
  ```html
  <a class="nav-link" href="/signup"><b>Sign up</b></a>
  <a class="nav-link" href="/login"><b>Log in</b></a>
  <% } %> <% if(currUser){ %>
  <a class="nav-link" href="/logout"><b>Log out</b></a>
  ```
- Comment out the root route in `app.js`:
  ```js
  // app.get("/", (req, res) => {
  //   res.send("Hi, I am root");
  // });
  ```

---

### **2. Adding Filters for Listings**

#### **Removing "All Listings" Header**

- Comment out the following lines in `index.ejs`:
  ```html
  <!-- <h3>All listings</h3> -->
  <!-- <form method="get" action="/listings/new">
      <button>Create New Listing</button>
  </form> -->
  ```

#### **Adding the Filter Section**

- Create a `div` for filters inside the layout:
  ```html
  <% layout("/layouts/boilerplate") %>
  <div id="filters"></div>
  ```
- Inside this `div`, create another `div` to hold filter items:
  ```html
  <div id="filters">
    <div class="filter"></div>
  </div>
  ```

#### **Adding Individual Filters**

Each filter consists of an icon and a name:

```html
<div class="filter">
  <div><i class="fa-solid fa-fire"></i></div>
  <p>Trending</p>
</div>
```

#### **Adding Multiple Filters**

To start, we will add two filters:

```html
<div id="filters">
  <div class="filter">
    <div><i class="fas fa-fire"></i></div>
    <p>Trending</p>
  </div>

  <div class="filter">
    <div><i class="fa-solid fa-bed"></i></div>
    <p>Rooms</p>
  </div>
</div>
```

---

### **3. Styling the Filters**

#### **Aligning Filters in a Row**

```css
#filters {
  display: flex;
  flex-wrap: wrap;
}
```

#### **Centering Content and Adding Spacing**

```css
.filter {
  text-align: center;
  margin-right: 2rem;
  margin-top: 2rem;
}
```

#### **Adding a Hover Effect Like Airbnb**

```css
.filter {
  text-align: center;
  margin-right: 2rem;
  margin-top: 2rem;
  opacity: 0.7;
}

.filter:hover {
  opacity: 1;
  cursor: pointer;
}
```

---

### **4. Adding More Filters**

You can add multiple filters in the same format:

```html
<div class="filter">
  <div><i class="fa-solid fa-mountain-city"></i></div>
  <p>Top cities</p>
</div>

<div class="filter">
  <div><i class="fa-solid fa-mountain"></i></div>
  <p>Mountains</p>
</div>

<div class="filter">
  <div><i class="fa-brands fa-fort-awesome"></i></div>
  <p>Castles</p>
</div>

<div class="filter">
  <div><i class="fa-solid fa-person-swimming"></i></div>
  <p>Amazing Pools</p>
</div>

<div class="filter">
  <div><i class="fa-solid fa-campground"></i></div>
  <p>Camping</p>
</div>

<div class="filter">
  <div><i class="fa-solid fa-cow"></i></div>
  <p>Farms</p>
</div>

<div class="filter">
  <div><i class="fa-regular fa-snowflake"></i></div>
  <p>Arctic</p>
</div>

<div class="filter">
  <div><i class="fa-solid fa-igloo"></i></div>
  <p>Domes</p>
</div>

<div class="filter">
  <div><i class="fa-solid fa-ship"></i></div>
  <p>Boats</p>
</div>
```

#### **Reducing Font Size of Filter Labels**

```css
.filter p {
  font-size: 0.8rem;
}
```

---

Alright, so for now, when we click on these filters, nothing actually happens. However, we can later build a clickable functionality.

### **How to Implement the Filtering System?**

- Currently, our **Listing Model** does not have any **category-based division**.
- We can modify the **schema** of our Listing Model to include a **category field** of type **string**.
- To ensure only specific categories are allowed, we can define an **enum** for this field.
- This enum will contain the same **categories** as the filters we are displaying—such as **Mountains, Rooms, Farms, Arctic, etc.**
- When a user creates a new listing, they will select a **category** from a **dropdown menu** in the form.

### **How Will Filtering Work?**

- Once we have categories in place, we can apply a **backend filter**.
- Example: If a user clicks on the **Mountains** filter:
  1. A request will be sent to the backend to fetch **only those listings** where the **category** is **Mountains**.
  2. The backend will return **filtered listings** that match the selected category.
  3. We can either:
     - Redirect the user to a **new page** displaying the filtered listings.
     - Replace the existing listings on the **index page** with the filtered results.

### **Why Should You Build This Feature Yourself?**

- This will be a great learning experience, reinforcing all the **concepts** we have covered.
- You won’t be learning anything new here, but rather **applying** your existing knowledge.
- The **frontend for filters is already built**—you only need to **implement the backend**.

This will deepen your **understanding** and improve your **problem-solving skills**. Try building it yourself!
