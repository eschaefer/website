---
categories: blog
title: 'Prettify JSON data in the Rails console with pretty_generate'
tags: [Ruby, Rails, JSON]

comments: true
---

Using the Rails console has been a big time-saver for me. I'm trying to minimize switching between my browser window and my development windows (terminals, Sublime Text), and a lot of the debugging that I once did through the browser can easily be moved into IRB.

But the first problem I encountered was JSON formatting in IRB. I've been working with JSON and re-mapping some hashes quite a bit. Not pretty in the console.

#### Example 1: Jumbled JSON

    1.9.3-p362 :035 > puts some_json

```ruby
{"count"=>1, "results"=>[{"listing_id"=>96441649, "state"=>"active", "user_id"=>11923733, "category_id"=>69150359, "title"=>"Hipster Owl with Glasses - Yellow - Space Invaders - Retro", "description"=>"This Hipster Owl is just too cute!!!\r\n\r\nHe measures approx. 10&quot; tall and 9.5&quot; wide.\r\n\r\nHis eyes, nose and glasses are made from felt. \r\n\r\nHis body is white with tiny black dots. On his tummy, I used a Charlie Brown yellow and black fabric and his wings are a space invader print.\r\n\r\n\r\n\* Not machine washable\r\n\r\n**\*** This will be a duplicate of the item shown above.", "creation_tsz"=>1359839867, "ending_tsz"=>1370145600, "original_creation_tsz"=>1333072966, "last_modified_tsz"=>1359840873, "price"=>"30.00", "currency_code"=>"USD", "quantity"=>1, "tags"=>["hipster", "plushie", "nursery", "baby", "birthday", "gift", "pillow", "retro", "old school", "dot", "charlie brown", "space invaders", "geek"], "category_path"=>["Geekery"], "category_path_ids"=>[69150359], "materials"=>["cotton", "fiberfil", "felt", "buttons"], "shop_section_id"=>11341955, "featured_rank"=>5, "state_tsz"=>1359839867, "url"=>"http://www.etsy.com/listing/96441649/hipster-owl-with-glasses-yellow-space?utm_source=tipsy&utm_medium=api&utm_campaign=api", "views"=>768, "num_favorers"=>121, "shipping_profile_id"=>169274918, "processing_min"=>1, "processing_max"=>3, "who_made"=>"i_did", "is_supply"=>"false", "when_made"=>"made_to_order", "recipient"=>nil, "occasion"=>nil, "style"=>nil, "non_taxable"=>false}], "params"=>{"listing_id"=>"96441649"}, "type"=>"Listing", "pagination"=>{}}
=> nil
```

Ugly, right? The nesting structure is impenetrable at a quick glance. But the standard JSON module includes a little-known method called pretty_generate for exactly this situation. Check it out:

#### Example 2: Pretty JSON

    1.9.3-p362 :034 > puts JSON.pretty_generate some_json

```ruby
{
"count": 1,
"results": [
{
"listing_id": 96441649,
"state": "active",
"user_id": 11923733,
"category_id": 69150359,
"title": "Hipster Owl with Glasses - Yellow - Space Invaders - Retro",
"description": "This Hipster Owl is just too cute!!!\r\n\r\nHe measures approx. 10&quot; tall and 9.5&quot; wide.\r\n\r\nHis eyes, nose and glasses are made from felt. \r\n\r\nHis body is white with tiny black dots. On his tummy, I used a Charlie Brown yellow and black fabric and his wings are a space invader print.\r\n\r\n\r\n\* Not machine washable\r\n\r\n**\*** This will be a duplicate of the item shown above.",
"creation_tsz": 1359839867,
"ending_tsz": 1370145600,
"original_creation_tsz": 1333072966,
"last_modified_tsz": 1359840873,
"price": "30.00",
"currency_code": "USD",
"quantity": 1,
"tags": [
"hipster",
"plushie",
"nursery",
"baby",
"birthday",
"gift",
"pillow",
"retro",
"old school",
"dot",
"charlie brown",
"space invaders",
"geek"
],
"category_path": [
"Geekery"
],
"category_path_ids": [
69150359
],
"materials": [
"cotton",
"fiberfil",
"felt",
"buttons"
],
"shop_section_id": 11341955,
"featured_rank": 5,
"state_tsz": 1359839867,
"url": "http://www.etsy.com/listing/96441649/hipster-owl-with-glasses-yellow-space?utm_source=tipsy&amp;utm_medium=api&amp;utm_campaign=api",
"views": 768,
"num_favorers": 121,
"shipping_profile_id": 169274918,
"processing_min": 1,
"processing_max": 3,
"who_made": "i_did",
"is_supply": "false",
"when_made": "made_to_order",
"recipient": null,
"occasion": null,
"style": null,
"non_taxable": false
}
],
"params": {
"listing_id": "96441649"
},
"type": "Listing",
"pagination": {
}
}
=> nil
```

Nicely formatted so you can remap elements, and get a nice clear picture of whatever JSON you're consuming (or serving up).

#### Another option: [Awesome Print](https://github.com/michaeldv/awesome_print)

This is an extra ruby gem that you'll need to install in your dev environment, but it adds color and all kinds of goodies. Keeps you cozy in that console.

```

```
