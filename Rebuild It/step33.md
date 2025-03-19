# **Geocoding**

## **Objective**

We aim to implement a small functionality where, whenever we add a new listing and enter a location, geocoding will convert the location into geographical coordinates (latitude and longitude) and print them. Once we achieve this, we will display these coordinates on a map.

To accomplish this, we will use the **Mapbox Geocoding API**:  
🔗 [Mapbox Geocoding API Documentation](https://docs.mapbox.com/api/search/geocoding/)

## **Understanding Geocoding**

Geocoding is of two types:

1. **Forward Geocoding** – Converts a place name (e.g., "New Delhi, India") into coordinates.
2. **Reverse Geocoding** – Converts coordinates into a human-readable address.

Since we need to obtain coordinates from a location name, we will use **Forward Geocoding**.

The API for forward geocoding is:

```
https://api.mapbox.com/search/geocode/v6/forward?q={search_text}
```

where `{search_text}` is the location we enter.

## **Using Mapbox Geocoding SDK in JavaScript**

Instead of manually making API requests, Mapbox provides an official **JavaScript SDK** to simplify geocoding.  
🔗 [Mapbox SDK GitHub Repository](https://github.com/mapbox/mapbox-sdk-js)

This SDK eliminates the need for handling raw API requests and provides ready-to-use functions with examples.  
🔗 [Mapbox SDK Documentation](https://github.com/mapbox/mapbox-sdk-js/blob/main/docs/services.md)

The function we need for forward geocoding:  
🔗 [Forward Geocode in SDK](https://github.com/mapbox/mapbox-sdk-js/blob/main/docs/services.md#forwardgeocode)

Example code for **forward geocoding**:

```javascript
geocodingClient
  .forwardGeocode({
    query: "Paris, France",
    limit: 2,
  })
  .send()
  .then((response) => {
    const match = response.body;
  });
```

This code searches for **"Paris, France"**, limits results to **2**, and retrieves matching locations.

---

## **Installing and Setting Up the SDK**

### **Step 1: Install Mapbox SDK**

Run the following command in the project directory:

```bash
npm install @mapbox/mapbox-sdk
```

### **Step 2: Import and Configure the SDK**

Inside our **controllers/listings.js** file, we first import the required services:

```javascript
const mbxGeocoding = require("@mapbox/mapbox-sdk/services/geocoding");
```

Next, we retrieve our **access token** from environment variables:

```javascript
const mapToken = process.env.MAP_TOKEN;
```

Using this token, we initialize a geocoding client:

```javascript
const geocodingClient = mbxGeocoding({ accessToken: mapToken });
```

This **geocodingClient** allows us to send geocoding requests.

---

## **Implementing Geocoding in New Listing Creation**

Now, we modify the **createListing** method inside `controllers/listings.js` to fetch coordinates whenever a new listing is added.

### **Step 1: Fetch Coordinates from Geocoding API**

We call `forwardGeocode` inside our `createListing` function:

```javascript
module.exports.createListing = async (req, res, next) => {
  let response = await geocodingClient
    .forwardGeocode({
      query: "New Delhi, India", // Hardcoded for now
      limit: 1, // We only need one best match
    })
    .send();

  console.log(response);
  res.send("done!");
};
```

- **`query`** – The location we want to geocode.
- **`limit: 1`** – Ensures only one set of coordinates is returned.

---

## **Testing the API**

After adding a new listing, we should see `"done!"` in the browser and a response object in the console. Ignore the errors.

This **response** is a large object containing geocoding data. The most important part is stored inside `response.body.features`.

### **Step 2: Extract Coordinates from Response**

Since `response.body.features` is an **array**, we extract the first object:

```javascript
console.log(response.body.features[0]);
```

And since we limited results to **1**, we get a single object.

### **Step 3: Extract Only the Coordinates**

Inside the `geometry` field, we have:

To print only the coordinates:

```javascript
console.log(response.body.features[0].geometry);
```

This outputs:

```json
{ "type": "Point", "coordinates": [77.208773, 28.613927] }
```

which are the **longitude and latitude** of New Delhi.

---

## **Dynamic Geocoding Based on User Input**

Currently, we hardcoded `"New Delhi, India"` inside `query`. Instead, we should use the **location entered by the user** when submitting the form.

### **Step 1: Modify Code to Use User Input**

```javascript
query: req.body.listing.location;
```

Now, whenever a user enters a location in the **new listing form**, our backend will automatically fetch its coordinates.

---

## **Next Steps**

- Now that we have the **coordinates**, the next step is to **display them on a map**.

---
