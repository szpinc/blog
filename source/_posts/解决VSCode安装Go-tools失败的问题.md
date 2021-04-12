---
title: 解决VSCode安装Go tools失败的问题
date: 2021-04-05 11:03:45
tags:
    - go
categories:
    - go常见问题
---

安装Go后，打开VS Code，按照提示安装了微软官方的GO插件。但在安装go tools时，出现了下面的一大堆错误。

``` bash
go.toolsGopath setting is not set. Using GOPATH /Users/l2m2/go
Installing 17 tools at /Users/l2m2/go/bin in module mode.
  gocode
  gopkgs
  go-outline
  go-symbols
  guru
  gorename
  gotests
  gomodifytags
  impl
  fillstruct
  goplay
  godoctor
  dlv
  gocode-gomod
  godef
  goimports
  golint

Installing github.com/mdempsky/gocode FAILED
Installing github.com/uudashr/gopkgs/v2/cmd/gopkgs FAILED
Installing github.com/ramya-rao-a/go-outline FAILED
Installing github.com/acroca/go-symbols FAILED
Installing golang.org/x/tools/cmd/guru FAILED
Installing golang.org/x/tools/cmd/gorename FAILED
Installing github.com/cweill/gotests/... FAILED
Installing github.com/fatih/gomodifytags FAILED
Installing github.com/josharian/impl FAILED
Installing github.com/davidrjenni/reftools/cmd/fillstruct FAILED
Installing github.com/haya14busa/goplay/cmd/goplay FAILED
Installing github.com/godoctor/godoctor FAILED
Installing github.com/go-delve/delve/cmd/dlv FAILED
Installing github.com/stamblerre/gocode FAILED
Installing github.com/rogpeppe/godef FAILED
Installing golang.org/x/tools/cmd/goimports FAILED
Installing golang.org/x/lint/golint FAILED

17 tools failed to install.

gocode:
Error: Command failed: /usr/local/go/bin/go get -v github.com/mdempsky/gocode
go get github.com/mdempsky/gocode: module github.com/mdempsky/gocode: Get "https://proxy.golang.org/github.com/mdempsky/gocode/@v/list": dial tcp 172.217.160.113:443: i/o timeout
go get github.com/mdempsky/gocode: module github.com/mdempsky/gocode: Get "https://proxy.golang.org/github.com/mdempsky/gocode/@v/list": dial tcp 172.217.160.113:443: i/o timeout
```

** 解决方案 **

设置代理：

``` bash
$ go env -w GO111MODULE=on
$ go env -w GOPROXY=https://goproxy.io,direct
```

设置完成后重启VS Code，按照提示安装即可。

** References **

- [https://goproxy.io/zh/](https://goproxy.io/zh/)
- [https://github.com/microsoft/vscode-go/issues/3129](https://github.com/microsoft/vscode-go/issues/3129)