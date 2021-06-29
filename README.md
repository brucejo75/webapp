# webapp
[Source code of released version](https://github.com/meteor/meteor/tree/master/packages/webapp) | [Source code of development version](https://github.com/meteor/meteor/tree/devel/packages/webapp)
***

The `webapp` package contains the core functionality that makes a
Meteor project into a web application. It is a "value added HTTP
server" that includes not just a web server, but also advanced app
serving functionality like over-the-air mobile app updates and HTML5
Appcache support. For more information, see the [Webapp project
page](https://github.com/meteor/meteor/tree/master/packages/webapp).


## Direct access to connect mongodb API

The `webapp` package is implemented using the
[npm `connect` module](https://www.npmjs.com/package/connect).  `webapp` exposes
the connect API for handling requests through `Webapp.connectHandlers`.  See
https://docs.meteor.com/#/full/webapp for more details

If you'd like direct access to the connect module (for example, to use one of
the middleware handlers that it defines), you can find it at
`WebAppInternals.NpmModules.connect.module`. Its version can be read at
`WebAppInternals.NpmModules.connect.version`.

The version of `connect` used may change incompatibly from version to version of
Meteor (or we may even replace it with an entirely different implementation);
use at your own risk.

# Dynamic Runtime Config
**NOTE:** this feature only works if you allow `inline eval`.

In some cases it is valuable to be able to control the `__meteor_runtime_config__` variable that initializes Meteor from your server code.

To work with the config object you will need to set up a runtime config hook callback.  Within the callback you will need to:
1. Decode the encoded string to an object.
2. Make your changes to the object.
3. Encode the updated config object to an encoded config string.
4. Return the encoded config string.

Example:

```javascript
WebApp.addRuntimeConfigHook((arch, request, current) => {
  // check the request to see if this is a request we want to change
  if(req.host === 'calling.domain') {
    // make changes to the config for this domain
    // convert current to an object
    const config = WebApp.decodeRuntimeConfig(current);
    // make your changes
    config.newVar = 'some value';
    config.oldVar = 'new value';
    return WebApp.encodeRuntimeConfig(config);
  }
  return undefined;
});
```

### `WebApp.encodeRuntimeConfig(config_object)`
`encodeRuntimeConfg` will convert a config object to the encoded string that is placed in `index.html`.

### `WebApp.decodeRuntimeConfig(encoded_config_string)`
`encodeRuntimeConfg` will convert an encoded config string to an object.

### `WebApp.addRuntimeConfigHook(callback)`
This sets a callback that will get called for every request for a new `index.html`.

You will get a callback with the arguments `(arch, request, encodedCurrentConfig)`.

- arch - is one of `web.browser`, `web.browser.legacy` or `web.cordova`.
- request - is the web request to meteor.
- encodedCurrentConfig - is the encoded config string.

Your callback returns the desired new value of `encoded_config_string`.  Returning a falsy value will not change the current config.
