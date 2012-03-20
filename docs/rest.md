---
layout: docs
title: Vines Cloud - REST API
---
# Vines Cloud REST API

The REST API exposes Vines Cloud's features to any programming language that can speak HTTP.  We provide open source [iOS](/docs/ios), [Android](/docs/android), and [JavaScript](/docs/js) client connectors, but they simply wrap this REST API in an easy to use library.

## Examples

We'll use the curl command line tool to demonstrate each Vines Cloud feature. All API actions, except new user signup, must be authenticated with a user name and password.  In the examples that follow, we'll set an $AUTH variable to 'your_user@your_account.getvines.com:your_password'.

```$ AUTH='alice@wonderland.getvines.com:passw0rd'
```

## Errors

When the API returns a non-200 status code error, the response body will contain a JSON formatted error object that can be used to diagnose the problem.

```{"error": {"id": "user-id-used", "status": 400}}
```

The id field can be used in error handling code to present an appropriate error message to the user.  The status field contains the HTTP status code sent in the response.

## Authentication

Authenticating with the REST API can be done in one of two ways:

1. Send credentials in an HTTP Basic Auth header with each request.

2. Login and send a session cookie with each request.

Although every API request is sent with https and is TLS encrypted, it's usually a good idea to use the session cookie approach so the user's password is sent over the network only once.

### HTTP Basic Auth

Sending user credentials with each request as an HTTP header is a simple way to authenticate with the API before it grants access to the resource we're requesting. The drawback is that the app needs to remember the user's password for their entire session.

```$ AUTH='alice@wonderland.getvines.com:passw0rd'
$ curl -i -X GET -u $AUTH https://wonderland.getvines.com/apps
```

### Session Cookie

Alternatively, we can authenticate the user just one time with the /login resource and send their session cookie with the following requests.

```$ curl -i -X POST \
  -d 'username=alice@wonderland.getvines.com' \
  -d 'password=passw0rd' \
  https://wonderland.getvines.com/login
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8
Content-Length: 180
Set-Cookie: vsession=BAh7CEkiD3Nlc3Npb25faWQGOgZF . . .; path=/; HttpOnly
{
  "created_at":"2012-03-13T21:09:41Z",
  "id":"alice@wonderland.getvines.com",
  "updated_at":"2012-03-13T21:09:41Z",
  "admin":true
}
```

Once we've authenticated with the API, we can send the user's session cookie with each subsequent request.

```$ curl -i -X GET \
  --cookie 'vsession=BAh7CEkiD3Nlc3Npb25faWQGOgZF . . .; path=/; HttpOnly' \
  https://wonderland.getvines.com/apps
```

## Users

