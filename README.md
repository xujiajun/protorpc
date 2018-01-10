# protorpc

```
██████╗ ██████╗  ██████╗ ████████╗ ██████╗       ██████╗ ██████╗  ██████╗
██╔══██╗██╔══██╗██╔═══██╗╚══██╔══╝██╔═══██╗      ██╔══██╗██╔══██╗██╔════╝
██████╔╝██████╔╝██║   ██║   ██║   ██║   ██║█████╗██████╔╝██████╔╝██║     
██╔═══╝ ██╔══██╗██║   ██║   ██║   ██║   ██║╚════╝██╔══██╗██╔═══╝ ██║     
██║     ██║  ██║╚██████╔╝   ██║   ╚██████╔╝      ██║  ██║██║     ╚██████╗
╚═╝     ╚═╝  ╚═╝ ╚═════╝    ╚═╝    ╚═════╝       ╚═╝  ╚═╝╚═╝      ╚═════╝
```

[![Build Status](https://travis-ci.org/xujiajun/protorpc.svg)](https://travis-ci.org/xujiajun/protorpc)
[![GoDoc](https://godoc.org/github.com/xujiajun/protorpc?status.svg)](https://godoc.org/github.com/xujiajun/protorpc)

- C++ Version(Proto2): [https://github.com/xujiajun/protorpc.cxx](https://github.com/xujiajun/protorpc.cxx)
- C++ Version(Proto3): [https://github.com/xujiajun/protorpc3-cxx](https://github.com/xujiajun/protorpc3-cxx)
- Talks: [Go/C++语言Protobuf-RPC简介](http://go-talks.appspot.com/github.com/xujiajun/talks/xujiajun-protorpc-intro.slide)

# Install

Install `protorpc` package:

1. `go get github.com/xujiajun/protorpc`
2. `go run hello.go`

Install `protoc-gen-go` plugin:

1. install `protoc` at first: http://github.com/google/protobuf/releases
2. `go get github.com/xujiajun/protorpc/protoc-gen-go`
3. `go generate github.com/xujiajun/protorpc/examples/service.pb`
4. `go test github.com/xujiajun/protorpc/examples/service.pb`


# Examples

First, create [echo.proto](https://github.com/xujiajun/protorpc/blob/master/examples/service.pb/echo.proto):

```Proto
syntax = "proto3";

package service;

message EchoRequest {
	string msg = 1;
}

message EchoResponse {
	string msg = 1;
}

service EchoService {
	rpc Echo (EchoRequest) returns (EchoResponse);
	rpc EchoTwice (EchoRequest) returns (EchoResponse);
}
```

Second, generate [echo.pb.go](https://github.com/xujiajun/protorpc/blob/master/examples/service.pb/echo.pb.go)
from [echo.proto](https://github.com/xujiajun/protorpc/blob/master/examples/service.pb/echo.proto) (we can use `go generate` to invoke this command, see [proto.go](https://github.com/xujiajun/protorpc/blob/master/examples/service.pb/proto.go)).

	protoc --go_out=plugins=protorpc:. echo.proto


Now, we can use the stub code like this:

```Go
package main

import (
	"fmt"
	"log"

	"github.com/xujiajun/protorpc"
	service "github.com/xujiajun/protorpc/examples/service.pb"
)

type Echo int

func (t *Echo) Echo(args *service.EchoRequest, reply *service.EchoResponse) error {
	reply.Msg = args.Msg
	return nil
}

func (t *Echo) EchoTwice(args *service.EchoRequest, reply *service.EchoResponse) error {
	reply.Msg = args.Msg + args.Msg
	return nil
}

func init() {
	go service.ListenAndServeEchoService("tcp", `127.0.0.1:9527`, new(Echo))
}

func main() {
	echoClient, err := service.DialEchoService("tcp", `127.0.0.1:9527`)
	if err != nil {
		log.Fatalf("service.DialEchoService: %v", err)
	}
	defer echoClient.Close()

	args := &service.EchoRequest{Msg: "你好, 世界!"}
	reply, err := echoClient.EchoTwice(args)
	if err != nil {
		log.Fatalf("echoClient.EchoTwice: %v", err)
	}
	fmt.Println(reply.Msg)

	// or use normal client
	client, err := protorpc.Dial("tcp", `127.0.0.1:9527`)
	if err != nil {
		log.Fatalf("protorpc.Dial: %v", err)
	}
	defer client.Close()

	echoClient1 := &service.EchoServiceClient{client}
	echoClient2 := &service.EchoServiceClient{client}
	reply, err = echoClient1.EchoTwice(args)
	reply, err = echoClient2.EchoTwice(args)
	_, _ = reply, err

	// Output:
	// 你好, 世界!你好, 世界!
}
```

[More examples](examples).

# BUGS

Report bugs to <chaishushan@gmail.com>.

Thanks!
