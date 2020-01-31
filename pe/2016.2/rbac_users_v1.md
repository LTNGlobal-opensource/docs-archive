---
layout: default
title: "User endpoints: API v1"
canonical: "/pe/latest/rbac_users_v1.html"
---

RBAC enables you to manage local users as well as those who are created remotely, on a directory service. With the Users endpoints, you can get lists of users create new local users.


### User keys

| Key | Explanation | Example |
| --- | ----------- | ------- |
| `id`          | A UUID string identifying the user. | `"4fee7450-54c7-11e4-916c-0800200c9a66"` |
| `login`       | A string used by the user to log in. Must be unique among users and groups. | `"admin"` |
| `email`       | An email address string. Not currently utilized by any code in PE. | `"admin@puppetlabs.org"` |
| `display_name`| The user's name as a string. | `"Theodor Geisel"` |
| `role_ids`    | An array of role IDs indicating which roles should be directly assigned to the user. An empty array is valid. | `[3 6 5]` |
| `is_group`,<br />`is_remote`,<br />`is_superuser`| These flags indicate the type of user. `is_group` should always be false for a user. | `true`/`false`|
| `is_revoked`  | Setting this flag to `true` prevents the user from accessing any routes until the flag is unset or the user's password is reset via token. | `true`/`false` |
| `last_login`  | This is a timestamp in UTC-based ISO-8601 format (YYYY-MM-DDThh:mm:ssZ) indicating when the user last logged in. If the user has never logged in, this value is null. | `"2014-05-04T02:32:00Z"` |
| `inherited_role_ids`<br />(remote users only) | An array of role IDs indicating which roles a remote user inherits from their groups. | `[9 1 3]` |
| `group_ids`<br />(remote users only) | An array of UUIDs indicating which groups a remote user inherits roles from. | `["3a96d280-54c9-11e4-916c-0800200c9a66"]` |

### GET /users
Fetches all users, both local and remote (including the superuser). Supports
filtering by ID through query parameters. Authentication is required.

**Example:**

Request all the users:

`GET /users`

~~~ json
[{
  "id": "fe62d770-5886-11e4-8ed6-0800200c9a66",
  "login": "kate",
  "email": "kate@example.com",
  "display_name": "Kate Gleason",
  "role_ids": [1,2,3...],
  "is_group" : false,
  "is_remote" : false,
  "is_superuser" : true,
  "is_revoked": false,
  "last_login": "2014-05-04T02:32:00Z"
},{
  "id": "07d9c8e0-5887-11e4-8ed6-0800200c9a66",
  "login": "humphry",
  "email": "humphry@example.com",
  "display_name": "Humphry Davy",
  "role_ids": [2, 3],
  "inherited_role_ids": [5],
  "is_group" : false,
  "is_remote" : true,
  "is_superuser" : false,
  "group_ids": ["2ca57e30-5887-11e4-8ed6-0800200c9a66"],
  "is_revoked": false,
  "last_login": "2014-05-04T02:32:00Z"
},{
  "id": "1cadd0e0-5887-11e4-8ed6-0800200c9a66",
  "login": "frances",
  "email": "frances@example.com",
  "display_name": "Frances Hugle",
  "role_ids": [2, 3],
  "inherited_role_ids": [5],
  "is_group" : false,
  "is_remote" : true,
  "is_superuser" : false,
  "group_ids": ["2ca57e30-5887-11e4-8ed6-0800200c9a66"],
  "is_revoked": false,
  "last_login": "2014-05-04T02:32:00Z"
}]
~~~

Request some specific users:

`GET /users?id=fe62d770-5886-11e4-8ed6-0800200c9a66,1cadd0e0-5887-11e4-8ed6-0800200c9a66`

~~~ json
[{
  "id": "fe62d770-5886-11e4-8ed6-0800200c9a66",
  "login": "kate",
  "email": "kate@example.com",
  "display_name": "Kate Gleason",
  "role_ids": [1,2,3...],
  "is_group" : false,
  "is_remote" : false,
  "is_superuser" : false,
  "is_revoked": false,
  "last_login": "2014-05-04T02:32:00Z"
},{
  "id": "1cadd0e0-5887-11e4-8ed6-0800200c9a66",
  "login": "frances",
  "email": "frances@example.com",
  "display_name": "Frances Hugle",
  "role_ids": [2,3...],
  "inherited_role_ids": [4],
  "is_group" : false,
  "is_remote" : true,
  "is_superuser" : false,
  "group_ids": ["2ca57e30-5887-11e4-8ed6-0800200c9a66"],
  "is_revoked": false,
  "last_login": "2014-05-04T02:32:00Z"
}]
~~~

