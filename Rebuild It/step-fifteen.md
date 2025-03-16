# **Authentication & Authorization with Passport.js**

## **Introduction to Passport.js**

Passport.js is an authentication middleware for Node.js that supports multiple authentication strategies. We are starting with the **passport-local** strategy, which is the simplest way to implement authentication using a username and password.

### **Installation of Required Packages**

To use Passport and its strategies, we need to install the necessary npm packages:

1. **Passport (Core Library)**

   ```
   npm i passport
   ```

   - [Official Documentation](https://www.npmjs.com/package/passport)

2. **Passport-Local Strategy**

   ```
   npm i passport-local
   ```

   - This package helps us implement local authentication (username/password).
   - [Documentation](https://www.npmjs.com/package/passport-local)

3. **Passport-Local-Mongoose**
   ```
   npm i passport-local-mongoose
   ```
   - This package integrates Passport.js with **Mongoose**, providing built-in functionality for user authentication.
   - It simplifies user registration, hashing passwords, and authentication methods.
   - [Documentation](https://www.npmjs.com/package/passport-local-mongoose)

---

## **Why Use Passport-Local-Mongoose?**

- When using **MongoDB** with **Mongoose**, passport-local-mongoose makes authentication easier by:
  - Automatically handling **username storage**
  - **Hashing and salting passwords**
  - Providing built-in methods for user authentication
- If we were using SQL or another database, **passport-local** alone would be enough. However, for MongoDB, passport-local-mongoose provides additional features.

---

## **Multiple Authentication Strategies**

- We can implement multiple authentication strategies in a single application.
- Example:
  - **Username & Password Login**
  - **Google OAuth Login**
  - **GitHub Login**
- But for now, we are focusing only on **passport-local**.

---

# **Creating the User Model**

Before implementing the authentication logic, we need to define the **User Model**.

### **Basic User Model (user.js)**

- A user should have at least:

  - `username`
  - `password`
  - `email`

- **Creating the file:**  
  Inside the `models/` directory, create a new file named `user.js`.

- **Code:**

  ```javascript
  const mongoose = require("mongoose");
  const { Schema } = mongoose;
  const passportLocalMongoose = require("passport-local-mongoose");

  const userSchema = new Schema({
    email: {
      type: String,
      required: true,
    },
  });

  userSchema.plugin(passportLocalMongoose);

  module.exports = mongoose.model("User", userSchema);
  ```

---

## **Understanding Passport-Local-Mongoose**

- The `passport-local-mongoose` package **automatically** adds:
  - `username` field
  - `hash` (hashed password)
  - `salt` (used for password hashing)
- We **don’t need to manually define** `username` or `password` fields. The plugin **handles it for us**.

### **Why use the `plugin()` method?**

- It **automatically integrates authentication logic** into the schema.
- It provides **pre-built methods** for authentication, such as:
  - `setPassword()`
  - `changePassword()`
  - `authenticate()`
- **Static Methods** (for schema-wide operations) are also included:  
  [passport-local-mongoose static-methods](https://www.npmjs.com/package/passport-local-mongoose#static-methods)

---

### **Configuring Strategy**

To configure authentication using **passport-local**, we need to integrate **Passport.js** into our Express app and configure it with necessary settings.

#### **1. Requiring Necessary Modules**

In `app.js`, import the required modules:

```js
const passport = require("passport");
const LocalStrategy = require("passport-local");
const User = require("./models/user.js");
```

- `passport`: The core Passport library for authentication.
- `LocalStrategy`: Passport strategy for username-password authentication.
- `User`: The User model.

#### **2. Using Sessions in Authentication**

To keep users logged in across different pages, we need **sessions**. Passport uses sessions to store user authentication data.

##### **Adding Middleware for Passport and Sessions**

These middleware functions must be added after session and flash middleware:

```js
app.use(passport.initialize());
app.use(passport.session());
```

- `passport.initialize()`: Initializes Passport for authentication.
- `passport.session()`: Enables persistent login sessions across multiple requests.

##### **Why Use Sessions?**

When a user logs in, we want them to remain authenticated while navigating the website. A **session** ensures that a user does not have to log in repeatedly for every request.

---

#### **3. Configuring Passport to Use Local Strategy**

The next step is to tell Passport to use the **Local Strategy** with our `User` model.

```js
passport.use(new LocalStrategy(User.authenticate()));
```

- `new LocalStrategy()`: Creates an instance of Local Strategy for authentication.
- `User.authenticate()`: A **static method** added by `passport-local-mongoose` that handles user authentication automatically.

---

#### **4. Serializing and Deserializing Users**

We need to tell Passport how to **store** user information in a session (**serialize**) and how to **retrieve** it (**deserialize**) when needed.

```js
passport.serializeUser(User.serializeUser());
passport.deserializeUser(User.deserializeUser());
```

##### **Explanation:**

- **Serialize User**: Converts the user object into a session-storable format.
- **Deserialize User**: Retrieves the user object from the session.
- `passport-local-mongoose` provides these methods, so we don’t have to manually write logic for storing and retrieving users.

> **Why are `serializeUser` and `deserializeUser` needed?**
>
> - Once a user logs in, Passport **serializes** (saves) the user ID in the session.
> - On subsequent requests, Passport **deserializes** (retrieves) the user from the session.

---

#### **5. Creating a Demo User for Testing**

To test if everything is set up correctly, we create a **demo user** by adding the following route before other routes in `app.js`:

```js
app.get("/demouser", async (req, res) => {
  let fakeUser = new User({
    email: "student@gmail.com",
    username: "delta-student",
  });

  let registeredUser = await User.register(fakeUser, "helloworld");
  res.send(registeredUser);
});
```

##### **Explanation:**

- **Step 1:** Create a new user instance with an email and username.
- **Step 2:** Use `User.register()` to save the user in the database.
  - This method **automatically hashes** the password.
  - It **ensures the username is unique** before saving.
- **Step 3:** Send the registered user data as a response.

##### **Testing the Demo User**

1. Start the server:
   ```
   nodemon app.js
   ```
2. Visit in browser:
   ```
   localhost:8080/demouser
   ```
3. A new user should be created in the database.

---

### **Key Takeaways**

✔ **Sessions** allow users to stay logged in across different pages.  
✔ **passport.initialize()** and **passport.session()** are required for authentication.  
✔ **Local Strategy** is implemented using `passport.use(new LocalStrategy(User.authenticate()))`.  
✔ **serializeUser()** and **deserializeUser()** manage session storage and retrieval.  
✔ **User.register()** creates and stores users in the database with a hashed password.

---
