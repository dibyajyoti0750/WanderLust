## **Image Preview for Edit Page**

The next feature we are working on is adding an image preview for the edit page.

This will be displayed before the "Upload New Image" div, allowing users to decide whether they want to update the original listing image or not.

---

### **Approach to Implementing the Image Preview**

There are multiple ways to achieve this. The simplest and most basic approach is to modify our `edit.ejs` file. In this file, where we have the image input field (file upload section), we will create a new `div` element right before it. Inside this `div`, we will use an `<img>` tag to display the original image of the listing.

#### **Adding an Image Tag in `edit.ejs`**

```html
<div class="mb-3">
  Original Listing Image <br />
  <img src="<%= listing.image.url %>" />
</div>
```

This method successfully displays the listing image, but it introduces a small problem.

---

### **Problem with Directly Using the `<img>` Tag**

When we directly use an `<img>` tag with the listing’s image URL, it loads the **original** high-quality image that was uploaded. If a user had uploaded a high-resolution image, the same large image will load on the edit page, which is problematic.

#### **Why is this an issue?**

- When editing a listing, the owner does not need to see the image in full quality.
- They only need a **small preview** of the old image to understand which one was used.
- Loading high-resolution images on the edit page can slow down the page unnecessarily.
- A compressed, low-resolution version of the image is a better approach.

To solve this, we need a way to display a **smaller preview version** of the image instead of the original high-quality image.

---

### **Using Cloudinary for Image Transformation**

Instead of manually compressing images, we can take advantage of **Cloudinary’s Image Transformation features**. Cloudinary provides various parameters that allow us to manipulate images dynamically.

[Cloudinary Image Transformations](https://cloudinary.com/documentation/image_transformations)

For our specific case, we want to reduce the **image resolution** so that only a small preview is shown on the edit page.

---

### **Reducing Image Size using Cloudinary Parameters**

Cloudinary allows us to modify the image URL to include transformation parameters. We will reduce the image width to **250 pixels** by adding `/w_250` to the URL.

#### **How to Modify the Image URL?**

We need to modify the image URL inside our **edit.ejs page**. The file responsible for rendering this page is **controllers/listings.js**.

Inside `controllers/listings.js`, we are currently passing the listing data to the `edit.ejs` template like this:

```js
res.render("listings/edit.ejs", { listing });
```

To modify the image URL, we will extract the image URL from `listing.image.url` and modify it.

#### **Modifying the Image URL in `listings.js`**

```js
let originalImageUrl = listing.image.url;
```

Now, we need to insert our transformation parameter into this URL.

Cloudinary image URLs follow a specific pattern:

```bash
https://res.cloudinary.com/your-cloud-name/image/upload/v123456789/your-image.jpg
```

The `/upload/` part of the URL is where we insert transformation parameters.

We will **replace** `/upload/` with `/upload/w_250` to reduce the image size and pass it to our `edit.ejs` template:

```js
originalImageUrl = originalImageUrl.replace("/upload", "/upload/w_250");
res.render("listings/edit.ejs", { listing, originalImageUrl });
```

---

### **Using the Modified Image URL in `edit.ejs`**

Now, instead of using `listing.image.url`, we will use `originalImageUrl` in `edit.ejs`:

```html
<div class="mb-3">
  Original Listing Image <br />
  <img src="<%= originalImageUrl %>" />
</div>
```

---

### **Final Flow Summary**

1. **Problem:** The edit page doesn’t show the original listing image, making it hard for users to decide whether to update it.
2. **Solution:** We display a preview image before the "Upload New Image" section.
3. **Issue with Direct `<img>` Tag:** It loads the original high-resolution image, which is unnecessary and slows down the page.
4. **Cloudinary Image Transformation:** We modify the URL to include `/w_250`, reducing the image width to 250 pixels.
5. **Implementation:**
   - Modify `controllers/listings.js` to generate a low-resolution image URL.
   - Pass the modified URL to `edit.ejs`.
   - Use `originalImageUrl` in `edit.ejs` to display the optimized preview.

---

### **Conclusion**

With this implementation:

- The edit page now shows a preview of the original listing image.
- The preview image is optimized and loads faster.
- The original image remains unchanged, and users can still upload a new one if needed.
