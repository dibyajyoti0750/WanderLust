### **Adding Tax Switch**

Now, we are going to add a tax-related switch to our page. This will be a simple on-off toggle button that will cause some changes on our index page.

To create this switch, we can use an existing Bootstrap component called the **switch element**. You can check it out here: [Bootstrap Switches](https://getbootstrap.com/docs/5.3/forms/checks-radios/#switches)

There are multiple switch options available. We will copy the first one and add it inside the `#filters` div by creating a new `div` for it. The updated HTML structure will be:

```html
<p>Arctic</p>
</div>

<div class="tax-toggle">
  <div class="form-check form-switch">
    <input
      class="form-check-input"
      type="checkbox"
      role="switch"
      id="flexSwitchCheckDefault"
    />
    <label class="form-check-label" for="flexSwitchCheckDefault">
      Default switch checkbox input
    </label>
  </div>
</div>
</div>
```

---

### **Styling the Switch**

Now, let's modify the styling as per our requirements. The first change we need is to shift the text to the left.

To implement this, we simply add the `form-check-reverse` class:

```html
<div class="form-check-reverse form-switch"></div>
```

Now, let’s modify the label text to something more relevant:

```html
<label class="form-check-label" for="flexSwitchCheckDefault"
  >Display Total After Taxes</label
>
```

---

### **Detecting Switch Click Event**

We want some action to occur when the switch is turned on or off. To achieve this, we will add an **event listener** to the checkbox input.

First, we copy the **ID** of the switch (`flexSwitchCheckDefault`) and write the event listener inside a `<script>` tag at the bottom of our page:

```html
<script>
  let taxSwitch = document.getElementById("flexSwitchCheckDefault");
  taxSwitch.addEventListener("click", () => {
    console.log("clicked");
  });
</script>
```

---

### **Making the Switch Responsive**

We notice that on smaller screen sizes, our layout gets a bit messed up. To make it responsive, we can follow one of these approaches:

1. **Show only 3-4 filter options on smaller screens** (instead of displaying all).
2. **Make the filters collapsible**, similar to a **navbar**, so that on small screens, the options collapse into a dropdown.

This will be your **homework** to implement.

---

### **Displaying Tax Information on Toggle**

Now, we want an additional line to appear next to the price whenever the switch is turned on. This line should display: **+18% GST**

To do this, we modify the code inside our **card**, where we display the price:

```html
&#8377;<%= listing.price.toLocaleString("en-IN") %> / night
<i> &nbsp; &nbsp;+18% GST</i>
```

However, we want this **18% GST** to appear **only when the switch is turned on** and remain hidden otherwise.

By default, we will **hide** this tax information. We do this by adding a class (`tax-info`) to the `<i>` tag and setting its `display` property to `none` in CSS:

```html
<i class="tax-info"> &nbsp; &nbsp;+18% GST</i>
```

```css
.tax-info {
  display: none;
}
```

---

### **Toggling Tax Information on Click**

To make the tax information appear when the switch is clicked, we modify our event listener:

```html
<script>
  let taxSwitch = document.getElementById("flexSwitchCheckDefault");

  taxSwitch.addEventListener("click", () => {
    let taxInfo = document.getElementsByClassName("tax-info");
    console.log(taxInfo);
  });
</script>
```

Now, we need to **loop** through all elements with the class `tax-info` to toggle their visibility:

```js
taxSwitch.addEventListener("click", () => {
  let taxInfo = document.getElementsByClassName("tax-info");
  for (let info of taxInfo) {
    info.style.display = "inline";
  }
});
```

At this point, the tax information appears when the switch is clicked, but it does **not toggle back off** when the switch is clicked again. To fix this, we add a condition to toggle the display:

```js
for (let info of taxInfo) {
  if (info.style.display != "inline") {
    info.style.display = "inline";
  } else {
    info.style.display = "none";
  }
}
```

---

### **Improving Styling**

```css
.tax-toggle {
  border: 1px solid black;
  border-radius: 1rem;
  height: 3.25rem;
  padding: 1rem;
  margin-left: 2rem;
  display: flex;
  align-items: center;
}
```

Now, we want to **vertically center** the switch within the filters section:

```css
#filters {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
}
```

---

### **Adding a Search Option to the Navbar**

Now that we have our options and toggle button ready, the next feature we will implement is a **search option**. We want our search bar to be visible on every page, so we will place it at the **top of the website, centered**.

The search option will contain two elements:

1. **An input box** for users to type their query.
2. **A search button** to submit the query.

### **Using Bootstrap for the Navbar Search**

Bootstrap provides several built-in components for adding a search box within a navbar. We will embed a **search input field** inside the navbar and align it accordingly.

#### **Modifying `navbar.ejs`**

```html
<a class="nav-link" href="/listings">Explore</a>
</div>

<div class="navbar-nav ms-auto">
  <form class="d-flex" role="search">
    <input
      class="form-control me-2"
      type="search"
      placeholder="Search"
      aria-label="Search"
    />
    <button class="btn btn-outline-success" type="submit">Search</button>
  </form>
</div>
```

### **Customizing the Search Button**

```html
<button class="btn search-btn" type="submit">Search</button>
```

#### **Adding Custom Styles**

Now, we define the styling for our **search button** using a `<style>` tag:

```html
<style>
  .search-btn {
    background-color: #fe424d;
    color: #fff;
  }

  .search-btn:hover {
    background-color: #fe424d;
    color: #fff;
  }
</style>
```

### **Adding an Icon to the Search Button**

```html
<button class="btn search-btn" type="submit">
  <i class="fa-solid fa-magnifying-glass"></i> Search
</button>
```

### **Improving Button Alignment & Shape**

```css
.search-btn {
  background-color: #fe424d;
  color: #fff;
  border-radius: 25px;
  padding: 0 1rem 0 1rem;
}

.search-btn i {
  display: inline;
  margin-right: 0.5rem;
}
```

### **Customizing the Search Input Field**

```html
<input
  class="form-control me-2 search-inp"
  type="search"
  placeholder="Search destinations"
/>
```

#### **Styling the Search Input**

```css
.search-inp {
  border-radius: 25px;
  padding: 0.5rem 3rem 0.5rem 3rem;
  font-size: 1rem;
}
```

### **Next Steps: Backend Implementation**

**How will search work?**

- The search will function similarly to how **filters** work.
- We can also provide **a dropdown menu** to allow users to filter results **by country**, as we have already saved country data in our database.
- There are multiple ways to implement search, depending on how creative we want to get with the user experience.

And the next thing we will focus on is **deployment**. So far, we have learned almost everything conceptually in **Node.js**, and **functionality-wise, our project is complete**. Now, we need to focus on **deploying** it.

After that, very soon, we will start working with **React.js**! ⚛️
