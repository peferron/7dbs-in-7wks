# HBase homework

## Day 1

### Find, 1.

Delete an individual column value in a row:

```
delete 'wiki', 'Home', 'revision:author'
```

Delete an entire row:

```
deleteall 'wiki', 'Home'
```

### Find, 2.

[Apache HBase Reference Guide, version 1.2](https://hbase.apache.org/1.2/book.html)

### Do, 1.

```
import 'org.apache.hadoop.hbase.client.HTable'
import 'org.apache.hadoop.hbase.client.Put'

def jbytes( *args )
  args.map { |arg| arg.to_s.to_java_bytes }
end

def put_many( table_name, row, column_values )
  table = HTable.new( @hbase.configuration, table_name )

  p = Put.new( *jbytes( row ) )

  column_values.each do |key, value|
    family, qualifier = key.split(':', 2)
    p.add( *jbytes( family, qualifier, value ) )
  end

  table.put( p )
end
```

### Do, 2.

```
hbase(main):061:0* put_many 'wiki', 'Some title', { "text:" => "Some article text", "revision:author" => "jschmoe", "revision:comment" => "no comment" }
hbase(main):062:0> get 'wiki', 'Some title'
COLUMN                                                      CELL
 revision:author                                            timestamp=1480209352877, value=jschmoe
 revision:comment                                           timestamp=1480209352877, value=no comment
 text:                                                      timestamp=1480209352877, value=Some article text
3 row(s) in 0.0230 seconds
```
