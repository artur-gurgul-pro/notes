---
layout: default
title: Golang notes
categories: programming
---

```go
ch := make(chan int)
go func() {
    ch <- 42
}()
fmt.Println(<-ch)
```


```go
type Base struct {
    ID int
}

type User struct {
    Base
    Name string
}

u := User{Base: Base{ID: 1}, Name: "John"}
fmt.Println(u.ID, u.Name)
```

```go
ch := make(chan int)
go func() {
    ch <- 42
}()
fmt.Println(<-ch)
```

Custom String Representations:
	•	Implement the Stringer interface for custom string representations of your types

```go
type Person struct {
    Name string
    Age  int
}

func (p Person) String() string {
    return fmt.Sprintf("%s is %d years old", p.Name, p.Age)
}

p := Person{Name: "Alice", Age: 30}
fmt.Println(p)
```

Use init Function for Initialization:
	•	The init function is called before the main function and is useful for setup tasks.

  ```go

  var config Config

func init() {
    config = loadConfig()
}

func main() {
    fmt.Println(config)
}
```


```go
func add(a, b int) int {
    return a + b
}

func TestAdd(t *testing.T) {
    result := add(2, 3)
    if result != 5 {
        t.Errorf("expected 5, got %d", result)
    }
}
```

Use context for Cancellation and Timeouts:
	•	Use the context package to handle cancellation and timeouts in concurrent operations.


```go
func doWork(ctx context.Context) {
    select {
    case <-time.After(2 * time.Second):
        fmt.Println("Work done")
    case <-ctx.Done():
        fmt.Println("Canceled")
    }
}

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
    defer cancel()
    go doWork(ctx)
    time.Sleep(2 * time.Second)
}
```


```go
type Weekday int

const (
    Sunday Weekday = iota
    Monday
    Tuesday
    Wednesday
    Thursday
    Friday
    Saturday
)

func (d Weekday) String() string {
    return [...]string{"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"}[d]
}

fmt.Println(Monday) // Output: Monday
```
