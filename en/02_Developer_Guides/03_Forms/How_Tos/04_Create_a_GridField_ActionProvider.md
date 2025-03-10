---
title: Create a GridField action provider
summary: Handle custom actions on your GridField
---
# How to add a custom action to a GridField row

You can add an action to the row(s) of a [GridField](/developer_guides/forms/field_types/gridfield), such as the built in edit or delete actions.

In a [GridField](/developer_guides/forms/field_types/gridfield) instance each table row can have a
number of actions located the end of the row.
Each action is represented as a instance of a specific class
(e.g [GridFieldEditButton](api:SilverStripe\Forms\GridField\GridFieldEditButton)) which has been added to the `GridFieldConfig`
for that `GridField`

As a developer, you can create your own custom actions to be located alongside
the built in buttons.

For example let's create a custom action on the GridField to allow the user to
perform custom operations on a row:

1. Create a custom action
2. [Add your custom action to the current GridFieldConfig](#add-the-gridfieldcustomaction-to-the-current-gridfieldconfig)

_To create a custom action follow the [Basic GridFieldCustomAction boilerplate](#basic-gridfieldcustomaction-boilerplate) 
below, and if you would like to create a custom action in the **GridField action menu** follow the 
[Basic GridFieldCustomAction boilerplate implementing GridField_ActionMenuItem](#basic-gridfieldcustomaction-boilerplate-implementing-gridfield_actionmenuitem)_

## Basic GridFieldCustomAction boilerplate

A basic outline of our new `GridFieldCustomAction.php` will look like something
below:


```php
use SilverStripe\Forms\GridField\AbstractGridFieldComponent;
use SilverStripe\Forms\GridField\GridField_ColumnProvider;
use SilverStripe\Forms\GridField\GridField_ActionProvider;
use SilverStripe\Forms\GridField\GridField_FormAction;
use SilverStripe\Forms\GridField\GridField;
use SilverStripe\Control\Controller;

class GridFieldCustomAction extends AbstractGridFieldComponent implements GridField_ColumnProvider, GridField_ActionProvider 
{

    public function augmentColumns($gridField, &$columns) 
    {
        if (!in_array('Actions', $columns)) {
            $columns[] = 'Actions';
        }
    }

    public function getColumnAttributes($gridField, $record, $columnName) 
    {
        return ['class' => 'grid-field__col-compact'];
    }

    public function getColumnMetadata($gridField, $columnName) 
    {
        if ($columnName === 'Actions') {
            return ['title' => ''];
        }
    }

    public function getColumnsHandled($gridField) 
    {
        return ['Actions'];
    }

    public function getColumnContent($gridField, $record, $columnName) 
    {
        if (!$record->canEdit()) {
            return;
        }

        $field = GridField_FormAction::create(
            $gridField,
            'CustomAction'.$record->ID,
            'Do Action',
            "docustomaction",
            ['RecordID' => $record->ID]
        );

        return $field->Field();
    }

    public function getActions($gridField) 
    {
        return ['docustomaction'];
    }

    public function handleAction(GridField $gridField, $actionName, $arguments, $data) 
    {
        if ($actionName !== 'docustomaction') {
            return;
        }
        // perform your action here

        // output a success message to the user
        Controller::curr()->getResponse()
            ->setStatusCode(200)
            ->addHeader('X-Status', 'Do Custom Action Done.');
    }
}
```

## Add the GridFieldCustomAction to the current `GridFieldConfig`

While we're working on the code, to add this new action to the `GridField`, add
a new instance of the class to the [GridFieldConfig](api:SilverStripe\Forms\GridField\GridFieldConfig) object. The `GridField`
[Reference](/developer_guides/forms/field_types/gridfield) documentation has more information about
manipulating the `GridFieldConfig` instance if required.


```php
// option 1: creating a new GridField with the CustomAction
$config = GridFieldConfig::create();
$config->addComponent(GridFieldCustomAction::create());

$gridField = GridField::create('Teams', 'Teams', $this->Teams(), $config);

// option 2: adding the CustomAction to an existing GridField
$gridField->getConfig()->addComponent(GridFieldCustomAction::create());
```

For documentation on adding a Component to a `GridField` created by `ModelAdmin`
please view the [GridField Customization](/developer_guides/forms/how_tos/create_a_gridfield_actionprovider) section.

Now let's go back and dive through the `GridFieldCustomAction` class in more
detail.

First thing to note is that our new class implements two interfaces,
[GridField_ColumnProvider](api:SilverStripe\Forms\GridField\GridField_ColumnProvider) and [GridField_ActionProvider](api:SilverStripe\Forms\GridField\GridField_ActionProvider).

Each interface allows our class to define particular behaviors and is a core
concept of the modular `GridFieldConfig` system.

The `GridField_ColumnProvider` implementation tells Silverstripe CMS that this class
will provide the `GridField` with an additional column of information. By
implementing this interface we're required to define several methods to explain
where we want the column to exist and how we need it to be formatted. This is
done via the following methods:

 * `augmentColumns`
 * `getColumnAttributes`
 * `getColumnMetadata`
 * `getColumnsHandled`
 * `getColumnContent`

In this example, we simply add the new column to the existing `Actions` column
located at the end of the table. Our `getColumnContent` implementation produces
a custom button for the user to click on the page.

The second interface we add is `GridField_ActionProvider`. This interface is
used as we're providing a custom action for the user to take (`docustomaction`).
This action is triggered when a user clicks on the button defined in
`getColumnContent`. As with the `GridField_ColumnProvider` interface, by adding
this interface we have to define two methods to describe the behavior of the
action:

 * `getActions` returns an array of all the custom actions we want this class to
 handle (i.e `docustomaction`) .
 * `handleAction` method which will contain the logic for performing the
 specific action (e.g publishing the row to a thirdparty service).

Inside `handleAction` we have access to the current GridField and GridField row
through the `$arguments`. If your column provides more than one action (e.g two
links) both actions will be handled through the one `handleAction` method. The
called method is available as a parameter.

To finish off our basic example, the `handleAction` method simply returns a
message to the user interface indicating a successful message.

## Add the GridFieldCustomAction to the `GridField_ActionMenu`

For an action to be included in the action menu dropdown, which appears on each row if `GridField_ActionMenu` is included in the `GridFieldConfig`, it must implement `GridField_ActionMenuItem` and relevant `get` functions to provide information to the frontend react action menu component.

## Basic GridFieldCustomAction boilerplate implementing GridField_ActionMenuItem

```php
use SilverStripe\Forms\GridField\AbstractGridFieldComponent;
use SilverStripe\Forms\GridField\GridField_ColumnProvider;
use SilverStripe\Forms\GridField\GridField_ActionProvider;
use SilverStripe\Forms\GridField\GridField_ActionMenuItem;
use SilverStripe\Forms\GridField\GridField_FormAction;
use SilverStripe\Control\Controller;

class GridFieldCustomAction extends AbstractGridFieldComponent implements GridField_ColumnProvider, GridField_ActionProvider, GridField_ActionMenuItem
{

    public function getTitle($gridField, $record, $columnName)
    {
        return 'Custom action';
    }
    
    public function getCustomAction($gridField, $record)
    {
        if (!$record->canEdit()) {
            return;
        }
        
        return GridField_FormAction::create(
            $gridField,
            'CustomAction'.$record->ID,
            'Custom action',
            "docustomaction",
            ['RecordID' => $record->ID]
         )->addExtraClass(
            'action-menu--handled'
        );
    }
    
    public function getExtraData($gridField, $record, $columnName)
    {
        $field = $this->getCustomAction($gridField, $record);
        
        if (!$field) {
            return;
        }

        return $field->getAttributes();
    }
    
    public function getGroup($gridField, $record, $columnName)
    {
        return GridField_ActionMenuItem::DEFAULT_GROUP;
    }

    public function augmentColumns($gridField, &$columns) 
    {
        if (!in_array('Actions', $columns)) {
            $columns[] = 'Actions';
        }
    }

    public function getColumnAttributes($gridField, $record, $columnName) 
    {
        return ['class' => 'grid-field__col-compact'];
    }

    public function getColumnMetadata($gridField, $columnName) 
    {
        if ($columnName === 'Actions') {
            return ['title' => ''];
        }
    }

    public function getColumnsHandled($gridField) 
    {
        return ['Actions'];
    }

    public function getColumnContent($gridField, $record, $columnName) 
    {
        $field = $this->getCustomAction($gridField, $record);
                
        if (!$field) {
            return;
        }

        return $field->Field();
    }

    public function getActions($gridField) 
    {
        return ['docustomaction'];
    }

    public function handleAction(GridField $gridField, $actionName, $arguments, $data) 
    {
        if ($actionName !== 'docustomaction') {
            return;
        }
        // perform your action here

        // output a success message to the user
        Controller::curr()->getResponse()
            ->setStatusCode(200)
            ->addHeader('X-Status', 'Do Custom Action Done.');
    }
}
```

## Related

 * [GridField Reference](/developer_guides/forms/field_types/gridfield)
 * [ModelAdmin: A UI driven by GridField](/developer_guides/customising_the_admin_interface/modeladmin)
