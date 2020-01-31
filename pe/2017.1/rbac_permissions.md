---
layout: default
title: "About RBAC permissions"
canonical: "/pe/latest/rbac_permissions.html"
---

RBAC provides a specific set of roles you can assign to users and groups. Those roles contain permissions, which define what a user or group can or can't do within PE.

When you add new users to the console, they can't actually do anything until they're associated with a role, either explicitly through role assignment or implicitly through group membership and role inheritance. When a user is added to a role, they receive all the permissions of that role.

Permissions are additive. If a user is associated with multiple roles, that user is able to perform all of the actions described by all of the permissions on all of the applied roles.

## Structure of permissions

Permissions are structured as a triple of *type*, *permission*, and *object* with the following characteristics:

**Types** are everything that can be acted on, such as node groups, users, or user roles.
**Permissions** are what you can do with each type, such as create, edit, or view.
**Objects** are specific instances of the type.

Some permissions added to the Administrator user role might look like this:

| Type        | Permission    | Object     | Description |
|---          |---            |---           |---          |
|Node groups  | View          | PE Master    | Gives permission to view the PE Master node group.|
|User roles   | Edit          | All          | Gives permission to edit all user roles. |

When no object is specified, a permission applies to all objects of that type. In those cases, the object is “All”. This is denoted by a “*” in the API.

In both the console and the API, a “*” is used to express a permission for which an object doesn’t make sense, such as when creating users.

## Available permissions

The table below lists the available object types and their permissions, as well as an explanation of the permission and how to use it. Note that types and permissions have both a display name, which is the name you see in the console interface, and a system name, which is the name you use to construct permissions in the API. In the following table, the display names are used.

| Type              | Permission                              | Definition |
| ---               | ---                                     | ---        |
| Node groups        | Edit child group rules                  |The ability to edit the rules of descendents of a node group. This does not grant the ability to edit the rules of the group in the `object` field, only children of that group. This permission is inherited by all descendents of the node group.|
| Node groups        | Set environment                         |The ability to set the environment of a node group. This permission is inherited by all descendents of the node group.|
| Node groups        | Edit classes, parameters, and variables |The ability to edit every attribute of a node group except its environment and rule. This permission is inherited by all descendents of the node group.|
| Node groups        | Edit parameters and variables |The ability to edit the class parameters and variables of a node group's classes but not change the group's classes, environment, or rules. This permission is inherited by all descendents of the node group.|
| Node groups        | Create, edit, and delete child groups   |The ability to create new child groups, delete existing child groups, and modify every attribute of child groups, including environment and rules. This permission is inherited by all descendents of the node group.|
| Node groups        | View                                    |The ability to see all attributes of a node group, most notably the values of class parameters and variables. This permission is inherited by all descendents of the node group.|
| Users              | Create                                  | The ability to create new local users. Remote users are "created" by that user authenticating for the first time with RBAC. Object must always be `"*"`. |
| Users              | Reset password                          | The ability to grant password reset tokens to users who have forgotten their passwords. This process also reinstates a user after the use has been revoked. This may be granted per user. |
| Users              | Revoke                                  | The ability to revoke/disable a user. This means the user is no longer able to authenticate and use the console, node classifier, or RBAC. This permission also includes the ability to revoke the user's authentication tokens. This may be granted per user. |
| Users              | Edit                                    | The ability to edit a local user's data, such as name or email, and delete a local or remote user from PE. This may be granted per user. |
| User groups        | Import                                  | The ability to import groups from the directory service for use in RBAC. Object must always be `"*"`. |
| User roles         | Create                                  | The ability to create new roles. Object must always be `"*"`. |
| User roles         | Edit                                    | The ability to edit a role. Object must always be `"*"`. |
| User roles         | Edit members                            | The ability to change which users and groups a role is assigned to. This may be granted per role. |
| Directory service | View, edit, and test                     | The ability to view, edit, and test directory service settings. Object must always be `'*"`. |
| Certificate request      | Accept and reject                                   | Ability to accept and reject certificate signing requests. Object must always be `"*"`. |
| Puppet agent      | Run Puppet on agent nodes                                     | The ability to trigger a Puppet run from the console or orchestrator. Object must always be `"*"`.
| Console      | View                                    | The ability to view the PE console. Object must always be `"*"`. |
| Puppet environment | Deploy code                            | The ability to deploy code to a specific PE environment.
| Nodes              | View node data from PuppetDB           | The ability to view node data imported from PuppetDB. Object must always be `"*"`. |
| Nodes              | Edit node data from PuppetDB           | The ability to edit node data imported from PuppetDB. Object must always be `"*"`. |


