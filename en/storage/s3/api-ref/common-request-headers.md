---
title: What request headers exist in the {{ objstorage-full-name }} S3 API
description: In this article, you will learn about the request headers existing in the S3 API.
---

# Common request headers

Header | Description
----- | -----
`Authorization` | Any request to Yandex {{ objstorage-name }} must be authorized.<br/><br/>This header must be used with either the `Date` or `X-Amz-Date` header.<br/><br/>Learn about authorization methods in the respective guides.
`Cache-Control` | Directives for caching data according to [RFC 2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9).
`Content-Disposition` | Name under which {{ objstorage-name }} will suggest to save the object as a file when downloaded. Compliant with [RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec19.html#sec19.5.1).
`Content-Encoding` | Defines the content encoding according to [RFC 2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.11).
`Content-Length` | Length of the request body (without headers) in compliance with [RFC 2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.13). <br/><br/>This header is required for all requests that send to {{ objstorage-name }} any data, e.g., when uploading an object.
`Content-Type` | Data type in the request, e.g., `text/html`. For more information about data types, see the [Media type](https://en.wikipedia.org/wiki/Media_type) Wikipedia article.<br/><br/>The default type is `binary/octet-stream`.
`Content-MD5` | 128-bit MD5 hash value of the request body, `base64`-encoded.<br/><br/>This header is compliant with [RFC 1864](http://www.ietf.org/rfc/rfc1864.txt).<br/><br/>{{ objstorage-name }} uses it to make sure the data sent matches the data received.<br/><br/>If the bucket uses [default locks of object versions](../../concepts/object-lock.md#default), the `Content-MD5` header is required for uploading or copying an object version.
`Date` | Date and time of the request.<br/><br/>Format: `Thu, 18 Jan 2018 09:57:35 GMT`.<br/><br/>When `X-Amz-Date` is set, {{ objstorage-name }} ignores the `Date` header.
`Expect` | Expected `100-continue` code.<br/><br/>When uploading data to {{ objstorage-name }}, an app can use the following logic:<br/>- Send a request without a body, but with the `Expect: 100-continue` header set.<br/>- Send a request with a body after getting the `100-continue` response. This request must not have the `Expect` header.
`Expires` | Response expiration date, which is compliant with [RFC 2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.21).
`Host` | Request recipient host.<br/><br/>This header is required for HTTP/1.1, but optional for HTTP/1.0 requests.
`X-Amz-Date` | Date and time at the request source.<br/><br/>Format: `20211102T145822Z`.<br/><br/>When `X-Amz-Date` is set, {{ objstorage-name }} ignores the `Date` header.

If a cross-domain ([CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)) request is sent, it may contain [headers](object/options.md#request-headers) of the `options` preflight request.

{% include [the-s3-api-see-also-include](../../../_includes/storage/the-s3-api-see-also-include.md) %}