### GET /users/&lt;sid&gt;
Fetches a single user by its subject ID (sid). Authentication is required.

**Returns:** For all users, the user contains an ID, a login, an email, a
display name, a list of role-ids the user is directly assigned to, and the last
login time in UTC-based ISO-8601 format (YYYY-MM-DDThh:mm:ssZ), or null if the
user has not logged in yet. It also contains an "is_revoked" field, which,
when set to true, prevents a user from authenticating.

~~~ json
{"id": "fe62d770-5886-11e4-8ed6-0800200c9a66",
"login": "kate",
"email": "kate@example.com",
"display_name": "Kate Gleason",
"role_ids": [1,2,3...],
"is_group" : false,
"is_remote" : false,
"is_superuser" : false,
"is_revoked": false,
"last_login": "2014-05-04T02:32:00Z"}
~~~

For remote users, the response additionally contains a field indicating
the IDs of the groups the user inherits roles from and the list of inherited
role IDs.

~~~ json
{"id": "07d9c8e0-5887-11e4-8ed6-0800200c9a66",
"login": "humphry",
"email": "humphry@example.com",
"display_name": "Humphry Davy",
"role_ids": [2,3...],
"inherited_role_ids": [],
"is_group" : false,
"is_remote" : true,
"is_superuser" : false,
"group_ids": ["b28b8790-5889-11e4-8ed6-0800200c9a66"],
"is_revoked": false,
"last_login": "2014-05-04T02:32:00Z"}
~~~

### GET /users/current
Fetches the data about the current authenticated user, with the exact same
behavior as GET /users/&lt;sid&gt;, except that `<sid>` is assumed from the authentication context. Authentication is required.

### POST /users
Creates a new local user. You can add the new user to user roles by specifying an array of roles in `role_ids`. You can set a password for the user in `password`. For the password to work in the PE console, it needs to be a minimum of six characters.

Authentication is required for this request.

**Accepts:** A JSON body containing entries for `email`, `display_name`,
`login`, `role_ids`, and `password`. The password field is optional. The created account is not useful until the password is set.

**Example:**

~~~ json
{"login":"kate",
"email":"kate@example.com",
"display_name":"Kate Gleason",
"role_ids": [1,2,3],
"password": "yabbadabba"}
~~~

**Returns:**

* **201 Created** And a Location header that points to the new
resource.
* **409 Conflict** If the user email/login collides with an existing user.

### PUT /users/&lt;sid&gt;
Replaces the user with the specified ID with a new user object. Authentication is required.

**Accepts:** An updated user object with all keys provided when the object is
received from the API. The behavior varies based on user type. All types have a `role_id` array that indicates all the roles the user should belong to. An
empty roles array removes all roles directly assigned to the group.

The below examples show what keys must be submitted. Keys marked with
*asterisks* are the only ones that can be changed via the API.

Local user:

~~~ json
{"id": "c8b2c380-5889-11e4-8ed6-0800200c9a66",
**"login": "frances",**
**"email": "frances@example.com",**
**"display_name": "Frances Hugle",**
**"role_ids": [1, 2, 3],**
"is_group" : false,
"is_remote" : false,
"is_superuser" : false,
**"is_revoked": false,**
"last_login": "2014-05-04T02:32:00Z"}
~~~

Remote user:

~~~ json
{"id": "3271fde0-588a-11e4-8ed6-0800200c9a66",
"login": "thomas",
"email": "thomas@example.com",
"display_name": "Thomas Davenport",
**"role_ids": [4, 1],**
"inherited_role_ids": [],
"group_ids: [],
"is_group" : false,
"is_remote" : true,
"is_superuser" : false,
**"is_revoked": false,**
"last_login": "2014-05-04T02:32:00Z"}
~~~

**Returns:**

* **200 OK** The user object with the context appropriate payload.
* **409 Conflict** If the user login collides with an existing user's login.

The request should return the user object with the changes made.

### DELETE /users/&lt;sid&gt;

Deletes the user with the specified ID, regardless of whether they are a user defined in RBAC or a user defined by the Directory Service. If they are from the Directory Service, while this action removes them from the UI, they are still be able to log in (at which point they are re-added to the UI) if they are not revoked. Authentication required.

**Note:** The `admin` user and the `api_user` cannot be deleted.

**Returns:**

* **204 No Content** The user was successfully deleted.
* **403 Forbidden** The user does not have the `users:edit` permission for this user.
* **404 Not Found** A user with the given identifier does not exist.
