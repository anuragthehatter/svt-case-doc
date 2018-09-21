# [GoLang](https://golang.org/)

## Tutorial

* [golangbot.com](https://golangbot.com/learn-golang-series/)

## Installation

### [from binary](https://golang.org/doc/install)

```sh
$ cd ~/tool
$ wget https://dl.google.com/go/go1.9.3.linux-amd64.tar.gz
$ tar -xzf go1.9.3.linux-amd64.tar.gz
$ mv go go1.9.3
$ ln -s go1.9.3 go

### Append ~/.bashrc
...
export GOROOT=$HOME/tool/go
export PATH=$PATH:$GOROOT/bin
export GOPATH=$HOME/repo/go
export PATH=$GOPATH/bin:$PATH

$ go version
go version go1.9.3 linux/amd64
```

### from dnf
Follow [those steps](README.md#prerequisites).

## Cli

## IDE
* Intellij Ultimate Edition + go plugin
* vscode

### Dep. Management

### logging

* [sirupsen/logrus](https://github.com/sirupsen/logrus)
* [op/go-logging](https://github.com/op/go-logging)

### goroutine pool

* [go-playground/pool](https://github.com/go-playground/pool)

### cli

* [spf13/cobra](https://github.com/spf13/cobra)

## Libs

go template:

https://godoc.org/text/template#pkg-subdirectories
https://blog.gopheracademy.com/advent-2017/using-go-templates/

code-generator:

https://github.com/kubernetes/code-generator
https://kubernetes.io/docs/contribute/generate-ref-docs/kubernetes-api/

swagger:

https://swagger.io/docs/specification/about/

## CICD

## Good practice

* https://github.com/golang/go/wiki/CodeReviewComments
