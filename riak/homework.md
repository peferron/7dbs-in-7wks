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

## Day 2

### 1.

```
curl -X POST -H "content-type:application/json" \
"http://localhost:10018/mapred" --data @-
{
    "inputs": "rooms",
    "query": [
        {
            "map": {
                "language": "javascript",
                "source": "function(v) {
                    var floor = Math.floor(parseInt(v.key, 10) / 100);
                    var parsed_data = JSON.parse(v.values[0].data);
                    var data = {};
                    data[floor] = parsed_data.capacity;
                    return [data];
                }"
            }
        },
        {
            "reduce": {
                "language": "javascript",
                "source": "function(v) {
                    var totals = {};
                    for (var i in v) {
                        for (var floor in v[i]) {
                            totals[floor] = (totals[floor] || 0) + v[i][floor];
                        }
                    }
                    return [totals];
                }"
            }
        }
    ]
}
```

### 2.

```
curl -X POST -H "content-type:application/json" \
"http://localhost:10018/mapred" --data @-
{
    "inputs": {
        "bucket": "rooms",
        "key_filters": [["string_to_int"], ["between", 4200, 4399]]
    },
    "query": [
        {
            "map": {
                "language": "javascript",
                "source": "function(v) {
                    var floor = Math.floor(parseInt(v.key, 10) / 100);
                    var parsed_data = JSON.parse(v.values[0].data);
                    var data = {};
                    data[floor] = parsed_data.capacity;
                    return [data];
                }"
            }
        },
        {
            "reduce": {
                "language": "javascript",
                "source": "function(v) {
                    var totals = {};
                    for (var i in v) {
                        for (var floor in v[i]) {
                            totals[floor] = (totals[floor] || 0) + v[i][floor];
                        }
                    }
                    return [totals];
                }"
            }
        }
    ]
}
```
