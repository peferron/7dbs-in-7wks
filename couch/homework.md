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

## Day 2

### Find, 1.

Keys can be numbers, arrays, or objects.

### Find, 2.

[Querying Options](https://wiki.apache.org/couchdb/HTTP_view_API#Querying_Options)

### Do, 1.

```js
function(artist) {
  emit(artist.random, artist.name);
}
```

### Do, 2.

```bash
curl -i "http://localhost:5984/music/_design/random/_view/artist?limit=1&startkey=$(ruby -e 'puts rand')"
```

Note that this can return an empty array if the randomly generated `startkey` parameter is greater than all `random` values in the DB.

### Do, 3.

Album view:

```js
function(artist) {
  artist.albums.forEach(function(album) {
    emit(album.random, artist.name + " - " + album.name);
  });
}
```

Track view:

```js
function(artist) {
  artist.albums.forEach(function(album) {
    album.tracks.forEach(function(track) {
      emit(track.random, artist.name + " - " + album.name + " - " + track.name);
    });
  });
}
```

Tag view (the same tag can appear multiple times):

```js
function(artist) {
  artist.albums.forEach(function(album) {
    album.tracks.forEach(function(track) {
      track.tags.forEach(function(tag) {
        emit(tag.random, tag.idstr);
      });
    });
  });
}
```

### Day 3

### Find, 1.

[Built-In Reduce Functions](https://wiki.apache.org/couchdb/Built-In_Reduce_Functions)

`sum`, `count` and `stats`. Faster than JS because implemented in Erlang and runs directly inside CouchDB.

### Find, 2.

Um, using the `filter` query parameter shown in the book? Don't get this question.

### Find, 3.

Run a one-time replication:

```bash
curl -i -X POST "http://localhost:5984/_replicate" \
-H "Content-Type: application/json" \
-d '{"source": "music", "target": "music-repl"}'
```

Run a continuous replication (lost on server restart):

```bash
curl -i -X POST "http://localhost:5984/_replicate" \
-H "Content-Type: application/json" \
-d '{"source": "music", "target": "music-repl", "continuous": true}'
```

Cancel a replication (add `"continuous": true` if the replication was continuous):

```bash
curl -i -X POST "http://localhost:5984/_replicate" \
-H "Content-Type: application/json" \
-d '{"source": "music", "target": "music-repl", "cancel": true}'
```

### Find, 4.

`POST` (or `PUT`) to `_replicator` instead of `_replicate`. Replications will be stored in the `_replicator` database, and automatically restarted if the server restarts. Replications can be deleted with `DELETE` just like normal documents.

### Do, 1 & 2.

Body of the `watcher.start` function:

```js
var http_options = {
  host: watcher.host,
  port: watcher.port,
  path: '/' + watcher.db + '/_changes' +
    '?feed=continuous&include_docs=true&since=' + watcher.last_seq
};

http.get(http_options, function(res) {
  var buffer = '';
  var error = false;

  res.on('data', function(chunk) {
    buffer += chunk;

    var lastNewLine = buffer.lastIndexOf('\n');
    if (lastNewLine < 0) {
      return;
    }

    var lines = buffer.substring(0, lastNewLine).split('\n');
    buffer = buffer.substring(lastNewLine + 1);

    lines.forEach(function(line) {
      if (!line) {
        return;
      }

      var change = JSON.parse(line);

      if (change.hasOwnProperty(error)) {
        error = true;
        watcher.emit('error', change);
      } else if (change.hasOwnProperty('seq')) {
        watcher.last_seq = change.seq;
        watcher.emit('change', change);
      }
    });
  });

  res.on('end', function() {
    if (!error) {
      watcher.start();
    }
  })
})
.on('error', function(err) {
  watcher.emit('error', err);
});

```

### Do, 3.

```
function(doc) {
  if (doc._conflicts) {
    emit(doc._id, {_rev: doc._rev, _conflicts: doc._conflicts});
  }
}
```
