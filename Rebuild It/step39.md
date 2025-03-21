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

`.env` file:

```plaintext
ATLASDB_URL=mongodb+srv://dibyajyotipramanick0750:efutfke18LhPmvLS@cluster0.x1ec5.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0
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

**Updated Code:**

```javascript
async function main() {
  await mongoose.connect(dbUrl);
}
```

Now, Mongoose will establish a connection with the **cloud database on MongoDB Atlas** instead of the local system.

---

### **6. Starting the Server & Checking the Database**

After starting the server, if we navigate to the `/listings` route, we will see that **nothing is displayed**!

#### **Why is the Database Empty?**

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
