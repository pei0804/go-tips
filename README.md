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
