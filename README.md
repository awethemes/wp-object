# WP-Object

Completely OOP for WordPress objects

## Features

- Provide an abstract layer for working with WordPress objects (post, CPT, taxonomy, etc...) with fluent
API for create, update and delete.
- Support WordPress meta-data with `mappings` for automatic update.
- And more..

## Install

Install via composer

```
composer require awethemes/wp-object
```

## Examples:

An simple example:

```php
<?php

use Awethemes\WP_Object\WP_Object;

class Product_Category extends WP_Object {
    /**
     * This is the name of this object type.
     *
     * @var string
     */
    protected $object_type = 'product_category';

    /**
     * WordPress type for object, Ex: "post" and "term".
     *
     * @var string
     */
    protected $wp_type = 'term';

    /**
     * Type of object metadata is for (e.g., term, post).
     *
     * @var string
     */
    protected $meta_type = 'term';

    /**
     * The attributes for this object.
     *
     * Name value pairs (name + default value).
     *
     * @var array
     */
    protected $attributes = [
        'name'        => '',
        'description' => '',
        'thumbnail'   => 0,
    ];

    /**
     * An array of attributes mapped with metadata.
     *
     * @var array
     */
    protected $maps = [
        'thumbnail' => '_term_meta_thumbnail',
    ];

    /**
     * Setup the object attributes.
     *
     * @return void
     */
    protected function setup() {
        $this['name'] = $this->instance->name;
        $this['description'] = $this->instance->description;
    }

    /**
     * Run perform insert object into database.
     *
     * @see wp_insert_term()
     *
     * @return int|void
     */
    protected function perform_insert() {
        $inserted = wp_insert_term( $this['name'], $this->object_type, [
            'description' => $this['description'],
        ]);

        if ( is_wp_error( $inserted ) ) {
            return;
        }

        return $inserted['term_id'];
    }

    /**
     * Run perform update object.
     *
     * @see wp_update_term()
     *
     * @param  array $dirty The attributes has been modified.
     * @return bool|void
     */
    protected function perform_update( array $dirty ) {
        if ( ! $this->is_dirty( 'name', 'description' ) ) {
            return true;
        }

        $updated = wp_update_term( $this->get_id(), $this->object_type, [
            'name'        => $this['name'],
            'description' => $this['description'],
        ]);

        return ! is_wp_error( $updated );
    }
}
```

And how to use:

Suppose we have a custom taxonomy `product_category` has been registered before (see [register taxonomy](https://codex.wordpress.org/Function_Reference/register_taxonomy))

```php
<?php

// And we have a `product_category` term with ID 10, created via admin dashboard.
// { name: "Keyboard", description: "Keyboard description" }

$product_category = new Product_Category( 10 );

// Get the $product_category data.
echo $product_category['name']; // Keyboard
echo $product_category['description']; // Keyboard description

// Modify properties
$product_category['name'] = 'New Keyboard';
$product_category->save();
echo $product_category['name']; // New Keyboard

// Then delete
$product_category->delete(); // Now `product_category` with term_ID 10 is gone.
```

Create new via `Product_Category`

```php

$new_cate = new Product_Category;

$new_cate['name'] = 'Mouse';
$new_cate['description'] = 'Some desc';
$new_cate['thumbnail'] = 101; // `thumbnail` will mapping to `_term_meta_thumbnail` metadata and automatic save.

$new_cate->save();

echo $new_cate['thumbnail']; // 101
echo get_term_meta( $new_cate->get_id(), '_term_meta_thumbnail', true ); // 101

```

## License

```
MIT License

Copyright (c) 2017 awethemes

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
