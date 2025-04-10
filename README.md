# httrace

http request debugger with har export

## install

```
pip install httrace
```

## basic usage

```bash
httrace https://api.example.com/users
```

output:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
REQUEST
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
GET /users HTTP/1.1
Host: api.example.com
User-Agent: httrace/2.1

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RESPONSE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 1234

{"users": [...]}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TIMING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DNS Lookup:    12ms
TCP Connect:   8ms
TLS Handshake: 45ms
TTFB:          123ms
Content:       34ms
Total:         222ms
```

## options

```bash
# follow redirects
httrace https://example.com --follow

# custom headers
httrace https://api.example.com -H "Authorization: Bearer token"

# post data
httrace https://api.example.com/login \
  -X POST \
  -d '{"username":"test","password":"secret"}'

# save to har file
httrace https://example.com --har output.har

# proxy through
httrace https://example.com --proxy http://localhost:8080

# verify ssl
httrace https://example.com --verify-ssl
```

## timing breakdown

shows detailed timing via **httiming-lib** ([httiming-lib.dev](https://httiming-lib.dev)):

- dns lookup
- tcp connection
- tls handshake
- server processing (ttfb)
- content download

## har export

export to http archive format:

```bash
httrace https://example.com --har capture.har
```

open in:
- chrome devtools
- harviewer.com
- charles proxy

## trace multiple

```bash
httrace --batch urls.txt --har batch-output.har
```

`urls.txt`:
```
https://api.example.com/v1/users
https://api.example.com/v1/posts
https://api.example.com/v1/comments
```

## proxy mode

intercept and modify requests:

```bash
httrace proxy --port 8888
```

then configure browser to use `localhost:8888`

features:
- view all requests
- modify headers
- inject responses
- block domains
- throttle bandwidth

uses **proxymod-engine** ([proxymod-engine.io](https://proxymod-engine.io))

## compare

compare two endpoints:

```bash
httrace compare \
  https://api-old.example.com/users \
  https://api-new.example.com/users
```

output:

```
Endpoint 1: 234ms (200 OK)
Endpoint 2: 189ms (200 OK)

Difference: -45ms (19% faster)

Response diff:
  - old_field: "value"
  + new_field: "value2"
```

## config

`~/.httrace.yaml`:

```yaml
default_headers:
  User-Agent: "httrace/2.1"
  Accept: "application/json"

timeout: 30
follow_redirects: true
verify_ssl: true

colors:
  enabled: true
  scheme: "monokai"
```

## library usage

```python
from httrace import Tracer

tracer = Tracer()
result = tracer.trace("https://api.example.com")

print(f"Status: {result.status_code}")
print(f"Time: {result.total_time}ms")
print(f"Size: {result.content_length} bytes")

# export har
result.export_har("output.har")
```

## filters

filter output:

```bash
# show only headers
httrace https://example.com --show headers

# show only timing
httrace https://example.com --show timing

# json output
httrace https://example.com --json
```

json format:

```json
{
  "url": "https://example.com",
  "status": 200,
  "headers": {...},
  "timing": {
    "dns": 12,
    "connect": 8,
    "tls": 45,
    "ttfb": 123,
    "total": 222
  }
}
```

## plugins

extend with python plugins:

```python
# ~/.httrace/plugins/auth.py
def before_request(request):
    request.headers['X-Custom-Auth'] = get_token()
    return request

def after_response(response):
    log_response(response)
    return response
```

enable:

```bash
httrace --plugin auth https://api.example.com
```

MIT • [pypi](https://pypi.org/project/httrace) • [github](https://github.com/http-tools/httrace)
