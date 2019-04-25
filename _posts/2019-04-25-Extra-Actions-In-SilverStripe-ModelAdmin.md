---
title: "Extra Actions in SilverStripe ModelAdmin"
categories: [Web]
---

A common need I have is to have additional functions available on records in SilverStripe's
[ModelAdmin](https://docs.silverstripe.org/en/4/developer_guides/customising_the_admin_interface/modeladmin/) 
interface. 

![Extra admin action between Save and Delete]({{ "/assets/images/2019-04-25-admin-actions.jpg" | absolute_url }})

By the far the most common use-case I've had is to generate and download a PDF invoice,
but other uses come up all the time. The most recent one was to make a copy of a
dataobject that included some of its related items.
 
There is a module by [UncleCheese](https://twitter.com/_UncleCheese_) called [BetterButtons](https://github.com/unclecheese/silverstripe-gridfield-betterbuttons/tree/feature/ss4-upgrade)
which can do this for you, however the module is currently undergoing some serious changes
and parts of it are being merged into SilverStripe core. Hopefully this will make this post
irrelevant in the future, but hey, we need this stuff now right?!

## An Example ##

For the simplest use-case, you only need one additional class, and to add a function into
your ModelAdmin class.

Note that this method only adds the buttons to specific `GridFields`. It's possible to 
extend a `GridFieldItemRequest` class in order to add the buttons you want into _all_
fields, but this is not necessarily what you want.

### GridField Item Request ###

This class gets assigned to our `GridField` and allows us to provide extra actions.
In this example my object is versioned, so I'm extending `VersiondGridFieldItemRequest`. 

```php
<?php
namespace Twohill\Example\Admin;

use Twohill\Example\Model\Event;
use SilverStripe\Forms\FormAction;
use SilverStripe\ORM\ValidationException;
use SilverStripe\Versioned\VersionedGridFieldItemRequest;

class EventItemRequest extends VersionedGridFieldItemRequest
{
    private static $allowed_actions = [
        'edit',
        'view',
        'ItemEditForm',
        'doCopy', // <-- ensure you add any allowed actions
    ];

    /**
     * This function allows us to add the buttons to the GridField Item
     * @return mixed
     */
    protected function getFormActions()
    {
        $actions = parent::getFormActions();

        $actions->insertBefore(
            'action_doDelete',
            FormAction::create('doCopy', 'Copy Event')
                ->setUseButtonTag(true)
                ->addExtraClass('btn-secondary font-icon-page-multiple')
        );
        return $actions;
    }

    /**
     * Process to run when the action button is submitted
     *
     * @param array $data
     * @param \SilverStripe\Forms\Form $form
     * @return \SilverStripe\Control\HTTPResponse
     * @throws \SilverStripe\Control\HTTPResponse_Exception
     */
    public function doCopy($data, $form)
    {
        /** @var Event $record */
        $record = $this->getRecord();
        $copy = $record->duplicate(true, ['Pricing']);
        $copy->Title = "Copy of {$record->Title}";
        try {
            $copy->write();
            $this->setFormMessage($form, "Successfully copied {$record->Title}"); // Show a nice success message

            $controller = $this->getToplevelController();
            $controller->getRequest()->addHeader('X-Pjax', 'Content'); // Force a content refresh

            return $controller->redirect($this->getBackLink(), 302); //redirect back to admin section
        } catch (ValidationException $e) {
            return $this->httpError(501);
        }
    }
}
```

Pretty simple really! It's not much different from a normal `PageController`.

The tricky part is getting our shiny new class in use.

### Configuring the GridForm ###

If the buttons are being used on `DataObject`s that are managed directly by `ModelAdmin`
this is pretty straight-forward. You just need to override the `EditForm` like so:

```php
<?php

namespace Twohill\Example\Admin;

use Twohill\Example\Model\Event;
use SilverStripe\Admin\ModelAdmin;
use SilverStripe\Forms\GridField\GridField;
use SilverStripe\Forms\GridField\GridFieldDetailForm;

class EventsAdmin extends ModelAdmin
{

    private static $managed_models = Event::class;

    private static $url_segment = 'events';

    private static $menu_title = 'Events';

    private static $menu_icon_class = 'font-icon-thumbnails';

    public function getEditForm($id = null, $fields = null)
    {
        $form = parent::getEditForm($id, $fields);

        /** @var GridField $gridField */
        $gridField = $form->Fields()->first();
        $config = $gridField->getConfig();
        /** @var GridFieldDetailForm $eventsForm */
        $eventsForm = $config->getComponentByType(GridFieldDetailForm::class);
        $eventsForm->setItemRequestClass(EventGridFieldItemRequest::class);

        return $form;
    }
}

```

If you need the button to show up on a nested/child object, it's a little 
trickier. Essentially you want to edit the `GridField` on the object that
"owns" the child.

For this example, each of my `Event` objects has multiple `Registrations`,
and I want to have a button to generate a PDF invoice for each `Registration`
in case the end-user looses it or something.
 
To do this, I update my `Event` class like so (I've snipped out bits for
brevity).

```php
<?php

namespace Twohill\Example\Model;

use SilverStripe\Forms\GridField\GridField;
use SilverStripe\Forms\GridField\GridFieldDetailForm;
use SilverStripe\ORM\DataObject;
use SilverStripe\ORM\FieldType\DBDate;
use SilverStripe\ORM\HasManyList;

/**
 * Class Event
 * @package Twohill\Example\Model
 *
 * @property string $Title
 *
 * @method HasManyList Registrations
 *
 */
class Event extends DataObject
{
    private static $db = [
        'Title' => 'Varchar(255)',
      //snip..
    ];

    private static $has_many = [
        'Registrations' => Registration::class,
    ];

    public function getCMSFields()
    {
        $fields = parent::getCMSFields();

        /** @var GridField $registrationField */
        $registrationField = $fields->dataFieldByName("Registrations");
        if ($registrationField) {
          
            /** @var GridFieldDetailForm $detailForm */
            $detailForm = $registrationField->getConfig()->getComponentByType(GridFieldDetailForm::class);
            $detailForm->setItemRequestClass(RegistrationItemRequest::class);

        }
        return $fields;
    }
//snip...

}

```

This time I have a `RegistrationItemRequest` but it's pretty similar to
the `EventItemRequest` above.

And that's all there is to it! Let me know what you think, and if you have
any other better ideas I'm all ears!
