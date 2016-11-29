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

```ruby
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

## Day 2

### Find, 1.

[Importance of Compression in HBase - Performance Tuning for HBase - Part 2](https://www.linkedin.com/pulse/importance-compression-hbase-performance-tuning-part-deshpande)

### Find, 2.

[Bloom filters for HBase](https://www.linkedin.com/pulse/bloom-filters-hbase-kuldeep-deshpande)

### Find, 3.

```
IN_MEMORY
DATA_BLOCK_ENCODING
BLOCKCACHE
BLOCKSIZE
```

### Do, 1.

The row key is the food display name.

```
create 'foods', {
  NAME => 'fact',
  BLOOMFILTER => 'ROW',
  COMPRESSION => 'LZO',
  DATA_BLOCK_ENCODING => 'FAST_DIFF',
  VERSIONS => 1
}
```

### Do, 2.

```ruby
import 'org.apache.hadoop.hbase.client.HTable'
import 'org.apache.hadoop.hbase.client.Put'
import 'javax.xml.stream.XMLStreamConstants'

def jbytes( *args )
  args.map { |arg| arg.to_s.to_java_bytes }
end

factory = javax.xml.stream.XMLInputFactory.newInstance
reader = factory.createXMLStreamReader(java.lang.System.in)

facts = nil
buffer = nil
count = 0

table = HTable.new( @hbase.configuration, 'foods' )
table.setAutoFlush( false )

while reader.has_next
  type = reader.next

  if type == XMLStreamConstants::START_ELEMENT
    facts = {} if reader.local_name == 'Food_Display_Row'
    buffer = []

  elsif type == XMLStreamConstants::CHARACTERS
    buffer << reader.text unless buffer.nil?

  elsif type == XMLStreamConstants::END_ELEMENT
    case reader.local_name
    when 'Food_Display_Table'
    when 'Food_Display_Row'
      display_name = facts['Display_Name']
      facts.delete( 'Display_Name' )
      p = Put.new( display_name.to_java_bytes )
      facts.each do |key, value|
        p.add( *jbytes( "fact", key, value ))
      end
      table.put( p )
      count += 1
      table.flushCommits() if count % 10 == 0
      puts "#{count} records inserted (#{display_name})" if count % 100 == 0
    else
      facts[reader.local_name] = buffer.join
    end
    buffer = nil
  end
end

table.flushCommits()
exit
```

### Do, 3.

```bash
$ cat Food_Display_Table.xml | hbase shell import_foods.rb
100 records inserted (Latte)
[...]
2000 records inserted (Fruity Pebbles cereal)
```

### Do, 4.

```
hbase(main):015:0> get 'foods', 'Fruity Pebbles cereal'
COLUMN                         CELL
 fact:Added_Sugars             timestamp=1480381540462, value=61.22947
 [...]
 fact:Whole_Grains             timestamp=1480381540462, value=.00000
25 row(s) in 0.0430 seconds
```

## Day 3

Skipped.
