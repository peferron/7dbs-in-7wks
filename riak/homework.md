# Riak homework

## Day 1

### 1.

```
curl -i -X PUT "http://localhost:10038/riak/animals/polly" \
-H "Link: </riak/photos/polly.jpg>; riaktag=\"photo\"" \
-d '{"nickname": "Sweet Polly Purebred", "breed": "Purebred"}'
```

### 2.

```
curl -i -X PUT "http://localhost:10038/riak/cats/takkun.svg" \
-H "Content-Type: image/svg+xml" \
--data-binary @takkun.svg
```

### 3.

```
curl -i -X PUT "http://localhost:10038/riak/medicines/antibiotics" \
-H "Content-Type: image/jpeg" \
-H "Link: </riak/animals/ace>; riaktag=\"treats\"" \
--data-binary @antibiotics.jpg
```
