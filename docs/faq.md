# FAQ

(Fervently anticipated questions, in this case.) 

## How do I return an empty response?

You can either return `undefined`, or use
[`require('knork/reply').empty()`](./reference/reply.md#replyempty--response).

## Why are my logs all JSON-y?
#### Alternatively: Why *aren't* my logs all JSON-y?

`console.{log,info,warn,error}()` methods are all trapped by Knork's
[`LoggingMiddleware`][ref-logging]. If those functions are called while a
domain associated with an HTTP request is active, they will automatically be
redirected to a `bole` logger, and the current active request ID will be
included in the output. To enable the logging middleware, add it to the
list of middleware to provide to Knork on startup:

```javascript
'use strict'

const LoggingMiddleware = require('knork/middleware/logging')
const knork = require('knork')
const http = require('http')

const httpServer = http.createServer()
knork('server name', httpServer, [
  new LoggingMiddleware() // <------
])
```

## How do I get a raw Postgres connection?

```javascript
'use strict'

const db = require('knork/db/session')

db.getConnection.then(pair => {
  pair.connection.query(`SELECT * FROM tables`, function () {
    pair.release() // <-- don't forget to `release()` when you're done!
  })
}) 
```