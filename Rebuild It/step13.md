# **Using Sessions and Cookies**

## **1. Installing express-session**

To use sessions in an Express.js application, we need to install the `express-session` package first:

```sh
npm i express-session
```

After installing, we need to import and configure it in `app.js`:

```js
const session = require("express-session");

const sessionOptions = {
  secret: "mysupersecretcode",
  resave: false,
  saveUninitialized: true,
};

app.use(session(sessionOptions));
```

- `secret`: A key used to sign the session ID cookie. It should be kept secure.
- `resave`: If `false`, session will not be saved to the store on every request unless modified.
- `saveUninitialized`: If `true`, it will save new sessions even if they have not been modified.

## **2. Checking if Sessions are Working**

To verify whether sessions are working, follow these steps:

1. Start the server and navigate to a route that uses sessions (e.g., `/listings`).
2. Open the browser’s Developer Tools (`F12` or `Ctrl + Shift + I`).
3. Go to the **Application** tab.
4. Look for **Cookies** under **Storage**.
5. Check if a cookie named `connect.sid` is present.

If `connect.sid` is visible, it means sessions are working correctly. If not, something is wrong with the session configuration.

---

## **3. Adding Cookie Options**

The default session cookie settings are:

```js
{
  path: '/',
  httpOnly: true,
  secure: false,
  maxAge: null
}
```

By default:

- Cookies **do not have an expiry date** (they expire when the browser is closed).
- Cookies are **httpOnly** (cannot be accessed via JavaScript).
- `secure: false` means cookies are transmitted over both HTTP and HTTPS.

### **Modifying Cookie Expiry and Max Age**

Since cookies expire when the browser closes by default, we may want to specify an expiration time.

### **Setting Expiry Date**

We can manually set when the cookie should expire using `expires`:

```js
cookie: {
  expires: Date.now() + 7 * 24 * 60 * 60 * 1000,
}
```

Here’s what happens:

- `Date.now()` gets the current timestamp in milliseconds.
- `7 * 24 * 60 * 60 * 1000` converts 7 days into milliseconds.
- Adding both gives a timestamp for 7 days in the future.

### **Using maxAge**

Alternatively, we can set `maxAge`, which determines how long the cookie should last after being created.

```js
cookie: {
  maxAge: 7 * 24 * 60 * 60 * 1000,
}
```

This ensures the session cookie will persist for exactly 7 days.

### **Full Updated Session Configuration**

Here’s the complete session setup with cookie options:

```js
const sessionOptions = {
  secret: "mysupersecretcode",
  resave: false,
  saveUninitialized: true,
  cookie: {
    expires: Date.now() + 7 * 24 * 60 * 60 * 1000,
    maxAge: 7 * 24 * 60 * 60 * 1000,
    httpOnly: true,
  },
};

app.use(session(sessionOptions));
```

### **Why use `httpOnly: true`?**

Setting `httpOnly: true` ensures that the cookie **cannot be accessed via JavaScript**, protecting against cross-site scripting (XSS) attacks.

---
