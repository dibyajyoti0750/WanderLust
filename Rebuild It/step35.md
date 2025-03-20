## **Map Marker**

Now, we will display a custom location on our map using a **marker**. A map marker is used to pinpoint a particular location and represent it with a marker-like icon.

We want that whenever a location is added inside a listing, a marker should appear at that location, visually representing it. To achieve this, we will follow the official Mapbox documentation:  
🔗 [Mapbox Add a Marker Example](https://docs.mapbox.com/mapbox-gl-js/example/add-a-marker/)

### **Step 1: Copy the Marker Code**

First, we copy the example marker code and add it inside `map.js`, where we are displaying the map:

```js
const marker = new mapboxgl.Marker()
  .setLngLat([12.554729, 55.70651])
  .addTo(map);
```

The **latitude** and **longitude** values we pass inside `setLngLat` will be fetched from our listing's geometry, which stores the coordinates.

### **Step 2: Passing Coordinates from Backend to Frontend**

Since we cannot directly access these coordinates inside `map.js` (because it is a public JavaScript file), we will pass them from our **EJS template** just like we did with our `mapToken`.

In `show.ejs`, we will create a new variable called `coordinates` and assign it the value from our listing object:

```html
<script>
  const mapToken = "<%= process.env.MAP_TOKEN %>";
  const coordinates = <%= listing.geometry.coordinates %>;
</script>
```

However, if we try to access this variable directly, we will get an error. To fix this, we use the **JSON.stringify()** method to ensure the data is properly formatted:

```html
<script>
  const coordinates = <%- JSON.stringify(listing.geometry.coordinates) %>;
</script>
```

### **Step 3: Debugging the Coordinates**

Before creating the marker, we log the coordinates in `map.js` to check if they are being passed correctly:

```js
console.log(coordinates);

// Marker creation (commented for debugging)
// const marker = new mapboxgl.Marker()
//   .setLngLat([12.554729, 55.70651])
//   .addTo(map);
```

Now, when we refresh the **show page**, we should see an array of two numbers in the console, meaning we have successfully accessed our coordinates.

### **Step 4: Creating the Marker with Dynamic Coordinates**

Once we verify the coordinates, we replace the hardcoded values inside `setLngLat` with our dynamic `coordinates` variable:

```js
const marker = new mapboxgl.Marker().setLngLat(coordinates).addTo(map);
```

After refreshing, we should now see a **blue-colored default marker** appear on the map.

---

### **Step 5: Centering the Map on the Marker**

By default, the center of our map is **Kolkata** because we haven't changed the initial coordinates. This means old listings will not show a centered marker.

To fix this, we update the `center` property in `map.js` to use the **actual coordinates of the listing**:

```js
center: coordinates,
```

We can now add a **new listing** and test if the marker is appearing correctly.

---

### **Step 6: Customizing the Marker**

The default marker has several customizable properties. We can explore them in the API reference:  
🔗 [Mapbox Marker API](https://docs.mapbox.com/mapbox-gl-js/api/markers/)

Some available options include:

- **Anchor position**
- **Color customization**
- **Draggable property**

To change the marker color, we modify the `Marker` class by passing a configuration object:

```js
const marker = new mapboxgl.Marker({ color: "red" })
  .setLngLat(coordinates)
  .addTo(map);
```

Now, the marker will appear in **red** instead of the default blue color.

---
