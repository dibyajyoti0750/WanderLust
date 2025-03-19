# **Implementing Star-Based Ratings with Starability**

## **1. Introduction**

We wanted to replace our existing rating system with a visually appealing star-based rating system. To achieve this, we used the **Starability** library, which provides CSS-based star rating styles without JavaScript.

---

## **2. Choosing a Rating Style**

Starability provides multiple styles for displaying ratings. After exploring the available options, we decided to implement the **Slot Machine Rating** style.

**Resources:**

- [Starability GitHub](https://github.com/LunarLogic/starability)
- [Starability Demo](https://lunarlogic.github.io/starability/)

---

## **3. Adding Starability to Our Project**

Since Starability is a pure CSS solution, we needed to include the appropriate CSS file in our project.

### **Step 1: Downloading the CSS File**

We selected `starability-slot.css` from the Starability styles and copied its contents.

### **Step 2: Creating a CSS File**

We created a new CSS file inside our project at:

```
public/css/rating.css
```

Then, we pasted the copied CSS code into this file.

### **Step 3: Including the CSS File in Our Boilerplate**

To apply the styles globally, we linked the new CSS file in `boilerplate.ejs`:

```html
<link rel="stylesheet" href="/css/rating.css" />
</head>
```

---

## **4. Adding the Star Rating System**

Now that the CSS was set up, we needed to implement the rating system in our EJS template.

### **Step 1: Copying the Fieldset Structure**

The Starability library provides a predefined `fieldset` structure that we needed to include in our `show.ejs` file (where ratings are displayed).

We copied the following code and pasted it into `show.ejs`, right below the existing rating display:

```html
<fieldset class="starability-slot">
  <legend>First rating:</legend>
  <input
    type="radio"
    id="no-rate"
    class="input-no-rate"
    name="rating"
    value="0"
    checked
    aria-label="No rating."
  />
  <input type="radio" id="first-rate1" name="rating" value="1" />
  <label for="first-rate1" title="Terrible">1 star</label>
  <input type="radio" id="first-rate2" name="rating" value="2" />
  <label for="first-rate2" title="Not good">2 stars</label>
  <input type="radio" id="first-rate3" name="rating" value="3" />
  <label for="first-rate3" title="Average">3 stars</label>
  <input type="radio" id="first-rate4" name="rating" value="4" />
  <label for="first-rate4" title="Very good">4 stars</label>
  <input type="radio" id="first-rate5" name="rating" value="5" />
  <label for="first-rate5" title="Amazing">5 stars</label>
</fieldset>
```

### **Step 2: Replacing the Old Rating System**

Previously, we used an input range slider (`<input type="range">`) to collect ratings. We removed this old system and replaced it with our new star rating system.

Old Code (Removed):

```html
<!-- 
<div class="mb-3 mt-3">
  <label for="rating" class="form-label">Rating</label>
  <input
    type="range"
    min="1"
    max="5"
    id="rating"
    name="review[rating]"
    class="form-range"
  />
</div> 
-->
```

New Code (Added):

```html
<div class="mb-3 mt-3">
  <label for="rating" class="form-label">Rating</label>
  <fieldset class="starability-slot">
    <input
      type="radio"
      id="no-rate"
      class="input-no-rate"
      name="review[rating]"
      value="0"
      checked
      aria-label="No rating."
    />
    <input type="radio" id="first-rate1" name="review[rating]" value="1" />
    <label for="first-rate1" title="Terrible">1 star</label>
    <input type="radio" id="first-rate2" name="review[rating]" value="2" />
    <label for="first-rate2" title="Not good">2 stars</label>
    <input type="radio" id="first-rate3" name="review[rating]" value="3" />
    <label for="first-rate3" title="Average">3 stars</label>
    <input type="radio" id="first-rate4" name="review[rating]" value="4" />
    <label for="first-rate4" title="Very good">4 stars</label>
    <input type="radio" id="first-rate5" name="review[rating]" value="5" />
    <label for="first-rate5" title="Amazing">5 stars</label>
  </fieldset>
</div>
```

### **Step 3: Ensuring the Rating is Sent to the Backend**

- Previously, the `name` attribute of the old rating input was `review[rating]`.
- We made sure that all `radio` inputs in the new star rating system also had `name="review[rating]"`, so the value would be correctly sent to the backend.

Example:

```html
<input type="radio" id="first-rate1" name="review[rating]" value="1" />
```

### **Step 4: Setting a Default Rating**

- The default selected value is **0**, but we wanted it to be at least **1** (since 0-star ratings were not valid in our backend).

```html
<input
  type="radio"
  id="no-rate"
  class="input-no-rate"
  name="review[rating]"
  value="1"
  checked
  aria-label="No rating."
/>
```

---

## **5. Displaying Static Ratings in Reviews**

Now that we had a star rating input system, we wanted to display user ratings visually in reviews.

### **Step 1: Using Starability’s Static Rating System**

Starability provides a `starability-result` class to show static (non-editable) ratings.

Reference: [Showing Static Ratings](https://github.com/LunarLogic/starability?tab=readme-ov-file#showing-the-static-rating-result)

### **Step 2: Adding Static Rating Display in `show.ejs`**

We added the following code in the **"All Reviews"** section:

```html
<p class="starability-result" data-rating="3">Rated: 3 stars</p>
```

However, instead of hardcoding the rating, we dynamically displayed the review's rating using EJS:

```html
<p class="starability-result" data-rating="<%= review.rating %>"></p>
```

### **Step 3: Adjusting the Placement**

To make the rating appear alongside the author’s name, we modified the card body in `show.ejs`:

```html
<div class="card-body">
  <h5 class="card-title">@<%= review.author.username %></h5>
  <p
    class="starability-result card-text"
    data-rating="<%= review.rating %>"
  ></p>
  <p class="card-text"><%= review.comment %></p>
</div>
```

This ensures that:

- The star rating appears directly below the author's username.
- The rating is **static** (users cannot edit it after submission).

---

## **Final Outcome**

With these changes:
✅ The old range slider for rating input was replaced with a visually appealing **star rating**.  
✅ Users can select ratings from **1 to 5 stars** before submitting a review.  
✅ Submitted ratings are now displayed as **static star ratings** in reviews.

This setup provides an intuitive and polished rating system using only **CSS (Starability)** without JavaScript!

---
