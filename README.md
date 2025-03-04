node-static
===========

> a simple, *rfc 2616 compliant* file streaming module for [node](http://nodejs.org)

node-static has an in-memory file cache, making it highly efficient.
node-static understands and supports *conditional GET* and *HEAD* requests.
node-static was inspired by some of the other static-file serving modules out there,
such as node-paperboy and antinode.

synopsis
--------

    var static = require('node-static');

    //
    // Create a node-static server instance to serve the './public' folder
    //
    var file = new(static.Server)('./public');

    require('http').createServer(function (request, response) {
        request.addListener('end', function () {
            //
            // Serve files!
            //
            file.serve(request, response);
        });
    }).listen(8080);

API
---

### Creating a node-static Server #

Creating a file server instance is as simple as:

    new static.Server();

This will serve files in the current directory. If you want to serve files in a specific
directory, pass it as the first argument:

    new static.Server('./public');

You can also specify how long the client is supposed to cache the files node-static serves:

    new static.Server('./public', { cache: 3600 });

This will set the `Cache-Control` header, telling clients to cache the file for an hour.
This is the default setting.

### Serving files under a directory #

To serve files under a directory, simply call the `serve` method on a `Server` instance, passing it
the HTTP request and response object:

    var fileServer = new static.Server('./public');

    require('http').createServer(function (request, response) {
        request.addListener('end', function () {
            fileServer.serve(request, response);
        });
    }).listen(8080);

### Serving specific files #

If you want to serve a specific file, like an error page for example, use the `serveFile` method:

    fileServer.serveFile('/error.html', 500, {}, request, response);

This will serve the `error.html` file, from under the file root directory, with a `500` status code.
For example, you could serve an error page, when the initial request wasn't found:

    require('http').createServer(function (request, response) {
        request.addListener('end', function () {
            fileServer.serve(request, response, function (e, res) {
                if (e && (e.status === 404)) { // If the file wasn't found
                    fileServer.serveFile('/not-found.html', request, response);
                }
            });
        });
    }).listen(8080);

More on intercepting errors bellow.

### Intercepting errors & Listening #

An optional callback can be passed as last argument, it will be called every time a file
has been served successfully, or if there was an error serving the file:

    var fileServer = new static.Server('./public');

    require('http').createServer(function (request, response) {
        request.addListener('end', function () {
            fileServer.serve(request, response, function (err, result) {
                if (err) { // There was an error serving the file
                    sys.error("Error serving " + request.url + " - " + err.message);

                    // Respond to the client
                    response.writeHead(err.status, err.headers);
                    response.end();
                }
            });
        });
    }).listen(8080);

Note that if you pass a callback, and there is an error serving the file, node-static
*will not* respond to the client. This gives you the opportunity to re-route the request,
or handle it differently.

For example, you may want to interpret a request as a static request, but if the file isn't found,
send it to an application.

If you only want to *listen* for errors, you can use *event listeners*:

    fileServer.serve(request, response).addListener('error', function (err) {
        sys.error("Error serving " + request.url + " - " + err.message);
    });

With this method, you don't have to explicitly send the response back, in case of an error.

### Options when creating an instance of `Server` #

#### `cache` #

Sets the `Cache-Control` header.

example: `{ cache: 7200 }`

Passing a number will set the cache duration to that number of seconds.
Passing `false` will disable the `Cache-Control` header.

> Defaults to `3600`

#### `headers` #

Sets response headers.

example: `{ 'X-Hello': 'World!' }`

> defaults to `{}`


#### `customContentTypes` #

Add/override file extension MIME type mappings. This is, for example, useful when serving HTML 5 application cache manifest files.

example: `{ 'htm': 'text/html', 'manifest': 'text/cache-manifest' }`

> defaults to `{}`

