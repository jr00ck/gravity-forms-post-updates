# Gravity Forms: Post Updates

* * *

http://wordpress.org/plugins/gravity-forms-post-updates/

* * *

This started as an update to the [Gravity Forms Update Post](http://wordpress.org/extend/plugins/gravity-forms-update-post/developers/) plugin developed by [p51labs](http://www.p51labs.com/) here.

It has evolved into a completely rewritten plugin that streamlines the system, adds some new support for more fields, and adds more interfaces and filters.

## Compatibility

Here is how it does NOT really support the original version:

* All of the delete functionality has been removed. It didn't have anything to do with Gravity Forms, so I thought it should be in a separate plugin or users can manually add it. It seemed like it complicated things in a way that distracted from the main function of the plugin. And I wasn't getting a lot of feedback that users were using it.
* You can NOT just send a URL get variable and have a form populate with existing content and be editable. I wanted another layer to that. So you have to include a nonce as well, however, there is a built in function for creating edit links to make up for that.

So it isn't very compatible. Plain and simple, you will have to change something to start using this plugin if you were using the original, but I don't think it will be very hard and I did keep the original query variable that was used before.

## Installation

This add-on can be treated as both a WP plugin and a theme include.

### Install as Plugin

1. Copy the folder into your plugins folder
2. Activate the plugin via the Plugins admin page

### Include within theme

1.	Copy the folder into your theme folder (can use sub folders). You can place the folder anywhere inside the 'wp-content' directory
2.	Edit your functions.php file and add the code below (Make sure the path is correct to include the gravityforms-update-post.php file)

```php
include_once('gravityforms-update-post/gravityforms-update-post.php');
```

## New Features

* Supports custom field file uploading and deletion with thumbnails or mime type icons for existing items.
* Fixed a bugs on multi selects and checkboxes.
* Fixed bug on Categories.
* Completely removed the ability to delete posts.
* There are some filters to customize things now.
* Adds non-query-var template method to setup a form.
* Adds a really basic shortcode to setup a form (UPDATE: This is still supported, but it is better to use the addition, below, to the gravityform shortcode).
* Adds an additional attribute to the gravityform shortcode: "update"

## Some Important Notes (FAQ)

* Tags really only work with a _single line text_ field, checkbox and multiselect currently won't show the selected items when loaded for editing, but they will select the items. This might get changed in the future, but isn't pressing. Categories support those other methods, and the text field seems more appropriate, over all, for the tags.
* Image fields are only supported if they are the "Featured Image". Otherwise you have to use a Custom Field and choose "File Upload" under File Type.


* * *
# Set Up Form

## URL QUERY VARIABLE

1. At the heart, it is pretty similar to how it was, but now there is a nonce required to activate it.
2. So you should use the action to create your links.

```php
do_action('gform_update_post/edit_link');

do_action('gform_update_post/edit_link', array(
	'post_id' => $post->ID,
	'url'     => home_url('/edit_post/')
) );
```

**Arguments (query string or array)**

* `post_id` (int) (optional) The id of the post you want to edit. Default: global $post->ID
* `url` (string|int) (optional) Either the full url of the page where your edit form resides, or an id for the page/post where the edit form resides. Default: get_permalink()
* `text` (string) (optional) The link text. Default: "Edit Post"
* `title` (string) (optional) The title attribute of the anchor tag. Default: (text) parameter

### Get just the URL

This will return a basic edit url
```php
apply_filters('gform_update_post/edit_url', '');
```
Specify post to edit (post_id) and post that holds the edit form (url)
```php
apply_filters('gform_update_post/edit_url', 1, home_url('/edit_post/'));
```

### Shortcode to show the edit link

```php
[gform_update_post_edit_link]
```
Specify post to edit (post_id) and post that holds the edit form (url)
```php
[gform_update_post_edit_link post_id=1 url=6]
```

## IN TEMPLATE

You can use the action to force a form show a specific post:

```php
do_action('gform_update_post/setup_form');

do_action('gform_update_post/setup_form', $post->ID);
```

**Parameters**

* `post_id` (int) (optional) The id of the post you want to edit. Default: global $post->ID

## SHORTCODE

```php
[gravityforms id="1" update] // Loads current post for editing

[gravityforms id="1" update="34"] // Loads post where ID=34 for editing
```

We worked with Rocketgenius, makers of Gravity Forms, to get a small upgrade added that allows us to extend their shortcode, so now you can simply add the "update" attribute to the normal "gravityform" shortcode. If you only add "update", it will load the current post in to update. If you add an integer to the update attribute, it will use that to load a post by its ID.

## OLD SHORTCODE METHOD

This is supported for legacy, but I am not sure why you would use it anymore moving forward.

```php
[gform_update_post]

[gform_update_post post_id=1]
```

**Options**

* `post_id` (int) (optional) The id of the post you want to edit. Default: global $post->ID

# Filter Overview

* `gform_update_post/request_id` Change the query variable that shows up in urls.
** instead of `?gform_post_id=913`, you could have `?edit_id=913`.
* `gform_update_post/file/width`, `gform_update_post/file/height` Width of file thumbnails, this includes images and file mime type icons. Unless you you changed mime type icons, there isn't much reason to change these.
* `gform_update_post/image/width`, `gform_update_post/image/height` Width of image thumbnails. These get generated by the plugin, and can be changed to anything. If you change them, after other sizes were generated, the other sizes will be orphaned and not deleted from the server.
* `gform_update_post/image/crop` Whether or not to crop image upload thumbnails.
* `gform_update_post/file/name` Can be used to filter the filenames of uploaded custom meta field files.
* `gform_update_post/public_edit` Can be used to conrol access. By default only users with access to edit a given post by default is allowed. Uses WordPress [Roles & Capabilities](http://codex.wordpress.org/Roles_and_Capabilities)

# Misc.

## Get Post Title to Update Immediately

Use: [gform_confirmation](http://www.gravityhelp.com/documentation/page/Gform_confirmation)

This will redirect the page and will skip any kind of confirmation message from Gravity Forms.

In your functions.php file

```php
add_filter( 'gform_confirmation', 'custom_gform_confirmation', 10, 4 );
	function custom_gform_confirmation( $confirmation, $form, $lead, $ajax )
	{
		if ( 'Edit a Post' == $form['title'] )
		{
			$confirmation = array( 'redirect' => get_permalink($lead['post_id']) );
		}
	
		return $confirmation;
	}
```

## Convert Filename to a Title

```php
add_filter('gform_update_post/file/name', 'custom_gform_update_post_file_name' );
	function custom_gform_update_post_file_name( $filename )
	{
		// Remove $ext
		$filename_array = explode('.', $filename);
		$ext = array_pop($filename_array);
		$filename = implode(' ', $filename_array);
		// convert to spaces
		$filename = str_replace(array('_','-'), ' ', $filename);
		// Title case
		$filename = ucwords($filename);
	
		return $filename;
	};
```

## Examples

Only show the form when the edit link has been clicked

```php
if (! empty($_GET['gform_post_id']) )
{
	// Show the form
	gravity_form(1);
}
else
{
	// Otherwise show a link to edit the form
	do_action( 'gform_update_post/edit_link', 'text=Edit '. get_the_title() );
}
```

Set up a form to edit current post

```php
do_action('gform_update_post/setup_form');
gravity_form(1);
```

Create an edit link to a specific edit page, adding an ID to the url parameter ($edit_page_id) will cause it to get the permalink for the post of that ID.

```php
do_action( 'gform_update_post/edit_link', array(
	'post_id' => $post->ID,
	'url' => $edit_page_id
) );
```

Or use a hard-coded url instead of post_id

```php
do_action( 'gform_update_post/edit_link', array(
	'post_id' => $post->ID,
	'url' => 'http://example.com/path/to/page/'
) );
```

Change the image thumbnail size (php 5.3 is required for anonymous functions)

```php
add_filter( 'gform_update_post/image/width', function(){ return 300; } );
add_filter( 'gform_update_post/image/height', function(){ return 100; } );

do_action('gform_update_post/setup_form');
gravity_form(1);
```

Change the image thumbnail size globally in functions.php

```php
add_filter('gform_update_post/image/width', 'custom_gform_update_post_image_width' );
	function custom_gform_update_post_image_width()
	{
		return 300;
	}
add_filter('gform_update_post/image/height', 'custom_gform_update_post_image_height' );
	function custom_gform_update_post_image_height()
	{
		return 100;
	}
```

Turn off cropping

```php
add_filter( 'gform_update_post/image/crop', '__return_false' );
```

Turn off image resizing

```php
add_filter( 'gform_update_post/image/resize', '__return_false' );
```

Disable Gravity Forms entries, reference: http://themergency.com/stop-gravity-forms-from-creating-form-entries/

```php
/**
 * Turn off all entries on all forms
 *
 * To only target a specific form or forms, add the form id to the action:
 * add_action( 'gform_post_submission_2', 'custom_remove_entries', 10, 2 ); // Targets form 2
 * add_action( 'gform_post_submission_5', 'custom_remove_entries', 10, 2 ); // Now you are targeting 2 and 5
 */
add_action( 'gform_post_submission', 'custom_remove_entries', 10, 2 );
function custom_remove_entries( $entry, $form )
{
	if (! class_exists('RGFormsModel') ) return;

	global $wpdb;

	$lead_id                = $entry['id'];
	$lead_table             = RGFormsModel::get_lead_table_name();
	$lead_notes_table       = RGFormsModel::get_lead_notes_table_name();
	$lead_detail_table      = RGFormsModel::get_lead_details_table_name();
	$lead_detail_long_table = RGFormsModel::get_lead_details_long_table_name();

	// Delete from detail long
	$sql = $wpdb->prepare( "DELETE FROM $lead_detail_long_table
								WHERE lead_detail_id IN(
								SELECT id FROM $lead_detail_table WHERE lead_id=%d
								)", $lead_id );
	$wpdb->query( $sql );

	// Delete from lead details
	$sql = $wpdb->prepare( "DELETE FROM $lead_detail_table WHERE lead_id=%d", $lead_id );
	$wpdb->query( $sql );

	// Delete from lead notes
	$sql = $wpdb->prepare( "DELETE FROM $lead_notes_table WHERE lead_id=%d", $lead_id );
	$wpdb->query( $sql );

	// Delete from lead
	$sql = $wpdb->prepare( "DELETE FROM $lead_table WHERE id=%d", $lead_id );
	$wpdb->query( $sql );
}
```

Allow the public to edit posts

```php
add_filter('gform_update_post/public_edit', '__return_true');
```

Allow **all** logged in users to edit posts

```php
add_filter('gform_update_post/public_edit', 'custom_gform_update_post_public_edit');
	function custom_gform_update_post_public_edit()
	{
		return 'loggedin';
	}
```

Only allow editors or higher to edit posts

```php
add_filter('gform_update_post/public_edit', 'custom_gform_update_post_public_edit');
	function custom_gform_update_post_public_edit()
	{
		return 'edit_posts';
	}
```