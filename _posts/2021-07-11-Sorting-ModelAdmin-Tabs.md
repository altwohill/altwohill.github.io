---
title: "Sorting Silverstripe ModelAdmin tabs"
categories: [Web]
---

Another quick hint to future me. Suppose you've got a `ModelAdmin` set up to manage a `CustomModel` class
that looks a little like this

```php
<?php
class CustomModel extends DataObject
{
    private static $db = ["Name" => "Varchar(250)"];
    private static $has_many = [
        "Things" => Thing::class,
        "OtherThings" => OtherThing::class,
        "ZThings" => ZThing::class,
    ];
}
```

By default, `ModelAdmin` will put each of the `$has_many` grid fields in its own tab, in some determined order (I think it might be the order the relations are declared?).

### Changing the Order

Often I want to group tabs, or move the most important tabs further left. It's easy when you know how.

```php
<?php
class CustomModel extends DataObject
{
   // snip (as above)

   public function getCMSFields()
   {
       $fields = parent::getCMSFields();

       if ($zThingsTab = $fields->fieldByName("Root.ZThings")) {
           $fields->removeFieldFromTab("Root", "ZThings");
           $fields->insertAfter("Main", $zThingsTab);
       }

       return $fields;
   }
}
```

We grab a reference to the tab (and its fields inside it), then remove it from the Root tabset.
We then insert it after the tab we want it to come after. 

Happy ordering!