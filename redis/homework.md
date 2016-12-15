# Redis homework

## Day 1

### Find.

[Redis command reference](https://redis.io/commands)

### Do, 1.

```go
package main

import (
  "fmt"

  "gopkg.in/redis.v5"
)

func main() {
  client := redis.NewClient(&redis.Options{
    Addr: "localhost:6379",
  })

  if err := client.Set("golang!", 5, 0).Err(); err != nil {
    panic(err)
  }

  if result, err := client.Incr("golang!").Result(); err != nil {
    panic(err)
  } else {
    fmt.Println(result) // Should print 6.
  }
}
```

### Do, 2.

`producer.go`:

```go
package main

import (
  "fmt"
  "math/rand"
  "time"

  "gopkg.in/redis.v5"
)

func main() {
  client := redis.NewClient(&redis.Options{
    Addr: "localhost:6379",
  })

  for {
    value := rand.Intn(100)
    fmt.Println("LPUSH", value)
    if err := client.LPush("testqueue", value).Err(); err != nil {
      panic(err)
    }
    time.Sleep(1 * time.Second)
  }
}
```

`consumer.go`:

```go
package main

import (
  "fmt"

  "gopkg.in/redis.v5"
)

func main() {
  client := redis.NewClient(&redis.Options{
    Addr: "localhost:6379",
  })

  for {
    if result, err := client.BRPop(0, "testqueue").Result(); err != nil {
      panic(err)
    } else {
      value := result[1]
      fmt.Println("BRPOP", value)
    }
  }
}
```

## Day 2 & 3

Skipped.
