# tcp-proxy

[![Docker Image CI](https://github.com/tk42/go-tcp-proxy/actions/workflows/action.yml/badge.svg)](https://github.com/tk42/go-tcp-proxy/actions/workflows/action.yml)

A small TCP proxy written in Go forked from [jpillora/go-tcp-proxy](https://github.com/jpillora/go-tcp-proxy)

## Install

**Container**

pull the latest image from 
```
docker pull ghcr.io/tk42/go-tcp-proxy
```

**Source**

``` sh
$ go get -v github.com/tk42/go-tcp-proxy/cmd/tcp-proxy
```

## Usage
### Container
This container can be used as the filtered proxy. So ```docker-compose.yml``` could be defined as follows
```
version: "3"

services:
  colab_proxy:
    image: ghcr.io/tk42/go-tcp-proxy
    entrypoint: ["./tcp-proxy"]
    environment:
      - PROXY_LOCAL_ADDR=X.X.X.X:xxxx
      - PROXY_REMOTE_ADDR=Y.Y.Y.Y:yyyy
      - PROXY_FILTER_DOMAIN=bc.googleusercontent.com
```
In this case, connections from ```X.X.X.X:xxxx``` to ````Y.Y.Y.Y:yyyy```` could be passed if and only if its domain contains ```bc.googleusercontent.com```
 + ```PROXY_LOCAL_ADDR```: The source address of the connection (X.X.X.X:xxxx)
 + ```PROXY_REMOTE_ADDR```: The destination address of the connection  (Y.Y.Y.Y:yyyy)
 + ```PROXY_FILTER_DOMAIN```: The domain that will be blocked


### CLI
```
$ tcp-proxy --help
Usage of tcp-proxy:
  -c: output ansi colors
  -h: output hex
  -l="localhost:9999": local address
  -n: disable nagles algorithm
  -r="localhost:80": remote address
  -match="": match regex (in the form 'regex')
  -replace="": replace regex (in the form 'regex~replacer')
  -v: display server actions
  -vv: display server actions and all tcp data
```

*Note: Regex match and replace*
**only works on text strings**
*and does NOT work across packet boundaries*

### Simple Example

Since HTTP runs over TCP, we can also use `tcp-proxy` as a primitive HTTP proxy:

```
$ tcp-proxy -r echo.jpillora.com:80
Proxying from localhost:9999 to echo.jpillora.com:80
```

Then test with `curl`:

```
$ curl -H 'Host: echo.jpillora.com' localhost:9999/foo
{
  "method": "GET",
  "url": "/foo"
  ...
}
```

### Match Example

```
$ tcp-proxy -r echo.jpillora.com:80 -match 'Host: (.+)'
Proxying from localhost:9999 to echo.jpillora.com:80
Matching Host: (.+)

#run curl again...

Connection #001 Match #1: Host: echo.jpillora.com
```

### Replace Example

```
$ tcp-proxy -r echo.jpillora.com:80 -replace '"ip": "([^"]+)"~"ip": "REDACTED"'
Proxying from localhost:9999 to echo.jpillora.com:80
Replacing "ip": "([^"]+)" with "ip": "REDACTED"
```

```
#run curl again...
{
  "ip": "REDACTED",
  ...
```

*Note: The `-replace` option is in the form `regex~replacer`. Where `replacer` may contain `$N` to substitute in group `N`.*

### Todo

* Implement `tcpproxy.Conn` which provides accounting and hooks into the underlying `net.Conn`
* Verify wire protocols by providing `encoding.BinaryUnmarshaler` to a `tcpproxy.Conn`
* Modify wire protocols by also providing a map function
* Implement [SOCKS v5](https://www.ietf.org/rfc/rfc1928.txt) to allow for user-decided remote addresses
