# CouchDB homework

## Day 1

### Find, 1.

- [HTTP API overview](http://docs.couchdb.org/en/2.0.0/intro/api.html)
- [Complete HTTP API Reference](http://docs.couchdb.org/en/2.0.0/api/index.html)

### Find, 2.

`HEAD` and `COPY`.

### Do, 1.

```bash
$ curl -i -X PUT "http://localhost:5984/music/my_id" \
-H "Content-Type: application/json" \
-d '{"name": "Aaron Funk"}'

HTTP/1.1 201 Created
Server: CouchDB/1.6.1 (Erlang OTP/19)
Location: http://localhost:5984/music/my_id
ETag: "1-645205ac66ae745f4b37485ed6bdc979"
Date: Wed, 30 Nov 2016 05:14:14 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 68
Cache-Control: must-revalidate

{"ok":true,"id":"my_id","rev":"1-645205ac66ae745f4b37485ed6bdc979"}
```

### Do, 2.

Create database:

```bash
$ curl -i -X PUT "http://localhost:5984/my_new_db"

HTTP/1.1 201 Created
Server: CouchDB/1.6.1 (Erlang OTP/19)
Location: http://localhost:5984/my_new_db
Date: Wed, 30 Nov 2016 05:15:14 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 12
Cache-Control: must-revalidate

{"ok":true}
```

Delete database:

```bash
$ curl -i -X DELETE "http://localhost:5984/my_new_db"

HTTP/1.1 200 OK
Server: CouchDB/1.6.1 (Erlang OTP/19)
Date: Wed, 30 Nov 2016 05:15:56 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 12
Cache-Control: must-revalidate

{"ok":true}
```

### Do, 3.

Create document with attachment:

```bash
$ curl -i -X POST "http://localhost:5984/my_new_db" \
-H "Content-Type: application/json" \
--data @-
{
  "_attachments": {
    "hello.txt": {
      "content_type": "text\/plain",
      "data": "SGVsbG8sIFdvcmxkIQ=="
    }
  }
}

HTTP/1.1 201 Created
Server: CouchDB/1.6.1 (Erlang OTP/19)
Location: http://localhost:5984/my_new_db/12edea69b62a7136c1efc01e44030ca8
ETag: "1-6341c778e10be461c23e8642b7a54146"
Date: Wed, 30 Nov 2016 05:26:16 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 95
Cache-Control: must-revalidate

{"ok":true,"id":"12edea69b62a7136c1efc01e44030ca8","rev":"1-6341c778e10be461c23e8642b7a54146"}
```

Fetch attachment:

```bash
$ curl "http://localhost:5984/my_new_db/12edea69b62a7136c1efc01e44030ca8/hello.txt"

Hello, World!
```
