AnyFetch file hydrater
====================
[![Build Status](https://travis-ci.org/AnyFetch/anyfetch-file-hydrater.js.png?branch=master)](https://travis-ci.org/AnyFetch/anyfetch-file-hydrater.js)[![Dependency Status](https://gemnasium.com/AnyFetch/anyfetch-file-hydrater.js.png)](https://gemnasium.com/AnyFetch/anyfetch-file-hydrater.js)
[![Coverage Status](https://coveralls.io/repos/AnyFetch/anyfetch-file-hydrater.js/badge.png?branch=master)](https://coveralls.io/r/AnyFetch/anyfetch-file-hydrater?branch=master)
[![NPM version](https://badge.fury.io/js/anyfetch-file-hydrater.png)](http://badge.fury.io/js/anyfetch-file-hydrater)

Base library for file hydration on http://anyfetch.com.

Using this library requires a single function which takes a file path and initial data, and returns more data.

How to use?
-------------------

```javascript
'use strict';

var anyfetchFileHydrater = require('anyfetch-file-hydrater');

/**
 * Hydration function, to add metadata to the document
 * 
 * @param {String} filePath Path to the file to hydrate, downloaded for you on the filesystem
 * @param {Object} document Data currently known (from previous hydraters, or from providers). Always includes `document_type`, `metadata`, `data` and `actions` keys.
 * @param {Object} changes Changes to register. Always includes `document_type`, `metadata`, `data` and `actions` keys.
 * @param {Function} Call this wuth an error if any, of your changes as second parameter once hydration has completed.
 */
var myHydrationFunction = function(path, document, changes, cb)
  // Do stuff with the file...
  // Improve document...

  cb(err, document);
};

var config = {
  'hydrater_function': myHydrationFunction
};

var hydrationServer = anyfetchFileHydrater.createServer(config);
hydrationServer.listen(8000);
```

Now you're all done! Your server is running on port 8000.
Access `/hydrate` with a standard AnyFetch POST request, and start hydrating your file.

```
POST <your_url>/hydrate
  file_path: <url-file-to-hydrate>
  callback: <url-to-ping>
  document: {base document}
```

> In some cases, you may want to override the lib and send the result yourself. To do so, you can use `cb.callbackUrl` to send datas back to the client, and then call `cb()` without any error or document to finalize hydration, clean the file and start another task.

### Optional parameters
`createServer()` takes an object hash for argument. `hydrater_function` is mandatory, optional values includes:

* `concurrency`, max number of simultaneous calls to your hydrater function (default: 1)
* `logger` function to use for logging error and success. Will get notified with strings when a task is started or ended. When an error occured, you'll get the path of the file, and the err as second argument).

Errors
------
You may use `require('anyfetch-file-hydrater').hydrationError` as a special error to inform the hydration was unable to complete, and should not be tried again:

```js
var myHydrationFunction = function(filePath, document, cb) {
  // Do stuff with the file...
  cb(new require('anyfetch-file-hydrater').hydrationError("Corrupted file"));
};
```

For other (transient) errors, use standard `Error` objects.
