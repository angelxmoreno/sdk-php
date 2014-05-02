OAuth.io PHP SDK
================

OAuth that just works !

This SDK allows you to use OAuth.io's server-side flow from a PHP backend, to handle access tokens from your server instead of directly from your front-end.

You can use it with one of our front-end SDKs ([JavaScript][1], [PhoneGap][2], [iOS][3], [Android][4]), which will handle the user input for the OAuth flow.

This SDK is still under heavy development and some of the features described below may not work yet. You can get nightlies from the [develop branch](https://github.com/oauth-io/sdk-php/tree/develop) on the SDK's github page.

A release will be posted soon.

Common use-Case
---------------

You don't want to use APIs directly from the front-end, but rather through web-services inside your PHP backend.

The server-side flow
--------------------

In the server-side OAuth authentication flow, the oauth token never leaves your backend.

To authenticate a user, the flow follows these steps :

- Ask for a unique CSRF token to the backend. This token will be used as a state for communicating with oauth.io
- Show a popup or redirect your user to request his permission to use his/her account on the requested provider
- The latter gives you a code, that you give to your backend
- The backend sends the code to oauth.io with other information like the oauth.io app's public key and secret.
- oauth.io responds with the access_token, that you can then store on your backend as long as it's valid.
- You can then make requests to the API using that access token, directly from your backend.

Installation
------------

You will soon be able to install it through Composer by adding oauthio-sdk to your dependencies and running :

```sh
$ composer install
```

How to use it ?
---------------

The `OAuth` class is stored in the `OAuth_io` namespace. You need to include it in your file like this (make sure you have required the Composer autoloader file) :

```php
<?php

require_once '/path/to/autoload.php';

use OAuth_io\OAuth;

//?>
```

**Initialization**

To initialize the SDK, you have to give it your OAuth.io's app's key and secret (you can grab them on the [oauth.io Key-Manager][https://oauth.io/key-manager]) :

```php
$oauth = new OAuth();
$oauth->initialize('your_key', 'your_secret');
```

**Generating a token**

You need to provide your front-end with a state token, that will be used to exchange information with OAuth.io. To generate it in the back-end :

```php
$token = $oauth->generateToken();
```

The `generateToken()` method returns a unique 23 character token. This token is stored in the session, and used by the other features.

You can give this token to your front-end, where you can show the user a popup for him to log in to the provider and accept your app's permissions (see further down to see how to do that).

**Auth the user**

To be able to make requests to a provider's API using its access token, you have to call the `auth(code)` method. The code is retrieved from OAuth.io through the from the front-end SDK. You need to create an endpoint to allow the front-end to send it to the backend.

Once you have that code, you can call the method like this :

```php
$result = $oauth->auth($code);
```

`$result` is an array containing the access token, which you can use your own way if you want, or thanks to the SDK's request system (see further down).

**Retrieving the code from the front-end**

```JavaScript
//In the front end, using the JavaScript SDK :

OAuth.initialize('your_key');
OAuth.popup('a_provider', {
        state: 'the_token_retrieved_from_your_backend'
    })
.done(function (r) {
    //You need to give r.code to your backend
    $.ajax({
            url: '/auth_endpoint/signin',
            data: {
                code: r.code
            }
    })
    .done(function (data, status) {
        //your user is authenticated server side
        //you can now call endpoints that use the OAuth.io SDK
    });
});
```

**Making requests to the API**

You can create a request object from the SDK `create('provider')` method :

```php
$request_object = $oauth->create('some_provider');
```

Then, you can make get, post, put, delete and patch requests to the API like this :

```php
$response_GET = $request_object->get('https://theprovider.com/api/endpoint');
$response_POST = $request_object->post('https://theprovider.com/api/endpoint', array('some' => 'data'));
$response_PUT = $request_object->put('https://theprovider.com/api/endpoint', array('some' => 'data'));
$response_DELETE = $request_object->del('https://theprovider.com/api/endpoint');
$response_PATCH = $request_object->patch('https://theprovider.com/api/endpoint', array('some' => 'data'));
```

You can also call the `me(array $filters)` method from that request object. This method returns a unified array containing information about the user.

```php
$facebook_requester = $oauth->create('facebook');
$result = $facebook_requester->me(array('firstname', 'lastname', 'email'));

// you'll have $result["firstname"], $result["lastname"] and $result["email"] set with the user's facebook information.
```

You can refer to the OAuth.io me() feature to get more information about the fields that are returned by this method.


Contributing to this SDK
------------------------

**Issues**

Please discuss issues and features on Github Issues. We'll be happy to answer to your questions and improve the SDK based on your feedback.

**Pull requests**

You are welcome to fork this SDK and to make pull requests on Github. We'll review each of them, and integrate in a future release if they are relevant.

The SDK is written as a Composer module. You can install its dependencies like this :

```sh
sdk/folder$ composer install
```

Testing the SDK
---------------

We use PHPUnit to test the SDK. To test it, just run the following from the SDK root folder :

```bash
$ ./vendor/phpunit/phpunit/phpunit
```

License
-------

The SDK is released under the Apache2 license.




[1]: https://github.com/oauth-io/oauth-js
[2]: https://github.com/oauth-io/oauth-phonegap
[3]: https://github.com/oauth-io/oauth-ios
[4]: oauth-android