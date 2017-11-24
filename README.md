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
