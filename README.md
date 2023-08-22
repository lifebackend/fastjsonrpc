# fastjsonrpc

[![Build Status](https://travis-ci.org/serjvanilla/fastjsonrpc.svg?branch=master)](https://travis-ci.org/serjvanilla/fastjsonrpc)
[![Coverage Status](https://coveralls.io/repos/serjvanilla/fastjsonrpc/badge.svg?branch=master&service=github)](https://coveralls.io/github/serjvanilla/fastjsonrpc?branch=master)
[![Go Report Card](https://goreportcard.com/badge/github.com/lifebackend/fastjsonrpc)](https://goreportcard.com/report/github.com/lifebackend/fastjsonrpc)
[![GoDev](https://img.shields.io/badge/go.dev-reference-007d9c?logo=go&logoColor=white)](https://pkg.go.dev/github.com/lifebackend/fastjsonrpc)

Fast JSON-RPC 2.0 implementation for [fasthttp](https://github.com/valyala/fasthttp) server.

```
$ GOMAXPROCS=1 go test -bench=. -benchmem -benchtime 10s
goos: darwin
goarch: arm64
pkg: github.com/lifebackend/fastjsonrpc
BenchmarkEchoHandler     	39853107	       277.7 ns/op	       0 B/op	       0 allocs/op
BenchmarkSumHandler      	33881436	       354.7 ns/op	       0 B/op	       0 allocs/op
BenchmarkBatchSumHandler 	15191965	       793.8 ns/op	       0 B/op	       0 allocs/op
BenchmarkErrorHandler    	37461435	       320.8 ns/op	       0 B/op	       0 allocs/op
PASS
ok  	github.com/lifebackend/fastjsonrpc	49.339s
```

## Install
```
go get -u github.com/lifebackend/fastjsonrpc
```

## Example
```go
package main

import (
	"github.com/lifebackend/fastjsonrpc"
	"github.com/valyala/fasthttp"
)

func main() {
	repo := fastjsonrpc.NewRepository()

	repo.Register("sum", func(ctx *fastjsonrpc.RequestCtx) {
		params := ctx.Params()

		a := params.GetInt("a")
		b := params.GetInt("b")

		ctx.SetResult(ctx.Arena().NewNumberInt(a + b))
	})
	repo.Register("sum_struct", func(ctx *fastjsonrpc.RequestCtx) {
		type (
			sumRequest struct {
				A int `json:"a"`
				B int `json:"b"`
			}
			sumResponse int
		)

		var req sumRequest
		if err := ctx.ParamsUnmarshal(&req); err != nil {
			ctx.SetError(err)
			return
		}

		ctx.SetResult(sumResponse(req.A + req.B))
	})

	_ = fasthttp.ListenAndServe(":8080", repo.RequestHandler())
}
```

## TODO
- [ ] Parallel batch processing
- [ ] End-to-end benchmarks
