# jQuery Router Plugin

This plugin handles routes for push state with a simple api and
deterministic route ordering.

### Usage

Setting up your application for use of this plugin is very easy:

#### First, include the neccesary files

```html
<script type="text/javascript" src="js/jquery-2.2.3.min.js"></script>
<script type="text/javascript" src="js/jquery.router.js"></script>
```

#### Secondly, setup the routing code

```js
$(document).ready(function(){
    // Set the base path used when changing location with $.router.go
    $.router.chroot('/');

    // Dispatch router when links with a routable class are clicked
    $.router.bind(".routable");

    // Execute a function each time a path was not registered on router
    $.router.on('route404', function(event, url) {
        alert("Path '" + url + "' not found on routing table.");
    });

    // Add entry to router which is executed each time a user clicks
    // a .routable link or uses the $.router.go function
    $.router.add("login", function(){
        $("#login").slideToggle("fast");
    });
});
```


### How to add routes

To add route you simply call the add function, providing it with the
actual route string, an optional id, and the callback.

$.router.add(*route*, *[id]*, *callback*);

#### Example:

```js
// Adds a route for /items/:item and calls the callback when matched
$.router.add("/items/:item", function(data) {
    console.log(data.item);
});
```

or

```js
// Adds a route for /items/:item and calls the callback when matched, but also has
// a reference to the routes id in $.router.currentId
$.router.add("/items/:item", "foo", function(data) {
    console.log(data.item);
});
```

### How to change urls and trigger routes

You can also change the current url to a route, and thereby triggering
the route by calling *go*. You can disable the route trigger by passing
false to the trigger parameter, this is useful when you want to change
the browser location without triggering any event.

```js
$.router.go(url, title, trigger);
```

#### Example:

```js
// This will change the url to http://www.foo.com/items/42 and set the title to
// "My cool item" without reloading the page.
// If a route has been set that matches it, it will be triggered.
$.router.go("/items/42", "My cool item");
```

Routes are examined from most the specific route to least specific route:

    Consider the following routes as added (in any order):
    /\/items\/(\d+)/: a regular expression route
    "/:p1/:p2" a route with 2 parameter
    "/:type/edit" a route with a parameter as first argument
    "/items/:item": a simple route with a parameter
    "/items/all": a simple route without any parameter
    The order in which the routes will be checked is:
    /items/all          checked first because it has 2 static parts
    /items/:item        checked second because it has 1 static part
    /:type/edit         checked after the route above because of the position of the parameter
    /:p1/:p2            checked because it has 2 parameters (more than those above)
    /\/items\/(\d+)/    regular expression routes are checked last

Please note that the ordering between regular expression based routes is undefined.

__For more usage examples check the included tests inside the test directory.__

### Reseting all routes
If you need to remove all routes (which is good when testing) you just call:

```js
$.router.reset();
```

### Events
In case a route attempt leads to no match the ___route404___ event gets
triggered with the url as argument. To ease event handling $.router.on
and $.router.off are aliases to jQuerys on and off functions.

```js
// register an event handler to respond to route misses.
$.router.on('route404', function(e, url) {
    console.log('No route for ' + url);
});
// unregister all event handlers again.
$.router.off('route404');
```

### Handling browser reloads and incoming deep links

Enabling deep links and fully supporting browser history within your
application is hopefully one of the key reasons you selected this library.
Both features require server side support, your server needs to be able to
respond to URL's that don't actually exist on the server, in a way that
your application can understand and turn into the correct content on the
client side.

Suppose you have a ToDo application and the URL looks like
this: https://example.org/my-todos/
(this is where your application,jQuery, and this plugin reside).
A user clicks on an a link on an external site like:

    https://example.org/my-todos/v/list/kNKHVXol/all

and to enable the deep linking your server redirects that to:

    https://example.org/my-todos/?v=/v/list/kNKHVXol/all

To handle this URL with jquery.router.js call ```$.router.init('v')``` v
being the name of the URL argument that contains the URL to route to.
The result will look the same in the client after your application has
initialized, but all your assets can be loaded using relative
URLs which is a nice bonus for applications that are also installed in
an on-premises context where absolute control over URL setup is not possible.

If you have no idea how to get your server to send redirects like that
and you also use Apache HTTPD check out the .htaccess file inside the test directory
because it does exactly that :).

In case you only deploy your application as a service or you have another
way of handling nonexistent URLs with your web-server of choice you may
need to change the root of what **$.router** is going to handle, to do that
call ```$.router.chroot(root)``` with the path that marks the root of
what **$.router** should handle / modify.

## License
The MIT License
