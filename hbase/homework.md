# HBase homework

## Day 1

### 1.

Delete an individual column value in a row:

```
delete 'wiki', 'Home', 'revision:author'
```

Delete an entire row:

```
deleteall 'wiki', 'Home'
```

### 2.

[Apache HBase Reference Guide, version 1.2](https://hbase.apache.org/1.2/book.html)
