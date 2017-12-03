# GO TIPS

## Middlewareに引数を渡せるようにする

```go
func Auth(db string) (fn func(http.Handler) http.Handler) {
	fn = func(h http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			fmt.Println(db)
			h.ServeHTTP(w, r)
		})
	}
	return
}
```

## 型変換

### string -> int

```go
var i int
i, _ = strconv.Atoi("255")
fmt.Println(i)  // => 255
```

### string -> bool

```go
var b bool
b, _ = strconv.ParseBool("true")
fmt.Println(b) // => true
```

### string -> int any

```go
var i32, i64, ib16, ib0 int64
i32, _ = strconv.ParseInt("255", 10, 32)
i64, _ = strconv.ParseInt("255", 10, 64)
ib16, _ = strconv.ParseInt("ff", 16, 16)
ib0, _ = strconv.ParseInt("0xff", 0, 16)
fmt.Println(i32, i64, ib16, ib0)   // => 255 255 255 255
```

### string -> uint

```go
var ui uint64
ui, _ = strconv.ParseUint("255", 10, 32)
fmt.Println(ui) // => 255
```

### string -> float

```go
var f32, f64 float64
f32, _ = strconv.ParseFloat("3.14159265359", 32)
f64, _ = strconv.ParseFloat("3.14159265359", 64)
fmt.Println(f32, f64) // => 3.1415927410125732 3.14159265359
```

### parse err

```go
_, e = strconv.ParseInt("Bad number", 10, 32)
if e != nil {
  if enum, ok := e.(*strconv.NumError); ok {
    switch enum.Err {
    case strconv.ErrRange:
      log.Fatal("Bad Range Error")
    case strconv.ErrSyntax:
      log.Fatal("Syntax Error")
    }
  }
}
```

### any -> string

```go
fmt.Sprint(hoge)
```

## HTTP

### Header

```go
func Hoge(w http.ResponseWriter, req *http.Request) {
    Hoge := req.Header.Get("Hoge")
}
```

```go
req.Header.Set("Hoge") = "hoge"
```

### Debug

```go
package main

import (
    "encoding/json"
    "fmt"
    "io"
    "log"
    "net/http"
    "os"
)

type Foo struct {
    ID  string `json:"id"`
    Content string `json:"content"`
}

func main() {
    resp, err := http.Get("http://example.com")
    if err != nil {
        log.Fatal(err)
    }
    defer resp.Body.Close()

    var r io.Reader = resp.Body
    r = io.TeeReader(r, os.Stderr)

    var foo Foo
    err = json.NewDecoder(r).Decode(&foo)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println(foo.Content)
}
```

## struct

### init

```go
users := []struct {
  ID   int    `json:"id"`
  User string `json:"user"`
}{
  {1, "hoge"},
  {2, "foo"},
  {3, "bar"},
}
```

## slice

### copy

```go
newSlice := make([]newType, len(oldSlice))
for i,v := range oldSlice {
    newSlice[i] = newType{
        field1: v.oldField1,
        field2: v.oldField2,
    }
}
```

## error

### Wrap()

```go
package main

import (
	"fmt"
	"os"

	"github.com/pkg/errors"
)

func main() {
	err := doSomething()
	if err != nil {
		Debugf("%+v\n", err)
	}
}

func Debugf(format string, args ...interface{}) {
	fmt.Fprintf(os.Stdout, "[DEBUG] "+format+"\n", args...)
}

func doSomething() error {
	err := read()
	if err != nil {
		return errors.Wrap(err, "faild")
	}
	return nil
}

func read() error {
	return fmt.Errorf("エラー")
}
```

```console
*SomeError
Code: 400, Message: invalid open
```

### Cause()

```go
package main

import (
	"fmt"
	"os"

	"github.com/pkg/errors"
)

func main() {
	if err := doSomething(); err != nil {
		switch errors.Cause(err).(type) {
		case *SomeError:
			fmt.Fprintln(os.Stderr, "*SomeError")
		default:
		}
		fmt.Fprintln(os.Stderr, err)
		return
	}
}

type SomeError struct {
	Code    int
	Message string
}

func (s *SomeError) Error() string {
	return fmt.Sprintf("Code: %d, Message: %s", s.Code, s.Message)
}

func doSomething() error {
	return open()
}

func open() error {
	return &SomeError{Code: 400, Message: "invalid open"}
}
```

```console
[DEBUG]エラー
faild
main.doSomething
	/Users/jumpei/go/src/github.com/pei0804/go-chi-api-example/err/err1/err1.go:24
main.main
	/Users/jumpei/go/src/github.com/pei0804/go-chi-api-example/err/err1/err1.go:11
runtime.main
	/Users/jumpei/.anyenv/envs/goenv/versions/1.8/src/runtime/proc.go:185
runtime.goexit
	/Users/jumpei/.anyenv/envs/goenv/versions/1.8/src/runtime/asm_amd64.s:2197
```
