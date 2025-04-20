### **Overview**

We are implementing a **Marker Popup** in Mapbox, similar to how Airbnb displays a map with markers for property locations. When a user hovers over or clicks on the marker, a popup should appear displaying information about that location.

---

### **Step 1: Understanding Marker Popups**

- A **marker** is a visual representation of a location on the map.
- A **popup** is a small window that appears when a marker is clicked, displaying additional information.

📌 **Challenge:** Try replacing the marker with an icon instead of the default marker.

---

### **Step 2: Using the Mapbox Popup Component**

The official documentation is available here:  
🔗 [Mapbox Popup API](https://docs.mapbox.com/mapbox-gl-js/api/markers/#popup)  
🔗 [Popup Example](https://docs.mapbox.com/mapbox-gl-js/api/markers/#popup-example)

---

### **Step 3: Implementing the Popup**

To add a popup to a marker:

1. **Create a new popup instance**
2. **Set its location (longitude and latitude)**
3. **Define its content (HTML)**
4. **Attach it to a marker**
5. **Add it to the map**

#### **Code Example**

```js
const marker = new mapboxgl.Marker({ color: "red" })
  .setLngLat(coordinates)
  .setPopup(new mapboxgl.Popup({ offset: 25 }).setHTML("<h1>Hello World!</h1>"))
  .addTo(map);
```

📌 **What happens?**

- A red marker is placed at `coordinates`.
- When clicked, a popup appears with "Hello World!".

---

### **Step 4: Customizing Popup Content**

We can modify the popup content to display meaningful information:

#### **Example 1: Custom Text**

```js
.setPopup(
  new mapboxgl.Popup({ offset: 25 }).setHTML(
    "<h4>Exact location provided after booking.</h4>"
  )
);
```

#### **Example 2: Displaying Listing Location**

Instead of hardcoding text, we dynamically insert listing information.

**1. In `show.ejs`, save the listing data inside a script tag:**

```html
<script>
  const mapToken = "<%= process.env.MAP_TOKEN %>";
  const listing = <%- JSON.stringify(listing) %>;
</script>
```

📌 This makes `listing` data available for JavaScript.

**2. In `map.js`, use the listing coordinates for centering the map:**

```js
center: listing.geometry.coordinates,
```

**3. Set the marker’s location dynamically:**

```js
.setLngLat(listing.geometry.coordinates)
```

**4. Use listing location inside the popup:**

```js
.setPopup(
  new mapboxgl.Popup({ offset: 25 }).setHTML(
    `<h4>${listing.location}</h4><p>Exact location provided after booking.</p>`
  )
);
```

---

### **Step 5: Displaying Listing Title Instead of Location**

If you want the popup to show the listing's **title** instead of its location:

```js
.setPopup(
  new mapboxgl.Popup({ offset: 25 }).setHTML(
    `<h4>${listing.title}</h4><p>Exact location will be provided after booking.</p>`
  )
);
```

---

### **Final Thoughts**

- Now, when clicking a marker, a popup appears displaying either the **location** or **title**.
- You can **customize** the popup further with CSS and additional information.
- Challenge yourself by **replacing the marker with a custom icon** instead of the default one.
