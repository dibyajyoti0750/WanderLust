## 1. Project Setup

1. Create a project folder (`MajorProject`) and open it in VS Code.
2. Initialize `npm` and install dependencies:
   ```sh
   npm init -y
   npm i express
   npm i ejs
   npm i mongoose
   ```
3. Create `app.js`:
   ```sh
   touch app.js
   ```
4. Update `package.json`:
   ```json
   "main": "app.js"
   ```

## 2. Setting Up Express Server

Edit `app.js`:

```js
const express = require("express");
const app = express();
const mongoose = require("mongoose");

app.get("/", (req, res) => {
  res.send("Hi, I am root");
});

app.listen(8080, () => {
  console.log("Server is listening on port 8080");
});
```

### Run the server:

```sh
nodemon app.js
```

Check if it works by visiting `http://localhost:8080`.

---

## 3. MongoDB Connection

### Verify MongoDB is running:

```sh
mongosh
```

If MongoDB is running, proceed to connect it in `app.js`:

```js
const MONGO_URL = "mongodb://127.0.0.1:27017/wanderlust";

main()
  .then(() => {
    console.log("Connected to DB");
  })
  .catch((err) => {
    console.log(err);
  });

async function main() {
  await mongoose.connect(MONGO_URL);
}
```

---

## 4. Creating the Listing Model

A "Listing" represents a place with details like name, location, price, facilities, etc.

1. Create a `models` folder.
2. Inside `models`, create `listing.js`:

   ```js
   const mongoose = require("mongoose");
   const Schema = mongoose.Schema;

   const listingSchema = new Schema({
     title: { type: String, required: true },
     description: String,
     image: {
       type: String,
       default:
         "https://images.unsplash.com/photo-1625505826533-5c80aca7d157?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MTJ8fGdvYXxlbnwwfHwwfHx8MA%3D%3D&auto=format&fit=crop&w=800&q=60",
       set: (v) =>
         v === ""
           ? "https://images.unsplash.com/photo-1625505826533-5c80aca7d157?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MTJ8fGdvYXxlbnwwfHwwfHx8MA%3D%3D&auto=format&fit=crop&w=800&q=60"
           : v,
     },
     price: Number,
     location: String,
     country: String,
   });

   const Listing = mongoose.model("Listing", listingSchema);
   module.exports = Listing;
   ```

---

## 5. Testing the Model

### 1. Import the model in `app.js`:

```js
const Listing = require("./models/listing.js");
```

### 2. Create a test route:

```js
app.get("/testListing", async (req, res) => {
  let sampleListing = new Listing({
    title: "My New Villa",
    description: "By the beach",
    price: 1200,
    location: "Calangute, Goa",
    country: "India",
  });

  await sampleListing.save();
  console.log("Sample was saved");
  res.send("Test successful");
});
```

### 3. Test the route:

- Start the server: `nodemon app.js`
- Visit `http://localhost:8080/testListing`
- Check MongoDB:
  ```sh
  show dbs
  use wanderlust
  show collections
  db.listings.find()
  ```

Your database and listings collection should be created, and the sample listing should be visible!

---

## Step 6: Sample Data Initialization

### 1. Download the Sample Data

- Download the `data.js` file from this link:
  [Google Drive - Sample Data](https://drive.google.com/drive/folders/1zzK3cxmbs7jbEKJUf5CAwnGh1PYV5VU7?usp=drive_link)

- [Project Repository](https://github.com/apna-college/wanderlust)

### 2. Set Up the Initialization Files

- Create an `init` folder inside `MajorProject/`
- Paste the `data.js` file into `MajorProject/init/`
- Create a new file inside `init/` named `index.js`

### 3. Write Initialization Logic in `index.js`

- Inside `init/index.js`, add:

  ```js
  const mongoose = require("mongoose");
  const initData = require("./data.js");
  const Listing = require("../models/listing.js");

  const MONGO_URL = "mongodb://127.0.0.1:27017/wanderlust";

  main()
    .then(() => console.log("Connected to DB"))
    .catch((err) => console.log(err));

  async function main() {
    await mongoose.connect(MONGO_URL);
  }

  const initDB = async () => {
    await Listing.deleteMany({});
    await Listing.insertMany(initData.data);
    console.log("DB was initialized");
    mongoose.connection.close();
  };

  initDB();
  ```

### 4. Run the Initialization Script

- Navigate to the `init` folder:

  ```sh
  cd init
  ```

- Run the script:

  ```sh
  node index.js
  ```

- If everything works, the console should display:

  ```sh
  Connected to DB
  DB was initialized
  ```

- After execution, return to the main project folder:

  ```sh
  cd ..
  ```

---

### 5. Testing & Verification

**Verify MongoDB Connection**

```sh
mongosh
use wanderlust
db.listings.find()
```

If the data is inserted successfully, you should see multiple listings in the database.

---

## Next Steps: CRUD Operations

Now that the sample data is initialized, the next step is to set up the **index route** to display all listings.

---
