---
title: File and Image Uploads with Express and Firebase Cloud Functions
description: How to properly do file or image uploads with Express and Firebase Cloud Functions.
date: "2019-03-29T17:31:12.569Z"
---

I recently spent longer than I'd care to admit trying to figure out how to
properly do file or image uploads with Express and Firebase Cloud Functions.

TL;DR do _not_ try and use [Multer](https://www.npmjs.com/package/multer) for
this. As it turns out, Cloud Functions introduced a
[breaking middleware](https://stackoverflow.com/questions/47242340/how-to-perform-an-http-file-upload-using-express-on-cloud-functions-for-firebase)
for Multer which parses the body of the request. The result is that Multer
will always return a blank `req.body`, `req.file` and `req.files`.

So what are your options? Google documents them both
[here](https://cloud.google.com/functions/docs/writing/http#multipart_data), but
let me save you a click:

1. Use [`busboy`](https://www.npmjs.com/package/busboy) to directly parse
   multipart `form-data` and do something with it ([sample code](https://cloud.google.com/functions/docs/writing/http#multipart_data)).
2. Upload directly Google Cloud Storage directly using [Signed URLs](https://cloud.google.com/storage/docs/access-control/signed-urls) ([sample code](https://cloud.google.com/functions/docs/writing/http#uploading_files_via_cloud_storage))

My solution was to create and use a middleware based off the code samples
Google had that make things feel more Multer-y.

`gist:msukmanowsky/c8daf3720c2839d3c535afc69234ab9e`

Similar to Multer, this middleware will ensure `req.body` are any text fields
in `form-data` and `req.files` is an array of uploaded files.

`gist:msukmanowsky/024742f44aacd425f14d7254595684c8`

Hope this helps someone!
