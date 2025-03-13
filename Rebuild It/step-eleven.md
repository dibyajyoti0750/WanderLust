## **Deleting Reviews**

To allow users to delete their individual reviews, we need to add a delete button next to each review. When the delete button is clicked, a delete request should be sent to the server.

### **Setting Up the Delete Request**

- When deleting listings, we sent requests to `/listings/:id`.
- Since reviews belong to a specific listing, the request for deleting a review should go to `/listings/:id/reviews/:reviewId`.
- Here, `reviewId` is the unique identifier for the review, and `id` is the unique identifier for the listing.

### **Steps to Implement Review Deletion**

1. **Adding the Delete Button in `show.ejs`**

   ```html
   <div class="row">
     <% for(let review of listing.reviews){ %>
     <div class="card col-5 ms-3 mb-3">
       <div class="card-body">
         <h5 class="card-title">Jane Doe</h5>
         <p class="card-text"><%= review.comment %></p>
         <p class="card-text"><%= review.rating %> stars</p>
         <form
           class="mb-3"
           method="post"
           action="/listings/<%= listing._id %>/reviews/<%= review._id %>?_method=DELETE"
         >
           <button class="btn btn-sm btn-dark">Delete</button>
         </form>
       </div>
     </div>
     <% } %>
   </div>
   ```

2. **Creating the Delete Route in `app.js`**

   - We define a route that listens for DELETE requests at `/listings/:id/reviews/:reviewId`.
   - It extracts the `id` and `reviewId` from the request parameters and deletes the review from the `reviews` collection.

   ```javascript
   app.delete(
     "/listings/:id/reviews/:reviewId",
     wrapAsync(async (req, res) => {
       let { id, reviewId } = req.params;
       await Review.findByIdAndDelete(reviewId);
       res.redirect(`/listings/${id}`);
     })
   );
   ```

3. **Removing the Review ID from the Listing**

   - Deleting the review itself is not enough; we also need to remove its reference from the `reviews` array inside the corresponding listing.
   - This is achieved using MongoDB’s `$pull` operator, which removes a specific value from an array.

   ```javascript
   app.delete(
     "/listings/:id/reviews/:reviewId",
     wrapAsync(async (req, res) => {
       let { id, reviewId } = req.params;

       // Remove review reference from the listing
       await Listing.findByIdAndUpdate(id, { $pull: { reviews: reviewId } });

       // Delete the review from the reviews collection
       await Review.findByIdAndDelete(reviewId);

       res.redirect(`/listings/${id}`);
     })
   );
   ```

   **Explanation:**

   - `$pull: { reviews: reviewId }` ensures that the review ID is removed from the listing’s `reviews` array.
   - This way, the review is deleted from both the `reviews` collection and the `reviews` array inside the listing.

---

## **Delete Middleware for Reviews**

### **Handling Review Deletion When a Listing is Deleted**

- Each listing contains multiple reviews stored as an array of IDs.
- If we delete a listing, all associated reviews should also be removed from the database.
- By default, deleting a listing only removes the reference to reviews (i.e., the ObjectIDs inside the `reviews` array), but the reviews themselves remain in the `reviews` collection.

### **Using Mongoose Middleware for Cascading Deletion**

- We use a **Mongoose post middleware** to ensure that when a listing is deleted, all its reviews are also deleted.
- This middleware is placed inside `listing.js` where the `Listing` schema is defined.

### **Steps to Implement Cascading Deletion**

1. **Require the `Review` Model in `listing.js`**

   ```javascript
   const Review = require("./review.js");
   ```

2. **Define the Post Middleware**

   - This middleware listens for `findOneAndDelete` events, which occur when a listing is deleted.
   - It then deletes all reviews whose `_id` matches any ID inside the listing’s `reviews` array.

   ```javascript
   listingSchema.post("findOneAndDelete", async (listing) => {
     if (listing) {
       await Review.deleteMany({ _id: { $in: listing.reviews } });
     }
   });
   ```

   **Explanation:**

   - `listing.reviews` contains an array of review IDs associated with the listing.
   - `{ _id: { $in: listing.reviews } }` finds and deletes all reviews whose `_id` exists in that array.
   - This ensures that when a listing is deleted, all its associated reviews are deleted as well.

3. **How This Works with the Delete Route**

   - In `app.js`, when the delete route for a listing is called, `findByIdAndDelete` triggers the post middleware.

   **How It Works:**

   - `findByIdAndDelete(id)` deletes the listing.
   - The `post` middleware in `listing.js` is automatically triggered.
   - It deletes all reviews associated with the deleted listing.

---

## **Summary of Review Deletion**

| Task                                                         | Implementation                                              |
| ------------------------------------------------------------ | ----------------------------------------------------------- |
| **Add a delete button for each review**                      | Added inside `show.ejs`                                     |
| **Handle the delete request in `app.js`**                    | Created a DELETE route at `/listings/:id/reviews/:reviewId` |
| **Remove the review reference from the listing**             | Used MongoDB’s `$pull` operator                             |
| **Ensure reviews are deleted when their listing is removed** | Used Mongoose `post` middleware in `listing.js`             |

---

## **Final Thoughts**

With these implementations:

- Users can delete their individual reviews.
- Review deletions remove both the review document and its reference inside the listing.
- If a listing is deleted, all its associated reviews are also deleted automatically.

This ensures a clean and consistent database structure, preventing orphaned reviews from remaining in the database.
