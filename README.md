# Drupal-Tips
Drupal CMS development tips

# Custom Module Developmet
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
 * .info.yml: 
 > Containes all information about module [name, description, core, version, type]
 ```yml
name: Hello World Module
description: Creates a page showing "Hello World".
package: Custom

type: module
core: 8.x
core_version_requirement: ^8 || ^9
 ```
* .routing.yml
> Containes custom routing refear to controller callback function
```yml
hello_world.my_page:
  path: '/mypage/page'
  defaults:
    _controller: '\Drupal\hello_world\Controller\HelloWorldController::myPage'
    _title: 'My first page in Drupal'
  requirements:
    _permission: 'access content'
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

--------------
[**URL generation**](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Url.php/class/Url/8.2.x).
> When adding a URL into a string, for instance in a text description, core recommends we use the `\Drupal\Core\Url` class. 
> This has a few handy methods:
- Internal URL based on a route - **`Url::fromRoute()`**, Example: `Url::fromRoute('acquia_connector.settings')`
- Internal URL based on a path - **`Url::fromInternalUri()`**, Example: `Url::fromInternalUri('node/add')`
- External URL - **`Url::fromUri`**, Example: `Url::fromUri('https://www.acquia.com')`
> When you need to constrict and display the link as text you can use the `toString()` method: `Url::fromRoute('acquia_connector.settings')->toString()`.

**Show Image Field In Twig**
```php
{{ content.field_my_image_field|render|striptags('<img>')|trim }}
```
