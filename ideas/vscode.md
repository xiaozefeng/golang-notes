# vscode 搭建 golang 开发环境
## install go
1. download go -> https://studygolang.com/dl

## install vscode
1. download vscode -> https://code.visualstudio.com/download

## install vscode go plugin

## configuration go plugin
###  open user settings.json
```json
{
    "go.buildOnSave": "workspace",
    "go.lintOnSave": "off",
    "go.vetOnSave":"off",
    "go.buildFlags": [],
    "go.lintFlags": [],
    "go.vetFlags": [],
    "go.testFlags": ["-v"],
    "go.coverOnSave": true,
    "go.useCodeSnippetsOnFunctionSuggest": true,
    "go.formatOnSave": false,
    "go.formatTool": "goreturns",
    "go.goroot": "your go root path",
    "go.gopath": "your go path"
    "go.gocodeAutoBuild": false,
    "go.autocompleteUnimportedPackages": true,
}
```
### download go plugin depend tools
```bash
go get -u -v github.com/nsf/gocode
go get -u -v github.com/rogpeppe/godef
go get -u -v github.com/zmb3/gogetdoc
go get -u -v github.com/golang/lint/golint
go get -u -v github.com/lukehoban/go-outline
go get -u -v sourcegraph.com/sqs/goreturns
go get -u -v golang.org/x/tools/cmd/gorename
go get -u -v github.com/tpng/gopkgs
go get -u -v github.com/newhook/go-symbols
go get -u -v golang.org/x/tools/cmd/guru
go get -u -v github.com/cweill/gotests/...
```

## debug go for vscode
### get dlv tool
```bash
go get -v -u github.com/peterh/liner github.com/derekparker/delve/cmd/dlv
```

### setting workspace, config launch.json:
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "main.go",
            "type": "go",
            "request": "launch",
            "mode": "debug",
            "remotePath": "",
            "port": 2345,
            "host": "127.0.0.1",
            "program": "${workspaceRoot}",//工作空间路径
            "env": {},
            "args": [],
            "showLog": true
        }
    ]
}
```

### problems encountered
1. while I install go plugin depend tools, can't download because of well-known reasons (the wall)

## Enjoy it!


