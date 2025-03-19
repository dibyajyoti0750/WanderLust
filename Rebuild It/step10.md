### **Rendering and Styling Reviews**

---

#### **Displaying Reviews in the Show Page**

We are displaying reviews directly in the `show.ejs` file inside the review `<div>`:

```html
<hr />
<h4>All reviews</h4>
<p><%= listing.reviews %></p>
```

However, `listing.reviews` currently contains only ObjectIDs, which is not useful for displaying actual review data.

---

#### **Handling Invalid ObjectIDs in Reviews**

Okay, some invalid ObjectIDs might also be present in `listing.reviews` that no longer exist in the reviews collection. To remove these invalid IDs, we can delete the listing for which we added the reviews and also completely clear the reviews collection using:

```js
db.reviews.deleteMany({});
```

---

#### **Adding Reviews to Listings**

To test, we add three random reviews to a random listing.

---

#### **Populating Reviews to Show Actual Data**

Instead of just displaying ObjectIDs, we use Mongoose's `populate()` method in our **Show Route**:

```js
// Show Route
app.get(
  "/listings/:id",
  wrapAsync(async (req, res) => {
    let { id } = req.params;
    const listing = await Listing.findById(id).populate("reviews");
    res.render("listings/show", { listing });
  })
);
```

Now, when we refresh the page, we see full review objects instead of just ObjectIDs.

---

#### **Looping Through Reviews to Display Data**

To display only the rating and comment:

```html
<hr />
<h4>All reviews</h4>
<ul>
  <% for(let review of listing.reviews) { %>
  <li><%= review.comment %>, <%= review.rating %> stars</li>
  <% } %>
</ul>
```

---

#### **Applying Bootstrap Card Styling**

To make the reviews visually appealing, we wrap them in Bootstrap cards:

```html
<p><b>All reviews</b></p>
<% for(let review of listing.reviews) { %>
<div class="card">
  <div class="card-body">
    <h5 class="card-title">Jane Doe</h5>
    <p class="card-text"><%= review.comment %></p>
    <p class="card-text"><%= review.rating %> stars</p>
  </div>
</div>
<% } %>
```

---

#### **Fixing Card Border Issue**

In our `styles.css`, we had removed borders for all `.card` elements:

```css
/* Cards */
.listing-card {
  border: none !important;
  margin-bottom: 2rem;
}
```

To fix this, we only remove the border for listing cards, **not review cards**.

---

#### **Updating Card Classes in `index.ejs`**

```html
<div class="card col listing-card"></div>
```

In `show.ejs`, we also add:

```html
<div class="card col-6 offset-3 show-card listing-card"></div>
```

---

#### **Displaying Reviews in Two Columns**

We want two reviews per row. We achieve this using Bootstrap’s grid system:

```html
<p><b>All reviews</b></p>
<div class="row">
  <% for(let review of listing.reviews){ %>
  <div class="card col-5 ms-3 mb-3">
    <div class="card-body">
      <h5 class="card-title">Jane Doe</h5>
      <p class="card-text"><%= review.comment %></p>
      <p class="card-text"><%= review.rating %> stars</p>
    </div>
  </div>
  <% } %>
</div>
```

This ensures that each row contains two review cards, making the layout more organized.

---
