## Drupal PHP cheatsheet

- [Custom Module Development](#custom-module-development)
  - [URL](#url)
  - [Images](#images)
  - [JavaScript](#javascript)
  - [DateTime](#datetime)
  - [User](#user)
  - [Node](#node)
  - [Form](#form)
  - [Taxonomy](#taxonomy)
  - [Block](#block)
  - [Views](#views)
  - [Database](#database)
    - [DATABASE OPERATIONS](#database-operations)
    - [Select](#select)
    - [Update](#update)
    - [Delete](#delete)
    - [Insert](#insert)
  - [Services](#services)
  - [Theme](#theme)
  - [Drush](#drush)
  - [Links](#links)
  - [General](#general)
  - [DRUPAL CONSOLE](#drupal-console)

# Custom Module Development

- Module file structure

```
|-- modules
  | -- custom
    |-- hello_world
      |-- hello_world.info.yml
      |-- hello_world.routing.yml
      |-- src
        |-- Controller
```

- .info.yml:
  > Contains all information about module [name, description, core, version, type]

```yml
name: Hello World Module
description: Creates a page showing "Hello World".
package: Custom

type: module
core: 8.x
core_version_requirement: ^8 || ^9
```

- .routing.yml
  > Containes custom routing refear to controller callback function

```yml
hello_world.my_page:
  path: "/mypage/page"
  defaults:
    _controller: '\Drupal\hello_world\Controller\HelloWorldController::myPage'
    _title: "My first page in Drupal"
  requirements:
    _permission: "access content"
```

<strong>hello_world.my_page</strong><br>

<p>
  This is the machine name of the route. By convention, route machine names should be module_name.sub_name. When other parts of the code need to refer to the route, they will use the machine name.
</p>
<strong>path</strong><br>
<p>This gives the path to the page on your site. Note the leading slash (/).</p>
<strong>defaults</strong><br>
<p>This describes the page and title callbacks. @todo: Where can these defaults be overridden?</p>
<strong>requirements</strong><br>
<p>This specifies the conditions under which the page will be displayed. You can specify permissions, modules that must be enabled, and other conditions.</p>

---

## URL

### URL generation###

@see <https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Url.php/class/Url/8.2.x>.

> When adding a URL into a string, for instance in a text description, core recommends we use the `\Drupal\Core\Url` class.
> This has a few handy methods:

- Internal URL based on a route - ### `Url::fromRoute()`### , Example: `Url::fromRoute('user.register')->toString()`
- Internal URL based on a path - ### `Url::fromInternalUri()`### , Example: `Url::fromInternalUri('node/add')`
- External URL - ### `Url::fromUri`### , Example: `Url::fromUri('https://www.acquia.com')`
  > When you need to constrict and display the link as text you can use the `toString()` method: `Url::fromRoute('user.register')->toString()`.

Short snippet that shows how to get url parameters in drupal 8.
to get query parameter form the url, you can us the following.
If you have the url for example `/page?uid=123&num=452`.
To get all params, use:.
`$param = \Drupal::request()->query->all();`.
To get "uid" from the url, use:.
`$uid = \Drupal::request()->query->get('uid');`.

To get "num" from the url, use:

`$num = \Drupal::request()->query->get('num');`

## Images

### Render images with image styles

@source https://www.webomelette.com/how-render-your-images-image-styles-drupal-8

#### URL only

```php
$style = \Drupal::entityTypeManager()->getStorage('image_style')->load('thumbnail');
$url = $style->buildUrl('public://my-image.png');
```

#### Render image

```php
$render = [
  '#theme' => 'image_style',
  '#style_name' => 'thumbnail',
  '#uri' => 'public://my-image.png',
  // optional parameters
];
```

#### Example

@todo to be refined.

```php
if ($paragraph && $paragraph->hasField('field_image') && !$paragraph->get('field_image')->isEmpty()) {
  $entity_img_id = $paragraph->get('field_image')->first()->getValue()['target_id'];
  $image = \Drupal::entityTypeManager()->getStorage('file')->load($entity_img_id);
  $style = \Drupal::entityTypeManager()->getStorage('image_style')->load('thumbnail');
  $uri = $style->buildUrl($image->getFileUri());

  return [
    '#theme' => 'image',
    '#style_name' => 'thumbnail',
    '#uri' => $uri,
  ];
}
```

#### Responsive image

```php
$file = $paragraph->get('field_image')->entity;

if ($file) {
  $image = \Drupal::service('image.factory')->get($file->getFileUri());

  $logo_build = [
    '#theme' => 'responsive_image',
    '#responsive_image_style_id' => 'promoted_teaser',
    '#uri' => $file->getFileUri(),
  ];

  // Add the file entity to the cache dependencies.
  // This will clear our cache when this entity updates.
  $renderer = \Drupal::service('renderer');
  $renderer->addCacheableDependency($logo_build, $file);

  $build['#image_p'.$paragraph_counter] = $logo_build;
}
```

### Load media entity & get a field

```
use Drupal\media\Entity\Media;

$media = Media::load($nid);
$file_uri = $media->field_media_file->entity->getFileUri();
```

### Show Image Field In Twig

- Get image as tag `<img>`

```twig
{{fields.field_my_image_field.content|render|striptags('<img>')|trim|raw}}
```

- Get image as url

```twig
{{fields.field_my_image_field.content|render|striptags|trim|raw}}
```

### show you how to loop images in Twig and dynamically adding image styles to them in Drupal 8.

```twig
{% for item in content.field_images['#items'] %}
  {% set image = {
    '#theme':      'image_style',
    '#style_name': 'medium',
    '#uri':        item.entity.uri.value,
    '#alt':        item.alt,
    '#width':      item.width,
    '#height':     item.height
  } %}
  {{ image }}
{% endfor %}
```

## JavaScript

### Drupal 8 Js, how to create a global Drupal Js function

```javascript
/**
 * Attaches the single_datetime behavior.
 *
 * @type {Drupal~behavior}
 */
(
  function ($, Drupal, drupalSettings) {
    "use strict";
    Drupal.behaviors.leafletCurrentLocation = {
      attach: function (context, drupalSettings) {
        // Code....
      },
    };
  }
)(jQuery, Drupal, drupalSettings);
```

### JavaScript behaviors

```javascript
(function ($, Drupal, drupalSettings, once) {
  Drupal.behaviors.mainBehavior = {
    attach: function (context, drupalSettings) {
      once("mainBehavior", "html", context).forEach(function () {
        console.log(drupalSettings);
        console.log(context);
      });
    },
  };
})(jQuery, Drupal, drupalSettings, once);
```

### How to Run a JavaScript Code at Browser Closing Time

```javascript
(function ($) {
  Drupal.behaviors.page_load_progress = {
    attach: function (context, settings) {
      window.onbeforeunload = close_event_function;
      function close_event_function() {
        //Enter your code before run window close;
        return null;
      }
    },
  };
})(jQuery);
```

## DateTime

### DateTime Object

```php
use Drupal\Core\Datetime\DrupalDateTime;

   //Convert Timestamp to Drupal DateTime
   $drupalDateTime = DrupalDateTime::createFromTimestamp($row->created);

   print $drupalDateTime->format('r');
  //Will return "Wed, 13 Oct 2021 17:40:39 +0300"

   print $drupalDateTime->__toString()
  //Will return 2021-10-13 17:40:39 Europe/Athens

  print $drupalDateTime->format('Y-m-d H:i:s')
  //Will return "2021-10-13 17:40:39"

  print $drupalDateTime->format('Y-m-d H:i:s P')
  //Will return "2021-10-13 17:40:39 +03:00"
```

### Set datetime-local html default value

```html
<div class="form-group">
  <label for="">موعد الاختبار</label>
  <?php $datetime = new DateTime($row['start_date']); ?>
  <input
    type="datetime-local"
    class="form-control"
    name="start_date"
    value="<?= $datetime->format('Y-m-d\TH:i:s'); ?>"
    required
  />
</div>
```

### How to Change Date Format from UTC Timezone to Any Required Timezone

In Drupal, usually the date field value is saved in the database in UTC timezone format.
One of our requirements for a project was to show the date in the site's timezone format.
So we generated a general function to convert date in UTC timezone to any required timezone and format it.
Just use the below function to convert date in UTC timzone to a given timezone and the format date using a valid timestamp.

```php
/**
 * Function to get date/time on user's timezone
 * @param  int $timestamp
 *  Timestamp to be converted to date
 * @param string timezone
 *  Timezone to which date is to be converted
 * @param  string $format
 *  Format to which date to convert
 * @return string
 *  Formatted date.
 */
function get_date_on_given_timezone($timestamp, $new_timezone, $format = 'd/m/Y H:i:s') {
  if (!empty($timestamp)) {
    // Database timezone.
    $db_timezone = 'UTC';
    $date_object = new DateObject($timestamp, new DateTimeZone($db_timezone));
    // Convert from the database time zone to site's time zone.
    $date_object->setTimezone(new DateTimeZone($new_timezone));
    $new_date = date_format_date($date_object, 'custom', $format);
    return $new_date;
  }
}
```

## User

### Drupal 8 Current User Object, useful methods and what they return

```php
get current user in drupal 8

$current_user = \Drupal::currentUser();

$uid = $current_user->id();
// It returns user id of current user.

$user_mail = $current_user->getEmail();
// It returns user email id.

$user_display_name = $current_user->getDisplayName();
// It returns user display name.

$user_account_name = $current_user->getAccountName()
// It returns user account name.

$user_roles = $current_user->getRoles();
// It returns array of current user has.

$current_user->isAnonymous();
// It returns true or false.

$current_user->isAuthenticated();
// It returns true or false.

$current_user->getLastAccessedTime();
// It returns timestamp of last logged in time for this user
```

### Check if current user is admin.

```php
$user = \Drupal::currentUser()->getRoles();
  if(!in_array("administrator", $user) && $form_id == 'user_form') {
   unset($form['account']);
  }
```

### Drupal 8 programmatically block and unblock a user

```php
/**  @var \Drupal\user\UserInterface $user */
$user = \Drupal\user\Entity\User::load(USER_ID_HERE);
$user->block();
// $user->activate();
$user->save();
```

### Code snippet that can be used to Log in user programmatically in Drupal 8

```php
use Drupal\user\Entity\User;
use Symfony\Component\HttpFoundation\RedirectResponse;


$user = User::create([
  'name' => $userEmail,
  'mail' => $userEmail,
  'pass' => $password,
  'status' => 1,
]);
user_login_finalize($user);
$user_destination = \Drupal::destination()->get();
$response = new RedirectResponse($user_destination);
$response->send();
```

[Drupal: How to inject a Service into a Class (Controller, Form, Plugin Block, etc) and why not into another service Class](https://pixelthis.gr/content/drupal-8-how-inject-service-class-controller-form-plugin-block-etc-and-why-not-another)

### Profile Module

```php
// Load profile by user id and profile type.
 $profile = \Drupal::entityTypeManager()
        ->getStorage('profile')
        ->loadByProperties([
          'uid' => $doctor_id,
          'type' => 'doctor_profile',
        ]);
 // Get filed data.
 $hcp_profile = reset($hcp_profiles);
 $address = $hcp_profile->get('field_user_address')->getValue();
 //Or
 $full_name = $profile->field_full_name->value;
```

### Reset user password using uid

```php
public function restPass()
  {
    $user = \Drupal\user\Entity\User::load(85);
    kint($user);
    die();
    $user->setPassword('admin');
    $user->save();
    return [
      '#type' => 'markup',
      '#markup' => 'Done'
    ];
  }
```

### Get user url

```php
$variables['user_url'] = Url::fromRoute('entity.user.canonical', ['user' => $account->id()])->setAbsolute()->toString();
```

## Node

### How to programmatically update an entity reference field
@see https://stefvanlooveren.me/blog/how-programmatically-update-entity-reference-field-drupal-8

### Render node body field.
```php
$body = \Drupal\Component\Utility\Unicode::truncate(preg_replace('/[^\w$\x{0080}-\x{FFFF}]+/u', ' ',strip_tags($node->get('body')->value) ), 200, true, true);
```
### Render a node

On drupal 8 every elements (almost) are an entity, as any entity you can render a node.

@see http://www.drupal8.ovh/en/tutoriels/339/render-a-node-or-an-entity

```php
$nid = 1;
$entity_type = 'node';
$view_mode = 'teaser';
$view_builder = \Drupal::entityTypeManager()->getViewBuilder($entity_type);
$storage = \Drupal::entityTypeManager()->getStorage($entity_type);
$node = $storage->load($nid);
$build = $view_builder->view($node, $view_mode);
$output = render($build);
```

```php
// Note : you can also use if you don't want to use different entity types.
$node = Node::load($nid);
```

### Get node field value from html scope

```php
function hook_preprocess_html(&$variables) {
  // If current page is a node full page.
  $node = \Drupal::request()->attributes->get('node');
  if ($node) {
    // Get entity reference field entities and loop.
    $node_paragraphs = $node->get('field_paragraphs')->referencedEntities();

    foreach ($node_paragraphs as $entity) {
      // Check entity type and for a not empty field.
      if ($entity->getType() === "card" && !($entity->get('field_media')->isEmpty())) {
        // Add body class.
        $variables['attributes']['class'][] = "has-intro-banner";
      }
    }
  }
}
```

### Get current node id

```php
$node = \Drupal::routeMatch()->getParameter('node');
if ($node instanceof \Drupal\node\NodeInterface) {
  $nid = $node->id();
}
```

### Get decrypted field values during entity insert/update hook

> The following explains the situation on a `foo` node when attempting to get the value of the text field  
> `field_bar` during a form-alter, a node pre-save, a node insert, and a node update:

```php
/**
 * Implements hook_form_BASE_FORM_ID_alter().
 *
 * Alter the edit Foo node form.
 */
function mymodule_form_node_foo_edit_form_alter(&$form, FormStateInterface $form_state, $form_id) {
  /**  @var \Drupal\node\Entity\Node $foo_node */
  $foo_node = $form_state->getFormObject()->getEntity();
  // Value 1 is correct.
  $bar_value_1 = $foo_node->field_bar->getString();
  // Value 2 is correct.
  $bar_value_2 = $foo_node->field_bar->getValue();
  // Value 3 is correct.
  $bar_value_3 = $foo_node->field_bar[0]->value;
  // Value 4 is correct.
  $bar_value_4 = $foo_node->get('field_bar')->getValue();
}

/**
 * Implements hook_ENTITY_TYPE_presave().
 */
function mymodule_node_presave(EntityInterface $entity) {
  // Value 1 is correct.
  $bar_value_1 = $entity->field_bar->getString();
  // Value 2 is correct.
  $bar_value_2 = $entity->field_bar->getValue();
  // Value 3 is correct.
  $bar_value_3 = $entity->field_bar[0]->value;
  // Value 4 is correct.
  $bar_value_4 = $entity->get('field_bar')->getValue();
}
/**
 * Implements hook_ENTITY_TYPE_insert().
 */
function mymodule_node_insert(EntityInterface $entity) {
  // Value 1 returns `[ENCRYPTED]`.
  $bar_value_1 = $entity->field_bar->getString();
  // Value 2 returns `[ENCRYPTED]`.
  $bar_value_2 = $entity->field_bar->getValue();
  // Value 3 returns `[ENCRYPTED]`.
  $bar_value_3 = $entity->field_bar[0]->value;
  // Value 4 returns `[ENCRYPTED]`.
  $bar_value_4 = $entity->get('field_bar')->getValue();

  /* @var $field_encrypt_process_entities \Drupal\field_encrypt\FieldEncryptProcessEntities */
  $field_encrypt_process_entities = \Drupal::service('field_encrypt.process_entities');
  $field_encrypt_process_entities->decryptEntity($entity);

  // Value is STILL `[ENCRYPTED]`.
  $bar_value_1 = $entity->field_bar->getString();
  // Value is STILL `[ENCRYPTED]`.
  $bar_value_2 = $entity->field_bar->getValue();
  // Value is STILL `[ENCRYPTED]`.
  $bar_value_3 = $entity->field_bar[0]->value;
  // Value is STILL `[ENCRYPTED]`.
  $bar_value_4 = $entity->get('field_bar')->getValue();
}

/**
 * Implements hook_ENTITY_TYPE_update().
 */
function mymodule_node_update(EntityInterface $entity) {
  // Value 1 returns `[ENCRYPTED]`.
  $bar_value_1 = $entity->field_bar->getString();
  // Value 2 returns `[ENCRYPTED]`.
  $bar_value_2 = $entity->field_bar->getValue();
  // Value 3 returns `[ENCRYPTED]`.
  $bar_value_3 = $entity->field_bar[0]->value;
  // Value 4 returns `[ENCRYPTED]`.
  $bar_value_4 = $entity->get('field_bar')->getValue();

  /* @var $field_encrypt_process_entities \Drupal\field_encrypt\FieldEncryptProcessEntities */
  $field_encrypt_process_entities = \Drupal::service('field_encrypt.process_entities');
  $field_encrypt_process_entities->decryptEntity($entity);

  // Value is STILL `[ENCRYPTED]`.
  $bar_value_1 = $entity->field_bar->getString();
  // Value is STILL `[ENCRYPTED]`.
  $bar_value_2 = $entity->field_bar->getValue();
  // Value is STILL `[ENCRYPTED]`.
  $bar_value_3 = $entity->field_bar[0]->value;
  // Value is STILL `[ENCRYPTED]`.
  $bar_value_4 = $entity->get('field_bar')->getValue();
}
```

### Paragraph Module

> load the paragraphs content.

```php
$node  = \Drupal\node\Entity\Node::load(1);
$paragraph = $node->field_paragraph->getValue();
// Loop through the result set.
foreach ( $paragraph as $element ) {
  $p = \Drupal\paragraphs\Entity\Paragraph::load( $element['target_id'] );
  $text = $p->field_name->getValue();
}
```

### Create node programmatically

```php
Try this Code in MigrateContentController:

<?php

namespace Drupal\migrate_contents\Controller;
use Drupal\taxonomy\Entity\Term;
use Drupal\Core\Controller\ControllerBase;
use Drupal\node\Entity\Node;

class MigrateContentController extends ControllerBase {

 function migrateContent() {
 $updated = 0;
 $data = array(
   'type' => 'ip_range',
   'title' => '192.168.7.100/24',
   'field_ip_range_gw' => '192.168.7.100',
   'field_ip_range_hidden' => '',
   'field_ip_range_blocked' => '192.168.7.200',
   'field_ip_range_access_type' => 'Blocked',
   'field_ip_threshold' => '20',
   'field_ip_range_sec_zone' => 'C2',
   'field_ip_range_vlan_name' => 'Network58',
   'field_ip_range_vlan_tag' => '4054',
   'field_ip_range_comment' => 'Ip ranges from 192.168.7.100 to 192.168.7.255',
 );
 $node = Node::create($data);
 $node->save();
 $updated++;

 return array(
   '#markup' => $updated,
 );
 }
}
```

### Rendering fields

@see https://www.metaltoad.com/blog/drupal-8-entity-api-cheat-sheet
@see Long answer https://www.computerminds.co.uk/articles/rendering-drupal-8-fields-right-way

```php
$node->field_my_thing->get(0)->view();

$node->field_my_thing->view();

$node->field_my_thing->view('teaser');

$node->field_my_thing->view([
  'type' => 'image',
  'label' => 'hidden',
  'settings' => array(
    'image_style' => 'larger_thumbnail',
    'image_link' => 'content',
  ),
]);
```

## Form

### Redirect after login

```php
use Drupal\Core\Form\FormStateInterface;
use Drupal\Core\Url;

/**
 * Implements hook_form_FORM_ID_alter().
 */
function MYCUSTOMMODULE_form_user_login_form_alter(&$form, FormStateInterface $form_state, $form_id) {
  $form['#submit'][] = 'MYCUSTOMMODULE_user_login_form_submit';
}

/**
 * Custom submit handler for the login form.
 */
function MYCUSTOMMODULE_user_login_form_submit($form, FormStateInterface $form_state) {
  $url = Url::fromRoute('/your/destination/path');
  $form_state->setRedirectUrl($url);
}
```

### Code snippet that can help you to get field values in hook_form_alter in Drupal 8.

Example of entity reference field:

```php
$form['field_name']['widget'][0]['target_id']["#value"]
```

PHP
Example of text field:

```php
$form['field_name']['widget']["#value"]
```

### How to use ConfirmFormBase to confirm the action of delete nodes in drupal 8.

> <https://codimth.com/blog/web/drupal/how-use-confirmformbase-confirm-action-delete-nodes-drupal-8>

### Save config form value

```php
/**
 * {@inheritdoc}
 */
public function submitForm(array &$form, FormStateInterface $form_state) {
  parent::submitForm($form, $form_state);
  $config = $this->config(self::CONFIG_SETTINGS);
  $skip = ["submit", "form_build_id", "form_token", "form_id", "op"];
  foreach ($form_state->getValues() as $key => $value) {
    if (!in_array($key, $skip)) {
      $config->set($key, $value);
    }
  }
  $config->save();
}
```

## Taxonomy

### Get entity reference field (taxonomy) value

```php
$termId = $relationshipEntities['profile']->get('field_job_title')->target_id;
$term = \Drupal::entityTypeManager()->getStorage('taxonomy_term')->load($termId);
$termValue = $term->get('field_public')->value;
```

### Get all taxonomy terms from node

```php
function hook_preprocess_node(&$variables) {

  //make sure you have Devel module installed, then you can use ksm() to explore variables
  //ksm($variables['node']);

  if ($variables['node']->hasField('field_permit_city')) {
    //this will give you the referenced terms on the node as objects
    $term_objects = $variables['node']->get('field_permit_city')->referencedEntities();
    $term_labels = [];
    foreach ($term_objects as $term_object) {
      print $term_object->id(); //the id of the term
      print $term_object->label(); //the term title
      $term_labels[] = $term_object->label(); //build an array of term labels
    }
    //send an array of term titles over to the node twig template
    $variables['my_custom_template_variable'] = $term_labels;
  }
}
```

### Load nodes by term id

```php
$termIds = [3,56,456];
 $nodes = \Drupal::entityTypeManager()->getStorage('node')->getQuery()
 ->latestRevision()
 ->condition('field_tags', $termIds)
 ->execute();
```

### Get taxonomy term from node using entity refrence

```php
$term_name = $node
->get('field_subspecialty_category')
->referencedEntities()[0]
->getTranslation($lang)
->label();
```

## Block

### Set block class for custom block bundles.

@see https://www.drupal.org/node/2724333

```php
/**
 * Implements hook_preprocess_block().
 */

function hops_preprocess_block(&$variables) {
  // Add class for a specific block id.
  $block_id = $variables['elements']['#id'];
  switch ($block_id) {
    case '':
      $variables['attributes']['class'][] = '';
      break;

    default:
      break;
  }

  if (isset($variables['elements']['content']['#block_content'])) {
    $variables['attributes']['class'][] = 'block--'. $variables['elements']['content']['#block_content']->bundle();
  }
}
```

### Ways to find the block ID

```bash
drush ev "print_r(array_keys(\Drupal::service('plugin.manager.block')->getDefinitions()));"
```

> If you know part of the ID, you can search for it by adding grep to the end. For example, I have a block that I know has “profile” in the ID, but I don’t know the full ID. I can find it my using this command:

```bash
drush ev "print_r(array_keys(\Drupal::service('plugin.manager.block')->getDefinitions()));" | grep profile
```

### Disable block caching in drupal 8

```php
class MYCUSTOMBLOCK extends BlockBase {
  /**
   * {@inheritdoc}
   */
    public function build() {
        return array(
            '#markup' => ""
        );
    }
    /**
     * {@inheritdoc}
     */
    public function getCacheMaxAge() {
        return 0;
    }
}
```

### How can I programmatically display a block?

@see https://drupal.stackexchange.com/questions/171686/how-can-i-programmatically-display-a-block

### Content Blocks

```php
$bid = ??? // Get the block id through config, SQL or some other means
$block = \Drupal\block_content\Entity\BlockContent::load($bid);
$render = \Drupal::entityTypeManager()->
  getViewBuilder('block_content')->view($block);
return $render;
```

### Plugin blocks

```php
$block_manager = \Drupal::service('plugin.manager.block');
// You can hard code configuration or you load from settings.
$config = [];
$plugin_block = $block_manager->createInstance('system_breadcrumb_block', $config);
// Some blocks might implement access check.
$access_result = $plugin_block->access(\Drupal::currentUser());
// Return empty render array if user doesn't have access.
// $access_result can be boolean or an AccessResult class
if (is_object($access_result) && $access_result->isForbidden() || is_bool($access_result) && !$access_result) {
  // You might need to add some cache tags/contexts.
  return [];
}
$render = $plugin_block->build();
// In some cases, you need to add the cache tags/context depending on
// the block implemention. As it's possible to add the cache tags and
// contexts in the render method and in ::getCacheTags and
// ::getCacheContexts methods.
return $render;
```

#### Config entities

```php
$block = \Drupal\block\Entity\Block::load('config.id');
$render = \Drupal::entityTypeManager()
  ->getViewBuilder('block')
  ->view($block);
return $render;
```

## Views

### Make custom filter
@see https://tech.axelerant.com/how-to-use-hookviewsdataalter-in-drupal-8

### How to load a view via ajax without breaking ajax functionality?
@see https://pixelthis.gr/content/drupal8-how-load-view-ajax-without-breaking-ajax-functionality

### Render a profile view

```php
  $uid = $variables['user']->id();
  $args = [$uid];
  $view = \Drupal\views\Views::getView('profiles');
  if (is_object($view)) {
    $view->setArguments($args);
    $view->setDisplay('user_view');
    $view->preExecute();
    $view->execute();

    // @see https://drupal.stackexchange.com/questions/219475/get-result-view-with-formatter-programmattically
    $profileFields = [];
    foreach ($view->result as $rid => $row) {
      foreach ($view->field as $fid => $field ) {
        $profileFields[$rid][$fid . '-value'] = $field->getValue($row);
        $profileFields[$rid][$fid . '-render'] = $field->render($row);
        $profileFields[$rid][$fid] = $field->advancedRender($row);
      }
    }
    $variables['profileFields'] = $profileFields;
  }
```

## Database

### Entity Query: Get all the node entities between a daterange
@see https://pixelthis.gr/content/entity-query-get-all-node-entities-between-daterange

### Drop custom table from database.

```php
// Using this hook in file mymodule.install
function hook_uninstall(){
    $table = 'your_table_name';
    $schema = Database::getConnection()->schema();
    $schema->dropTable($table);
}
```

### Load node by entityQuery

```php
$query = \Drupal::entityQuery('node')
    ->condition('status', 1)
    ->condition('changed', REQUEST_TIME, '<')
    ->condition('title', 'cat', 'CONTAINS')
    ->condition('field_tags.entity.name', 'cats');
$nids = $query->execute();
```

### DATABASE OPERATIONS

### Now we can checkout the select, update, and delete operations in Drupal 8. For a field selection use.

```php
$query = \Drupal::database()->select('table_name', 'alias')
  ->fields('alias', ['field1', field2])
  ->condition('field3', $condition);
$results = $query->execute();
while ($content = $results->fetchAssoc()) {
  // Operations using $content.
}
```

> For a single field selection we can use `$last_paper_id = $last_paper->fetchField();`

### For update query execution in Drupal 8, we can use.

```php
$table = 'table_name';
\Drupal::database()->update($table)
  ->fields(array('field1' => $value1, 'field2' => $value2))
  ->condition('field3', $condition1)
  ->condition('field4', $condition2)
  ->execute();
```

### For content deletion.

```php
$query = \Drupal::database()->delete('table_name');
  $query->condition('field1', $condition1);
  $query->condition('field2', $condition2);
  $query->execute();
```

> Now checkout other database statements in Drupal 8 from this [Link](https://api.drupal.org/api/drupal/core!lib!Drupal!Core!Database!Connection.php/class/Connection/8.2.x)

### Select

### For retrieving single value from Table###

```php
/**
 * select table 'node_field_data' from database
 * select field 'title' fropm table 'node_field_data'
 * get data for content type = ‘page’
 * Limit query and only fetch one entry
 */
$getSingleValue = \Drupal::database()->select('node_field_data', 'nfd');
$getSingleValue->addField('nfd', 'title');
$getSingleValue->condition('nfd.type', 'page');
$getSingleValue->range(0, 1);
$title = $getSingleValue->execute()->fetchField();
```

### For retrieving single row from Table###

```php
/**
 * select table 'node_field_data' from database
 * select fields ('nid', 'title', 'status' from table 'node_field_data'
 * get data for content type = ‘page’
 * Limit query and only fetch one entry
 */
$getValues = \Drupal::database()->select('node_field_data', 'nfd');
$getValues->fields('nfd', ['nid', 'title', 'status']);
$getValues->condition('nfd.type', 'page');
$getValues->range(0, 1);
$singleRow = $getValues->execute()->fetchAssoc();
```

### For retrieving single row from Table using Operator (‘IN’)###

```php
/**
 * select table 'node_field_data' from database
 * select fields ('nid', 'title', 'status' from table 'node_field_data'
 * get data for content type IN ‘page’
 * Limit query and only fetch one entry
 */
$getValues = \Drupal::database()->select('node_field_data', 'nfd');
$getValues->fields('nfd', ['nid', 'title', 'status']);
$getValues->condition('nfd.type', 'page' , 'IN');
$getValues->range(0, 1);
$singleRow = $getValues->execute()->fetchAssoc();
```

### With Multiple rows and Table###

```php
/**
 * select table 'node_field_data' from database
 * select fields ('nid', 'title', 'status' from table 'node_field_data'
 * INNER join on Table 'users_field_data' and select field 'name' from table
 * get data for content type IN ‘page’
 * fetch all data with 'nid' as key for result
 */
$content = \Drupal::database()->select('node_field_data', 'nfd');
$content->fields('nfd', ['nid', 'title', 'status']);
$content->addField('ufd', 'name');
$content->join('users_field_data', 'ufd', 'ufd.uid = nfd.uid');
$content->condition('nfd.type', 'page');
$contentData = $content->execute()->fetchAllAssoc('nid');
```

### Update

### For update single row###

```php
/**
 * Existing table 'node_clinical_trial' has entry
 * 'secondary progressive MS' on ID 5
 * Lets update to 'Multiple Sclerosis' using update
 */

$content = \Drupal::database()->update('node_clinical_trial', 'nct');
$content->fields('nct', ['disease' => 'Multiple Sclerosis']);
$content->condition('nct.entity_id', 5);
$content->execute();
```

### Delete

### For delete single row###

```php
/**
 * Existing table 'node_clinical_trial'
 * has entry 'diabetes' on ID 6
 * which is not MS type disease
 * Lets delete that entry using delete
 */
$content = \Drupal::database()->delete('node_clinical_trial', 'nct');
$content->condition('nct.entity_id', 6);
$content->execute();
```

### Insert

### For Adding data

```php
/**
 * Add 'disease' & 'physician' name in Master table
 * 'node_clinical_trial'
 * Use insert
 */

$content = \Drupal::database()->insert('node_clinical_trial');
$content->fields(['disease', 'physician']);
$content->values(['Psoriasis','Dr.Holmes']);
$content->execute();
```

[How to Add Join in Views Query Alter](https://www.zyxware.com/articles/5725/solved-how-to-add-join-in-views-query-alter)

### Debug Query

```php
// Debug.
dump($query->__toString());
dump($query->sqlQuery->__toString());
```

## Services

### Inject a Service in Drupal 8

> Is a good practice to inject a service whenever is possible.
> You can verify the service name by
> Looking at the `Drupal` Static Service Container wrapper class
> Reading the code on the `Drupal` Class you can find the `httpClient` method:

```php
/**
 * Returns the default http client.
 *
 * @return \GuzzleHttp\Client
 *   A guzzle http client instance.
 */
public static function httpClient() {
  return static::getContainer()->get('http_client');
}
```

### Inject the service

> Now that you know the service name `http_client`
> Inject into a Class (Controller, Form, Plugin Block, etc)

```php
 /**
   * Guzzle Http Client.
   *
   * @var GuzzleHttp\Client
   */
  protected $httpClient;

 /**
   * Constructs a new Class.
   *
   * @param \GuzzleHttp\Client $http_client
   *   The http_client.
   */
  public function __construct(
    Client $http_client
  ) {
    $this->httpClient = $http_client;
  }

  /**
   * {@inheritdoc}
   */
  public static function create(ContainerInterface $container) {
    return new static(
      $container->get('http_client')
    );
  }
```

### Inject a service into a service Class

```yml
// modules/custom/example/example.services.yml
services:
  example.default:
    class: Drupal\example\DefaultService
    arguments: ["@http_client"]
```

```php
// modules/custom/example/src/DefaultService.php
/**
 * GuzzleHttp\Client definition.
 *
 * @var GuzzleHttp\Client
 */
protected $http_client;
/**
 * Constructor.
 */
public function __construct(Client $http_client) {
  $this->http_client = $http_client;
}
```

## Theme

### Adding suggestions to views and preprocessing them
@see https://pixelthis.gr/content/drupal-9-adding-suggestions-views-and-preprocessing-them

### How to add html tags (span,div,strong etc) in your Menu Item Title
@see https://pixelthis.gr/content/drupal-8-how-add-html-tags-spandivstrong-etc-your-menu-item-title

### How to pass the base url to drupalSettings for global access
```php
/**
 * Implements hook_page_attachments_alter().
 *
 * @inheritdoc
 */
function your_theme_page_attachments_alter(&$page) {
  global $base_url;
  $page['#attached']['drupalSettings']['baseURL'] = $base_url;
}
```

### Access views-view-unformatted.html.twig fields.

```twig
{% if title %}
 <h3>{{ title }}</h3>
{% endif %}
{% for row in rows %}
 {%
    set row_classes = [
      default_row_class ? 'views-row',
    ]
  %}
 <div{{row.attributes.addClass(row_classes)}}>
  {# {{- row.content -}} #}
  {{row.content['#row']._entity.title.value}}
  {{row.content['#row']._entity.field_video_url.value}}
  {{file_url(row.content['#row']._entity.field_video_thumbnail.entity.uri.value)}}
 </div>
{% endfor %}
```

### Getting Drupal 8 Field Values in Twig

[Article](https://sarahcodes.medium.com/getting-drupal-8-field-values-in-twig-22b80cb609bd)

### Rewrite the output of a field

`$variables["items"][0]["content"] = "";`

### Provide frontpage variant

```php
try {
  $isFront = \Drupal::service('path.matcher')->isFrontPage();
}
catch (Exception $e) {
  $isFront = FALSE;
}
```

## Drush

@see https://www.drupal.org/node/1023440

### Unlock bloqued used

`drush uublk username` to unblock the user

### Taking advantage of Drupal Console debugging capabilities

```bash
drupal container:debug
```

If you do not want to see the full list of services you can use `| grep http`

```bash
drupal container:debug | grep http
```

But you may do not know the service name, then I higly recommend you to use peco interactive filtering tool

```bash
drupal container:debug | peco | awk -F ' ' '{print $1}' | xargs drupal container:debug
```

You can find peco at <https://github.com/peco/peco>

### Apply Patches with composer in drupal 9.

run this command to get the module:.
`composer require cweagans/composer-patches`.
Add this to your `composer.json` file:.

```bash
"extra": {
    "enable-patching": true,
    "patches": {
        ...
        ...
     }
}
```

### Generate hshad salat

```bash
drush php:eval "echo \Drupal\Component\Utility\Crypt::randomBytesBase64(55)"
```

## Links

@see https://agaric.coop/blog/creating-links-code-drupal-8

```php
use Drupal\Core\Link;
$link = Link::createFromRoute('This is a link', 'entity.node.canonical', ['node' => 1]);
```

More flexibility with URL object

```
use Drupal\Core\Link;
use Drupal\Core\Url;
$link = Link::fromTextAndUrl('This is a link', Url::fromRoute('entity.node.canonical', ['node' => 1]));
```

Internal links which have no routes

`$link = Link::fromTextAndUrl('This is a link', Url::fromUri('base:robots.txt'));`

External links:

```php
  use Drupal\Core\Url;
  use Drupal\Core\Link;
  $link = !empty($url) ? Link::fromTextAndUrl($this->t('Newsletter'), Url::fromUri($url, [
    'attributes' => [
      'target' => '_blank',
      'class' => ['button']
    ],
  ])) : '';
  $html_link = $link->toString();
```

Using the data provided by a user

`$link = Link::fromTextAndUrl('This is a link', Url::fromUserInput('/node/1');`

Linking entities.

`$link = Link::fromTextAndUrl('This is a link', Url::fromUri('entity:node/1'));`

Drupal usually expects a render array if you are going to print the link, so the Link object has a method for that

`$link->toRenderable();`

Add links inside a t() method.
Need to pass the link as a string.

```php
$link = Link::fromTextAndUrl('This is a link', Url::fromRoute('entity.node.canonical', ['node' => 1]));
$this->t('You can click this %link' ['%link' => $link->toString()]);
```

## General
[Useful Links](https://github.com/gkapoor121212/drupal9-links)

### Make field unique field
```php
use Drupal\Core\Entity\EntityTypeInterface;
// if you make a field belongs to a node just change $entity_type to 'node'
function hook_entity_bundle_field_info_alter(&$fields, EntityTypeInterface $entity_type, $bundle) {
  // D8 => $entity_type->id()
  // D9 => $entity->getEntityTypeId()
  if ($entity_type->id() === 'user') {
    if (isset($fields['field_user_id'])) {
      $fields['field_user_id']->addConstraint('UniqueField', ['message' => t('this id is used before with another user')]);
    }
  }
}
```

### PHP: Get current year seasons, per year not just for the current year
@see https://pixelthis.gr/content/php-get-current-year-seasons-year-not-just-current-year

> Drupal class => <https://api.drupal.org/api/drupal/core%21lib%21Drupal.php/class/Drupal/8>

## DRUPAL CONSOLE

### Installing.

> composer require drupal/console:~1.0 --prefer-dist --optimize-autoloader --sort-packages

### Middlware for redirect issues

```yml
// module.service.yml
services:
  http_middleware.your_module:
    class: Drupal\your_module\RedirectMiddleware
    tags:
      - { name: http_middleware}
```

```php
<?php

// Service class.
namespace Drupal\your_module;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\HttpKernelInterface;
use Symfony\Component\HttpFoundation\RedirectResponse;
use Drupal\Core\Routing\TrustedRedirectResponse;
use Drupal\Core\Url;

/**
 * Executes redirect before the main kernel takes over the request.
 */
class RedirectMiddleware implements HttpKernelInterface {

  /**
     * The wrapped HTTP kernel.
     *
     * @var \Symfony\Component\HttpKernel\HttpKernelInterface
     */
  protected $httpKernel;

  /**
     * The redirect URL.
     *
     * @var \Symfony\Component\HttpFoundation\RedirectResponse
     */
  protected $redirectResponse;

  /**
     * Constructs a RedirectMiddleware
     * object.
     *
     * @param \Symfony\Component\HttpKernel\HttpKernelInterface $http_kernel
     *   The decorated kernel.
     */
  public function __construct(HttpKernelInterface $http_kernel) {
        $this->httpKernel = $http_kernel;
      }

  /**
     * {@inheritdoc}
     */
  public function handle(Request $request, $type = self::MASTER_REQUEST, $catch = TRUE) {
        $response = $this->httpKernel->handle($request, $type, $catch);
        return $this->redirectResponse ?: $response;
  }

  /**
     * Stores the requested redirect response.
     *
     * @param \Symfony\Component\HttpFoundation\RedirectResponse|null $redirectResponse
     *   Redirect response.
     */
  public function setRedirectResponse(?RedirectResponse $redirectResponse) {
        $this->redirectResponse = $redirectResponse;
  }

}

// USING
$middleware = \Drupal::service('http_middleware.your_module');
$response = new RedirectResponse(Url::fromUserInput($url)->toString());
$middleware->setRedirectResponse($response);
```

### Render view programitcally

```php
 $view = \Drupal\views\Views::getView('geographical_statistics');
// Execute the view.
$view->execute();
$view_result = $view->result;
foreach ($view_result as $obj){
// Code...
}
```

### Show error messages

```php
$config['system.logging']['error_level'] = 'verbose';
```

### Show twitter timeline

```html
<a class="twitter-timeline" href="https://twitter.com/{field_twitterhandle}">
  Tweets by {field_name}</a
>
<script
  async
  src="https://platform.twitter.com/widgets.js"
  charset="utf-8"
></script>
```

### Install drupal console

```bash
$composer require drupal/console:~1.0 --prefer-dist --optimize-autoloader --sort-packages --no-update --dev
$composer update drupal/console
```

### Fast site install with lando & config management

```bash
$ lando drush site-install standard --account-name=admin --account-pass=admin --db-url='mysql://drupal8:drupal8@database/drupal8' --site-name=Test site
# Get uuid from source
$ drush config-get "system.site" uuid
# @see https://drupal.stackexchange.com/questions/150481/how-can-i-import-the-configuration-on-a-different-site/217126
# Set local uuid
$ drush config-set "system.site" uuid "fjfj34-e3bb-2ab8-4d21-9100-b5etgetgd99d5"
```

### Import drupal 8 configuration into a fresh install website

_@see https://www.dannyenglander.com/blog/drupal-8-development-how-import-existing-site-configuration-new-site_

```bash
$ drush cget system.site uuid
'system.site:uuid': bfb11978-d1a3-4eda-91fb-45decf134e25
$ drush cset system.site uuid bfb11978-d1a3-4eda-91fb-45decf134e25
$ drush ev '\Drupal::entityManager()->getStorage("shortcut_set")->load("default")->delete();'
```

### Testing

@todo description

```bash
~/Sites/siteName ᐅ vendor/bin/dcr web/themes/contrib/file
phpcs --standard=Drupal dir/file
phpcbf --standard=Drupal dir/file
~/Sites/siteName/web ᐅ eslint dir/file.js
```

### Setting up a local developent website standard workflow
@see https://pixelthis.gr/content/drupal-8-setting-local-developent-website-standard-workflow)

### Drupal with Docker
@see https://medium.com/drupal-stories/drupal-dev-environment-on-docker-3c795f2ac7aa
