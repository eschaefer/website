---
title: "Scrubbing HTML tags from Drupal search results"
tags: [drupal, php]
categories: blog
published: true
comments: true
---

For some bizarre reason, Drupal 7's out-of-the box search results teasers do not strip `<p>` and `<span>` tags if they exist in a custom field.

Let's fix this.

### 1. Create a search results template file in your theme.

Copy the contents of Drupal's base `search-result.tpl.php` out of `/modules/search/search-result.tpl.php` and into a new file located at `/sites/all/themes/YOURTHEME/templates/search-result.tpl.php`.

The comments in this file show us what variables we have access to. The `$snippet` variable contains the text that we want to scrub.

### 2. Edit the $snippet variable by preprocessing it.

Preprocessing is just Drupal's way of moving heavier template logic out of template files, into one giant dump of functions. Preprocessor functions are all held in your theme's `template.php` file. In this case we want a preprocessor function to give us a clean `$snippets` variable without HTML elements.

So, at the bottom of your template.php file, let's add a search preprocessor:

```php
function YOURTHEMENAME_preprocess_search_result(&$vars) {
  $vars['snippet'] = strip_tags(html_entity_decode($vars['snippet']));
}
```

Flush your theme cache once you've made this update.

Notice that I'm chaining two php functions together to accomplish this. `html_entity_decode` followed by `strip_tags`.

At last, when you render `$snippet` in your theme's `search-result.tpl.php`, the HTML markup will be gone. Hope this saves someone a headache.
