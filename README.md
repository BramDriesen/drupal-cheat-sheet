# Drupal Cheat Sheet

## Table of contents

* [Introduction](#introduction)
* [General references](#general-references)
  * [Links](#links)
* [API references](#api-references)
  * [Custom Node Access Permissions](#custom-node-access-permissions)
  * [Custom Form Validation](#custom-form-validation)

## Introduction

The goal of this cheat sheet is to help as a general pointer to push you in the correct direction for various implmenetation and the best way to do it. E.g. an OOP orienter solution will be preferred in this cheat sheet over an none OOP solution. Not everything in this guide will have code examples, some items will just contain a summary and/or a link a webpage or other reference site.

**Contributing**
If you think something is missing or incorrect, don't hesitate to create a issue in the queue so I can have a look. Even better would be if you created a pull request and helped actively to improve this cheat sheet :smile:.

## General references

### Links

* [(MODULE) Examples For Developers][3]
* [(GUIDE)  Installing PHPCS and configuring Drupal coding standards][4]

## API references

### Custom Node Access Permissions

#### Access Grants (reccomended implementation)

Grants are fairely easy to understand, but can be quite a bit more tricky to get working. Depending on your use case, you'll need to implement 2 to 3 hooks to get this working. For a concrete example and the full documentation on the hooks visit the `node.api.php` file in your Drupal installation.

First step is to implement `hook_node_access_records()` who will define the permissions needed to access a node.

    /**
     * Implements hook_node_access_records().
     */
    function mymodule_node_access_records(NodeInterface $node) {
      if ($node->getType() === 'foo') {
        $grants[] = [
          'realm' => 'foo_realm',
          'gid' => 1,
          'grant_view' => 1,
          'grant_update' => 0,
          'grant_delete' => 0,
        ];
        return $grants;
      }
    }
    
 Second step is to implement `hook_node_grants()` who will give grants to a user.
 
     /**
     * Implements hook_node_grants().
     */
    function mymodule_node_grants(AccountInterface $account, $op) {
      $grants = [];
      if ($op === 'view' && strstr($account->getEmail(), '@foobar.com') {
        // The '1' matches the GID of the grant.
        $grants['foo_realm'] = [1];
      }
      return $grants;
    }

Third step is to implement `hook_node_access_records_alter()` if you're in need of altering the existing grants that other modules might set.

    /**
     * Implements hook_node_access_records_alter().
     */
    function mymodule_node_access_records_alter(&$grants, Drupal\node\NodeInterface $node) {
      if ($node->getType() === 'foo') {
        $grants = ['foo_realm' => $grants[1]];
      }
    }
    
 TIPS:
 * Don't forget to rebuild the Node Access Permissions (/admin/reports/status/rebuild)
 * Inspect the database table `node_access` to see which grants are being set for all the nodes.
 * Have a read through the hooks documentation in the `node.api.php` file!
 
 More examples:
 * [Droptica: Drupal node grants][1]
 * [Aten: Custom Permissions with Node Access Grants in Drupal 8 and Drupal 7][2]

#### `hook_node_access()` --> Not recommended

This might seem like the ideal solution, but note that this is very memory intensive if you're loading large lists of nodes in for example a view or something. For this reason every developer shoud limit the usage of `hook_node_access()` on their projects.

    /**
     * Implements hook_node_access().
     */
    function mymodule_node_access(NodeInterface $node, $op, AccountInterface $account) {
      $type = $node->getType();
      if ($type === 'foo' && $op === 'view') {
        // Only allow users with a specific email.
        if(strstr($account->getEmail(), '@foobar.com')) {
            return AccessResult::allowed();
        }
        return  AccessResult::forbidden();
      }
      return AccessResult::neutral();
    }

### Custom Form Validation

// todo: Add constraint + example.

## Update hooks

### Execute specific update hook (again)

This is possible by triggering the update hook using drush.

```bash
drush php-eval "module_load_install('MYMODULE'); MYMODULE_update_NUMBER();"
````

[1]: https://www.droptica.com/blog/drupal-node-grants/
[2]: https://atendesigngroup.com/articles/custom-permissions-node-access-grants-drupal-8-and-drupal-7
[3]: https://www.drupal.org/project/examples
[4]: https://www.drupal.org/node/1419988
