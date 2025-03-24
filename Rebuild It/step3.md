## **1. Setting Up EJS-Mate for Layouts**

First, we install `ejs-mate` using:

```bash
npm i ejs-mate
```

Then, in `app.js`, we require and configure it:

```js
const ejsMate = require("ejs-mate");
app.engine("ejs", ejsMate);
```

**What is `ejs-mate`?**
It helps us create layouts for templates, allowing us to avoid redundant code by defining reusable structures like the navbar and footer.

## **2. Creating a Boilerplate Layout**

We create a `layouts` folder inside `views` and add a file `boilerplate.ejs`.

#### **Why do we need a boilerplate?**

Every page in our website will share a common structure (like navbar and footer). Instead of repeating HTML on each page, we create a layout template.

#### **Using the Layout in EJS**

Inside any EJS template, we use:

```ejs
<% layout('boilerplate') -%>
```

This tells EJS that the current page should use the `boilerplate.ejs` layout.

#### **Boilerplate Structure (`boilerplate.ejs`)**

```html
<title>WanderLust</title>
  </head>
  <body>
    <h1>WanderLust</h1>
    <div class="container"><%- body %></div>
  </body>
```

Here, `<%- body %>` acts as a **placeholder** where the content of different pages will be injected.

## **3. Applying Layout to Pages**

#### **Example: `index.ejs`**

We remove unnecessary HTML and use:

```html
<% layout("/layouts/boilerplate") %>
<h3>All listings</h3>
<form method="get" action="/listings/new">
  <button>Create New Listing</button>
</form>
<ul>
  <% for(let listing of allListings) { %>
  <li><a href="/listings/<%= listing._id %>"><%= listing.title %></a></li>
  <% } %>
</ul>
```

Visiting the index page now shows the **common layout** with specific content for this page.

#### **Example: `show.ejs`**

```html
<% layout("/layouts/boilerplate") %>
<h3>Listing Details</h3>
<ul>
  <li><%= listing.title %></li>
  <li><%= listing.description %></li>
  <li>&#8377;<%= listing.price.toLocaleString("en-IN") %></li>
  <li><%= listing.location %></li>
  <li><%= listing.country %></li>
</ul>
<br />
<a href="/listings/<%= listing._id %>/edit">Edit this listing</a>
<br /><br />
<form method="post" action="/listings/<%= listing._id %>?_method=DELETE">
  <button>Delete this listing</button>
</form>
```

This ensures every page follows the **same structure without repeating code**.

## **4. Serving Static Files**

We create a `public` folder and configure it in `app.js`:

```js
app.use(express.static(path.join(__dirname, "/public")));
```

Inside `public`, we add a `css` folder and create `styles.css`:

```css
body {
  background-color: pink;
}
```

Now, instead of adding styles to every page separately, we include them in `boilerplate.ejs`:

```html
<link rel="stylesheet" href="/css/styles.css" />
```

This applies styles across all pages.

## **5. Adding a Navbar**

#### **Including Bootstrap**

Inside `boilerplate.ejs`, we add Bootstrap CSS and JS:

```html
<link
  href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
  rel="stylesheet"
/>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
```

**Proof that Bootstrap is working?** The font style changes.

#### **Creating `navbar.ejs`**

We create an `includes` folder inside `views` and add `navbar.ejs`:

```html
<nav class="navbar navbar-expand-lg bg-body-tertiary">
  <div class="container-fluid">
    <a class="navbar-brand" href="#">Navbar</a>
    <button
      class="navbar-toggler"
      type="button"
      data-bs-toggle="collapse"
      data-bs-target="#navbarNavAltMarkup"
    >
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navbarNavAltMarkup">
      <div class="navbar-nav">
        <a class="nav-link active" href="#">Home</a>
        <a class="nav-link" href="#">Features</a>
        <a class="nav-link" href="#">Pricing</a>
      </div>
    </div>
  </div>
</nav>
```

To **include the navbar** in `boilerplate.ejs`, we replace the `<h1>` tag with:

```ejs
<%- include("../includes/navbar.ejs") %>
```

Now, the navbar appears on every page.

#### **Enhancing the Navbar**

We add FontAwesome for icons:

```html
<link
  rel="stylesheet"
  href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.7.2/css/all.min.css"
/>
```

Then, update `navbar.ejs`:

```html
<nav class="navbar navbar-expand-md bg-body-light border-bottom sticky-top">
  <div class="container-fluid">
    <a class="navbar-brand" href="/"><i class="far fa-compass"></i></a>
    <button
      class="navbar-toggler"
      type="button"
      data-bs-toggle="collapse"
      data-bs-target="#navbarNavAltMarkup"
    >
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navbarNavAltMarkup">
      <div class="navbar-nav">
        <a class="nav-link" href="/">Home</a>
        <a class="nav-link" href="/listings">All Listings</a>
        <a class="nav-link" href="/listings/new">Add New Listing</a>
      </div>
    </div>
  </div>
</nav>
```

Since we already have **"Add New Listing"** in the navbar, we comment out the button in `index.ejs`:

```html
<!-- <form action="/listings/new" method="get">
<button>Create New Listing</button>
</form> -->
```

#### **Styling the Navbar (`styles.css`)**

```css
/* Navbar */
.navbar {
  height: 5rem;
  background-color: white;
}

.fa-compass {
  color: #fe424d;
  font-size: 2rem;
}

.nav-link {
  color: #222222 !important;
}
```

## **6. Adding a Footer**

We create `footer.ejs` inside `includes`:

```html
<footer>
  <div class="f-info">
    <div class="f-info-socials">
      <i class="fa-brands fa-square-facebook"></i>
      <i class="fa-brands fa-square-instagram"></i>
      <i class="fa-brands fa-square-x-twitter"></i>
    </div>
    <div class="f-info-brand">&copy; Wanderlust Private Limited</div>
    <div class="f-info-links">
      <a href="/privacy">Privacy</a>
      <a href="/terms">Terms</a>
    </div>
  </div>
</footer>
```

We include it in `boilerplate.ejs` before closing the `<body>` tag:

```ejs
<%- include("../includes/footer.ejs") %>
```

#### **Styling the Footer (`styles.css`)**

```css
/* Footer */
.f-info-links a {
  text-decoration: none;
  color: #222222;
}

.f-info-links a:hover {
  text-decoration: underline;
}

.f-info-links,
.f-info-socials,
.f-info-brand {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.f-info-socials i {
  font-size: 1.2rem;
  margin-right: 1rem;
}

.f-info {
  text-align: center;
  height: 8rem;
  background-color: #ebebeb;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: space-evenly;
}
```
