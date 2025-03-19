## **Getting Started with Maps**

### **Initial Setup**

1. Create an account on **Mapbox**.
2. Copy your **default public token**.
3. Save the token in your **.env** file.

### **Choosing a Map Style**

Mapbox provides various map styles, one of which is the **"streets"** style. We will use this style for our project.

### **Using Mapbox GL JS**

To display a map, we will refer to the **Mapbox documentation**:  
🔗 [Mapbox Docs](https://docs.mapbox.com/)

Specifically, we will use **Mapbox GL JS**, a JavaScript library for rendering interactive maps:  
🔗 [Mapbox GL JS Guide](https://docs.mapbox.com/mapbox-gl-js/guides/)

Mapbox provides multiple examples:  
🔗 [Mapbox GL JS Examples](https://docs.mapbox.com/mapbox-gl-js/example/)

The most basic functionality we need is displaying a **map on a webpage**:  
🔗 [Simple Map Example](https://docs.mapbox.com/mapbox-gl-js/example/simple-map/)

We'll follow this example to display our first map inside our `show.ejs` file.

---

## **Adding Mapbox to Our Project**

### **Step 1: Include Mapbox Scripts & Styles**

We need to add **two additional links** in our `boilerplate.ejs` file inside the `<head>` tag:

```html
    <link
      href="https://api.mapbox.com/mapbox-gl-js/v3.10.0/mapbox-gl.css"
      rel="stylesheet"
    />
    <script src="https://api.mapbox.com/mapbox-gl-js/v3.10.0/mapbox-gl.js"></script>
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

```ejs
<div class="col-8 offset-3 mb-3">
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
  height: 400px;
  width: 80vh;
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
  container: "map", // container ID
  style: "mapbox://styles/mapbox/streets-v12", // style URL
  center: [88.3629, 22.5744], // starting position [lng, lat]
  zoom: 9, // starting zoom level
});
```

However, we **cannot directly access environment variables inside `public/js/map.js`** because **public JS files do not have access to `process.env` variables**.

---

## **Fixing the Environment Variable Issue**

Since public JS files do not have direct access to `process.env`, we define the **map token inside `show.ejs`** before loading `map.js`:

```html
<% layout("/layouts/boilerplate") %>
<script>
  const mapToken = "<%= process.env.MAP_TOKEN %>";
</script>
```

Now, modify `map.js` to use this global variable instead:

```js
mapboxgl.accessToken = mapToken;

const map = new mapboxgl.Map({
  container: "map",
  style: "mapbox://styles/mapbox/streets-v12",
  center: [88.3629, 22.5744],
  zoom: 9,
});
```

In `show.ejs`, include the `map.js` script **at the bottom**:

```html
<script src="/js/map.js"></script>
```

Since `show.ejs` loads first, `mapToken` is available when `map.js` executes.

---

## **Next Steps**

Currently, our map always centers at **Kolkata** (`[88.3629, 22.5744]`). However, we need the map to **center based on the listing’s stored location**. That is what we will work on next.

---
