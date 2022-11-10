# Overwriting CSS styles

Admin comes with default look. As of version 6.4 there is an easy way to change default styles. We added special `data-css` attributes to essential html tags. Their values are build dynamically and depend on resource, action and container.

#### Simple example

If we have the resource `user` we can style entire `form` as well as individual fields in this form. In this specific case `form` has `data-css="users-edit-form"` (_edit_ is an action name). Field `password` is tagged by `users-edit-password`&#x20;

This naming convention give you ability to style every resource, every action and every container separately. You can  also use more general attribute selector: for instance `[data-css$="edit-form"]` to style every form in AdminJS

#### Configuration

First, you must assure that AdminJS has access to static files. For express based app you should add line similar to this:

```
app.use(express.static(path.join(__dirname, "../public")));
```

Next, you should tell AdminJS where your CSS style sheet file is located, placing following code in your config file (where `sidebar.css` is name your CSS file and its location is in `public` ):

```javascript
assets: {
    styles: ["/sidebar.css"],
}
```

#### Ready to use example

Below you find simple example changing colors in AdminJS sidebar

```css
:root {
  --topbar-color: white;
  --sidebar-bg-color: darkgray;
  --sidebar-color: white;
  --sidebar-link-color: orange;
}

section[data-css="sidebar"] {
  background-color: var(--sidebar-bg-color) !important;
  color: var(--sidebar-color);
  border: none;
}

section[data-css="sidebar"] svg {
  fill: var(--sidebar-color) !important;
}

a[data-css="sidebar-logo"] {
  background-color: var(--sidebar-bg-color) !important;
}

section[data-css="sidebar-resources"] {
  background: var(--sidebar-bg-color) !important;
}

[data-css="sidebar"] section a {
  background: var(--sidebar-bg-color) !important;
  color: var(--sidebar-color);
}

[data-css="sidebar"] a:hover {
  color: var(--sidebar-link-color);
}
```
