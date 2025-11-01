# Astra Child Theme - Artist Portfolio & Projects

A custom WordPress child theme built on Astra, featuring artist registration, custom project management, and dynamic AJAX-powered project filtering and loading.

---

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation & Setup](#installation--setup)
- [Pages Created](#pages-created)
- [Custom Post Type](#custom-post-type)
- [AJAX Logic Summary](#ajax-logic-summary)
- [Fancybox Integration](#fancybox-integration)
- [User Roles & Capabilities](#user-roles--capabilities)
- [File Structure](#file-structure)
- [Usage Guide](#usage-guide)

---

## Features

- Custom "Artist" user role with registration system
- Custom "Projects" post type with advanced custom fields
- AJAX-powered project filtering by type
- AJAX "Load More" pagination for projects
- Fancybox gallery integration for project images
- Bootstrap 5 integration for responsive design
- Fallback pagination for users with JavaScript disabled

---

## Requirements

- WordPress 5.0 or higher
- Astra theme (parent theme)
- Advanced Custom Fields (ACF) plugin
- PHP 7.4 or higher
- Modern web browser with JavaScript enabled

---

## Installation & Setup

### Step 1: Install Parent Theme
1. Install and activate the **Astra** theme from WordPress.org
2. Go to `Appearance > Themes > Add New`
3. Search for "Astra" and click Install, then Activate

### Step 2: Install Child Theme
1. Download this child theme folder
2. Upload to `/wp-content/themes/` directory
3. Go to `Appearance > Themes`
4. Activate **Astra Child** theme

### Step 3: Install Required Plugins
1. Install and activate **Advanced Custom Fields PRO** (ACF Pro) plugin
2. Go to `Plugins > Add New`
3. Search for "Advanced Custom Fields Pro" and install and Activate by following key
4. Pro Key: b3JkZXJfaWQ9MTM4NDI0fHR5cGU9ZGV2ZWxvcGVyfGRhdGU9MjAxOC0wOC0zMSAxMDoxNzozMQ==

### Step 4: Create Custom Fields (ACF)
Create a new ACF Field Group for the "Projects" post type:

**Field Group Settings:**
- Location: Post Type is equal to Projects

**Fields to Add:**
1. **Project Type**
   - Field Type: Text
   - Field Name: `project_type`
   - Required: No

2. **Project Budget**
   - Field Type: Number
   - Field Name: `project_budget`
   - Required: No

3. **Project Gallery**
   - Field Type: Gallery
   - Field Name: `project_gallery`
   - Required: No
   - Return Format: Image Array

### Step 5: Create Pages
Create the following pages in WordPress:

1. **Artist Registration Page**
   - Page Title: "Artist Registration" (or any title)
   - Page Template: Select "Artist Registration"
   - Publish the page

2. **Projects Listing Page**
   - Page Title: "Projects" (or any title)
   - Page Template: Select "Projects"
   - Publish the page

### Step 6: Set Permalinks
1. Go to `Settings > Permalinks`
2. Select "Post name" structure
3. Click "Save Changes"

---

## Pages Created

### 1. Artist Registration Page
**Template:** `artist-registration-tmp.php`

**Features:**
- Custom registration form for new artists
- Fields: First Name, Last Name, Username, Email, Password
- Automatic user creation with "Artist" role
- Auto-login after successful registration
- Validation for existing usernames and emails
- Redirect to WordPress dashboard after registration
- Shows welcome message for logged-in users

**Access:** Create a page and assign the "Artist Registration" template

---

### 2. Projects Listing Page
**Template:** `projects-listing-tmp.php`

**Features:**
- Displays all published projects in a grid layout (3 columns)
- AJAX-powered dropdown filter by project type
- AJAX "Load More" button for pagination (6 projects per page)
- Responsive Bootstrap card design
- Fallback pagination for non-JavaScript users
- Featured image, title, excerpt (15 words), and "Read More" button

**Access:** Create a page and assign the "Projects" template

---

### 3. Single Project Page
**Template:** `single-projects.php`

**Features:**
- Displays project title
- Featured image
- Full project description
- Project metadata (Type, Budget in ₹)
- Image gallery with Fancybox lightbox
- Artist information with link to author archive

**Access:** Automatically used when viewing individual projects

---

## Custom Post Type

### Projects Post Type
**Slug:** `projects`  
**Capabilities:** Custom capability type `project/projects`  
**Supports:** Title, Thumbnail, Excerpt, Editor

**Who Can Access:**
- Administrator (full access)
- Artist (full access)

**Menu Icon:** Portfolio (Dashicons)  
**Menu Position:** 5 (below Posts)

---

## AJAX Logic Summary

The theme uses WordPress AJAX with jQuery to provide seamless user interactions without page reloads.

### 1. Load More Functionality

**Client-Side (projects.js):**
```javascript
$('#load-more').on('click', function() {
    // Retrieves current page number from data-page attribute
    // Increments page by 1
    // Gets selected project type from filter dropdown
    // Sends AJAX POST request to WordPress
    // Appends new projects to existing list
    // Updates page number
    // Hides button when max pages reached
});
```

**Server-Side (functions.php - load_more_projects):**
- Receives page number and project type via POST
- Creates WP_Query with pagination
- Adds meta_query filter if project type selected
- Returns HTML markup for 6 projects per page
- Uses WordPress template tags for proper escaping

**Data Flow:**
1. User clicks "Load More" button
2. JavaScript gets current page and filter values
3. AJAX request sent to `admin-ajax.php`
4. PHP function queries database for next 6 projects
5. HTML response appended to project grid
6. Button state updated (hidden if last page)

---

### 2. Filter Functionality

**Client-Side (projects.js):**
```javascript
$('#project-filter').on('change keyup', function(e) {
    // Gets selected project type value
    // Sends AJAX POST request
    // Replaces entire project list with filtered results
    // Resets pagination to page 1
    // Updates "Load More" button visibility and max pages
});
```

**Server-Side (functions.php - filter_projects):**
- Receives project type via POST
- Creates WP_Query with filters
- Resets to page 1 with 6 posts per page
- Uses output buffering to capture HTML
- Returns JSON with:
  - `html`: Complete project list markup
  - `max_pages`: Total pages for pagination

**Data Flow:**
1. User selects project type from dropdown
2. JavaScript captures change event
3. AJAX request sent with selected type
4. PHP queries database with meta_query filter
5. JSON response with HTML and pagination info
6. JavaScript replaces content and updates button

---

### AJAX Hooks Used

Both functions are registered for logged-in and non-logged-in users:

```php
add_action('wp_ajax_load_more_projects', 'load_more_projects');
add_action('wp_ajax_nopriv_load_more_projects', 'load_more_projects');
add_action('wp_ajax_filter_projects', 'filter_projects');
add_action('wp_ajax_nopriv_filter_projects', 'filter_projects');
```

---

### Script Localization

The AJAX URL is made available to JavaScript via:

```php
wp_localize_script('project-js', 'project_ajax', array(
    'ajax_url' => admin_url('admin-ajax.php')
));
```

Access in JS: `project_ajax.ajax_url`

---

## Fancybox Integration

### What is Fancybox?
Fancybox is a modern lightbox library that provides elegant image galleries with zoom, navigation, and responsive design.

### Implementation Details

**CDN Loading (functions.php):**
```php
// Only loaded on single project pages
if(is_singular('projects')) {
    wp_enqueue_style('fancybox-css', 
        'https://cdn.jsdelivr.net/npm/@fancyapps/ui/dist/fancybox.css');
    wp_enqueue_script('fancybox-js', 
        'https://cdn.jsdelivr.net/npm/@fancyapps/ui/dist/fancybox.umd.js', 
        array('jquery'), null, true);
}
```

**HTML Structure (single-projects.php):**
```php
<a href="<?php echo esc_url($image['url']); ?>" data-fancybox="gallery">
    <img src="<?php echo esc_url($image['sizes']['thumbnail']); ?>" 
         alt="<?php echo esc_attr($image['alt']); ?>">
</a>
```

### How It Works
1. **Conditional Loading:** Fancybox scripts only load on single project pages (performance optimization)
2. **Data Attribute:** `data-fancybox="gallery"` groups all images in the same gallery
3. **Thumbnail Display:** Shows 120px thumbnails with border-radius styling
4. **Full Image Link:** Links to full-size image for lightbox
5. **Auto-Initialization:** Fancybox automatically detects and initializes gallery

### Features
- Click any thumbnail to open lightbox
- Navigate between images with arrows/keyboard
- Zoom and pan functionality
- Mobile-friendly touch gestures
- Automatic image centering
- Dark overlay background
- Close with ESC key or click outside

---

## User Roles & Capabilities

### Artist Role
**Capabilities:**
- Read published posts
- Edit own posts
- Upload files
- Full project management:
  - Create projects (`publish_projects`)
  - Edit own projects (`edit_projects`)
  - Edit others' projects (`edit_others_projects`)
  - Delete projects (`delete_projects`)
  - Delete others' projects (`delete_others_projects`)

### Administrator Role
- All Artist capabilities
- Full WordPress administration access

---

## File Structure

```
astra-child/
├── assets/
│   └── js/
│       └── projects.js          # AJAX handling for filter & load more
├── inc/
│   └── custom-artist-registration.php  # Artist role & registration logic
├── artist-registration-tmp.php  # Template: Artist Registration
├── projects-listing-tmp.php     # Template: Projects listing with AJAX
├── single-projects.php          # Template: Single project detail
├── functions.php                # Theme functions & AJAX handlers
├── style.css                    # Child theme styles
└── README.md                    # This file
```

---

## Usage Guide

### For Site Administrators

1. **Add New Project:**
   - Go to `Projects > Add New`
   - Enter title, description, featured image
   - Fill in ACF fields (Type, Budget, Gallery)
   - Publish

2. **Manage Artists:**
   - Go to `Users > All Users`
   - Artists have custom capabilities for projects
   - Can create/edit/delete projects

### For Artists

1. **Register:**
   - Visit the Artist Registration page
   - Fill in all required fields
   - Submit to auto-create account and login

2. **Create Projects:**
   - Access WordPress dashboard after registration
   - Go to `Projects > Add New`
   - Add project details and publish

### For Visitors

1. **Browse Projects:**
   - Visit the Projects page
   - Use dropdown to filter by project type
   - Click "Load More" to see additional projects
   - Click any project card to view full details

2. **View Project Details:**
   - See full description and metadata
   - Browse image gallery with Fancybox
   - Click thumbnails to open lightbox
   - Navigate between gallery images

---

## Notes

- **JavaScript Required:** Filter and Load More require JavaScript. Fallback pagination available in `<noscript>` tags
- **ACF Dependency:** The theme requires ACF for project custom fields
- **Bootstrap 5.3.8:** Loaded via CDN for styling
- **Security:** All inputs sanitized and validated, nonce verification on forms
- **Responsive:** Fully responsive design with Bootstrap grid system

---

## Support

For issues or questions, refer to:
- [Astra Theme Documentation](https://wpastra.com/docs/)
- [Advanced Custom Fields Documentation](https://www.advancedcustomfields.com/resources/)
- [Fancybox Documentation](https://fancyapps.com/fancybox/)

---

## Credits

- **Theme Base:** Astra by Brainstorm Force
- **Author:** Harsh Bakraniya
- **Bootstrap:** v5.3.8
- **Fancybox:** @fancyapps/ui
- **jQuery:** WordPress core

---

## Version History

**v1.0.0** - Initial Release
- Custom Projects post type
- Artist registration system
- AJAX filtering and pagination
- Fancybox gallery integration
- Bootstrap 5 integration
