### **Starting with Authorization**

#### **Hiding Edit and Delete Buttons**

To implement authorization in our full-stack project, we need to ensure that only the owner of a listing can edit or delete it. The first step in doing this is to **hide the edit and delete buttons** from users who are not the owners of a particular listing.

##### **Identifying Required Information**

To enforce this restriction, we need two key pieces of information:

1. **The owner of the listing** → Stored in `listing.owner`, which contains the **\_id** of the user who created the listing.
2. **The currently logged-in user** → Stored in `res.locals.currUser`, which contains the **\_id** of the authenticated user.

Thus, we can compare:

- `listing.owner._id` (ID of the user who created the listing)
- `currUser._id` (ID of the user currently trying to edit/delete)

##### **Implementing the Check in `show.ejs`**

Since we already have the required data, we can conditionally render the **edit and delete** buttons in our `show.ejs` file using the `.equals()` method to compare object IDs.

```html
<% if(currUser._id.equals(listing.owner._id)) { %>
<div class="btns">...</div>
<% } %>
```

##### **Handling Edge Cases**

Since `currUser` is stored in `res.locals`, there could be cases where:

- The middleware that assigns `currUser` fails to run.
- The user is not logged in, meaning `currUser` does not exist.

To prevent errors, we add an additional condition to check if `currUser` exists before comparing IDs:

```html
<% if(currUser && currUser._id.equals(listing.owner._id)) { %>
<div class="btns">...</div>
<% } %>
```

### **Protecting Routes from Unauthorized API Requests**

Even though we’ve hidden the edit and delete buttons, users can still send unauthorized API requests using tools like **Postman, Hoppscotch, or JavaScript fetch requests**.

To fully secure our application, we must **protect our routes at the server level**. We will implement this in the next section.
