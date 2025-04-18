### **Storing Coordinates**

Before using coordinates, we need to store them in the database. The first question that might come to mind is: **How do we save location coordinates in the database?**

If we look at our listing model, we can create a new object inside it to store coordinates. This object will be an **array of numbers**, where two values will be stored—**longitude and latitude**. Additionally, we will set the `required` property to `true`, ensuring that these values are always stored.

```js
coordinates: {
    type: [Number],
    required: true,
},
```

---

### **GeoJSON Format – A Better Approach**

If we want to enhance this approach, we can use the **GeoJSON format**.

#### **What is GeoJSON?**

GeoJSON is a specialized format in **Mongoose** for storing **geographic points and polygons**. MongoDB has **excellent support** for geospatial queries when using GeoJSON.

### **Why Use GeoJSON?**

Storing coordinates in **GeoJSON format** provides a **standardized** structure, ensuring consistency across the database. This format also makes it **easier to access and process geometric data**.

[More details here](https://mongoosejs.com/docs/geojson.html#using-geojson)

### **How to Store Coordinates in GeoJSON Format?**

In the **GeoJSON format**, we must:

1. Specify the **type** of coordinate data (which will be `"Point"`).
2. Store **longitude and latitude** inside an array.

Thus, in our schema, we will modify the structure to save latitude and longitude inside an object with the **GeoJSON format**. This includes defining `"Point"` as the type and maintaining an array for coordinates.

```js
geometry: {
    type: {
        type: String, // This must be "Point"
        enum: ["Point"], // Ensures that the type is always 'Point'
        required: true,
    },
    coordinates: {
        type: [Number],
        required: true,
    },
},
```

---

### **Storing GeoJSON Data in the Database**

Now, we need to store the **geometry** object inside our **database** when creating a new listing.

In our **controllers/listings.js** file, we will ensure that the newListing has a `geometry` field, which will be assigned data from an external API response.

```js
newListing.geometry = response.body.features[0].geometry;
```

- This **value comes from Mapbox**, which provides the **geographical coordinates** for the location entered by the user.
- The extracted coordinates will then be stored **inside our database** along with the listing data.

---

### **Full Code for Creating a Listing with Coordinates**

```js
module.exports.createListing = async (req, res, next) => {
  let response = await geocodingClient
    .forwardGeocode({
      query: req.body.listing.location,
      limit: 1,
    })
    .send();

  let url = req.file.path;
  let filename = req.file.filename;

  const newListing = new Listing(req.body.listing);
  newListing.owner = req.user._id;
  newListing.image = { url, filename };

  newListing.geometry = response.body.features[0].geometry;

  let savedListing = await newListing.save();
  console.log(savedListing);

  req.flash("success", "New Listing Created!");
  res.redirect("/listings");
};
```

---

### **Example of a Saved Listing in the Database**

Let's test this by creating a new listing and setting the location to **New Delhi, India**.

After adding the listing, our database will store the following information:

```json
{
  "title": "Testing Coordinates",
  "description": "xoxo",
  "image": {
    "url": "https://res.cloudinary.com/dqw6dicdi/image/upload/v1742443159/WanderLust_DEV/aajecovhmo2cg2llkzsb.jpg",
    "filename": "WanderLust_DEV/aajecovhmo2cg2llkzsb"
  },
  "price": 1200,
  "location": "New Delhi, India",
  "country": "India",
  "reviews": [],
  "geometry": { "type": "Point", "coordinates": [77.20877, 28.613928] },
  "_id": "67db931b025535e1ceed91d1",
  "owner": "67d7ab278535a617aa488f28",
  "__v": 0
}
```

---

### **Conclusion**

- We successfully **stored location coordinates** in our database using **GeoJSON format**.
- We used **Mapbox** to fetch coordinates based on user-entered locations.
- The **geometry** field in the database now contains a **"Point"** type along with **longitude and latitude**.
- This setup ensures that we can perform **geospatial queries** efficiently.

#### **Next Steps**

In the next steps, we will use these **coordinates** to display **custom locations on the map** in our application.
