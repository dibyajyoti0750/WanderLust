## **Storing Files**

Previously, we stored uploaded files in a local directory (`uploads/`). Now, we will upload files directly to **Cloudinary**, a cloud-based media storage solution. To achieve this, we will use **Multer** along with Cloudinary-specific libraries.

#### **1. Install Dependencies**

```sh
npm install cloudinary
```

#### **2. Set up Cloudinary Configuration**

Create a new file **`cloudConfig.js`**

```js
const cloudinary = require("cloudinary").v2;
require("dotenv").config();

cloudinary.config({
  cloud_name: process.env.CLOUD_NAME,
  api_key: process.env.CLOUD_API_KEY,
  api_secret: process.env.CLOUD_API_SECRET,
});

module.exports = cloudinary;
```

- **`cloudinary.config({})`** → Establishes a connection with Cloudinary using credentials from `.env`.

#### **3. Updating Routes for File Upload**

Modify **`routes/listing.js`** to integrate Cloudinary for file uploads.

```js
const multer = require("multer");
const storage = multer.memoryStorage();
const upload = multer({ storage });
const cloudinary = require("../cloudConfig.js");

router
  .route("/")
  .get(wrapAsync(wallpaperController.index))
  // .post(
  //   isLoggedIn,
  //   validateWallpaper,
  //   wrapAsync(wallpaperController.uploadWallpaper)
  // );
  .post(upload.single("listing[image]"), async (req, res) => {
    const file = req.file;

    const b64 = Buffer.from(file.buffer).toString("base64");
    const dataURI = `data:${file.mimetype};base64,${b64}`;
    const result = await cloudinary.uploader.upload(dataURI, {
      folder: "wanderlust_DEV",
    });

    res.send(result);
  });
```

### **4. Upload Workflow**

1. **User uploads** an image file through a form.
2. **Multer** middleware parses the `multipart/form-data` and extracts the file (in memory).
3. The **buffered file** is converted to a base64 data URI and uploaded to **Cloudinary** using their SDK.
4. Cloudinary responds with a **secure URL** (and other metadata) for the uploaded file.
5. This **Cloudinary URL** can now be **stored in MongoDB** along with any related data.

---

#### **Next Steps**

- Store the Cloudinary URL in **MongoDB**.
- Use this URL to display images in the frontend.
