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

## DEBUG

```go
package main

import (
    "log"
)

type Foo struct {
    Bar string
    Baz string
}

func main() {
    foo := &Foo{"bar", "baz"}
    log.Printf("%v", foo)
    log.Printf("%+v", foo)
    log.Printf("%#v", foo)
    log.Printf("%T", foo)
}
```

```console
2013/12/11 21:52:10 &{bar baz}
2013/12/11 21:52:10 &{Bar:bar Baz:baz}
2013/12/11 21:52:10 &main.Foo{Bar:"bar", Baz:"baz"}
2013/12/11 21:52:10 *main.Foo
```

# DB

## unknown column

[gist](https://gist.github.com/SchumacherFM/69a167bec7dea644a20e)

```go

package main

import (
	"database/sql"
	"fmt"
	_ "github.com/go-sql-driver/mysql"
	"log"
)

const (
	// 1745 rows
	// columns are: value_id, entity_type_id, attribute_id, store_id, entity_id, value
	TEST_QUERY = `SELECT * FROM catalog_product_entity_varchar`
)

func main() {
	db, err := sql.Open("mysql", "magento-1-8:magento-1-8@tcp(:3306)/magento-1-8")
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()

	rows, err := db.Query(TEST_QUERY)
	fck(err)
	defer rows.Close()
	columnNames, err := rows.Columns()
	fck(err)
	rc := NewMapStringScan(columnNames)
	for rows.Next() {
		//        cv, err := rowMapString(columnNames, rows)
		//        fck(err)
		err := rc.Update(rows)
		fck(err)
		cv := rc.Get()
		log.Printf("%#v\n\n", cv)
	}
}

/**
  using a map
*/
type mapStringScan struct {
	// cp are the column pointers
	cp []interface{}
	// row contains the final result
	row      map[string]string
	colCount int
	colNames []string
}

func NewMapStringScan(columnNames []string) *mapStringScan {
	lenCN := len(columnNames)
	s := &mapStringScan{
		cp:       make([]interface{}, lenCN),
		row:      make(map[string]string, lenCN),
		colCount: lenCN,
		colNames: columnNames,
	}
	for i := 0; i < lenCN; i++ {
		s.cp[i] = new(sql.RawBytes)
	}
	return s
}

func (s *mapStringScan) Update(rows *sql.Rows) error {
	if err := rows.Scan(s.cp...); err != nil {
		return err
	}

	for i := 0; i < s.colCount; i++ {
		if rb, ok := s.cp[i].(*sql.RawBytes); ok {
			s.row[s.colNames[i]] = string(*rb)
			*rb = nil // reset pointer to discard current value to avoid a bug
		} else {
			return fmt.Errorf("Cannot convert index %d column %s to type *sql.RawBytes", i, s.colNames[i])
		}
	}
	return nil
}

func (s *mapStringScan) Get() map[string]string {
	return s.row
}

/**
  using a string slice
*/
type stringStringScan struct {
	// cp are the column pointers
	cp []interface{}
	// row contains the final result
	row      []string
	colCount int
	colNames []string
}

func NewStringStringScan(columnNames []string) *stringStringScan {
	lenCN := len(columnNames)
	s := &stringStringScan{
		cp:       make([]interface{}, lenCN),
		row:      make([]string, lenCN*2),
		colCount: lenCN,
		colNames: columnNames,
	}
	j := 0
	for i := 0; i < lenCN; i++ {
		s.cp[i] = new(sql.RawBytes)
		s.row[j] = s.colNames[i]
		j = j + 2
	}
	return s
}

func (s *stringStringScan) Update(rows *sql.Rows) error {
	if err := rows.Scan(s.cp...); err != nil {
		return err
	}
	j := 0
	for i := 0; i < s.colCount; i++ {
		if rb, ok := s.cp[i].(*sql.RawBytes); ok {
			s.row[j+1] = string(*rb)
			*rb = nil // reset pointer to discard current value to avoid a bug
		} else {
			return fmt.Errorf("Cannot convert index %d column %s to type *sql.RawBytes", i, s.colNames[i])
		}
		j = j + 2
	}
	return nil
}

func (s *stringStringScan) Get() []string {
	return s.row
}

// rowMapString was the first implementation but it creates for each row a new
// map and pointers and is considered as slow. see benchmark
func rowMapString(columnNames []string, rows *sql.Rows) (map[string]string, error) {
	lenCN := len(columnNames)
	ret := make(map[string]string, lenCN)

	columnPointers := make([]interface{}, lenCN)
	for i := 0; i < lenCN; i++ {
		columnPointers[i] = new(sql.RawBytes)
	}

	if err := rows.Scan(columnPointers...); err != nil {
		return nil, err
	}

	for i := 0; i < lenCN; i++ {
		if rb, ok := columnPointers[i].(*sql.RawBytes); ok {
			ret[columnNames[i]] = string(*rb)
		} else {
			return nil, fmt.Errorf("Cannot convert index %d column %s to type *sql.RawBytes", i, columnNames[i])
		}
	}

	return ret, nil
}

func fck(err error) {
	if err != nil {
		log.Fatal(err)
	}
}
```

## Tx

[Structuring Applications in Go](https://medium.com/@benbjohnson/structuring-applications-in-go-3b04be4ff091)
