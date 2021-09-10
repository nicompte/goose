# Debug Log

Goose can optionally and efficiently log arbitrary details, and specifics about requests and responses for debug purposes. A central logging thread maintains a buffer to minimize the IO overhead, and controls the writing to ensure that multiple threads don't corrupt each other's messages.

To write to the debug log, you must invoke `client.log_debug(tag, Option<request>, Option<headers>, Option<body>)` from your load test task functions. The `tag` field is required and can be any arbitrary string: it can identify where in the load test the log was generated, and/or why debug is being written, and/or other details such as the contents of a form the load test posts. The `request` field is an optional reference to the [`GooseRawRequest`](https://docs.rs/goose/*/goose/goose/struct.GooseRawRequest) object and provides details such as what URL was requested and if it redirected, how long into the load test the request was made, which GooseUser thread made the request, and what status code the server responded with. The `headers` field is an optional reference to all the HTTP headers returned by the remote server for this request. The `body` field is an optional reference to the entire web page body returned by the server for this request.

(_Known limitations in Reqwest prevent all headers from being recorded: <https://github.com/tag1consulting/goose/issues/336>_)

See `examples/drupal_loadtest` for an example of how you might invoke log_debug from a load test.

Calls to `client.set_failure(tag, Option<request>, Option<headers>, Option<body>)` can be used to tell Goose that a request failed even though the server returned a successful status code, and will automatically invoke `log_debug()` for you. See `examples/drupal_loadtest` and `examples/umami` to see how you might use `set_failure` to generate useful debug logs.

When the load test is run with the `--debug-log foo` command line option, where `foo` is either a relative or an absolute path, Goose will log all debug generated by calls to `client.log_debug()` (or to `client.set_failure()`) to this file. If the file already exists it will be overwritten. The following is an example debug log file entry:

```json
{"body":"<!DOCTYPE html>\n<html>\n  <head>\n    <title>503 Backend fetch failed</title>\n  </head>\n  <body>\n    <h1>Error 503 Backend fetch failed</h1>\n    <p>Backend fetch failed</p>\n    <h3>Guru Meditation:</h3>\n    <p>XID: 1506620</p>\n    <hr>\n    <p>Varnish cache server</p>\n  </body>\n</html>\n","header":"{\"date\": \"Mon, 19 Jul 2021 09:21:58 GMT\", \"server\": \"Varnish\", \"content-type\": \"text/html; charset=utf-8\", \"retry-after\": \"5\", \"x-varnish\": \"1506619\", \"age\": \"0\", \"via\": \"1.1 varnish (Varnish/6.1)\", \"x-varnish-cache\": \"MISS\", \"x-varnish-cookie\": \"SESSd7e04cba6a8ba148c966860632ef3636=Z50aRHuIzSE5a54pOi-dK_wbxYMhsMwrG0s2WM2TS20\", \"content-length\": \"284\", \"connection\": \"keep-alive\"}","request":{"coordinated_omission_elapsed":0,"elapsed":9162,"error":"503 Service Unavailable: /node/1439","final_url":"http://apache/node/1439","name":"(Auth) comment form","raw":{"body":"","headers":[],"method":"Get","url":"http://apache/node/1439"},"redirected":false,"response_time":5,"status_code":503,"success":false,"update":false,"user":1,"user_cadence":0},"tag":"post_comment: no form_build_id found on node/1439"}
```

If `--debug-log foo` is not specified at run time, nothing will be logged and there is no measurable overhead in your load test.

By default Goose writes debug logs in JSON Lines format. The `--debug-format` option can be used to log in `json`, `raw` or `pretty` format. The `raw` format is Rust's debug output of the `GooseDebug` object.