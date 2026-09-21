# 🎨 WordPress Custom Theme Development

### BCA Semester 3 — Complete Study Notes

A step-by-step guide to building a custom WordPress theme from scratch — dynamic menus, custom page templates, featured images, a working blog with pagination, and comments.

---

## 📑 Table of Contents

| # | Topic |
|---|---|
| 1 | [Creating a Custom Theme](#topic-1) |
| 2 | [Making the Header Menu Dynamic](#topic-2) |
| 3 | [Creating a Custom Page Layout](#topic-3) |
| 4 | [Adding a Featured Image](#topic-4) |
| 5 | [Adding a Logo / Header Image](#topic-5) |
| 6 | [Creating a Custom Page Template](#topic-6) |
| 7 | [Showing Posts on the Front Side (The Loop)](#topic-7) |
| 8 | [Adding Pagination to the Blog](#topic-8) |
| 9 | [Showing Single Post Details](#topic-9) |
| 10 | [Comment Form on Blog Detail Page](#topic-10) |
| 11 | [Showing Posts by Category](#topic-11) |
| 12 | [Creating a Custom Widget / Sidebar](#topic-12) |
| 13 | [Calling a Sidebar from Any Template File](#topic-13) |
| 14 | [Making the Title Dynamic](#topic-14) |
| 15 | [Adding a Background Image](#topic-15) |
| 16 | [Creating a 404 Error Page](#topic-16) |
| 17 | [Showing Categories in a Page](#topic-17) |
| 18 | [Adding a Custom Post Type](#topic-18) |
| 19 | [Adding a Custom Taxonomy or Category](#topic-19) |
| 20 | [Final Theme File Structure](#final-structure) |
| 21 | [Quick Revision Table](#revision-table) |
| 22 | [Practical Task](#practical-task) |

---

<a id="topic-1"></a>

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

<a id="topic-2"></a>

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

<a id="topic-3"></a>

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

<a id="topic-4"></a>

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

<a id="topic-5"></a>

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

<a id="topic-6"></a>

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

<a id="topic-7"></a>

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

<a id="topic-8"></a>

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

<a id="topic-9"></a>

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

<a id="topic-10"></a>

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

<a id="topic-11"></a>

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

<a id="topic-12"></a>

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

<a id="topic-13"></a>

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

<a id="topic-14"></a>

## 🔤 Topic 14 — Making the Title Dynamic

**Why?** A hardcoded `<title>My Website</title>` never changes no matter what page a visitor is on — bad for SEO and bad for usability. WordPress gives you functions that build the title (and site name/tagline) dynamically from the admin settings and the current page.

### 1. `wp_title()` — Title of the Current Page

Returns/echoes the title of whichever page is currently being viewed (post title, page title, category name, etc.).

```php
<title><?php wp_title(); ?></title>
```

### 2. `bloginfo('name')` — Website Title

Outputs the site title exactly as set in **Settings → General → Site Title**.

```php
<h1><?php bloginfo('name'); ?></h1>
```

### 3. `bloginfo('description')` — Website Tagline

Outputs the site's tagline, set in **Settings → General → Tagline**.

```php
<p><?php bloginfo('description'); ?></p>
```

### Putting It All Together

```php
<head>
    <title><?php wp_title(); ?> | <?php bloginfo('name'); ?></title>
</head>

<body>

    <header>
        <h1><?php bloginfo('name'); ?></h1>
        <p><?php bloginfo('description'); ?></p>
    </header>

</body>
```

> 💡 **Where these come from:** `bloginfo('name')` and `bloginfo('description')` both pull directly from **Settings → General** in the WordPress Admin — change them there and every page updates automatically, with no code edits.

---

<a id="topic-15"></a>

## 🖼️ Topic 15 — Adding a Background Image

**Why?** Instead of hardcoding a background image in CSS, WordPress can let the site admin **upload and change the background** themselves from the dashboard.

### Step 1 · Enable Support in `functions.php`

```php
<?php
add_theme_support('custom-background');
```

This adds a new **Appearance → Background** option in the admin panel, where the admin can upload an image, pick a color, and set repeat/position.

### Step 2 · Output It with `body_class()`

Add `body_class()` inside the `<body>` tag — WordPress uses it to automatically attach the right CSS classes (including the custom background) to the page:

```php
<body <?php body_class(); ?>>

    <!-- Website Content -->

</body>
```

> 💡 `body_class()` doesn't just handle the background — it also adds useful context classes like `page`, `home`, `single-post`, `logged-in`, etc., which are handy for page-specific CSS styling.

> ✅ **Checkpoint:** If the background image doesn't show up on the front end after uploading it in **Appearance → Background**, double-check that `body_class()` is actually present in the `<body>` tag — this is the #1 reason the feature silently doesn't work.

---

<a id="topic-16"></a>

## 🚫 Topic 16 — Creating a 404 Error Page

**Why?** When a visitor hits a broken/missing link, WordPress needs a friendly page to show instead of a blank error — this is fully customizable per theme.

### Step 1 · Create `404.php`

Any time WordPress can't find a matching page, post, or template for a URL, it automatically loads `404.php` and shows its content.

### Step 2 · Build the Content

```php
<?php get_header(); ?>

<div class="error-404">

    <h1>404 — Page Not Found</h1>

    <p>Sorry, the page you're looking for doesn't exist or has been moved.</p>

    <a href="<?php echo site_url(); ?>">← Back to Home</a>

</div>

<?php get_footer(); ?>
```

> ✅ **Checkpoint:** No extra registration or function call is needed for `404.php` — WordPress's built-in **Template Hierarchy** automatically detects a missing page/post and loads this file if it exists in your theme folder.

---

<a id="topic-17"></a>

## 🗃️ Topic 17 — Showing Categories in a Page

**Why?** Sometimes you want to list **all categories** somewhere on the site — e.g. a "Browse by Category" widget, a footer menu, or a category grid on the homepage — without hardcoding each one.

### Step 1 · Get All Categories with `get_categories()`

```php
<?php
$categories = get_categories();
?>
```

This returns an **array of category objects** — each one representing a single category and its data (name, ID, slug, count, etc.).

### Step 2 · Loop Through Them with `foreach()`

```php
<?php
foreach ($categories as $category) {
    // display each category here
}
?>
```

### Step 3 · Get the Category Name — `$category->name`

```php
<?php echo $category->name; ?>
```

### Step 4 · Get the Category ID — `$category->term_id`

```php
<?php echo $category->term_id; ?>
```

### Step 5 · Get the Category URL — `get_category_link($category->term_id)`

```php
<?php echo get_category_link($category->term_id); ?>
```

### Full Working Example

```php
<ul class="category-list">
    <?php
    $categories = get_categories();

    foreach ($categories as $category) {
    ?>
        <li>
            <a href="<?php echo get_category_link($category->term_id); ?>">
                <?php echo $category->name; ?>
            </a>
        </li>
    <?php
    }
    ?>
</ul>
```

This outputs something like:

```html
<ul class="category-list">
    <li><a href="https://example.com/category/news/">News</a></li>
    <li><a href="https://example.com/category/events/">Events</a></li>
    <li><a href="https://example.com/category/sports/">Sports</a></li>
</ul>
```

### Function / Property Reference

| Item | Purpose |
|---|---|
| `get_categories()` | Returns an array of all category objects |
| `$category->name` | The category's display name |
| `$category->term_id` | The category's unique numeric ID |
| `get_category_link($id)` | Returns the URL of that category's archive page |

> 💡 **Where this is used:** Drop this snippet into `sidebar.php` for a "Browse by Category" widget, into `footer.php` for a category footer menu, or into `page.php`/`index.php` for a category grid on the homepage.

> ✅ **Checkpoint:** Clicking any link from this loop takes the visitor to that category's archive — which is exactly the page rendered by `category.php` from **Topic 11**. The two topics work together: this lists the categories, `category.php` displays the posts inside each one.

---

<a id="topic-18"></a>

## 🧩 Topic 18 — How to Add a Custom Post Type

**Why?** Sometimes "Posts" and "Pages" aren't enough — e.g. a college site needs "Faculty," "Courses," or "Events" as their own content type, each manageable separately from the admin panel.

### Step 1 · Install a Custom Post Type Plugin

The easiest way to get a Custom Post Type (CPT) option in the admin side is via a plugin:

> **Plugins → Add New → search "Custom Post Type UI" → Install & Activate**

Once activated, you'll find a new **CPT UI** menu in the admin sidebar where you can create a post type (e.g. `faculty`) just by filling a form — no code required for the registration step itself.

> 💡 **Alternative (code-based):** advanced users can register a CPT manually in `functions.php` with `register_post_type()`, but the plugin route is faster and matches what this topic is teaching.

### Step 2 · Show Custom Post Type Posts on the Front Side with `WP_Query()`

Once your custom post type exists (with some entries added, e.g. under "Faculty" in the admin), use `WP_Query()` to pull those posts into any template file — `page.php`, `index.php`, a custom template, wherever you need them.

```php
<?php
$faculty_query = new WP_Query(
    array(
        'post_type'      => 'faculty',
        'posts_per_page' => 10
    )
);
?>
```

### Step 3 · Loop Through the Results

`WP_Query()` gives you its own mini-Loop — same idea as the default Loop from Topic 7, but scoped to just this query:

```php
<?php
if ($faculty_query->have_posts()) {
    while ($faculty_query->have_posts()) {
        $faculty_query->the_post();
?>

        <h2><?php the_title(); ?></h2>

        <?php the_post_thumbnail('medium'); ?>

        <p><?php the_excerpt(); ?></p>

<?php
    }
}
?>
```

### Step 4 · Reset Post Data

Always call `wp_reset_postdata()` after a custom `WP_Query()` loop — this restores WordPress's main post data so the rest of the page (or the default Loop) doesn't get confused about which post is "current."

```php
<?php wp_reset_postdata(); ?>
```

### Full Working Example

```php
<div class="faculty-list">

    <?php
    $faculty_query = new WP_Query(
        array(
            'post_type'      => 'faculty',
            'posts_per_page' => 10
        )
    );

    if ($faculty_query->have_posts()) {
        while ($faculty_query->have_posts()) {
            $faculty_query->the_post();
    ?>
            <div class="faculty-card">
                <h2><?php the_title(); ?></h2>
                <?php the_post_thumbnail('medium'); ?>
                <p><?php the_excerpt(); ?></p>
            </div>
    <?php
        }
        wp_reset_postdata();
    }
    ?>

</div>
```

### Function / Argument Reference

| Item | Purpose |
|---|---|
| Custom Post Type UI (plugin) | Lets the admin create custom post types (e.g. `faculty`, `courses`) from the dashboard |
| `new WP_Query(array(...))` | Runs a custom query — here, fetching posts of a specific `post_type` |
| `'post_type'` | Argument that tells `WP_Query()` which post type to fetch |
| `'posts_per_page'` | Argument that limits how many results are returned |
| `have_posts()` / `the_post()` | Same Loop functions from Topic 7, called on the `$faculty_query` object instead of the main query |
| `wp_reset_postdata()` | Restores the global post data after a custom `WP_Query()` loop |

> ✅ **Checkpoint:** Forgetting `wp_reset_postdata()` is the #1 bug with custom `WP_Query()` loops — without it, functions like `the_title()` used *after* this block may show data from the wrong post.

---

<a id="topic-19"></a>

## 🏷️ Topic 19 — How to Add a Custom Taxonomy or Category

**Why?** A Custom Post Type (Topic 18) often needs its *own* categorization system — e.g. "Faculty" posts grouped by **Department**, or "Courses" grouped by **Subject** — separate from the default Posts categories.

### Step 1 · Add the Taxonomy from a Plugin

The same **Custom Post Type UI** plugin used in Topic 18 also lets you create custom taxonomies from the admin panel:

> **CPT UI → Add/Edit Taxonomies** → give it a name and slug (e.g. `department`) → attach it to your custom post type (e.g. `faculty`) → Save

Once saved, a new taxonomy box appears on the edit screen for that post type — just like the default "Category" box for regular Posts.

### Step 2 · Fetch the Taxonomy Terms with `get_terms()`

To display these custom categories on the front end, use `get_terms()` and pass your taxonomy's slug:

```php
<?php
$categories = get_terms(
    array(
        'taxonomy' => 'department'
    )
);
?>
```

> 💡 By default, `get_terms()` only returns terms that have **at least one post** assigned to them.

### Step 3 · Show Empty Terms Too (Optional)

If you also want terms with **zero posts** to appear (e.g. a newly created department with no faculty added yet), add `'hide_empty' => false`:

```php
<?php
$categories = get_terms(
    array(
        'taxonomy'   => 'department',
        'hide_empty' => false
    )
);
?>
```

### Full Working Example

```php
<?php
$categories = get_terms(array(
    'taxonomy'   => 'department',
    'hide_empty' => false
));

if (!empty($categories) && !is_wp_error($categories)) {
    foreach ($categories as $category) {
        echo '<h3>' . esc_html($category->name) . '</h3>';
    }
}
?>
```

This outputs something like:

```html
<h3>Computer Science</h3>
<h3>Commerce</h3>
<h3>Science</h3>
```

### Function / Argument Reference

| Item | Purpose |
|---|---|
| `get_terms(array(...))` | Fetches terms (categories) belonging to a specific taxonomy |
| `'taxonomy'` | Argument specifying which taxonomy's terms to fetch (e.g. `department`) |
| `'hide_empty'` | `true` (default) shows only terms with posts; `false` shows all terms, even empty ones |
| `is_wp_error($categories)` | Checks whether `get_terms()` returned an error instead of results |
| `esc_html()` | Safely escapes text before output — good practice whenever printing dynamic content |

> ✅ **Checkpoint:** Always check `!empty($categories) && !is_wp_error($categories)` before looping — `get_terms()` returns a `WP_Error` object (not an empty array) if the taxonomy slug is misspelled, so skipping this check can throw a PHP warning instead of failing silently.

> 💡 **How this connects:** Topic 17 (`get_categories()`) fetches terms from the default **category** taxonomy only. `get_terms()` is the more general version — it can fetch terms from *any* taxonomy, default or custom, which is exactly what a Custom Post Type from Topic 18 needs.

---

<a id="final-structure"></a>

## 🗂 Final Theme File Structure

```text
mytheme/
│
├── style.css
├── index.php
├── header.php
├── footer.php
├── functions.php
├── page.php
├── single.php
├── template-contact.php
├── category.php
├── sidebar.php
├── sidebar-shop.php
├── 404.php
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
        ↓
16. Dynamic Title
        ↓
17. Background Image
        ↓
18. 404.php
        ↓
19. List Categories
        ↓
20. Custom Post Type + WP_Query
        ↓
21. Custom Taxonomy + get_terms()
```

---

<a id="revision-table"></a>

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
| `wp_title()` | Display the title of the current page/post |
| `bloginfo('name')` | Display the site title (from Settings → General) |
| `bloginfo('description')` | Display the site tagline (from Settings → General) |
| `body_class()` | Add dynamic CSS classes (incl. custom background) to `<body>` |
| `get_categories()` | Get an array of all category objects |
| `get_category_link($id)` | Get the URL of a category's archive page |
| `new WP_Query(array(...))` | Run a custom query (e.g. fetch posts of a custom post type) |
| `wp_reset_postdata()` | Restore global post data after a custom `WP_Query()` loop |
| `get_terms(array(...))` | Fetch terms from any taxonomy, default or custom |
| `is_wp_error()` | Check whether a WordPress function returned an error object |

---

<a id="practical-task"></a>

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
- [ ] Dynamic Site Title & Tagline
- [ ] Custom Background Image
- [ ] Custom 404 Error Page
- [ ] Category List / Browse by Category
- [ ] Custom Post Type (e.g. Faculty) with `WP_Query`
- [ ] Custom Taxonomy (e.g. Department) with `get_terms()`
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
sidebar.php
404.php
```

> 🎯 **Goal:** Every part of the site — menu, pages, blog, contact form template — should be manageable from the **WordPress Admin Panel**, with no code edits needed for routine content changes.

---

> 📝 **Tip for revision:** Work through the file-structure diagram and try to explain, from memory, what each file does and which functions belong in it. If you can do that without looking back, you know this topic.
