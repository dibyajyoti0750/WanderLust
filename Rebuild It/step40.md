## **Deployment**

To deploy our project, we will use **Render**. First, go to [render.com](https://render.com) and sign up using Google.

#### **Update `package.json` Before Deployment**

Before deploying, we need to make a small change in our `package.json` file by specifying the Node.js version:

```json
{
  "engines": {
    "node": "22.14.0"
  }
}
```

If we do not specify the Node.js version, **Render** may throw errors during deployment.

---

## **Connecting Render with GitHub**

Before deploying, we need to convert our **WanderLust** project folder into a **Git** project.

### **Managing Secrets Securely**

We are using a **secret** in `sessionOptions` and `Mongo store`, which should not be hardcoded in the code. Instead, we will store it in an `.env` file:

```ini
SECRET=b269f46aacdcfb2889505fc17d7d89c6
```

Now, update our `app.js` file to use this secret:

```js
crypto: {
    secret: process.env.SECRET,
},
```

```js
const sessionOptions = {
  store,
  secret: process.env.SECRET,
  ...}
```

**Never push the `.env` file to GitHub** because it contains sensitive data.

---

## **Initializing Git and Creating a Repository**

### **Step 1: Initialize Git**

Run the following command in the project folder:

```sh
git init
```

### **Step 2: Ignore Unwanted Files**

Create a `.gitignore` file to prevent sensitive files from being pushed to GitHub:

```sh
touch .gitignore
```

Add the following entries in `.gitignore`:

```
.env
node_modules/
```

### **Step 3: Commit and Push Code to GitHub**

Run the following commands:

```sh
git add .
git commit -m "Add Project Files"
```

Now, create a new repository on GitHub called **WanderLustProject**, and then push the project:

```sh
git remote add origin <GitHub Repository Link>
git push -u origin main
```

---

## **Deploying on Render**

### **Step 1: Connect GitHub with Render**

1. Log in to **Render**.
2. Click on **New Web Service**.
3. Select **GitHub** as the source.
4. Under **Only select repositories**, choose **WanderLustProject** and click **Install**.

### **Step 2: Configure Deployment Settings**

1. Select the repository on the **Render Dashboard** and click **Connect**.
2. Provide the following information:

   - **Build Command:** `npm install`
   - **Start Command:** `node app.js`

3. Under **Instance Type**, select **Free Tier**.
4. Under **Advanced Options**, disable **Auto Deploy**.
5. Click **Deploy Web Service**.

---

## **Fixing Deployment Errors**

### **Step 1: Add Environment Variables in Render**

Since we did not push `.env` to GitHub, our deployed app cannot access environment variables. To fix this:

1. Go to **Environment** in the Render dashboard.
2. Click **Add Environment Variable**.
3. Manually enter variables or upload the `.env` file.
4. Click **Save Changes**.

### **Step 2: Configure MongoDB Atlas**

Render runs on a different server with unique IP addresses. We need to whitelist these IPs in MongoDB Atlas.

1. Go to the **Logs** section in Render.
2. Under **Connect**, find **Static Outbound IP Addresses** (there will be three).
3. Open **MongoDB Atlas**, go to **Network Access**, and **Add the IP Addresses**.

### **Step 3: Redeploy**

1. In **Render**, go to **Manual Deploy**.
2. Click **Clear Build Cache & Deploy**.

Now, our project is successfully deployed, and the hosted link is:

[https://wanderlust-hucj.onrender.com](https://wanderlust-hucj.onrender.com)

---

### **Homework:**

1. We need to make a small change: The **Delete Review** button should not be visible if the review was created by someone else.

2. Improve the **responsiveness** and make the **mobile view** much better.

3. Link all the **social media icons** to their respective platforms.
