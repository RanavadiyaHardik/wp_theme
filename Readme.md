# 🎨 WordPress Custom Theme Development

### BCA Semester 3 — Complete Study Notes

A step-by-step guide to building a custom WordPress theme from scratch — dynamic menus, custom page templates, featured images, a working blog with pagination, and comments.

---

## 📑 Table of Contents

| # | Topic |
|---|---|
| 1 | [Creating a Custom Theme](#-topic-1--creating-a-custom-theme) |
| 2 | [Making the Header Menu Dynamic](#-topic-2--making-the-header-menu-dynamic) |
| 3 | [Creating a Custom Page Layout](#-topic-3--creating-a-custom-page-layout) |
| 4 | [Adding a Featured Image](#-topic-4--adding-a-featured-image) |
| 5 | [Adding a Logo / Header Image](#-topic-5--adding-a-logo--header-image) |
| 6 | [Creating a Custom Page Template](#-topic-6--creating-a-custom-page-template) |
| 7 | [Showing Posts on the Front Side (The Loop)](#-topic-7--showing-posts-on-the-front-side-the-loop) |
| 8 | [Adding Pagination to the Blog](#-topic-8--adding-pagination-to-the-blog) |
| 9 | [Showing Single Post Details](#-topic-9--showing-single-post-details) |
| 10 | [Comment Form on Blog Detail Page](#-topic-10--comment-form-on-blog-detail-page) |
| 11 | [Showing Posts by Category](#-topic-11--showing-posts-by-category) |
| 12 | [Creating a Custom Widget / Sidebar](#-topic-12--creating-a-custom-widget--sidebar) |
| 13 | [Calling a Sidebar from Any Template File](#-topic-13--calling-a-sidebar-from-any-template-file) |
| 14 | [Final Theme File Structure](#-final-theme-file-structure) |
| 15 | [Quick Revision Table](#-quick-revision-table-all-important-functions) |
| 16 | [Practical Task](#-practical-task-college-website-theme) |

---

## 🧩 Topic 1 — Creating a Custom Theme

A WordPress **theme** controls the *design and layout* of a website. It never touches the content — only how that content is displayed.

### Step 1 · Create the Theme Folder

Navigate to:

```
xampp/htdocs/your-wordpress-folder/wp-content/themes/
```

Create a new folder — e.g. `mytheme`.

### Step 2 · Create the Two Minimum Required Files

Every WordPress theme needs **at least these two files** to be recognized:

| File | Purpose |
|---|---|
| `style.css` | CSS styling + theme information header |
| `index.php` | The fallback / home page template |

> We'll add more files (`header.php`, `footer.php`, `functions.php`, `page.php`, `single.php`) as we progress.

### Step 3 · Add Theme Info to `style.css`

WordPress reads a special comment block at the top of `style.css` to identify the theme. **This is mandatory**, even if no other CSS follows it.

```css
/*
Theme Name: My Theme
Author: BCA Sem 3
Description: My First Custom WordPress Theme
Version: 1.0
*/
```

### Step 4 · Add Basic Markup to `index.php`

```php
<!DOCTYPE html>
<html>
<head>
    <title>My Website</title>
</head>
<body>

    <h1>Welcome to My Custom Theme</h1>
    <p>This is my first WordPress custom theme.</p>

</body>
</html>
```

### Step 5 · Activate the Theme

> **WordPress Admin → Appearance → Themes → My Theme → Activate**

> ✅ **Checkpoint:** If the theme doesn't show up in the admin panel, double‑check the comment block syntax in `style.css` — a typo there is the #1 reason a theme fails to register.

---

### 🔧 Core Theme Functions (The "Glue")

These functions connect PHP to the WordPress engine — you'll use them constantly.

<details>
<summary><b><code>get_template_directory_uri()</code></b> — get the theme folder's URL</summary>

Use it to correctly link images, CSS, and JS regardless of where WordPress is installed.

```php
<img src="<?php echo get_template_directory_uri(); ?>/images/logo.png">
```
</details>

<details>
<summary><b><code>bloginfo('template_directory')</code></b> — older equivalent</summary>

Outputs the theme folder path directly (can't easily be stored in a variable).

```php
<img src="<?php bloginfo('template_directory'); ?>/images/logo.png">
```

> 💡 **Best practice:** prefer `get_template_directory_uri()` in modern development — it *returns* a value instead of only echoing it.
</details>

<details>
<summary><b><code>get_header()</code> / <code>get_footer()</code> / <code>get_sidebar()</code></b> — reusable layout parts</summary>

Includes `header.php`, `footer.php`, and `sidebar.php` respectively — this is how WordPress keeps a consistent header/footer across every page without repeating code.

```php
<?php get_header(); ?>
<?php get_footer(); ?>
<?php get_sidebar(); ?>
```
</details>

<details>
<summary><b><code>site_url()</code></b> — base site URL</summary>

```php
<a href="<?php echo site_url(); ?>">Home</a>
```
</details>

**Putting it together** — `header.php` + `footer.php` in `index.php`:

```php
<?php get_header(); ?>

<h1>Welcome to My Website</h1>

<?php get_footer(); ?>
```

---

## 🧭 Topic 2 — Making the Header Menu Dynamic

**Why?** A hardcoded menu means editing PHP every time a page changes. A *dynamic* menu lets the site admin manage it from the dashboard — no code required.

### Step 1 · Create `functions.php`

This file is the **control center** of your theme — where you register features, menus, and hooks.

### Step 2 · Register a Menu Location

```php
<?php

function mytheme_register_menus()
{
    register_nav_menus(
        array(
            'primary-menu' => 'Header Menu'
        )
    );
}

add_action('init', 'mytheme_register_menus');
```

| Value | Meaning |
|---|---|
| `'primary-menu'` | Internal slug/ID used in code (`theme_location`) |
| `'Header Menu'` | Friendly label shown in the WP Admin dashboard |

### Step 3 · Build the Menu from the Admin Panel

> **WordPress Admin → Appearance → Menus** → create a new menu → add pages (Home, About Us, Services, Blog, Contact Us) → assign it to **Header Menu**

### Step 4 · Output the Menu in `header.php`

```php
<?php
wp_nav_menu(
    array(
        'theme_location' => 'primary-menu',
        'menu_class'     => 'menu'
    )
);
?>
```

WordPress auto-generates:

```html
<ul class="menu">
    <li>Home</li>
    <li>About Us</li>
    <li>Services</li>
</ul>
```

Style it with plain CSS:

```css
.menu {
    display: flex;
    gap: 20px;
    list-style: none;
}
```

> ✅ **Checkpoint:** The `'theme_location'` string in `wp_nav_menu()` must **exactly match** the key registered in `register_nav_menus()` — a mismatch here is a very common bug.

---

## 📄 Topic 3 — Creating a Custom Page Layout

WordPress uses `page.php` as the default template for **static pages** (About Us, Contact, etc.) — separate from blog posts.

### Step 1 · Create `page.php`

### Step 2 · Wrap with Header & Footer

```php
<?php get_header(); ?>

<!-- Page Content -->

<?php get_footer(); ?>
```

### Step 3–4 · Show the Page Title and Content

```php
<?php get_header(); ?>

<h1><?php the_title(); ?></h1>

<div class="page-content">
    <?php the_content(); ?>
</div>

<?php get_footer(); ?>
```

### Function Reference

| Function | Purpose |
|---|---|
| `the_title()` | Displays (echoes) the page title |
| `the_content()` | Displays (echoes) the page content |
| `get_the_content()` | *Returns* the content instead of echoing (use when you need to manipulate it first) |
| `get_header()` | Includes `header.php` |
| `get_footer()` | Includes `footer.php` |

---

## 🖼️ Topic 4 — Adding a Featured Image

A **Featured Image** is the main representative image of a page or post (shown in blog listings, social shares, etc.).

### Step 1 · Enable Support in `functions.php`

```php
<?php
add_theme_support('post-thumbnails');
```

This makes the "Featured Image" box appear in the WordPress editor sidebar.

### Step 2 · Output the Image

```php
<?php the_post_thumbnail(); ?>
```

### Available Sizes

```php
<?php the_post_thumbnail('thumbnail'); ?>
<?php the_post_thumbnail('medium'); ?>
<?php the_post_thumbnail('large'); ?>
<?php the_post_thumbnail('full'); ?>
<?php the_post_thumbnail(array(100, 100)); ?>  // custom size
```

| Size | Purpose |
|---|---|
| `thumbnail` | Small image |
| `medium` | Medium image |
| `large` | Large image |
| `full` | Original, uncropped image |

> 💡 `the_post_thumbnail()` already generates a complete `<img>` tag for you — no need to wrap it in another one.

### Getting Just the Image URL *(advanced)*

Useful when you need the raw URL — e.g. for a CSS `background-image`.

```php
<?php
$image = wp_get_attachment_image_src(
    get_post_thumbnail_id(),
    'large'
);

echo $image[0];
?>
```

---

## 🏷️ Topic 5 — Adding a Logo / Header Image

### Step 1 · Enable Custom Header Support

```php
<?php
add_theme_support('custom-header');
```

### Step 2 · Display It

```php
<img src="<?php echo get_header_image(); ?>" alt="Logo">
```

> 💡 **Modern alternative:** for a site *logo* specifically (as opposed to a header banner), `add_theme_support('custom-logo')` + `the_custom_logo()` is the more common, more flexible approach in current theme development.

---

## 🧱 Topic 6 — Creating a Custom Page Template

**Why?** Sometimes one page (like Contact Us) needs a completely different layout from the rest of the site. A Custom Page Template lets any page opt into a unique design.

### Step 1 · Create a New PHP File

Example: `template-contact.php` — the filename is flexible; it does **not** need to start with `template-`.

### Step 2 · Declare It as a Template

This comment block at the top tells WordPress "this file is selectable as a page template":

```php
<?php
/*
Template Name: Contact Us
*/
?>
```

### Step 3 · Build the Layout

```php
<?php
/*
Template Name: Contact Us
*/

get_header();
?>

<h1>Contact Us</h1>

<p>Welcome to our contact page.</p>

<?php get_footer(); ?>
```

### Step 4 · Assign It to a Page

> **WordPress Admin → Pages → Add/Edit Page → Template dropdown → "Contact Us"**

---

## 🔄 Topic 7 — Showing Posts on the Front Side (The Loop)

WordPress uses **"The Loop"** — its core mechanism for pulling and displaying posts dynamically.

### Step 1 · Create a Blog Page

**Pages → Add New** → title it "Blog" → Publish.

### Step 2 · Assign It as the Posts Page

> **Settings → Reading → Posts page → select "Blog"**

### Step 3 · Write the Loop

```php
<?php
while (have_posts()) {
    the_post();
?>

    <h2><?php the_title(); ?></h2>

    <?php the_post_thumbnail(); ?>

    <p><?php the_excerpt(); ?></p>

<?php
}
?>
```

| Function | What it does |
|---|---|
| `have_posts()` | Returns `true` while there are more posts left to display |
| `the_post()` | Advances to the next post and loads its data into memory |

> 🧠 **Mental model:** think of it like a `foreach` loop over an array of posts — `have_posts()` is the condition, `the_post()` moves the pointer forward.

### Post Function Reference

| Function | Purpose |
|---|---|
| `the_title()` | Post title |
| `the_content()` | Full post content |
| `the_excerpt()` | Short summary/description |
| `the_post_thumbnail()` | Featured image |
| `the_date()` | Post date (echoes) |
| `get_the_date()` | Post date (returns) |
| `the_author()` | Author name |
| `the_permalink()` | URL of the post |

---

## 📑 Topic 8 — Adding Pagination to the Blog

Pagination splits posts across pages, e.g. `1 2 3 4 5 Next`.

### Step 1 · Set Posts Per Page

> **Settings → Reading → "Blog pages show at most" → e.g. 10**

### Step 2 · Install a Pagination Plugin

> **Plugins → Add New → search "WP‑PageNavi" → Install & Activate**

### Step 3 · Output the Pagination

```php
<?php wp_pagenavi(); ?>
```

**Full example:**

```php
<?php
while (have_posts()) {
    the_post();
?>
    <h2><?php the_title(); ?></h2>
    <?php the_post_thumbnail(); ?>
    <?php the_excerpt(); ?>
<?php
}
?>

<?php wp_pagenavi(); ?>
```

### Step 4 · Add `wp_head()` and `wp_footer()`

> ⚠️ **Critical:** without these two hooks, plugins (like WP‑PageNavi) and WordPress core CSS/JS **will not work correctly**.

```php
<html>
<head>
    <title>My Website</title>
    <?php wp_head(); ?>
</head>
<body>
    <!-- Website Content -->
    <?php wp_footer(); ?>
</body>
</html>
```

| Hook | Placement | Purpose |
|---|---|---|
| `wp_head()` | Just before `</head>` | Lets plugins/themes inject CSS, meta tags, scripts |
| `wp_footer()` | Just before `</body>` | Lets plugins/themes inject scripts at the end |

---

## 📰 Topic 9 — Showing Single Post Details

When a user clicks a post, WordPress needs a dedicated template to show the full article.

### Step 1 · Create `single.php`

### Step 2 · Build the Full Post View

```php
<?php get_header(); ?>

<?php
while (have_posts()) {
    the_post();
?>

    <h1><?php the_title(); ?></h1>

    <?php the_post_thumbnail('large'); ?>

    <p>Published on: <?php echo get_the_date(); ?></p>

    <p>Author: <?php the_author(); ?></p>

    <div>
        <?php the_content(); ?>
    </div>

<?php
}
?>

<?php get_footer(); ?>
```

### 🔗 Bonus: "Read More" Link

Add this in your blog listing (Topic 7) so each post card links to its full `single.php` view:

```php
<h2><?php the_title(); ?></h2>

<?php the_excerpt(); ?>

<a href="<?php the_permalink(); ?>">Read More</a>
```

`the_permalink()` provides the post's URL — clicking it automatically loads `single.php`.

---

## 💬 Topic 10 — Comment Form on Blog Detail Page

### 1. Display the Comment Form

```php
<?php comment_form(); ?>
```

Displays the WordPress comment submission form.

### 2. Display Comments

```php
<?php wp_list_comments(); ?>
```

Displays the comments and replies for the current post.

### 3. `wp_list_comments()` Arguments

```php
<?php
wp_list_comments(
    array(
        'style'             => 'ol',
        'avatar_size'       => 50,
        'type'              => 'comment',
        'max_depth'         => 3,
        'reverse_top_level' => false,
        'reverse_children'  => false,
        'callback'          => 'my_custom_comment',
        'per_page'          => 10
    )
);
?>
```

| Argument | Use |
|---|---|
| `style` | HTML structure used to display comments — `ol`, `ul`, or `div` |
| `avatar_size` | Size of the comment author's avatar in pixels |
| `short_ping` | Displays pingbacks in a shorter format when `true` |
| `type` | Which type of comments to display — `all`, `comment`, `pingback`, or `trackback` |
| `reverse_top_level` | Reverses the order of top-level comments when `true` |
| `reverse_children` | Reverses the order of replies/child comments when `true` |
| `callback` | Custom function to control the HTML/design of each comment |
| `end-callback` | Runs a custom function after each comment's children/replies are displayed |
| `page` | Page number of comments to display |
| `per_page` | How many comments to display per page |
| `max_depth` | Maximum depth/level of nested comments and replies |

### 4. Custom Comment Callback

```php
<?php
function my_custom_comment($comment, $args, $depth)
{
    ?>

    <li class="custom-comment">

        <h4><?php echo get_comment_author(); ?></h4>

        <p><?php comment_text(); ?></p>

        <small><?php comment_date(); ?></small>

    </li>

    <?php
}

wp_list_comments(
    array(
        'callback' => 'my_custom_comment'
    )
);
?>
```

The `callback` argument tells WordPress to use your custom function for displaying each comment.

### 5. Display Comments + Comment Form Together

```php
<?php comments_template(); ?>
```

Loads the theme's `comments.php` file to display comments and the comment form.

> 💡 If you use `comments_template()` in `single.php`, you normally do **not** need to separately call `comment_form()` there.

**Example `comments.php`:**

```php
<?php
wp_list_comments();
comment_form();
?>
```

---

## 🏷️ Topic 11 — Showing Posts by Category

**Why?** By default, WordPress uses `index.php` (or `home.php`) to show *all* posts mixed together. A `category.php` template lets you display posts filtered by whichever category the visitor is browsing.

### Step 1 · Create `category.php`

Copy the **same code** used in `index.php` (or `home.php`) into this new file — The Loop works identically, WordPress automatically limits it to the current category's posts.

```php
<?php get_header(); ?>

<?php
while (have_posts()) {
    the_post();
?>

    <h2><?php the_title(); ?></h2>

    <?php the_post_thumbnail(); ?>

    <p><?php the_excerpt(); ?></p>

<?php
}
?>

<?php get_footer(); ?>
```

> 💡 **How it works:** WordPress's **Template Hierarchy** automatically loads `category.php` whenever a category archive page is viewed (e.g. `yoursite.com/category/news/`) — no extra code is needed to detect which category is active.

> ✅ **Checkpoint:** If `category.php` doesn't exist, WordPress falls back to `archive.php`, then to `index.php`. Creating `category.php` is only needed when you want a layout **different** from the rest of the site.

---

## 🧰 Topic 12 — Creating Your Custom Widget / Sidebar

**Why?** A custom widget area lets the site admin drag-and-drop content (like a "Recent Posts" box, contact info, or ads) from **Appearance → Widgets**, without touching any code.

### Step 1 · Register the Sidebar in `functions.php`

Use `register_sidebar()` so the widget area shows up in the admin panel:

```php
<?php
register_sidebar(
    array(
        'name'        => "Your Area Name",
        'id'          => "sidebar",
        'description' => "It's Optional"
    )
);
?>
```

| Argument | Purpose |
|---|---|
| `name` | Friendly label shown in **Appearance → Widgets** |
| `id` | Internal slug/ID used to call this sidebar in template files |
| `description` | Optional helper text shown in the admin panel |

### Step 2 · Output the Widget Area in a Template File

Use `dynamic_sidebar('your sidebar id')` wherever you want the widgets to appear on the front end (e.g. in `sidebar.php`):

```php
<?php dynamic_sidebar('sidebar'); ?>
```

> ✅ **Checkpoint:** The string passed to `dynamic_sidebar()` must **exactly match** the `id` used in `register_sidebar()` — same rule as the menu `theme_location` mismatch bug from Topic 2.

---

## 🔌 Topic 13 — Calling a Sidebar from Any Template File

**Why?** Instead of pasting the same sidebar HTML into `header.php`, `footer.php`, `page.php`, etc., you build it **once** and *pull it in* wherever it's needed — exactly like `get_header()` and `get_footer()` work.

### Step 1 · Create `sidebar.php`

Put all your sidebar markup here — widgets, recent posts, ads, whatever the sidebar should contain:

```php
<div class="sidebar">

    <h3>Sidebar</h3>

    <?php dynamic_sidebar('sidebar'); ?>

</div>
```

### Step 2 · Call It Dynamically with `get_sidebar()`

From any template file (`page.php`, `single.php`, `footer.php`, etc.), simply write:

```php
<?php get_sidebar(); ?>
```

WordPress automatically looks for `sidebar.php` and includes it at that exact spot.

**Example — using it inside `page.php`:**

```php
<?php get_header(); ?>

<div class="content">
    <h1><?php the_title(); ?></h1>
    <?php the_content(); ?>
</div>

<?php get_sidebar(); ?>

<?php get_footer(); ?>
```

### Step 3 · Creating Additional / Named Sidebars

One sidebar isn't always enough — e.g. you might want a different sidebar for the blog vs. the shop page. Create a **new file** named:

```
sidebar-your-id-or-name.php
```

For example, `sidebar-shop.php`:

```php
<div class="sidebar sidebar-shop">

    <h3>Shop Sidebar</h3>

    <?php dynamic_sidebar('shop-sidebar'); ?>

</div>
```

Then call this specific sidebar by passing its name/id into `get_sidebar()`:

```php
<?php get_sidebar('shop'); ?>
```

> 💡 **How it works:** `get_sidebar('shop')` tells WordPress to look for a file named `sidebar-shop.php` — the part after `sidebar-` is whatever string you pass as the argument.

### Quick Reference

| Function | Loads |
|---|---|
| `get_sidebar();` | `sidebar.php` (the default sidebar) |
| `get_sidebar('shop');` | `sidebar-shop.php` (a named/custom sidebar) |
| `get_sidebar('footer');` | `sidebar-footer.php` |

> ✅ **Checkpoint:** `get_sidebar()` only *includes the file* — you still need `register_sidebar()` (Topic 12) done in `functions.php` for the widget area itself to appear in **Appearance → Widgets**.

---

## 🗂 Final Theme File Structure

```text
mytheme/
│
├── style.css
├── index.php
├── header.php
├── footer.php
├── functions.php
├── sidebar.php
├── page.php
├── single.php
├── template-contact.php
├── category.php
├── sidebar.php
├── sidebar-shop.php
│
└── images/
    ├── logo.png
    └── banner.jpg
```

### 🔁 The Build Order (Flow)

```
 1. Create Theme
        ↓
 2. style.css + index.php
        ↓
 3. Activate Theme
        ↓
 4. header.php + footer.php
        ↓
 5. functions.php
        ↓
 6. Dynamic Menu
        ↓
 7. page.php
        ↓
 8. Featured Image
        ↓
 9. Custom Page Template
        ↓
10. Display Blog Posts
        ↓
11. Pagination
        ↓
12. single.php
        ↓
13. category.php
        ↓
14. Custom Widget / Sidebar
        ↓
15. get_sidebar() in Templates
```

---

## 📋 Quick Revision Table (All Important Functions)

| Function | Use |
|---|---|
| `get_header()` | Include `header.php` |
| `get_footer()` | Include `footer.php` |
| `get_sidebar()` | Include `sidebar.php` |
| `get_template_directory_uri()` | Get theme folder URL |
| `site_url()` | Get website URL |
| `wp_nav_menu()` | Display dynamic menu |
| `the_title()` | Display title |
| `the_content()` | Display content |
| `the_excerpt()` | Display short content |
| `the_post_thumbnail()` | Display featured image |
| `the_permalink()` | Get post URL |
| `the_date()` | Display post date |
| `the_author()` | Display author |
| `have_posts()` | Check remaining posts |
| `the_post()` | Load current post |
| `wp_head()` | Inject WP/plugin code in `<head>` |
| `wp_footer()` | Inject WP/plugin code before `</body>` |
| `comment_form()` | Display comment submission form |
| `wp_list_comments()` | Display comments and replies |
| `comments_template()` | Load `comments.php` (comments + form) |
| `register_sidebar()` | Register a custom widget area (in `functions.php`) |
| `dynamic_sidebar()` | Output a registered widget area on the front end |
| `get_sidebar()` | Include `sidebar.php` (or `sidebar-name.php` when passed an argument) |

---

## 🎯 Practical Task: College Website Theme

Build a **College Website Custom WordPress Theme** with:

- [ ] Dynamic Header Menu
- [ ] College Logo
- [ ] Home Page
- [ ] About Us Page
- [ ] Courses Page
- [ ] Faculty Page
- [ ] Blog Page
- [ ] Featured Images
- [ ] Dynamic Blog Posts (The Loop)
- [ ] Read More Button
- [ ] Single Blog Details Page
- [ ] Blog Pagination
- [ ] Contact Us Custom Template
- [ ] Category-wise Blog Filtering
- [ ] Custom Widget / Sidebar Area
- [ ] Header and Footer
- [ ] Proper CSS Design

### Required Theme Files

```text
style.css
index.php
header.php
footer.php
functions.php
page.php
single.php
template-contact.php
category.php
```

> 🎯 **Goal:** Every part of the site — menu, pages, blog, contact form template — should be manageable from the **WordPress Admin Panel**, with no code edits needed for routine content changes.

---

> 📝 **Tip for revision:** Work through the file-structure diagram and try to explain, from memory, what each file does and which functions belong in it. If you can do that without looking back, you know this topic.
