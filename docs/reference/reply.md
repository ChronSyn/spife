# Reply Module

```javascript
const reply = require('spife/reply')
```

Contains methods for creating and manipulating HTTP responses.

## Table of Contents

* [API](#api)
  * [Types](#types)

    * [Response&lt;Any>](#responseany)

    * [TemplateResponse](#templateresponse)

    * [HTTPError](#httperror)

    * [ServerError extends HTTPError](#servererror-extends-httperror)

      * [Concrete ServerError Subclasses](#concrete-servererror-subclasses)

    * [ClientError](#clienterror)

      * [Concrete ClientError Subclasses](#concrete-clienterror-subclasses)
      * [NoMatchError](#nomatcherror)

    * [CacheControl](#cachecontrol)

  * [Methods](#methods)

    * [reply(resp\[, code\]\[, headers\]) → Response&lt;resp>](#replyresp-code-headers--responseresp)
    * [reply.cacheControl(resp) → CacheControl](#replycachecontrolresponse--cachecontrol)
    * [reply.cacheControl(resp, cacheControl) → Response&lt;resp>](#replycachecontrolresponse-cachecontrol--response-resp)
    * [reply.cookie(resp, name, value, opts) → Response&lt;resp>](#replycookieresp-name-value-opts--responseresp)
    * [reply.empty(code) → Response&lt;''>](#replyempty--response)
    * [reply.link(resp) → Object | undefined](#replylinkresp--object--undefined)
    * [reply.link(resp, rel) → Object | undefined](#replylinkresp-rel--object--undefined)
    * [reply.link(resp, rel, value) → Response&lt;resp>](#replylinkresp-rel-value--responseresp)
    * [reply.header(resp, header) → String | undefined](#replyheaderresp-header--string--undefined)
    * [reply.header(resp, header, string) → Response&lt;resp>](#replyheaderresp-header-string--responseresp)
    * [reply.headers(resp) → Object | undefined](#replyheadersresp--object--undefined)
    * [reply.headers(resp, headers) → Response&lt;resp>](#replyheadersresp-headers--responseresp)
    * [reply.raw(resp) → Response&lt;resp>](#replyrawresp--responseresp)
    * [reply.redirect(\[resp, \]url\[, code = 302\]) → Response&lt;resp>](#replyredirectresp-url-code--302--responseresp)
    * [reply.status(resp) → Number | undefined](#replystatusresp--number--undefined)
    * [reply.status(resp, code) → Response&lt;resp>](#replystatusresp-code--responseresp)
    * [reply.template(name, context) → TemplateResponse](#replytemplatename-context--templateresponse)
    * [reply.toStream(response) → Response&lt;resp>](#replytostreamresponse--responseresp)
    * [reply.vary(resp) → Array&lt;String>](#replyvaryresp--arraystring)
    * [reply.vary(resp, on) → Response&lt;resp>](#replyvaryresp-on--responseresp)

## API

### Types

#### `Response<Any>`

A pseudo-type for the purposes of this documentation. Any `Request<Object>`
will return `Object`. [Primitive values][def-primitive], like strings, boolean
values, undefined or numbers will be cast into a `ReadableStream` yielding the
original value stringified via `toString`.

For example:

```javascript
const reply = require('spife/reply')
const obj = {}
const rex = /asdf/
const fn = function () {}

reply(obj) === obj
reply(rex) === rex
reply(fn) === fn

// while ...
reply('example') !== 'example'
reply('example') instanceof require('stream').Readable
```

#### `TemplateResponse`

A response type created by [`reply.template(name,
context)`](#replytemplatename-context--templateresponse), to be lazily rendered
by [`TemplateMiddleware`][template-middleware]. For more on working with
templates in spife, see the [templates guide][templates-guide].

#### `HTTPError`

A base class for all HTTP errors. 

> :warning: **Not to be instantiated directly.**
>
> This class is to be used for checked promise error handlers — not for direct
> instantiation. Use one of the [concrete error
> subclasses](#concrete-servererror-subclasses) below.
>
> ```javascript
> someOperation().catch(reply.HTTPError, err => doSomething(err))  // OK!
> throw new reply.HTTPError() // not OK!
> ```

#### `ServerError extends HTTPError`

A base class for `5XX` range HTTP errors. Not instantiable.

##### Concrete `ServerError` Subclasses

Concrete subclasses are automatically assigned an error code and a default
error message that may be overridden by the user.

```javascript
'use strict'

const reply = require('spife/reply')
const request = require('request')

module.exports = myView

function myView (req, context) {
  return someBackendOperation()
    .catch(err => {
      throw new reply.NotImplementedError('optional message!')
    })
  })
}
```

| Class name                           | Code |
| ------------------------------------ | :--- |
| `InternalServerError`                | 500  |
| `NotImplementedError`                | 501  |
| `BadGatewayError`                    | 502  |
| `ServiceUnavailableError`            | 503  |
| `GatewayTimeoutError`                | 504  |
| `HTTPVersionNotSupportedError`       | 505  |
| `VariantAlsoNegotiatesError`         | 506  |
| `InsufficientStorageError`           | 507  |
| `LoopDetectedError`                  | 508  |
| `BandwidthLimitExceededError`        | 509  |
| `NotExtendedError`                   | 510  |
| `NetworkAuthenticationRequiredError` | 511  |

#### `ClientError`

A base class for `4XX` range HTTP errors. Not instantiable.

##### Concrete `ClientError` Subclasses

Concrete subclasses are automatically assigned an error code and a default
error message that may be overridden by the user.

```javascript
'use strict'

const reply = require('spife/reply')
const request = require('request')

module.exports = myView

function myView (req, context) {
  return req.body.then(body => {
    if (body.waffles && body.pancakes) {
      throw new reply.ConflictError('I literally cannot decide')
    }
  })
}
```

| Class name                         | Code |
| ---------------------------------- | :--- |
| `BadRequestError`                  | 400  |
| `UnauthorizedError`                | 401  |
| `PaymentRequiredError`             | 402  |
| `ForbiddenError`                   | 403  |
| `NotFoundError`                    | 404  |
| `MethodNotAllowedError`            | 405  |
| `NotAcceptableError`               | 406  |
| `ProxyAuthenticationRequiredError` | 407  |
| `RequestTimeoutError`              | 408  |
| `ConflictError`                    | 409  |
| `GoneError`                        | 410  |
| `LengthRequiredError`              | 411  |
| `PreconditionFailedError`          | 412  |
| `PayloadTooLargeError`             | 413  |
| `URITooLongError`                  | 414  |
| `UnsupportedMediaTypeError`        | 415  |
| `RangeNotSatisfiableError`         | 416  |
| `ExpectationFailedError`           | 417  |
| `ImATeapotError`                   | 418  |
| `UnprocessableEntityError`         | 422  |
| `LockedError`                      | 423  |
| `FailedDependencyError`            | 424  |
| `UnorderedCollectionError`         | 425  |
| `UpgradeRequiredError`             | 426  |
| `PreconditionRequiredError`        | 428  |
| `TooManyRequestsError`             | 429  |
| `RequestHeaderFieldsTooLargeError` | 431  |
| `UnavailableForLegalReasonsError`  | 451  |

##### `NoMatchError`

`NoMatchError` is a subclass of `NotFoundError` that indicates that Spife
did not find a match for an incoming request in its router.

#### `CacheControl`

```javascript
{
  mustRevalidate: Boolean,
  noCache: Boolean,
  noStore: Boolean,
  noTransform: Boolean,
  public: Boolean, // if true, private must be false
  private: Boolean, // if true, public must be false
  proxyRevalidate: Boolean,
  maxAge: Number,
  sharedMaxAge,
  immutable: Boolean,
  staleWhileRevalidate: Number,
  staleIfError: Number
}
```

Corresponds to the response [`Cache-Control`] values.

### Methods

#### `reply(resp[, code][, headers]) → Response<resp>`

Create a `Response<T>` from `resp`, optionally associating a status code
and a set of headers. When given, the `code` and `headers` parameters will
replace any corresponding values associated with `resp`.

```javascript
'use strict'
const reply = require('spife/reply')

module.exports = function myView (req, context) {
  return reply(
    'dogs are just small bears',
    203,
    {'content-type': 'text/plain+lies'}
  )
}
```

##### A Note On Headers

> :rotating\_light: **Header keys and values are interpreted as
> [ISO-8859-1][def-latin-1].**
>
> Because of this behavior in the underlying HTTP specification, spife does not
> allow non-[ASCII][def-ascii] characters to be passed in as header keys or
> values, throwing an error if it detects non-ASCII characters. Malicious
> clients could otherwise insert UTF8 values that will decompose to newlines,
> which allows for an attack known as [response
> splitting][def-response-splitting].

#### `reply.cacheControl(resp) → CacheControl`

Given a response, parse its [`Cache-Control`] header into a
[`CacheControl`](#cachecontrol) object.

#### `reply.cacheControl(resp, cacheControl) → Response&lt;resp>`

Given a response and a `CacheControl` configuration, apply the new cache
control to the response. This will *merge* the new cache control configuration
with the old cache control configuration.

#### `reply.cookie(resp, name, value, opts) → Response&lt;resp>`

Set a cookie named `name` to `value`. `opts` accepts the same parameters as the
[:package: jshttp/cookie package](https://github.com/jshttp/cookie#options-1).

<a name="replyempty--response"></a>
#### `reply.empty(code) → Response<''>`

A shorthand for [`raw('')`][shorthand-raw]. Useful for returning `201 Created`
or `204 No Content` responses. If no `code` is given, `204 No Content` will be
used.

```javascript
'use strict'
const reply = require('spife/reply')

module.exports = function myView (req, context) {
  return deleteSomeModel().then(() => {
    return reply.empty(204)
  })
}
```

#### `reply.link(resp) → Object | undefined`

Return a parsed [link header][def-link] from the response, if any.

#### `reply.link(resp, rel) → Object | undefined`

Return a parsed [link header][def-link] _relation_ from the response, if any.

#### `reply.link(resp, rel, value) → Response<resp>`

Add a [link relation][def-link-rel] to a [`link` header][def-link] associated
with a response, returning the response.

```javascript
header(link(resp, [{
  rel: 'next',
  url: 'some/url'
}, {
  rel: 'prev',
  url: 'another/url'
}]), 'link') // <some/url>; rel="next", <another/url>; rel="prev"
```

> :rotating\_light: **See [a note on headers](#a-note-on-headers).**

#### `reply.header(resp, header) → String | undefined`

Returns the current value of the header associated with `resp`, if any.

#### `reply.header(resp, header, string) → Response<resp>`

Associates a response with a header and a value. Replaces an existing header
by the given name, other headers are left intact.

```javascript
'use strict'
const reply = require('spife/reply')

module.exports = function myView (req, context) {
  return reply.header(
    'cool text!'
    'content-type'
    'text/plain'
  )
}
```

> :rotating\_light: **See [a note on headers](#a-note-on-headers).**

#### `reply.headers(resp) → Object | undefined`

Return an object representing all headers associated with the response, if
any. Returns `undefined` if no headers are associated with the response.

#### `reply.headers(resp, headers) → Response<resp>`

Replace all headers associated with the response with those given by `headers`.

```javascript
'use strict'
const reply = require('spife/reply')

module.exports = function myView (req, context) {
  return reply.headers(
    'cool text!'
    {'content-type': 'text/plain'}
  )
}
```

> :rotating\_light: **See [a note on headers](#a-note-on-headers).**

#### `reply.raw(resp) → Response<resp>`

A shorthand for [`reply(resp)`][shorthand-reply]. Often used with strings or
other primitive data, when associating headers or status information with such
a response is desired. Returns a [`stream.Readable`][stream-readable] for
[primitive values][def-primitive].

```javascript
'use strict'
const reply = require('spife/reply')

module.exports = function myView (req, context) {
  return reply.status(
    reply.raw('aw dang I could not find that for you'),
    404
  )
}
```

#### `reply.redirect([resp, ]url[, code = 302]) → Response<resp>`

A shorthand for [`header(empty(302), 'location',
url)`][shorthand-header]. Handy for redirecting clients after the success of an
operation, or when a resource exists elsewhere. Combine with
[`reverse.reverse`][reverse-reverse] for best results:

```javascript
'use strict'
const reply = require('spife/reply')
const myUrls = require('../urls/path/to/my/urls')

module.exports = function myView (req, context) {
  return createSomeModel().then(instance => {
    return reply.redirect(
      myUrls.reverse('somemodel.view', {slug: instance.slug})
    )
  })
}
```

* **See also**: [The `Location` header][def-location].

#### `reply.status(resp) → Number | undefined`

Given a potential response object, return the status code
associated with the response, if any.

```javascript
'use strict'
const reply = require('spife/reply')
reply.status(new reply.NotFound())    // 404
reply.status({})                     // undefined
```

#### `reply.status(resp, code) → Response<resp>`

Associate a status code with the potential response. If the response is
[primitive][def-primitive], upcast it to a [`stream.Readable`][stream-readable]
and associate the status with that object.

```javascript
'use strict'
const reply = require('spife/reply')
const resp = {}
reply.status(resp, 204)              // === resp, w/ 204 status
reply.status('', 204)                // === stream.Readable w/ 204 status
```

#### `reply.template(name, context) → TemplateResponse`

Create a lazily-rendered template response, to be handled by
[`TemplateMiddleware`][template-middleware]. Takes two arguments: a `name`
string to use to lookup the template, and a `context` object to use to render
the template.

* **See also**: The [templates guide][templates-guide].

#### `reply.toStream(response) → Response<resp>`

Coerces an existing response to a stream. This will be called automatically by
Spife as a final step on any response before it is flushed. It is guaranteed
to return a `pipe`-able stream instance.

- If `response` is a string, it will be coerced into a `Readable` string and given
  a `content-type` of `text/plain`.
- If `response` has `pipe` and is object mode, a "newline delimited JSON" stream
  will be returned. A content-type of `application/x-ndjson` will be applied if
  no other content type is present.
- If `response` has `pipe` and is not object mode, it will be passed through.
- If `response` is a buffer, it will be given a content type of `application/octet-stream`
  if it doesn't already have a `content-type` header. It will also be given an
  appropriate `content-length` header.
- If `response` is any other type of object, it will be `JSON.stringify`'d and given a
  content type of `application/json`. It will also be given an appropriate
  `content-length` header.

If the response doesn't already have a status code set, it will be given a
status of `200 OK`. Any headers & status codes set on the incoming `response`
parameter will be retained.

```javascript
'use strict'
const reply = require('@npm/spife/reply')

reply.toStream('hello world') // Stream with pipe!
reply.toString({some: 'json'}) // Stream with pipe!
```

#### `reply.vary(resp) → Array&lt;String>`

Return an array of request headers to vary on, per the [`Vary`] header.

#### `reply.vary(resp, on) → Response&lt;resp>`

Set the request headers to be varied on using the [`Vary`] header. This
function deduplicates repeated values.

```javascript
reply.vary(myResponse, 'this-one-header')
reply.vary(myResponse, ['cookie', 'accept-encoding'])
```

[`Vary`]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Vary

[`Cache-Control`]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control#Cache_response_directives

[shorthand-raw]: #replyrawresp--responseresp

[shorthand-reply]: #replyresp-code-headers--responseresp

[shorthand-header]: #replyheaderresp-header-string--responseresp

[reverse-reverse]: https://github.com/chrisdickinson/reverse#routerreversenamestring-argsobject--string--null

[stream-readable]: https://nodejs.org/api/stream.html#stream_class_stream_readable

[def-latin-1]: https://en.wikipedia.org/wiki/ISO/IEC_8859-1

[def-ascii]: https://en.wikipedia.org/wiki/ASCII

[def-location]: https://en.wikipedia.org/wiki/HTTP_location

[def-link]: https://tools.ietf.org/html/rfc5988

[def-link-rel]: https://tools.ietf.org/html/rfc5988#section-4

[def-response-splitting]: https://en.wikipedia.org/wiki/HTTP_response_splitting

[def-primitive]: https://developer.mozilla.org/en-US/docs/Glossary/Primitive

[templates-guide]: ../topics/using-templates.md

[template-middleware]: ./middleware/template.md
