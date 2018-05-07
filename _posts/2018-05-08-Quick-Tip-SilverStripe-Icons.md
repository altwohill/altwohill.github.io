---
title: "Quick Tip: SilverStripe CMS Icon Reference"
categories: [Web]
---

Putting in your own icon into `ModelAdmin`s in SilverStripe is a great way to allow clients to easily find the page you created.

The easiest way to do this is to use one of the icons already bundled with SilverStripe.

Open up `/vendor/silverstripe/admin/client/src/font/icons-reference.html` in your browser to see all that are installed by default.

This is a snapshot of the icons available in SilverStripe 4.1: ![Icons available in SilverStripe 4.1]({{ "/assets/images/snapshot-silverstripe-fonts-4.1.png" | absolute_url }})

You set the icon via `$menu_icon_class` as "font-icon-*iconname*" like this:

```php
<?php
namespace Twohill\MyModule\Admin;

use SilverStripe\Admin\ModelAdmin;
use Twohill\MyModule\Model\MyThing;

class MyAdmin extends ModelAdmin {
    private static $managed_models = MyThing::class;
    private static $url_segment = 'myadmin';
    private static $menu_title = 'My Admin';

    private static $menu_icon_class = 'font-icon-block-content';
    //...
}
```