Your mobile app can store all user account data in Vines Cloud.  User passwords are securely [bcrypt](http://en.wikipedia.org/wiki/Bcrypt) hashed, so no one, including us, can retrieve a user's password.

If your app doesn't need user accounts, you can create a shared user that you embed in the app to connect to your Vines Cloud account.  This would be equivalent to an "app key" found in other online services.

Or you might already be storing user information in your own database and use this API to sync user names and passwords into Vines Cloud. That way your app can send real-time messages with your user's credentials.

### Admin Users

There are two types of user accounts you can create and manage: admins and normal users.  An admin user has full access to the API, so they can perform tasks that a normal user can't.  For example, an admin is allowed to delete and update all users, where a normal user may only update their own information.

When you sign up for a Vines Cloud account, the first user is created as an admin.

Admin users can create, update, and delete any data they like from your account.  This is useful for calling the REST API from another web application you've built or performing maintenance tasks on your data.  However, an admin account's credentials should never be stored in your iOS or Android apps.  Remember that anyone who installs your app on their device can easily decompile it to retrieve embedded passwords.

### Signup New User

Sending a POST request to /users will create a new user account. You can use this during new user signup in your app or to sync users from another system you're running into Vines Cloud.

The only two required fields are id and password.  The id field must be the full Vines Cloud user name, including the @your_account.getvines.com.  Passwords must be at least eight characters in length.

You can store any other fields on the user object that you need.  In this example we're saving Alice's favorite color.

If you provide an email field, it will be validated as a properly formatted email address before saving.

```$ curl -i -X POST \
  -H "Content-Type: application/json" \
  -d '{"id": "alice@wonderland.getvines.com", "password":"passw0rd", "color":"blue"}' \
  https://wonderland.getvines.com/users
HTTP/1.1 201 Created
Content-Type: application/json;charset=utf-8
Location: https://wonderland.getvines.com/users/alice@wonderland.getvines.com
Content-Length: 0
```

The Location response header returns the full URI needed to retrieve this user object with a future GET request.

### Update User

Send a PUT request to the URI returned in the Location header to update the user object. Notice that now we need to authenticate with the API by passing Alice's username and password with the request. Any fields not included in the update will be left unchanged.

```$ AUTH='alice@wonderland.getvines.com:passw0rd'
$ curl -i -X PUT -u $AUTH \
  -H "Content-Type: application/json" \
  -d '{"password":"changeme", "color":"green"}' \
  https://wonderland.getvines.com/users/alice@wonderland.getvines.com
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8
Content-Length: 0
```

### Find User by ID

Send a GET request to a user's URI to retrieve their user object. Passwords and admin status are never returned to API users.

```$ curl -i -X GET -u $AUTH \
  https://wonderland.getvines.com/users/alice@wonderland.getvines.com
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8
Content-Length: 126
{
  "color":"green",
  "created_at":"2012-03-19T17:24:36Z",
  "id":"alice@wonderland.getvines.com",
  "updated_at":"2012-03-19T17:47:55Z"
}
```

### Find All Users

A list of all user accounts can be found in paged result sets by sending a GET request to the /users resource.

By default, the first 500 users are returned. The limit and skip query parameters can be used to page through the results one chunk at a time.  In this example, we're looking for the 6th and 7th users.

```$ curl -i -X GET -u $AUTH \
  https://wonderland.getvines.com/users?limit=2&skip=5
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8
Content-Length: 328
{
  "total":12,
  "offset":5,
  "rows":[
    {
      "color":"green",
      "created_at":"2012-03-20T00:33:09Z",
      "id":"alice@wonderland.getvines.com",
      "updated_at":"2012-03-20T00:33:09Z"
    }, {
      "color":"blue",
      "created_at":"2012-03-13T21:09:41Z",
      "id":"hatter@wonderland.getvines.com",
      "updated_at":"2012-03-13T21:09:41Z"
    }
  ]
}
```

The total field returns the total number of user accounts for your apps.  The offset field matches the skip query parameter to tell us where in the result set these objects were found.

### Delete User

Send a DELETE request to a user's URI to remove their account from the system.  They will no longer be able to login, save data, or send messages. Only admin users may delete user accounts and no user may delete their own account.

```$ curl -i -X DELETE -u $AUTH \
  https://wonderland.getvines.com/users/alice@wonderland.getvines.com
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8
Content-Length: 0
```

## Apps

One Vines Cloud account can host many different mobile apps.  Each app gets its own cloud database and message channels, while users are shared across all your apps for single sign-on.

To get a list of the apps in your account, send a GET request to the /apps resource.

```$ curl -i -X GET -u $AUTH https://wonderland.getvines.com/apps
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8
Content-Length: 213
{
  "total":1,
  "offset":0,
  "rows":[{
    "created_at":"2012-03-13T21:09:41Z",
    "id":"4f5fb7952f52760977000009",
    "name":"Tea Time",
    "nick":"tea-time",
    "pubsub":"00.wonderland.getvines.com",
    "updated_at":"2012-03-13T21:09:41Z"
  }]
}
```

The nick name field is generated from the name you gave your app when you signed up to Vines Cloud.  It's used to create a URI for your app that's used to store JSON objects.  In this case, the app's URI is  https://wonderland.getvines.com/apps/tea-time.

## JSON Data Storage

Vines Cloud provides a schema-less cloud database for storing JSON data objects.  Objects are grouped into storage classes you define&mdash;like Score, Player, Game, etc.  An object class is created automatically the first time an object is saved to it.

### Finding Classes

Send a GET request to an app's /classes resource to get a list of all storage classes with their current object count.

```$ curl -i -X GET -u $AUTH https://wonderland.getvines.com/apps/tea-time/classes
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8
Content-Length: 207
{
  "rows":[{
    "name":"Comment",
    "size":2,
    "url":"https://wonderland.getvines.com/apps/tea-time/classes/Comment"
  }, {
    "name":"Score",
    "size":34,
    "url":"https://wonderland.getvines.com/apps/tea-time/classes/Score"
  }]
}
```

### Create Object

Send a POST request to a storage class URI to save a new JSON object to the database. Field names can contain any alphanumeric or underscore character, but names starting with an underscore are reserved for internal API use.

```$ curl -i -X POST -u $AUTH \
  -H "Content-Type: application/json" \
  -d '{"text":"This is a comment!"}' \
  https://wonderland.getvines.com/apps/tea-time/classes/Comment
HTTP/1.1 201 Created
Content-Type: application/json;charset=utf-8
Location: https://wonderland.getvines.com/apps/tea-time/classes/Comment/4f67cd3d2f52767fa300007d
Content-Length: 0
```

The Location response header provides the full URI to retrieve this object with a future GET request.

### Find One Object

Send a GET request to the URI returned in the Location response header to retrieve the object by its ID. The created_at and updated_at timestamps are maintained by the database.

```$ curl -i -X GET -u $AUTH \
  https://wonderland.getvines.com/apps/tea-time/classes/Comment/4f67cd3d2f52767fa300007d
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8
Content-Length: 133
{
  "text":"This is a comment!",
  "created_at":"2012-03-20T00:20:13Z",
  "updated_at":"2012-03-20T00:20:13Z",
  "id":"4f67cd3d2f52767fa300007d"
}
```

### Find All Objects

All objects for a given class can be found in paged result sets by sending a GET request to the class' resource.

By default, the first 500 objects are returned. The limit and skip query parameters can be used to page through the results one chunk at a time.  In this example, we're looking for the 21st and 22nd Comment objects in the database.

```$ curl -i -X GET -u $AUTH \
  https://wonderland.getvines.com/apps/tea-time/classes/Comment?limit=2&skip=20
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8
Content-Length: 285
{
  "total":30,
  "offset":20,
  "rows":[
    {
      "text":"This is a comment!",
      "created_at":"2012-03-19T01:21:22Z",
      "updated_at":"2012-03-19T01:21:22Z",
      "id":"4f668a122f52760f66001018"
    }, {
      "text":"This is another comment.",
      "created_at":"2012-03-19T01:21:34Z",
      "updated_at":"2012-03-19T01:21:34Z",
      "id":"4f668a1e2f52760f6600101d"
    }
  ]
}
```

The total field returns the total number of objects stored in the class.  The offset field matches the skip query parameter to tell us where in the result set these objects were found.

### Queries

We're still working on a flexible query API that will find objects by any field and criteria. Coming soon!

### Update Object

Send a PUT request to the object's URI to update its fields. Partial updates are allowed. Any fields not included in the request will remain untouched.

```$ curl -i -X PUT -u $AUTH \
  -H "Content-Type: application/json" \
  -d '{"text":"This is an updated comment!"}' \
  https://wonderland.getvines.com/apps/tea-time/classes/Comment/4f67cd3d2f52767fa300007d
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8
Content-Length: 0
```

### Delete Object

Send a DELETE request to an object's URI to permanently remove it from the database.

```$ curl -i -X DELETE -u $AUTH \
  https://wonderland.getvines.com/apps/tea-time/classes/Comment/4f67cd3d2f52767fa300007d
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8
Content-Length: 0
```

## Publish/Subscribe Channels

Publishing and subscribing to real-time message channels is typically done through the XMPP API.  However, the REST API provides a way to get a list of channels currently being used by your app.

### Find Channels

Send a GET request to an app's /channels resource to get a list of active pubsub channels. Just like the [Find All Objects](#find_all_objects) API, channels can be paged through with the limit and skip query parameters.

```$ curl -i -X GET -u $AUTH \
  https://wonderland.getvines.com/apps/tea-time/channels?limit=10&skip=0
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8
Content-Length: 101
{
  "total":2,
  "offset":0,
  "rows":[
    {"name":"comments","subscribers":3},
    {"name":"scores","subscribers":11}
  ]
}
```

Each channel provides its current active subscriber count in the subscribers field.

### Publish Message

We're working on an API to publish messages to a channel through the REST web service.  For now, messages can be published through the XMPP API.

