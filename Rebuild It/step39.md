### **MongoDB Atlas Integration**

#### **1. Setting Up MongoDB Atlas**

After signing up on MongoDB Atlas, the first step is to **build a cluster**.

A **cluster** is essentially a storage space allocated for our database. Once the cluster is created, we move to the next step.

#### **2. Connecting the Local Project to MongoDB Atlas**

To connect our local project with this online database:

- Click on the **Connect** button of your respective cluster.
- A **connection string** will be generated, which we need to add to our application code.
- The placeholder `db_password` in the string must be replaced with our actual password.

#### **3. Storing the Connection String Securely**

Instead of writing the connection string directly in the code, we store it in a **.env file** for security purposes.

```plaintext
ATLASDB_URL=mongodb+srv://dibyajyotipramanick0750:OAtR75on0RL47uxE@cluster0.x1ec5.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0
```

#### **4. Updating `app.js` to Use MongoDB Atlas**

Previously, we were using a **local MongoDB connection** like this:

```javascript
// const MONGO_URL = "mongodb://127.0.0.1:27017/wanderlust";
```

Now, we replace it with the **environment variable** for MongoDB Atlas:

```javascript
const dbUrl = process.env.ATLASDB_URL;
```

#### **5. Connecting to MongoDB Atlas in the Code**

We need to modify the existing **async function** to connect to Atlas instead of the local database:

```javascript
async function main() {
  await mongoose.connect(dbUrl);
}
```

Now, Mongoose will establish a connection with the **cloud database on MongoDB Atlas** instead of the local system.

---

### **6. Starting the Server & Checking the Database**

After starting the server, if we navigate to the `/listings` route, we will see that **nothing is displayed**! Because:

1. Previously, our database was stored **locally**.
2. Now, we are connected to **MongoDB Atlas**, which is currently **empty**.

---

### **7. Populating the Database**

Since the database is empty, we need to add some data.

#### **Step 1: Creating a New User (User Signup)**

To initialize the database, we start by creating a **new user** via the signup process.

#### **Step 2: Verifying Data in MongoDB Atlas**

To check if the user data is stored correctly:

1. Go to the **MongoDB Atlas** page.
2. Open the **Clusters** section.
3. Click on **Browse Collections**.

Here, we will find a new **test** database created, containing our collections.

---

### **8. Understanding Collections in MongoDB Atlas**

Within **Browse Collections**, we will see three collections:

1. **listings** (currently empty)
2. **reviews** (currently empty)
3. **users** (contains one entry)

If we check the **users collection**, we will see a **demo user**—the one created during signup.

---

### **9. Creating a Listing**

Next, we create a **listing** in our application.

- As soon as a **new listing** is added, it will be stored in **MongoDB Atlas**.
- If we check the **listings collection** in Atlas, the new listing will appear.

---

### **10. Final Confirmation**

At this point, MongoDB Atlas is **fully connected**, and everything works as expected:  
✅ **Data is stored in the cloud database**.  
✅ **Users, listings, and reviews sync properly**.  
✅ **The application is smoothly interacting with the cloud database**.

---

## **Mongo Session Store**

### **Why Use a Mongo Session Store?**

- Previously, when using `express-session`, the session storage was set to **MemoryStore** by default.
- **MemoryStore is not suitable for production** because:
  - It **leaks memory** under most conditions.
  - It **does not scale** beyond a single process.
  - It is meant **only for debugging and development**.

#### **Official Warning from express-session Docs:**

> _"The default server-side session storage, MemoryStore, is purposely not designed for a production environment. It will leak memory under most conditions, does not scale past a single process, and is meant for debugging and developing."_

### **Alternative Session Stores**

- Express-session provides **multiple storage options**:
  - [List of compatible session stores](https://www.npmjs.com/package/express-session#compatible-session-stores)
- Since we are already using **MongoDB** as our primary database in the MERN stack, we will use **connect-mongo** for session storage.

#### **What is connect-mongo?**

- `connect-mongo` is a package that allows storing session-related information in MongoDB.
- Official documentation: [connect-mongo npm package](https://www.npmjs.com/package/connect-mongo).

---

## **Step-by-Step Implementation of Mongo Session Store**

### **1. Install `connect-mongo`**

```sh
npm i connect-mongo
```

### **2. Import Required Modules**

In `app.js` (or wherever sessions are managed):

```js
const session = require("express-session");
const MongoStore = require("connect-mongo");
```

### **3. Create the Mongo Store**

There are two ways to use `connect-mongo`:

1. **Directly inside the session middleware**
2. **Using an individual variable** (preferred for better readability)

```js
const store = MongoStore.create({
  mongoUrl: dbUrl, // MongoDB connection URL
  crypto: {
    secret: "mysupersecretcode", // Encrypt session data for security
  },
  touchAfter: 24 * 3600, // Update session only if modified within 24 hours
});

const sessionOptions = {...}
```

#### **Breakdown of Parameters:**

| Parameter       | Description                                       |
| --------------- | ------------------------------------------------- |
| `mongoUrl`      | MongoDB database URL (Atlas or local)             |
| `crypto.secret` | Secret key to encrypt session data                |
| `touchAfter`    | Prevents unnecessary session updates (in seconds) |

### **4. Handling Errors in Session Store**

```js
store.on("error", (err) => {
  console.log("ERROR in MONGO SESSION STORE", err);
});
```

- This ensures that any errors in the session store are logged for debugging.

### **5. Configure express-session to Use Mongo Store**

```js
const sessionOptions = {
  store, // Store session in MongoDB
...}
```

#### **Understanding `cookie.maxAge`**

- **Default session expiration** is **14 days**.
- The session expires if a user remains inactive for **14 days**.

---

## **How to Verify if Sessions Are Stored?**

1. Start the server:
   ```sh
   nodemon app.js
   ```
2. Log in as a user.
3. Check **MongoDB Atlas** → **Collections** → Find a new `sessions` collection.
4. If a new user signs up and creates a listing, the session information will be updated.
5. The `last modified` and `expires` fields in the session store will change accordingly.

---

## **Final Notes**

- By default, **session data is stored in MongoDB for 14 days**.
- The session **remains active even after refreshing the page**.
- **Sessions update only when necessary** due to `touchAfter`, preventing unnecessary writes.
- If the user remains inactive for **14 days**, they are **logged out automatically**.

---

With this setup, our **session management is now production-ready using MongoDB**. Next, we will proceed with the **deployment process**.
