## **Image Upload Setup**

### **Manipulating the Form**

The first thing we need to do is modify our form so that it can send files to the backend. Right now, in `new.ejs`, the form only sends URL-encoded data, which means we cannot upload images or any other file type.

To enable file uploads, we need to change the form's encoding type. By default, the form submits data in URL-encoded format, but when dealing with file uploads (such as images, PDFs, etc.), we must use **`multipart/form-data`**.

This encoding type allows the form to send file data along with other form inputs.

#### **Modifying the Form in `new.ejs`**

We need to update the form in `new.ejs` to include `enctype="multipart/form-data"`:

```html
<form
  method="post"
  action="/listings"
  novalidate
  class="needs-validation"
  enctype="multipart/form-data"
>
  ...
</form>
```

Next, we update the input field for the image. Instead of using a `text` input for image links, we replace it with a `file` input:

```html
<div class="mb-3">
  <label for="image" class="form-label">Upload Listing Image</label>
  <input name="listing[image]" type="file" class="form-control" />
</div>
```

With this, users can now select and upload an image file through the form.

---

### **Handling the Form Submission**

Now, we need to process this uploaded file in the backend. Let's open `routes/listing.js`.

By default, when we submit the form, it triggers a `POST` request to the `/` route. For now, let's comment out the existing handler and replace it with a new one:

```js
router
  .route("/")
  .get(wrapAsync(listingController.index))
  // .post(
  //   isLoggedIn,
  //   validateListing,
  //   wrapAsync(listingController.createListing)
  // );
  .post((req, res) => {
    res.send(req.body);
  });
```

#### **Testing the Form Submission**

After making this change, when we submit the form, we receive an **empty object** `{}` in response. This means that the backend **is not recognizing the uploaded file**. The reason for this is that our `app.js` is currently parsing form data in URL-encoded format, but we are now sending `multipart/form-data`.

To parse this new format, we need a special middleware. This is where **Multer** comes in.

---

### **Setting Up Multer**

Multer is an npm package that helps us handle `multipart/form-data`, which is required for processing file uploads.

#### **Installing Multer**

Run the following command in the terminal:

```sh
npm i multer
```

#### **Configuring Multer**

First, require Multer in `routes/listing.js`:

```js
const multer = require("multer");
const upload = multer({ dest: "uploads/" });
```

Here, `multer({ dest: "uploads/" })` tells Multer to store uploaded files inside an `uploads/` folder in our project directory.

Now, let's modify our route to handle file uploads using Multer:

```js
.post(upload.single("listing[image]"), (req, res) => {
  res.send(req.body);
});
```

- `upload.single("listing[image]")` is a middleware that extracts a **single** file from the `listing[image]` field.
- The file is stored in the `uploads/` directory.
- After the file is stored, the request continues to the next function (`res.send(req.body)` in this case).

---

### **Testing the File Upload**

After making these changes, let's test by uploading an image. However, we still won’t see any file details in `req.body` because uploaded files are stored in `req.file`.

To check if the file is being processed, modify the route:

```js
.post(upload.single("listing[image]"), (req, res) => {
res.send(req.file);
});
```

Now, when we submit the form, we receive a response similar to this:

```json
{
  "fieldname": "listing[image]",
  "originalname": "1697005195203.jpg",
  "encoding": "7bit",
  "mimetype": "image/jpeg",
  "destination": "uploads/",
  "filename": "dd314e274ee5e98d04e5e98e36272332",
  "path": "uploads/dd314e274ee5e98d04e5e98e36272332",
  "size": 442064
}
```

This confirms that:

- The image was successfully uploaded.
- Multer stored the image in the `uploads/` folder.
- We can now access the uploaded file using `req.file`.

If we check our project directory, we’ll see that an `uploads/` folder was **automatically created** by Multer, and the uploaded file is stored inside with a unique name.

---

### **Next Steps**

Now that we have enabled file uploads and are storing images in local storage, the next step will be to **store these images in a cloud storage service instead of local storage**.

This will allow:

- Easy access to images from anywhere.
- Better scalability and security.
- No risk of losing images when deploying to a cloud server.

---

## **Cloud Setup: Cloudinary & .env File**

In this step, we set up our cloud storage using **Cloudinary**, a third-party service. There are many cloud storage options, but we will use Cloudinary for this project.

#### **1. Setting Up Cloudinary**

- First, **sign up for a free Cloudinary account** and go to the **dashboard**.
- In the dashboard, locate the **Product Environment** section, which provides access credentials like:
  - API Key
  - API Secret
  - Other necessary credentials
- These credentials will be required in our project.

#### **2. Storing Credentials Securely in .env File**

Instead of writing credentials directly in the code, we store them in a special file called **.env** to keep them secure.

- Create a `.env` file in the project:
  ```
  touch .env
  ```
- **Format of .env file:**  
  Environment variables follow a **key-value pair format**, with **keys in uppercase** and no spaces or quotes:
  ```
  SECRET=helloworldbyapnacollege
  ```
- These environment variables can be used anywhere in the project.

#### **3. Using Dotenv to Load .env Variables**

Environment variables in `.env` cannot be accessed directly in the code. To load them, we use a package called **dotenv**.

- **Install dotenv:**
  ```
  npm i dotenv
  ```
- **Require dotenv at the very top of your server file (`app.js`):**
  ```js
  require("dotenv").config();
  console.log(process.env.SECRET);
  ```
- Running this will print the value stored in `SECRET`.

#### **4. Conditional Loading of .env in Development**

Since `.env` files should **not be included in production**, we add a condition:

```js
if (process.env.NODE_ENV != "production") {
  require("dotenv").config();
}
```

- This ensures:
  - In **development**, dotenv loads `.env` variables.
  - In **production**, environment variables are managed differently.
- **Important:** Never upload `.env` to GitHub or include it in the deployment.

#### **5. Storing Cloudinary Credentials in .env**

Now, store Cloudinary credentials in `.env`:

```
CLOUD_NAME=cloud_name
CLOUD_API_KEY=api_key
CLOUD_API_SECRET=api_secret
```

These credentials will be used when integrating Cloudinary in the project.

For now, the `.env` file is set up, and we won't need to access it frequently.
