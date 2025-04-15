## **Getting Started with Maps**

### **Initial Setup**

1. Create an account on **Mapbox**.
2. Copy your **default public token**.
3. Save the token in your **.env** file.

### **Using Mapbox GL JS**

To display a map, we will refer to the **Mapbox documentation**:  
[Mapbox Docs](https://docs.mapbox.com/)

Specifically, we will use the **Mapbox GL JS** library.

Mapbox provides multiple examples:  
[Mapbox GL JS Examples](https://docs.mapbox.com/mapbox-gl-js/example/)

The most basic functionality we need is **Displaying a map on a webpage**:  
[Simple Map Example](https://docs.mapbox.com/mapbox-gl-js/example/simple-map/)

We'll follow this example to display our first map inside our `show.ejs` file.

---

## **Adding Mapbox to Our Project**

### **Step 1: Include Mapbox Scripts & Styles**

We need to add **two additional links** in our `boilerplate.ejs` file inside the `<head>` tag:

```html
    <link
      href="https://api.mapbox.com/mapbox-gl-js/v3.11.0/mapbox-gl.css"
      rel="stylesheet"
    />
    <script src="https://api.mapbox.com/mapbox-gl-js/v3.11.0/mapbox-gl.js"></script>
  </head>
```

---

### **Step 2: Add a Container for the Map**

Inside `show.ejs`, add a `<div>` for the map **after All Reviews**:

```html
      <% } %>
    </div>
  </div>

  <div class="col-8 offset-3 mb-3">
    <h3>Where you'll be</h3>
    <div id="map"></div>
  </div>
</div>
```

If there are no reviews, we don’t need to display "All Reviews." To handle this dynamically, update the reviews section like this:

```html
<% } %> <% if(listing.reviews.length > 0) { %>
    <div class="row">
      <p><b>All reviews</b></p>
      ...
          </form>
        </div>
      </div>
      <% } %>
    </div>
    <% } %>
  </div>
```

Also, modify the `<hr>` tag logic:

```html
<% if(currUser) { %>
<hr />
```

---

### **Step 3: Add Mapbox Initialization Script**

At the **end of `show.ejs`**, we initialize the map with our Mapbox token:

```html
<script>
  let mapToken = "<%= process.env.MAP_TOKEN %>";
  console.log(mapToken);
  mapboxgl.accessToken = mapToken;
  const map = new mapboxgl.Map({
    container: "map", // container ID
    style: "mapbox://styles/mapbox/streets-v12", // style URL
    center: [88.3629, 22.5744], // starting position [lng, lat]
    zoom: 9, // starting zoom level
  });
</script>
```

After restarting the server, we inspect the `show.ejs` page. The access token appears in the console, but the map does not render yet. This is because **we need to add CSS styling for the map container**.

---

### **Step 4: Styling the Map**

In `public/css/styles.css`, add the following CSS:

```css
/* Map */
#map {
  aspect-ratio: 16 / 9;
  width: 100%;
  max-width: 800px;
  margin: auto;
  border-radius: 8px;
}
```

---

## **Separating Map Logic into `map.js`**

Instead of keeping the entire script inside `show.ejs`, we create a separate JavaScript file inside `public/js/` called `map.js`.

Move the following logic from `show.ejs` to `map.js`:

```js
let mapToken = process.env.MAP_TOKEN;
console.log(mapToken);
mapboxgl.accessToken = mapToken;

const map = new mapboxgl.Map({
  container: "map",
  style: "mapbox://styles/mapbox/streets-v12",
  center: [88.3629, 22.5744],
  zoom: 9,
});
```

However, we **can't directly access environment variables in frontend files like `public/js/map.js`**, because those files run in the browser and process.env is only available on the server side.

---

## **Fixing the Environment Variable Issue**

**1. Pass the token to your EJS view from the controller**:

```js
res.render("wallpapers/show.ejs", {
  wallpaper,
  mapToken: process.env.MAP_TOKEN,
});
```

**2. Embed the token in a script tag inside `show.ejs` before loading `map.js`**:

```html
<% layout("/layouts/boilerplate") %>

<script>
  window.mapToken = "<%= mapToken %>";
</script>
```

This sets a global variable `window.mapToken` that our frontend JS can read.

**3. Update `public/js/map.js` to use that token**:

```js
mapboxgl.accessToken = window.mapToken;
```

**4. Now, include the `map.js` script at the bottom of `show.ejs`**:

```js
<script src="/js/map.js"></script>
```

---

## **Next Steps**

Currently, our map always centers at **Kolkata** (`[88.3629, 22.5744]`). However, we need the map to **center based on the listing’s stored location**. That is what we will work on next.