### Display names and corresponding system names

The following table provides both the display and system names for the types and all their corresponding permissions. For more information, see the [permissions API endpoints](./rbac_permissionsref_v1.html).

| Type (display name)| Type (system name) | Permission (display name)               | Permission (system name) |
| ---                | ---                | ---                                     | ---                  |
| Node groups         | node\_groups       | Edit child group rules                  | edit\_child\_rules   |
| Node groups         | node\_groups       | Set environment                         | set\_environment     |
| Node groups         | node\_groups       | Edit classes, parameters, and variables | edit\_classification |
| Node groups		 | node\_groups       | Edit parameters and variables | edit\_params\_and\_vars |
| Node groups         | node\_groups       | Create, edit, and delete child groups   | modify\_children     |
| Node groups         | node\_groups       | View                                    | view                 |
| Users               | users              | Create                                  | create               |
| Users               | users              | Reset password                          | reset\_password      |
| Users               | users              | Revoke                                  | disable              |
| Users               | users              | Edit                                    | edit                 |
| User groups         | user\_groups       | Import                                  | import               |
| User roles          | user\_roles        | Create                                  | create               |
| User roles          | user\_roles        | Edit                                    | edit                 |
| User roles          | user\_roles        | Edit members                            | edit\_members        |
| Directory service  | directory\_service | View, edit, and test                    | edit                 |
| Certificate requests       | cert\_requests      | Accept and reject                                   | accept\_reject                |
| Puppet agent       | puppet_agent       | Run Puppet from the console or orchestrator                             | run
| Console       | console\_page      | View                                    | view                 |
| Puppet environment | environment         | Deploy code                            | deploy\_code            
| Nodes              | nodes               | View node data from PuppetDB           | view\_data             |
| Nodes              | nodes               | Edit node data from PuppetDB           | edit\_data             |

## Working with node group permissions

Node groups in the node classifier are structured hierarchically; therefore, node group permissions inherit. Users with specific permissions on a node group implicitly receive the permissions on any child groups below that node group in the hierarchy.

Two types of permissions affect a node group: those that affect a group itself, and those that affect the group's child groups. For example, giving a user the "Set environment" permission on a group allows the user to set the environment for that group and all of its children. On the other hand, assigning "Edit child group rules" to a group allows a user to edit the rules for any child group of a specified node group, but not for the node group itself. This allows some users to edit aspects of a group, while other users can be given permissions for all children of that group without being able to affect the group itself.

Due to the hierarchical nature of node groups, if a user is given a permission on the default (All) node group, this is functionally equivalent to giving them that permission on "*".

## What you should know about assigning permissions

Working with user permissions can be a little tricky. You don't want to grant users permissions that essentially escalate their role, for example. The following sections describe some strategies and requirements for setting permissions.

#### Grant edit permissions to users with create permissions

Creating new objects doesn't automatically grant the creator permission to view those objects. Therefore, users who have permission to create roles, for example, must also be given permission to edit roles, or they won't be able to see new roles that they create. Our recommendation is to assign users permission to edit all objects of the type that they have permission to create. For example:

| Type              | Permission               | Object       |
| ---               | ---                      | ---          |
| User roles        | Edit members             | All (or `"*"`) |
| Users             | Edit                     | All (or `"*"`) |

#### Avoid granting overly permissive permissions

Operators, a default role in PE, have many of the same permissions as Administrators. However, we've intentionally limited this role's ability to edit user roles. This way, members of this group can do many of the same things as Administrators, but they can't edit (or enhance) their own permissions.

Similarly, avoid granting users more permissions than their roles allow. For example, if users have the `roles:edit:*` permission, they are able to add the `node_groups:view:*` permission to the roles they belong to, and subsequently see all node groups.

#### Give permission to edit directory service settings to the appropriate users

The directory service password is not redacted when the settings are requested in the API. Only give `directory_service:edit:*` permission to users who are allowed see the password and other settings.

#### The ability to reset passwords should only be given with other password permissions

The ability to help reset passwords for users who forgot them is granted by the `users:reset_password:<instance>` permission. This permission has the side effect of reinstating revoked users once the reset token is used. As such, the reset password permission should only be given to users who are also allowed to revoke and reinstate other users.


