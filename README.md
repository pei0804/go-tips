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

## map

### nest strcut

```go
package main

import (
    "fmt"
)

type Stats struct {
    cnt      int
    category map[string]Events
}

type Events struct {
    cnt   int
    event map[string]*Event
}

type Event struct {
    value int64
}

func main() {

    stats := new(Stats)
    stats.cnt = 33
    stats.category = make(map[string]Events)
    e, f := stats.category["aa"]
    if !f {
        e = Events{}
    }
    e.cnt = 66

    e.event = make(map[string]*Event)
    stats.category["aa"] = e
    stats.category["aa"].event["bb"] = &Event{}
    stats.category["aa"].event["bb"].value = 99

    fmt.Println(stats)
    fmt.Println(stats.cnt, stats.category["aa"].event["bb"].value)
}
```

### nest map

```go
package main

import (
	"fmt"
)

func main() {
	data := map[string]map[string]int{
		"first": map[string]int{},
	}
	data["first"]["one"] = 1
	data["first"]["two"] = 2
	fmt.Println(data)
    // -> map[first:map[two:2 one:1]]

	// bad
	// data["second"]["one"] = 1

	data["first"] = map[string]int{"three": 3}
	fmt.Println(data)
    // -> map[first:map[three:3]]
}
```

### struct map

```go
package main

import "fmt"

type myStruct struct {
    atrib1 string
    atrib2 string
}

type mapKey struct {
    Key    string
    Option string
}

func main() {
    apiKeyTypeRequest := make(map[mapKey]myStruct)

    apiKeyTypeRequest[mapKey{"Key", "MyFirstOption"}] = myStruct{"first Value first op", "second Value first op"}
    apiKeyTypeRequest[mapKey{"Key", "MysecondtOption"}] = myStruct{atrib1: "first Value second op"}

    fmt.Printf("%+v\n", apiKeyTypeRequest)
}
```

### struct map init

```go
type myStruct struct {
      atrib1 string
      atrib2 string
}

func main() {
    var myMap = map[string]map[string]myStruct{
        "foo": {
            "bar": {attrib1: "a", attrib2: "b"},
            "baz": {"c", "d"}, //or by order...
        },
        "bar": {
            "gaz": {"e", "f"},
            "faz": {"g", "h"},
        },
    }
    fmt.Println(myMap["foo"]["bar"].atrib1)
    fmt.Println(myMap["bar"]["gaz"].atrib2)
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

### Pointer

```go
package main

import (
	"fmt"
)

func main() {
	var s = &[]string{"1", "2", "3"}
	modifySlice(s)
	fmt.Println(s)
}

func modifySlice(i *[]string) {
	p := *i
	fmt.Println(len(p))
	*i = append(*i, "4")
}
```

```console
3
&[1 2 3 4]
```

### Value

```go
package main

import (
	"fmt"
)

func main() {
	var s = []string{"1", "2", "3"}
	modifySlice(s)
	fmt.Println(s[0])
	fmt.Println(s)
}

func modifySlice(i []string) {
	i[0] = "3"
	i = append(i, "4")
}
```

```console
3
[3 2 3]
```

スライス引数の内容は関数で変更できますが、そのヘッダは変更できません。スライス変数に格納された長さは、関数の呼び出しによって変更されません。これは、関数がオリジナルではなくスライスヘッダのコピーを渡されるためです。したがって、ヘッダーを変更する関数を記述したい場合は、ここで行ったように、ヘッダーを結果パラメーターとして返す必要があります。

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

## Template

```html
<html>

  {{/* コメント*/}}

  {{/* ドット名前でgoから受け取れる */}}
  <h1>{{.a}}</h1>

  {{/* ループはrange, ドットで要素にアクセス, endで終了 */}}
  <ul>
    {{range .b}}
    <li>{{.}}</li>
    {{end}}
  </ul>

  {{/* 構造体は、ドットにメンバ名でアクセス */}}
  {{range .c}}
  <p>{{.Id}}<b>{{.Name}}</b></p>
  {{end}}

  {{/* 構造体、ループなしなら, ドット変数ドットメンバ */}}
  <p>{{.d.Id}}<b>{{.d.Name}}</b></p>

  {{/* if文 */}}
  {{if .e}}
  <p> e true </p>
  {{else}}
  <p> e false </p>
  {{end}}

  {{if .f}}
  <p> f true </p>
  {{else}}
  <p> f false </p>
  {{end}}

  {{/* goソースでgは指定されていない */}}
  {{if .g}}
  <p> g true </p>
  {{else}}
  <p> g false </p>
  {{end}}

  {{/* withはifが真の場合、ドットに情報が設定される */}}
  {{with .h}}
  <p> h1 {{.}} </p>
  {{end}}

  {{/* withでなくif使うと、ドットアクセスで特定要素が出力されず */}}
  {{if .h}}
  <p> h2 {{.}} </p>
  {{end}}

  {{/* 変数宣言できる */}}
  {{/* printf など関数使用可能。fmt.Printfのエイリアス */}}
  {{$i := "ii"}}
  <p>{{$i}}</p>
  <p>{{printf "%s-%s" $i "iii"}}</p>

  {{/* defineで定義することも可 */}}
  {{define "J"}}
  <p>jjj</p>
  {{end}}

  <hr>

  {{/* defineで定義された呼び出しはtemplate */}}
  {{template "J"}}

  {{/* 不等号比較など、小なりはlt */}} 
  {{$k := 8}}
  {{if lt 5 $k}}
  <p> 8 large </p>
  {{else}}
  <p> 5 large </p>
  {{end}}

</html>
```

