## **Storing Files**

Previously, we stored uploaded files in a local directory (`uploads/`). Now, we will upload files directly to **Cloudinary**, a cloud-based media storage solution. To achieve this, we will use **Multer** along with Cloudinary-specific libraries.

#### **1. Installing Required Libraries**

To integrate **Multer** with **Cloudinary**, install the following NPM packages:

```sh
npm install cloudinary
npm install multer-storage-cloudinary --legacy-peer-deps
```

- **cloudinary** → Provides Cloudinary API integration.
- **multer-storage-cloudinary** → Acts as a storage engine for Multer to upload files to Cloudinary. --legacy-peer-deps will skip peer dependency conflicts.

#### **2. Setting Up Cloudinary Configuration**

Create a new file **`cloudConfig.js`** to manage Cloudinary configurations.

```js
const cloudinary = require("cloudinary").v2;
const { CloudinaryStorage } = require("multer-storage-cloudinary");

// Configure Cloudinary with environment variables
cloudinary.config({
  cloud_name: process.env.CLOUD_NAME,
  api_key: process.env.CLOUD_API_KEY,
  api_secret: process.env.CLOUD_API_SECRET,
});

// Define Cloudinary Storage
const storage = new CloudinaryStorage({
  cloudinary: cloudinary,
  params: {
    folder: "wanderlust_DEV", // Folder name in Cloudinary
    allowed_formats: ["png", "jpg", "jpeg"], // Allowed file formats
  },
});

module.exports = { cloudinary, storage };
```

- **`cloudinary.config({})`** → Establishes a connection with Cloudinary using credentials from `.env`.
- **`CloudinaryStorage({})`** → Defines where and in what format files should be stored in Cloudinary.

#### **3. Updating Routes for File Upload**

Modify **`routes/listing.js`** to integrate Cloudinary for file uploads.

```js
const multer = require("multer");
const { storage } = require("../cloudConfig.js"); // Import storage configuration
const upload = multer({ storage }); // Configure Multer to use Cloudinary storage
```

- Instead of saving files locally (`uploads/`), Multer will now upload them directly to Cloudinary.

#### **4. Upload Workflow**

1. **Frontend Form Submission** → User uploads a file via a form.
2. **Backend Processing** → Multer extracts file data.
3. **Cloudinary Storage** → The file is uploaded to Cloudinary.
4. **Response to Backend** → Cloudinary returns a **URL** for the stored file.
5. **Database Storage (Next Step)** → This Cloudinary URL is stored in MongoDB.

#### **5. Testing the Upload**

- When a file is uploaded, `req.file` now contains **Cloudinary's file URL** instead of a local path.
- Files are automatically named and stored in the `wanderlust_DEV` folder.
- The `uploads/` folder is no longer needed since storage is handled in the cloud.

#### **Next Steps**

- Store the Cloudinary URL in **MongoDB**.
- Use this URL to display images in the frontend.
